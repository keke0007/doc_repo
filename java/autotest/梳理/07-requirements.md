# requirements.txt 用法(知识点梳理)

## 一、requirements.txt 是什么 / 作用

`requirements.txt` 是 Python 项目用于声明依赖包及其版本的纯文本清单文件,通常放在项目根目录下。

核心作用:

- **依赖清单**:集中记录项目运行所需的全部第三方模块及版本号。
- **环境复现**:在新机器 / 新虚拟环境上,只需一条 `pip install -r requirements.txt` 即可还原相同的依赖环境。
- **协作交付**:随项目源码一起提交到 Git,团队成员、CI/CD、生产服务器都能拿到一致的依赖版本。
- **可追溯**:版本号锁定后,可避免「我这能跑、你那不能跑」的环境漂移问题。

---

## 二、生成方式(`pip freeze` vs `pipreqs`)

### 1. `pip freeze`(原生命令,无需额外安装)

在项目根目录执行:

```bash
pip freeze > requirements.txt
```

特点:导出**当前 Python 环境**中所有已安装的包(包括项目并未真正使用的包)。**因此最好配合虚拟环境使用。**

### 2. `pipreqs`(第三方工具,按需扫描)

先安装:

```bash
pip install pipreqs
```

在项目根目录执行:

```bash
pipreqs ./ --encoding=utf8
```

文件已存在时加 `--force` 覆盖:

```bash
pipreqs ./ --encoding=utf8 --force
```

只生成不带版本号的库名:

```bash
pipreqs ./ --encoding=utf8 --no-pin
```

### 3. 对比表

| 维度 | `pip freeze` | `pipreqs` |
| --- | --- | --- |
| 是否需安装 | 不需要,pip 自带 | 需 `pip install pipreqs` |
| 扫描范围 | 当前 Python 环境中**所有**已装包 | 扫描**项目源码**中的 `import` 语句 |
| 是否包含未使用包 | 是(全量) | 否(精准) |
| 输出版本号 | 始终带版本(`==x.y.z`) | 默认带版本,可用 `--no-pin` 去掉 |
| 文件已存在时 | 直接覆盖(shell 重定向) | 报错,需 `--force` |
| 推荐场景 | 配合 venv 使用,导出干净环境 | 全局环境下生成项目级精简清单 |
| 编码问题 | 一般无 | Windows 下中文路径需 `--encoding=utf8` |

---

## 三、安装方式

### 1. 基本安装

```bash
pip install -r requirements.txt
```

### 2. 使用国内镜像源加速

```bash
pip install -i https://pypi.doubanio.com/simple/ -r requirements.txt
```

参数含义:

- `-i`:更换镜像源(index-url),默认是国外 PyPI,容易慢甚至超时
- `-r`:read,遍历并安装文件中列出的所有包

### 3. 常用国内镜像源

| 名称 | URL |
| --- | --- |
| 豆瓣 | https://pypi.doubanio.com/simple/ |
| 清华 TUNA | https://pypi.tuna.tsinghua.edu.cn/simple |
| 阿里云 | https://mirrors.aliyun.com/pypi/simple/ |
| 中科大 | https://pypi.mirrors.ustc.edu.cn/simple/ |
| 腾讯云 | https://mirrors.cloud.tencent.com/pypi/simple |

---

## 四、版本约束符号

| 符号 | 含义 | 示例 | 解释 |
| --- | --- | --- | --- |
| `==` | 精确等于(锁定) | `django==4.2.1` | 必须是 4.2.1,常用于生产环境 |
| `>=` | 大于等于 | `requests>=2.28` | 最低 2.28,允许更高版本 |
| `<=` | 小于等于 | `flask<=2.0` | 不超过 2.0 |
| `>` | 严格大于 | `numpy>1.20` | 必须高于 1.20 |
| `<` | 严格小于 | `numpy<2.0` | 必须低于 2.0 |
| `!=` | 排除某版本 | `urllib3!=2.0.0` | 不允许 2.0.0,其他版本均可 |
| `~=` | 兼容版本(PEP 440) | `pandas~=1.4.2` | 等价于 `>=1.4.2, <1.5.0`,允许补丁更新 |
| `,` | 多条件组合(逻辑与) | `django>=3.2,<4.0` | 范围限定 |

组合示例:

```text
requests>=2.28,<3.0,!=2.30.0
```

含义:requests 大于等于 2.28、小于 3.0,且不能是 2.30.0。

---

## 五、文件语法

```text
# 这是注释行,以 # 开头会被忽略

# 空行也会被忽略(可用来分组)

requests==2.31.0          # 行尾也可写注释
flask>=2.0,<3.0
pandas~=1.4.2

# 引用另一个 requirements 文件(嵌套)
-r requirements-dev.txt

# 从 Git 仓库直接安装
git+https://github.com/psf/requests.git@v2.31.0#egg=requests

# 从指定分支 / commit / tag
git+https://github.com/user/repo.git@main
git+ssh://git@github.com/user/repo.git@abc1234

# 可编辑安装(开发模式,等价于 pip install -e)
-e .
-e git+https://github.com/user/repo.git#egg=mypkg

# 从本地 wheel / 目录 / 压缩包安装
./vendor/mypkg-0.1.0-py3-none-any.whl
./local_pkg/

# 指定额外特性(extras)
uvicorn[standard]==0.27.0

# 环境标记(只在某些环境安装)
pywin32==306; sys_platform == "win32"
```

常用语法要点:

- `#` 起首为注释,空行允许
- `-r other.txt`:嵌套引用,便于拆分 `requirements-dev.txt`、`requirements-prod.txt`
- `-c constraints.txt`:约束文件(只限制版本,不主动安装)
- `git+`、`hg+`、`svn+`:从版本控制系统直接安装
- `-e`:editable install,源码改动即时生效,常用于本地开发
- `;` 后跟环境标记(environment marker),按平台 / Python 版本条件安装

---

## 六、与 venv / virtualenv 配合

强烈建议在虚拟环境中维护 `requirements.txt`,避免把全局环境里无关包一起导出。

### Windows(`venv`,Python 3.3+ 自带)

```bash
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
pip freeze > requirements.txt
deactivate
```

### Linux / macOS

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
pip freeze > requirements.txt
deactivate
```

### `virtualenv`(老牌第三方,功能更全)

```bash
pip install virtualenv
virtualenv venv
# 激活同上
```

### PyCharm

在 `Settings → Project → Python Interpreter` 里选择 「Add Interpreter → Virtualenv Environment」,PyCharm 会自动隔离环境,在其 Terminal 中执行 `pip freeze` 即只导出本项目依赖。

---

## 七、跨机器复现环境 ASCII 流程图

```text
+--------------------------+              +--------------------------+
|       开发机 A           |              |       新机器 / 服务器 B  |
|--------------------------|              |--------------------------|
| 1. 激活 venv             |              | 1. 拉取项目源码          |
|    venv\Scripts\activate |              |    git clone <repo>      |
|                          |              |                          |
| 2. 安装依赖              |              | 2. 创建并激活 venv       |
|    pip install xxx       |              |    python -m venv venv   |
|                          |              |    source venv/bin/...   |
| 3. 导出依赖              |   提交        |                          |
|    pip freeze >          |   git push    | 3. 安装依赖              |
|    requirements.txt      | ============> |    pip install -i        |
|        |  (或 pipreqs)   |               |      <镜像源> -r         |
|        v                 |               |      requirements.txt    |
|  requirements.txt -------+               |        |                 |
|                          |               |        v                 |
| 4. git add / commit /    |               | 4. python main.py        |
|    push                  |               |    环境一致,运行成功    |
+--------------------------+               +--------------------------+
```

---

## 八、易错点 / 原文错误订正

| # | 原文 / 常见说法 | 问题 | 正确做法 / 建议 |
| --- | --- | --- | --- |
| 1 | "pip freeze 把整个 python 环境的依赖都生成出来" | 描述正确,但容易忽视后果——把无关包(如 IDE 工具、调试库)也写进去 | 在 venv 中执行 `pip freeze`,或改用 `pipreqs ./` 只导出项目实际 import 的依赖 |
| 2 | `pip install -i https://pypi.doubanio.com/simple/ -r requestment.txt` | 原文一处把 `requirements` 拼成了 `requestment` | 正确拼写为 **`requirements.txt`** |
| 3 | 「-r 遍历并安装 requestment.txt 中的包」 | 同上拼写错误,且「遍历」措辞不准 | 应为「读取 requirements.txt 并安装其中列出的包」 |
| 4 | 「里面是什么内容我们不用管」 | 误导。版本号、约束符号、嵌套引用都会影响安装结果 | 应当了解 `==` / `>=` / `~=` 等约束符号的含义,必要时手工编辑 |
| 5 | 直接 `pip freeze > requirements.txt` 而不进虚拟环境 | 会把全局环境的包都导出,污染清单 | 先创建并激活 venv,再 freeze |
| 6 | 使用 `pipreqs ./` 不加 `--encoding=utf8` | Windows 下源码含中文注释时会 `UnicodeDecodeError` | 始终加 `--encoding=utf8`,已存在文件加 `--force` |
| 7 | 镜像源 URL 末尾斜杠不统一(`/simple` vs `/simple/`) | 大多数 pip 版本兼容,但旧版可能有问题 | 推荐统一写成 `https://.../simple`(无尾斜杠),与 pip 官方示例一致 |
| 8 | 把 `requirements.txt` 当成绝对真理 | 不同 OS / Python 版本可能装不上某些包(如 `pywin32` 在 Linux) | 使用环境标记 `; sys_platform == "win32"`,或拆分 `requirements-windows.txt` |
| 9 | 在生产用 `>=` 而非 `==` | 上游小版本变更可能引入 Bug | 生产环境锁定 `==`,或使用 `pip-tools` / `poetry` 生成 lock 文件 |
| 10 | 修改源码后未更新 requirements.txt | 新增 import 在他人机器上 `ModuleNotFoundError` | 每次新增依赖后立即 `pip freeze` 或 `pipreqs --force` 重新生成并提交 |
