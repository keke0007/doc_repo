# Ansible02 知识点梳理

## 1. Playbook 基础

### 1.1 Playbook 是什么
- Playbook 是用 YAML 编写的任务编排文件。
- 一个 Playbook 由一个或多个 `play` 组成。
- 一个 `play` 里可以包含多个 `task`。

### 1.2 Playbook 和 ad-hoc 的区别
- ad-hoc：一次性、临时执行
- Playbook：可复用、可版本管理、能表达顺序和依赖
- 复杂部署、标准化交付、批量变更应优先使用 Playbook

### 1.3 YAML 编写要点
- 使用空格缩进，不用 tab
- 层级要对齐
- `-` 表示列表项
- `key: value` 冒号后面要有空格

### 1.4 Playbook 典型结构
```yaml
- hosts: webservers
  remote_user: root
  vars:
    pkg_name: httpd
  tasks:
    - name: install package
      yum:
        name: "{{ pkg_name }}"
        state: present
```

## 2. Playbook 实战案例

### 2.1 NFS 部署案例
核心任务链：
1. 安装 `nfs-utils`
2. 下发 `/etc/exports`
3. 创建共享目录
4. 启动 NFS 服务

重点：
- 一个服务部署通常就是“安装 + 配置 + 目录准备 + 启动”
- 已经开始体现“多任务串联”

### 2.2 Httpd 部署案例
核心任务链：
1. 安装 httpd
2. 启动 httpd
3. 启动 firewalld
4. 下发网页内容
5. 放行 http 服务

重点：
- 服务不是装完就结束，通常还要配防火墙和页面验证

### 2.3 LAMP 部署案例
核心任务链：
1. 安装 httpd、php、php-mysql、mariadb 等
2. 启动服务
3. 放行 http
4. 下载 `index.php`

重点：
- Playbook 可以把多组件环境一次性交付

### 2.4 集群部署思维
文档中开始把主机分组：
- `dbservers`
- `lbservers`
- `webservers`

核心思想：
- 先做 Inventory 分组
- 再按角色拆任务
- 最终形成可编排的集群部署流程

## 3. Variables 变量体系

### 3.1 为什么需要变量
- 避免硬编码
- 提高复用性
- 同一个 Playbook 可以适配不同环境

### 3.2 常见定义方式

#### 方式 1：`vars`
- 直接写在 Playbook 内
- 适合当前 Playbook 局部使用

#### 方式 2：`vars_files`
- 从外部变量文件导入
- 适合多个 Playbook 共用

#### 方式 3：Inventory 中定义
- 可定义主机变量
- 可定义组变量

#### 方式 4：`host_vars`
- 对单台主机单独定义变量

#### 方式 5：`group_vars`
- 对一组主机统一定义变量

#### 方式 6：命令行 `-e`
- 运行时临时传参
- 优先级高，适合环境切换和临时覆盖

### 3.3 变量使用原则
- 局部变量放 `vars`
- 通用变量放 `vars_files`
- 主机差异用 `host_vars`
- 组级共性用 `group_vars`
- 临时覆盖用 `-e`

### 3.4 变量优先级
原文有“变量优先级测试”部分，重点结论可以记为：
- 命令行 `-e` 优先级通常最高
- 主机变量通常高于组变量
- 更具体的定义高于更通用的定义

## 4. Register

### 4.1 Register 是什么
- `register` 用来接收某个任务的执行结果。
- 可保存：
  - `stdout`
  - `stderr`
  - `rc`
  - `stdout_lines`
  - `changed`

### 4.2 使用场景
- 先执行命令，再根据结果做判断
- 采集端口、进程、配置检查结果
- 配合 `when`、`debug`、`changed_when` 使用

## 5. Facts 事实变量

### 5.1 facts 是什么
- facts 是 Ansible 对被控端自动采集的系统信息。
- 如：
  - 主机名
  - IP 地址
  - CPU
  - 内存
  - 系统版本

### 5.2 典型用途
- 按主机 IP 生成配置
- 按 CPU 核心数生成 worker 数量
- 按内存生成缓存配置
- 按主机名生成主备配置

### 5.3 facts 的代价
- 默认 `Gathering Facts` 会增加执行时间
- 优化方式：
  - 不需要时 `gather_facts: no`
  - 对 facts 进行缓存

## 6. Task 控制

### 6.1 `when`
- 条件执行
- 常用于：
  - 按系统类型分发不同任务
  - 只对特定主机执行
  - 根据命令返回值决定后续步骤

### 6.2 `loop`
- 批量处理重复项
- 常见场景：
  - 批量安装软件
  - 批量创建用户
  - 批量启动服务
  - 批量复制文件

### 6.3 `handlers` 和 `notify`
- 当某个任务“发生变更”时，通知 handler
- handler 常用于重启或重载服务
- 只有被 notify 且有变更时才会执行

### 6.4 `tags`
- 给任务打标签
- 运行 Playbook 时可定向执行部分任务
- 适合：
  - 安装与配置分开跑
  - 同一入口选择不同版本安装

### 6.5 `include`
- 用于复用任务文件
- 把重复的 task 抽成独立文件
- 多个 Playbook 共同引用

这部分是文档里“多文件调用”最明确的部分。

### 6.6 异常与状态控制

#### `ignore_errors`
- 忽略当前任务失败，继续执行后续任务

#### `force_handlers`
- 即使后续任务失败，也强制执行已通知的 handlers

#### `changed_when`
- 手工定义任务是否算“changed”
- 常用于 `shell`、`command` 这类不天然幂等的任务

## 7. 这篇文档的核心能力图谱

### 7.1 从“执行任务”升级到“编排任务”
- Ansible01 更偏模块与 ad-hoc
- Ansible02 真正进入工程化交付

### 7.2 从“固定写法”升级到“参数化”
- 用变量复用 Playbook
- 用 facts 做动态配置

### 7.3 从“单文件”升级到“拆分复用”
- `vars_files`
- `host_vars`
- `group_vars`
- `include`

## 8. 多文件调用 ASCII 流程图

### 8.1 Playbook + vars_files
```text
+--------------+
| ansible-playbook
| site.yml      |
+------+-------+
       |
       v
+--------------+
| 读取 playbook |
+------+-------+
       |
       v
+--------------+
| vars_files    |
| vars.yml      |
+------+-------+
       |
       v
+--------------+
| 变量渲染任务  |
+------+-------+
       |
       v
+--------------+
| 执行 tasks    |
+--------------+
```

### 8.2 Inventory + host_vars/group_vars
```text
+------------------+
| Inventory hosts  |
+--------+---------+
         |
         +--------------------+
         |                    |
         v                    v
+----------------+   +------------------+
| host_vars/<主机>|   | group_vars/<组名>|
+--------+-------+   +---------+--------+
         \                   /
          \                 /
           v               v
         +-------------------+
         | 变量合并与优先级  |
         +---------+---------+
                   |
                   v
         +-------------------+
         | Playbook tasks    |
         +-------------------+
```

### 8.3 include 任务复用
```text
+------------------+
| a_project.yml    |
+--------+---------+
         |
         v
+------------------+        +----------------------+
| include          | -----> | restart_httpd.yml    |
| restart_httpd.yml|        | 仅保存可复用 tasks   |
+------------------+        +----------------------+

+------------------+
| b_project.yml    |
+--------+---------+
         |
         v
+------------------+        +----------------------+
| include          | -----> | restart_httpd.yml    |
| restart_httpd.yml|        | 被多个项目复用       |
+------------------+        +----------------------+
```

### 8.4 notify/handler 执行链路
```text
+---------------------+
| task: 修改配置文件  |
+----------+----------+
           |
           | changed
           v
+---------------------+
| notify handler      |
| 例如 Restart Httpd  |
+----------+----------+
           |
           v
+---------------------+
| handlers 区域执行   |
| 重启/重载服务       |
+---------------------+
```

## 9. 学习重点总结
1. 学会把 ad-hoc 迁移成 Playbook
2. 学会用变量做参数化
3. 学会用 facts 生成动态配置
4. 学会用 `when`、`loop`、`notify` 控制执行过程
5. 学会用 `include`、`vars_files`、`host_vars`、`group_vars` 拆分项目
