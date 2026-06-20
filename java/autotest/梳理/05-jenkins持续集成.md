# Jenkins 持续集成(知识点梳理)

> 源文档:`本地部署jenkins持续集成.html` / `定时构建时间设置.html`

---

## 一、环境准备

| 组件 | 说明 |
| --- | --- |
| **JDK** | Jenkins 是 Java 程序。Jenkins LTS 2.387+ 起**最低要求 JDK 11**(更新版本可能要求 JDK 17) |
| **Jenkins war 包** | 官网下载:<https://www.jenkins.io/download/> |
| **Tomcat** | Servlet 容器,用来跑 jenkins.war。**版本必须与 Jenkins 兼容**(详见易错点)|

> ⚠ **原文重要错误**:讲义示范用的是 **Tomcat 10**,但 Tomcat 10 已经从 `javax.*` 切到 `jakarta.*`,**Jenkins 的 war 包基于 Servlet 3.1(`javax.servlet`),在 Tomcat 10/11 上启动会直接失败**。  
> ✅ 正确做法:**部署 Jenkins.war 必须用 Tomcat 9.x(或 8.5.x)**,不能用 Tomcat 10+。

---

## 二、安装步骤

### 2.1 安装 JDK

```text
1. 双击 jdk-11.0.21_windows-x64_bin.exe 安装
2. 配置环境变量
     JAVA_HOME = D:\Java\jdk-11
     Path 追加 %JAVA_HOME%\bin
3. 验证
     java -version
```

> JDK 9 起 JDK 目录下不再带 jre,**也不再需要单独配 `CLASSPATH`**。如果有外部工具仍需 jre,可手动生成:
> ```bash
> bin\jlink.exe --module-path jmods --add-modules java.desktop --output jre
> ```

### 2.2 安装 Tomcat(选 9.x)

1. 解压 `apache-tomcat-9.0.x-windows-x64.zip` 到**路径不含空格**的目录
2. 清空 `webapps/` 下的默认应用(`ROOT/manager/host-manager/docs/examples`),避免干扰
3. 修改 `conf/logging.properties` 解决控制台中文乱码:
   ```properties
   # 原 = UTF-8 改为 GBK
   java.util.logging.ConsoleHandler.encoding = GBK
   ```
4. 把 `jenkins.war` 复制到 `webapps/` 下
5. 双击 `bin/startup.bat` 启动 Tomcat
6. 浏览器访问 <http://localhost:8080/jenkins>

> Tomcat 启动报错最常见两类:
> - **JDK 与 Jenkins 版本不兼容**:升级 JDK 到 11/17
> - **Tomcat 10 + javax/jakarta 包冲突**:换回 Tomcat 9

---

## 三、初始化向导

1. 进入 Jenkins 后,会要求填写 `secrets/initialAdminPassword`(终端启动时会输出该路径)
2. 选择 **「Install suggested plugins」** 一路下一步,装常用插件
3. 创建管理员用户名 / 密码
4. 实例 URL 默认 `http://localhost:8080/jenkins/`

常用插件(初始化时未装的可在 `Manage Jenkins → Plugins` 里搜)
- **Git plugin**(代码拉取)
- **Allure Jenkins Plugin**(直接展示 Allure 报告)
- **HTML Publisher**(发布 HTML 静态报告)
- **Email Extension**(邮件通知)
- **GitHub / Gitee Integration**(Webhook 触发)

---

## 四、新建 Free Style 项目

`New Item → Freestyle project`,常用配置面板:

| 区块 | 关键设置 |
| --- | --- |
| General | 项目描述、参数化构建(可选) |
| 源码管理 | 选 `Git`,填仓库地址 + 凭据 + 分支 |
| 构建触发器 | Poll SCM / GitHub hook / Build periodically(定时构建) |
| 构建环境 | `Delete workspace before build starts`(每次构建前清空工作空间)|
| 构建步骤 | `Execute Windows batch command` 或 `Execute shell` |
| 构建后操作 | `Allure Report` / `Editable Email Notification` |

---

## 五、构建步骤(Windows 示例)

```bat
:: 进入项目目录
cd %WORKSPACE%

:: 创建并激活虚拟环境(避免污染全局 Python)
python -m venv venv
call venv\Scripts\activate.bat

:: 装依赖
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple/

:: 跑 pytest,生成 allure 原始结果
pytest --alluredir=./allure-results --clean-alluredir
```

> ⚠ **Jenkins 构建不会让命令的非 0 退出码失败**——除非你显式让 pytest 失败时退码非 0。pytest 默认就是用例失败退码 1。**不要加 `|| exit 0` 之类「吞」错误码的写法**。

---

## 六、Allure 报告集成

1. 安装 **Allure Jenkins Plugin**
2. 安装 **Allure CLI** 到 Jenkins 服务器,并在 `Manage Jenkins → Tools → Allure Commandline` 中注册
3. 在项目 → 构建后操作 → 新增 **Allure Report**
   - Results 路径:`allure-results`(相对 `WORKSPACE`)
4. 构建完成后,项目首页出现 **Allure Report** 入口与趋势图

---

## 七、定时构建表达式

**Jenkins 使用 5 字段的类 cron 语法**:

```
 分(0-59)  时(0-23)  日(1-31)  月(1-12)  周(0-7,0/7 都是周日)
   *         *         *         *         *
```

### 7.1 通配符与高级语法

| 符号 | 含义 |
| --- | --- |
| `*` | 任意值 |
| `M-N` | 范围,如 `9-17` |
| `M-N/X` 或 `*/X` | 步进,如 `*/5` 每 5 分 |
| `A,B,C` | 列表,如 `0,30` |
| `H` | **Hash**,在允许范围内由 Jenkins **按项目稳定哈希** 计算一个值——用于错峰,避免所有任务同时撞在整点 |
| `@hourly @daily @midnight @weekly @monthly @yearly @annually` | 别名 |

> 「`H`」是 Jenkins 独有的占位符,**标准 Unix cron 没有**。这是讲义和很多博客最常混淆的点。

### 7.2 常用示例

```text
每 5 分钟构建一次
H/5 * * * *

每 2 小时一次
H H/2 * * *

每 8 小时一次
H H/8 * * *

每天中午 12 点
H 12 * * *

每天下午 18 点
H 18 * * *

每小时前半小时,每 10 分钟一次
H(0-29)/10 * * * *

工作日上午 9:45 到下午 16:45,每 2 小时
45 9-16/2 * * 1-5

工作日 9-16 点,每 2 小时(由 Jenkins 选随机分,例如 10:38/12:38/14:38/16:38)
H H(9-16)/2 * * 1-5

每天午夜
@midnight

每小时
@hourly
```

> ⚠ 注意:`H 12 * * *` 并不是「**12:00 整**」,而是「**12 点这一小时内的某一分钟**」,Jenkins 哈希计算稳定但是哪一分钟看项目名。要精确到 12:00,写成 `0 12 * * *`。

### 7.3 构建触发器(完整选项)

| 触发器 | 说明 |
| --- | --- |
| **Build periodically** | 按表达式定时构建,**不管代码是否更新** |
| **Poll SCM** | 表达式同上,但**先去仓库轮询**,有变更才触发构建,适合源码触发 |
| **GitHub hook trigger** | GitHub Webhook 推送即构建 |
| **Trigger builds remotely** | 通过 URL + token 远程触发 |
| **Build after other projects are built** | 上游项目构建完后链式触发 |

---

## 八、CI 流程图(代码提交 → 报告)

```
+-------------+      git push     +-----------------+
| 开发者本地  | -----------------> | GitHub/Gitee    |
| git commit  |                    | (远程仓库)      |
+-------------+                    +--------+--------+
                                            |
                            ┌───────────────┴───────────────┐
                            │ (1) Webhook                   │ (2) Poll SCM
                            │     仓库主动推送              │     Jenkins 定时拉取
                            ▼                               ▼
                                  +-----------------+
                                  |   Jenkins       |
                                  | (Tomcat 容器内) |
                                  +--------+--------+
                                           |
              +----------------------------+----------------------------+
              | Stage:Checkout                                          |
              |   git clone / pull -> $WORKSPACE                        |
              +----------------------------+----------------------------+
                                           |
              +----------------------------+----------------------------+
              | Stage:Install                                           |
              |   python -m venv venv                                   |
              |   pip install -r requirements.txt                       |
              +----------------------------+----------------------------+
                                           |
              +----------------------------+----------------------------+
              | Stage:Test                                              |
              |   pytest --alluredir=./allure-results                   |
              |   (失败用例 -> 退码 != 0 -> 构建标记为 FAILURE)         |
              +----------------------------+----------------------------+
                                           |
              +----------------------------+----------------------------+
              | Stage:Report                                            |
              |   Allure Jenkins Plugin                                 |
              |   读取 ./allure-results -> 生成 HTML 报告               |
              +----------------------------+----------------------------+
                                           |
              +----------------------------+----------------------------+
              | Stage:Notify                                            |
              |   Email Ext / 钉钉 / 企微 webhook                       |
              +---------------------------------------------------------+
```

---

## 九、易错点 / 原文错误集中订正

| # | 原文写法 / 现象 | 问题 | 正确做法 |
| --- | --- | --- | --- |
| 1 | 「准备 Tomcat 10」 | **Jenkins.war 基于 `javax.servlet`,在 Tomcat 10+(jakarta)上启动失败** | 改用 **Tomcat 9.x**(或 8.5.x),或直接用 `java -jar jenkins.war` 内置 Jetty |
| 2 | 截图里既出现 Tomcat 10 又出现 8.5.96 | 版本前后不一致,容易踩坑 | 全局统一使用 9.x |
| 3 | 「准备环境(jdk 版本跟 Tomcat 版本要匹配)」表述过粗 | 实际是**三者都要匹配**:JDK ↔ Tomcat ↔ Jenkins | 参考表:Jenkins LTS 2.387+ 需 JDK 11+;Tomcat 9.x 支持 JDK 8/11/17;Tomcat 10 需 JDK 11+ 且 Jakarta EE 9 |
| 4 | 「jdk 中不包含 jre 文件,需要自己生成 jre 文件」 | 这是 JDK 9 起的设计变更,**通常不需要生成**;只在某些老工具找 `jre/bin/java.exe` 时才用 `jlink` 补 | 多数情况下跳过这步 |
| 5 | 「关于 CLASSPATH 的配置问题」 | 现代 JDK **不需要** 配 `CLASSPATH` | 不要再手动设 `CLASSPATH=.;%JAVA_HOME%\lib\dt.jar;...` |
| 6 | `JAVA_HOME = D:\Java`(目录名截图含歧义) | 应指向具体 jdk 版本目录 | `JAVA_HOME = D:\Java\jdk-11.0.21`(实际 JDK 根) |
| 7 | logging.properties 把 `UTF-8` 改成 `GBK` | 仅解决**Windows 控制台中文乱码**;部署到 Linux 不要改 | 平台相关,Linux 保持 UTF-8 |
| 8 | 报错「jdk 版本与 Jenkins 版本不兼容」 | 原文未给版本对照 | Jenkins 2.357+ 起最低 JDK 11;2.426+ 起 LTS 也要 JDK 11;参考 <https://www.jenkins.io/doc/book/platform-information/support-policy-java/> |
| 9 | 定时构建未解释 `H` 占位符的含义 | `H` 是 Jenkins 独有,用于在允许范围内**随机但稳定**地分散负载 | 「`H/5 * * * *`」实际是「每小时 0/5/10... 中的某一组(由项目 hash 决定),并非每隔 5 分整」 |
| 10 | 「每天中午 12 点定时构建一次:`H 12 * * *`」 | `H 12 * * *` 是 12 点这一小时内**某个**分钟,不一定是 12:00 整 | 想精确 12:00 用 `0 12 * * *` |
| 11 | 「每两小时 45 分钟,从上午 9:45 开始……」措辞不通 | 表达式 `45 9-16/2 * * 1-5` 实际是 **9:45 / 11:45 / 13:45 / 15:45 工作日**,共 4 次,不是从 9:45 起一直到 15:45 | 中文描述应改为「工作日的 9:45 / 11:45 / 13:45 / 15:45 各构建一次」 |
| 12 | 字段顺序 | Jenkins 是 5 字段「分 时 日 月 周」,**没有秒位**;别与 Linux crond(5 字段)或 Quartz(6/7 字段)混淆 | 写表达式时先核对字段数 |
| 13 | 周字段 0/7 | Jenkins 中 **0 和 7 都代表周日**,1=周一 ... 6=周六 | 与某些 cron(0=Sunday/1=Monday/...)一致;Quartz 是 1=Sunday/.../7=Saturday,差一位 |
| 14 | 文档未提及别名 | 缺 `@hourly / @daily / @midnight / @weekly / @monthly / @yearly` | 简单场景优先用别名,可读性好 |
| 15 | 构建步骤未提到 venv | 直接在全局 Python 装依赖,容易污染服务器 | 用 `python -m venv venv && call venv\Scripts\activate.bat`,或在 Linux 用 Docker agent |
