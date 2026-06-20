# Ansible01 知识点梳理

## 1. Ansible 快速入门

### 1.1 Ansible 是什么
- Ansible 是自动化运维与配置管理工具。
- 核心价值是把重复性的远程运维动作标准化、批量化、可复用化。
- 常见使用场景：
  - 批量执行远程命令
  - 批量安装和配置服务
  - 自动化发布和环境初始化
  - 通过 Playbook 编排一组有顺序的任务

### 1.2 Ansible 主要能力
- 批量远程执行命令
- 批量分发文件
- 批量管理服务
- 批量管理用户、计划任务、挂载、主机名、防火墙等
- 用 Playbook 进行任务编排

### 1.3 Ansible 特点
- 无 Agent：通常基于 SSH 管理被控端
- 上手成本低：模块化能力强
- 幂等性好：重复执行目标一致
- 易迁移：Playbook 可以复制到其他控制端复用
- 安全性较高：依赖成熟的 SSH 体系

### 1.4 基础架构
- 控制端：执行 Ansible 的机器
- 被控端：被管理的远程主机
- Inventory：主机清单
- ad-hoc：一次性命令执行方式
- Playbook：任务编排文件
- Connection Plugin：连接插件，常见是 SSH
- Module：真正执行任务的功能单元

## 2. 安装与配置

### 2.1 安装方式
- `yum/rpm` 安装
- `pip` 安装

### 2.2 安装确认
- `ansible --version`：看版本与配置路径
- `ansible localhost -m ping`：验证基础可用性

## 3. 配置文件与优先级

### 3.1 常见路径
- `/etc/ansible/ansible.cfg`：主配置文件
- `/etc/ansible/hosts`：默认 Inventory
- `/etc/ansible/roles/`：默认角色目录

### 3.2 `ansible.cfg` 常见配置项
- `inventory`：Inventory 文件位置
- `forks`：并发数
- `remote_port`：远程端口
- `host_key_checking`：是否检查 SSH host key
- `log_path`：日志文件路径
- `become*`：提权相关配置

### 3.3 配置优先级
从高到低通常是：
1. 环境变量 `ANSIBLE_CONFIG`
2. 当前目录 `ansible.cfg`
3. 用户家目录 `~/.ansible.cfg`
4. `/etc/ansible/ansible.cfg`

### 3.4 普通用户统一管理被控端
关键步骤：
1. 控制端和被控端创建同名普通用户
2. 配置 SSH 免密
3. 被控端给该用户 sudo 权限
4. 在 `ansible.cfg` 中配置 `become`

这部分的重点不是命令本身，而是理解：
- SSH 登录用户可以不是 root
- 真正执行系统级操作时可以通过 `become` 提权

## 4. Inventory

### 4.1 Inventory 是什么
- Inventory 是主机与主机组定义文件。
- 它不仅能定义“有哪些机器”，还能定义“这些机器的连接参数和变量”。

### 4.2 常见写法
- 直接写 IP
- 写主机名
- 写主机别名并指定 `ansible_ssh_host`
- 组变量 `group:vars`
- 主机范围匹配：如 `web[1:2].example.com`

### 4.3 连接方式
- 密码方式：`ansible_ssh_user`、`ansible_ssh_pass`
- 密钥方式：推荐，维护成本更低

### 4.4 主机匹配模式
- `all`
- `*`
- `webservers`
- `webservers:appservers`：并集
- `webservers:&dbservers`：交集
- `webservers:!apps`：差集
- 正则匹配：`~(web|db).*`

## 5. ad-hoc

### 5.1 ad-hoc 是什么
- ad-hoc 就是一次性临时命令。
- 适合快速执行、临时排查、简单操作。
- 不适合复杂编排；复杂任务应转为 Playbook。

### 5.2 基本格式
```bash
ansible 主机模式 -m 模块 -a "模块参数"
```

示例：
```bash
ansible webservers -m command -a "df -h"
```

### 5.3 ad-hoc 执行过程
- 读取配置文件
- 读取 Inventory
- 解析主机模式
- 找到对应模块
- 在控制端生成临时模块脚本
- 通过 SSH 传到远端临时目录
- 赋权并执行
- 回收结果
- 删除临时文件

### 5.4 返回状态
- `green / ok`：无变更
- `yellow / changed`：发生了变更
- `red / failed`：执行失败

## 6. 常用模块

### 6.1 命令执行

#### `command`
- 默认模块
- 不经过 shell 解析
- 不支持管道、重定向、变量展开
- 常用参数：
  - `chdir`
  - `creates`
  - `removes`

#### `shell`
- 经过 shell 解析
- 支持管道、重定向、变量、复合命令
- 比 `command` 更灵活，但要注意安全性和幂等性

#### `script`
- 把控制端脚本传到被控端执行
- 适合已有脚本的快速复用

### 6.2 软件包与文件

#### `yum`
- 管理软件包安装、升级、删除
- 常用 `state`：
  - `present`
  - `latest`
  - `absent`

#### `copy`
- 从控制端复制文件到被控端
- 也能直接用 `content` 写文件
- 可配置：
  - `owner`
  - `group`
  - `mode`
  - `backup`

#### `file`
- 创建文件/目录/软链接
- 也可删除路径
- 常见 `state`：
  - `touch`
  - `directory`
  - `link`
  - `absent`

#### `lineinfile`
- 针对单行内容做增删改
- 常用于配置文件局部修改
- 常见参数：
  - `regexp`
  - `line`
  - `insertbefore`
  - `insertafter`
  - `state`
  - `backup`
  - `create`

#### `get_url`
- 从网络下载文件到被控端
- 可配校验和 `checksum`

### 6.3 服务与系统

#### `systemd` / `service`
- 管理服务启动、停止、重启、开机自启
- 常见参数：
  - `name`
  - `state`
  - `enabled`

#### `hostname`
- 修改主机名

#### `mount`
- 管理挂载和 `/etc/fstab`
- 常见 `state`：
  - `mounted`
  - `unmounted`
  - `present`
  - `absent`

#### `selinux`
- 配置 SELinux 模式

#### `firewalld`
- 管理服务放行、端口放行、富规则等

### 6.4 用户与计划任务

#### `group`
- 管理用户组

#### `user`
- 管理用户
- 可创建家目录、设置 shell、附加组、生成 ssh key

#### `cron`
- 管理计划任务
- 支持启用/禁用任务

### 6.5 压缩归档

#### `archive`
- 打包压缩

#### `unarchive`
- 解压
- `remote_src=yes` 表示压缩包已在被控端

## 7. 这篇文档最重要的理解点

### 7.1 ad-hoc 与 Playbook 的边界
- ad-hoc：临时、快速、单步
- Playbook：可复用、可审计、有顺序的多步任务

### 7.2 幂等意识
- `creates` / `removes`
- 模块本身的 `state`
- 避免一味用 `shell`

### 7.3 模块优先于脚本
- 优先用原生模块
- 无法表达时再退回 `shell` 或 `script`

## 8. 多文件/多组件执行流程图

### 8.1 ad-hoc 执行链路
```text
+----------------------+
| ansible 命令行       |
+----------+-----------+
           |
           v
+----------------------+
| 读取 ansible.cfg     |
+----------+-----------+
           |
           v
+----------------------+
| 读取 Inventory/hosts |
+----------+-----------+
           |
           v
+----------------------+
| 匹配目标主机         |
+----------+-----------+
           |
           v
+----------------------+
| 加载指定模块         |
| 如 command/copy/yum  |
+----------+-----------+
           |
           v
+----------------------+
| 控制端生成临时脚本   |
+----------+-----------+
           |
           v
+----------------------+
| SSH 传到被控端 tmp   |
+----------+-----------+
           |
           v
+----------------------+
| 被控端执行模块       |
+----------+-----------+
           |
           v
+----------------------+
| 返回结果并清理临时文件|
+----------------------+
```

### 8.2 `copy`/`lineinfile`/`systemd` 常见配置变更链路
```text
+-------------+      +------------------+      +------------------+
| 本地配置文件| ---> | copy/lineinfile  | ---> | 远端配置文件更新 |
+-------------+      +------------------+      +---------+--------+
                                                           |
                                                           v
                                                +------------------+
                                                | systemd/service  |
                                                | 重载或重启服务   |
                                                +------------------+
```

## 9. 建议的学习顺序
1. 先掌握 Inventory、`ansible.cfg`、SSH 免密
2. 再掌握 ad-hoc 与常用模块
3. 最后进入 Playbook、变量、模板、角色
