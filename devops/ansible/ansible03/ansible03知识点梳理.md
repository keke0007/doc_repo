# Ansible03 知识点梳理

## 1. delegate 委派

### 1.1 delegate 是什么
- 某个 task 本来应该在当前被控主机执行。
- 通过 `delegate_to` 可以把这一个 task 委派给别的主机执行。

本质上是：
- `hosts` 决定“当前 play 面向谁”
- `delegate_to` 决定“当前 task 实际在哪台机器上执行”

### 1.2 使用场景
- 更新某个节点前，先去负载均衡器摘流
- 在控制端生成 SSH key，再分发到被控端
- 只想让某一步在本地执行

### 1.3 相关关键字
- `delegate_to`
- `run_once`
- `connection: local`
- `delegate_facts`

### 1.4 文档中的三个典型场景

#### 场景 1：把单个任务委派给另一台主机
- 对 `172.16.1.7` 执行一组任务
- 其中一条写 `/etc/hosts` 的任务改为在 `172.16.1.5` 上执行

#### 场景 2：在控制端生成普通用户密钥，再下发到节点
核心思路：
1. 在控制端创建用户并生成 SSH 公钥
2. `register` 保存公钥内容
3. 在各被控端创建用户和 `.ssh`
4. 把公钥写入 `authorized_keys`

关键点：
- `delegate_to: localhost`
- `run_once: true`

#### 场景 3：滚动发布 + 负载均衡摘流/上流
核心思路：
1. 当前 web 节点先从 HAProxy 后端摘除
2. 在当前节点更新代码
3. 重载服务
4. 再把该节点加回 HAProxy
5. `serial: 1` 逐台执行，保证集群可用

这是整篇最有工程价值的部分。

## 2. Jinja2 模板

### 2.1 Jinja2 是什么
- Jinja2 是模板引擎。
- Ansible 用它把“变量 + 模板文件”渲染成最终配置文件。

### 2.2 与 `copy` 的区别
- `copy`：原样复制
- `template`：复制前先渲染变量和逻辑

### 2.3 基本语法
- 变量输出：`{{ var }}`
- 循环：
```jinja2
{% for item in list %}
...
{% endfor %}
```
- 条件：
```jinja2
{% if cond %}
...
{% elif cond %}
...
{% endif %}
```
- 注释：
```jinja2
{# comment #}
```

### 2.4 典型案例

#### 案例 1：渲染 `/etc/motd`
- 通过 facts 写入主机名、总内存、空闲内存
- 说明 template 能直接消费 facts

#### 案例 2：Nginx upstream 配置
- 用 `for` 循环生成多个后端节点
- 可以基于固定范围，也可以基于 `groups['webservers']`

#### 案例 3：Keepalived 主备配置
- 用 `if` 根据主机名输出不同的 `state` 和 `priority`
- 本质是“一套模板，多台机器渲染不同结果”

#### 案例 4：MySQL 配置开关
- 用 `if` 判断变量是否存在/开启
- 根据变量动态生成不同端口配置

## 3. Ansible Vault

### 3.1 Vault 是什么
- 用于加密敏感内容。
- 常见加密对象：
  - 含密码的变量文件
  - 含密钥的 Playbook
  - 不适合明文存放的配置

### 3.2 常见操作
- `ansible-vault encrypt`
- `ansible-vault view`
- `ansible-vault edit`
- `ansible-vault rekey`
- `ansible-vault decrypt`

### 3.3 密码文件
- 可通过 `--vault-password-file` 指定密码文件
- 也可以在 `ansible.cfg` 中配置 `vault_password_file`

### 3.4 实际意义
- 把“可自动化”与“可保密”结合起来
- 避免把敏感信息直接提交到仓库

## 4. Roles 角色

### 4.1 Roles 是什么
- Roles 是对 Playbook 的工程化拆分。
- 它把变量、任务、模板、文件、处理器按标准目录组织起来。

### 4.2 典型目录结构
- `tasks/main.yml`
- `handlers/main.yml`
- `vars/main.yml`
- `templates/`
- `files/`
- `meta/main.yml`

### 4.3 角色的价值
- 降低重复
- 增强可维护性
- 形成标准组件
- 便于组合和依赖管理

### 4.4 依赖关系
- 通过 `meta/main.yml` 中的 `dependencies` 声明
- 例如 `wordpress` 依赖 `nginx` 和 `php-fpm`

### 4.5 角色编写思路
1. 先规划目录结构
2. 把任务写入 `tasks`
3. 把配置模板放进 `templates`
4. 把静态文件放进 `files`
5. 把服务重启逻辑写入 `handlers`
6. 在主 Playbook 中引用角色

## 5. Roles 实战案例

### 5.1 Rsync 角色
关键组成：
- `tasks/main.yml`：安装、下发配置、启动服务
- `handlers/main.yml`：配置变更后重启 rsync
- `files/`：静态配置文件和密码文件

重点：
- 这是“静态文件分发型角色”

### 5.2 NFS 角色
关键组成：
- `tasks/main.yml`
- `handlers/main.yml`
- `templates/exports`
- `group_vars/all`

重点：
- 角色内的模板和外部变量结合使用
- 这是“模板渲染型角色”

### 5.3 Memcached 角色
关键组成：
- `tasks/main.yml` 再次 `include` 其他 task 文件
- `yum.yml`
- `template.yml`
- `start.yml`
- `templates/memcached.j2`

重点：
- 角色内部还可以继续拆任务文件
- 模板中使用 facts，比如按内存动态计算参数

## 6. 这篇文档最关键的工程能力

### 6.1 从“任务编排”升级到“集群变更控制”
- `delegate_to`
- `serial`
- 摘流/发布/回流

### 6.2 从“复制配置”升级到“渲染配置”
- `template`
- `for`
- `if`
- `groups`
- `facts`

### 6.3 从“单 Playbook”升级到“组件化角色”
- 标准目录
- 角色依赖
- 角色内部再拆分 tasks

## 7. 多文件调用 ASCII 流程图

### 7.1 Jinja2 模板渲染链路
```text
+------------------+
| playbook.yml     |
+--------+---------+
         |
         v
+------------------+
| vars / facts     |
+--------+---------+
         |
         v
+------------------+
| template 模块    |
+--------+---------+
         |
         v
+------------------+
| *.j2 模板文件    |
+--------+---------+
         |
         v
+------------------+
| 渲染后的远端配置 |
+--------+---------+
         |
         v
+------------------+
| notify handler   |
+--------+---------+
         |
         v
+------------------+
| 重启/重载服务    |
+------------------+
```

### 7.2 HAProxy 滚动发布委派流程
```text
+----------------------+
| play hosts=webservers|
| serial: 1            |
+----------+-----------+
           |
           v
+----------------------+
| 当前 web 节点        |
| inventory_hostname   |
+----------+-----------+
           |
           | delegate_to lbserver
           v
+----------------------+
| HAProxy 节点         |
| 禁用当前 backend 节点|
+----------+-----------+
           |
           v
+----------------------+
| 当前 web 节点        |
| 发布新代码/改页面    |
+----------+-----------+
           |
           v
+----------------------+
| handler 重载 nginx   |
+----------+-----------+
           |
           | delegate_to lbserver
           v
+----------------------+
| HAProxy 节点         |
| 重新启用 backend 节点|
+----------------------+
```

### 7.3 Role 标准调用链路
```text
+------------------+
| site.yml         |
| roles: - nfs     |
+--------+---------+
         |
         v
+------------------+
| roles/nfs/       |
+--------+---------+
         |
         +--------------------+
         |                    |
         v                    v
+------------------+   +------------------+
| tasks/main.yml   |   | handlers/main.yml|
+--------+---------+   +------------------+
         |
         +--------------------+
         |                    |
         v                    v
+------------------+   +------------------+
| templates/exports|   | files/           |
+--------+---------+   +------------------+
         |
         v
+------------------+
| group_vars/all   |
| 提供 share_dir 等|
+--------+---------+
         |
         v
+------------------+
| 渲染并执行部署   |
+------------------+
```

### 7.4 Role 内部 include 拆分
```text
+------------------------+
| roles/memcached/       |
| tasks/main.yml         |
+-----------+------------+
            |
            +------------------+
            |                  |
            v                  v
+------------------+   +------------------+
| include yum.yml  |   | include template |
+--------+---------+   +---------+--------+
         |                         |
         v                         v
+------------------+     +----------------------+
| 安装 memcached   |     | 渲染 memcached.j2    |
+--------+---------+     +---------+------------+
         \                         /
          \                       /
           v                     v
            +-------------------+
            | include start.yml |
            +---------+---------+
                      |
                      v
            +-------------------+
            | 启动服务          |
            +-------------------+
```

## 8. 学习总结
1. `delegate_to` 解决“目标主机不是实际执行主机”的问题
2. `template + jinja2` 解决“同一套配置模板适配多台主机”的问题
3. `vault` 解决“自动化里的敏感信息保护”问题
4. `roles` 解决“Playbook 规模变大后的结构化治理”问题
