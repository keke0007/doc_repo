# 第 2 章 requests 模块 — 知识点梳理

> 对应原文 2.1 ~ 2.16,共 16 节。本章涉及 retrying+requests、Session+多请求等多模块协作,绘制了流程图。

---

## 1. 安装

```bash
pip install requests -i https://pypi.tuna.tsinghua.edu.cn/simple
```

特性提要:Keep-Alive + 连接池、自动解码、Cookie 持久化(Session)、SSL 校验、流式下载、代理、分块上传、超时。

---

## 2. 发请求与响应对象常用属性

```python
import requests
r = requests.get(url, headers=..., params=..., timeout=5)
```

| 属性 | 含义 |
|---|---|
| `r.status_code` | 状态码 |
| `r.content` | bytes 响应体 |
| `r.text` | str 响应体(按 `r.encoding` 推断) |
| `r.encoding` | requests 推断的字符集,可手动改 |
| `r.apparent_encoding` | 基于内容用 chardet 嗅探的字符集(原文遗漏) |
| `r.headers` | 响应头 |
| `r.request.headers` | 实际发出的请求头 |
| `r.cookies` | `RequestsCookieJar`(原文叫 CookieJar,严谨说应是 RequestsCookieJar) |
| `r.url` | 最终 URL(经重定向后) |
| `r.history` | 重定向链上的历史响应 |
| `r.json()` | 把 JSON 响应体解析为 dict/list |

### 三步解决乱码

```python
r.encoding = r.apparent_encoding   # 推荐:嗅探后再 .text
# 或
r.content.decode('utf-8')          # 直接按已知字符集解码
```

---

## 3. `text` vs `content`

| | `text` | `content` |
|---|---|---|
| 类型 | str | bytes |
| 解码 | 按 `r.encoding`(HTTP 头里的 charset 或推断) | 不解码 |
| 适用 | HTML/JSON 文本 | 图片/视频/PDF 等二进制 |

---

## 4. 文件 / 大文件下载

普通图片:

```python
with open('logo.png', 'wb') as f:
    f.write(requests.get(url).content)
```

大文件 / 进度条(必须配合 `stream=True`):

```python
import requests
from tqdm import tqdm

r = requests.get(url, stream=True)
total = int(r.headers.get('content-length', 0))
with open('video.mp4', 'wb') as f, tqdm(total=total, unit='B', unit_scale=True) as bar:
    for chunk in r.iter_content(chunk_size=1024 * 8):
        if chunk:
            f.write(chunk)
            bar.update(len(chunk))
```

**关键点**:`stream=True` 时 body 并不会立即下载,只有迭代 `iter_content` / 读 `content` 时才下载;用完务必 `r.close()` 或用 `with` 语句包住,否则连接不会归还连接池。

---

## 5. 请求头 / UA

```python
headers = {"User-Agent": "Mozilla/5.0 ..."}
requests.get(url, headers=headers)
```

最常被服务端校验:`User-Agent`、`Referer`、`Cookie`、`X-Requested-With`、`Origin`、`Content-Type`。

---

## 6. 查询参数 vs 表单 vs JSON

```python
# GET:URL query string
requests.get(url, params={'wd': 'python'})

# POST:application/x-www-form-urlencoded
requests.post(url, data={'k': 'v'})

# POST:application/json
requests.post(url, json={'k': 'v'})   # 自动序列化 + 自动设 Content-Type

# 文件上传:multipart/form-data
requests.post(url, files={'file': open('a.png', 'rb')})
```

`data` 与 `json` 互斥:`json=` 时 requests 会自动把 dict 转成 JSON 字符串并设置 `Content-Type: application/json`,无需手动设置。

---

## 7. Cookie 处理

三种方式:

```python
# ① 放进 headers 的 Cookie 字符串
headers = {'Cookie': 'name1=v1; name2=v2'}

# ② 用 cookies 参数传字典
requests.get(url, cookies={'name': 'value'})

# ③ Session 自动维护(推荐,见下一节)
```

读出响应里设置的 cookie:

```python
requests.utils.dict_from_cookiejar(r.cookies)
```

---

## 8. 重定向 & 历史请求

- 默认行为:除 HEAD 外,所有方法都会自动跟随 3xx 重定向。
- 关闭:`allow_redirects=False`,此时 `r.status_code` 直接是 302/301 等。
- 整条链路:`r.history`(list,每一项是一次中间响应),最终响应仍在 `r` 本身。

```
http://www.baidu.com
  └─302→ https://www.baidu.com           ← 进入 r.history
       └─302→ https://m.baidu.com/?…     ← 最终 r
```

---

## 9. SSL 证书 / 超时 / 重试

```python
requests.get(url, verify=False)   # 跳过证书校验(配合 urllib3 警告关闭)
requests.get(url, timeout=5)      # 单值=连接+读取总超时
requests.get(url, timeout=(3, 10))# (connect, read)
```

⚠️ 关闭 `verify` 会触发 `InsecureRequestWarning`,可:

```python
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
```

`retrying` 装饰器配合 requests 做断网重试:

```python
from retrying import retry

@retry(stop_max_attempt_number=3, wait_fixed=1000)
def fetch(url):
    r = requests.get(url, timeout=5)
    assert r.status_code == 200
    return r
```

> 现代项目更常用 **`tenacity`**(`retrying` 已多年不维护)。tenacity 支持指数退避、按异常类型重试等。

---

## 10. Session 会话

`requests.Session()` 的两个核心好处:

1. **自动保持 cookie** 跨请求传递(登录后访问受保护页);
2. **复用底层 TCP 连接(连接池)**,显著加速同站多请求。

```python
s = requests.Session()
s.headers.update({'User-Agent': '...'})  # 全局头
s.post(login_url, data={...})            # 登录 → cookie 自动入袋
s.get(profile_url)                       # 自动带 cookie + 复用连接
```

---

## 11. 代理 proxies

```python
proxies = {
    'http':  'http://127.0.0.1:7890',
    'https': 'http://127.0.0.1:7890',   # 注意:https 走 http 隧道(CONNECT)
}
requests.get('http://httpbin.org/ip', proxies=proxies, timeout=10)
```

带账号密码:`http://user:pass@host:port`。socks5 需 `pip install requests[socks]` 再写 `socks5://host:port`。

---

## 12. retrying + requests 调用流程图

```
                        ┌────────────────────────────┐
                        │      业务调用 parse_url     │
                        └──────────────┬─────────────┘
                                       │ try/except
                                       ▼
                        ┌────────────────────────────┐
                        │   _parse_url (@retry)      │
                        │   stop_max_attempt=3       │
                        └──────────────┬─────────────┘
                                       │ 调用
                                       ▼
                ┌─────────────────────────────────────┐
                │    requests.get(url, timeout=...)   │
                └────────────┬────────────────────────┘
                             │
              ┌──────────────┴──────────────┐
        响应正常 │                       超时/异常 │
              ▼                              ▼
       assert status==200            ConnectTimeout/ReadTimeout
              │ 通过                       │ retrying 捕获
              ▼                              ▼
         返回 response             重试 (累计未超过 3 次)
                                            │
                                  达到 3 次 ▼
                                     抛异常 → 外层 except → 返回 None
```

---

## 13. 原文需要纠正/补充的点

| # | 原文/示例 | 问题 | 正确做法 |
|---|----------|------|----------|
| 1 | `for i in 100: print(i)` (2.13 retrying 示例) | `int` 对象不可迭代,会抛 `TypeError` | 应为 `for i in range(100):` |
| 2 | "`response.cookies` 是 CookieJar 类型" | 严格说是 `requests.cookies.RequestsCookieJar`(继承自 `http.cookiejar.CookieJar`) | 描述更准确为 `RequestsCookieJar` |
| 3 | "HEAD 之外都会自动重定向" | 默认 `allow_redirects=True` 适用于 GET/OPTIONS/POST 等;**HEAD 默认是 False**,这一点原文表述对,但还应指出 requests 在某些 POST→GET 重定向时会改方法 | 见原文表 + 注意 301/302 POST 的方法变化 |
| 4 | `verify=False` | 没提示警告抑制 | 建议加 `urllib3.disable_warnings(...)` |
| 5 | 推荐 retrying 库 | 该库 2014 年之后基本停更 | 生产中改用 **tenacity** |
| 6 | "session 实现和服务端的长连接,加快请求速度" | 说法对但不够具体 | 准确说法:Session 内部使用 `HTTPAdapter` 提供的连接池(默认 10 个连接 × 10 个主机),并复用 TCP/TLS 连接(Keep-Alive) |
| 7 | json 翻译案例里手写 `Content-Type: application/x-www-form-urlencoded` 同时给 `data=` 表单参数 | 没问题,但容易和后面 `json=` 混淆 | 强调:用 `json=` 时不要再手动指定 Content-Type,会被 requests 改为 `application/json` |
| 8 | 2.16 代理示例 `"%s:%d" % (ip, port)` | 写法 OK,可读性差 | 现代写法 `f"http://{ip}:{port}"` |

---

## 14. 速记卡

- GET → `params`;POST 表单 → `data`;POST JSON → `json`。
- 二进制内容(图片/视频)用 `content`,文本用 `text`;乱码先调 `encoding`。
- 大文件:`stream=True` + `iter_content`,务必关闭。
- 想保持登录态/跨请求 cookie/连接复用 → `Session`。
- 易错点:`for i in 100` ❌、`verify=False` 警告未抑制、retrying 已过时改 tenacity。
