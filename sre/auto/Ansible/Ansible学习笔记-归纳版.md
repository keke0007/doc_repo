# Ansible 学习笔记归纳版

> 本文由 `Ansible` 目录下 3 份 Markdown 文档归纳整理而成，按学习路径重新编排。每个知识点都包含：核心概念、代码示例、应用场景。

## 目录

- [1. Ansible 基础概念](#1-ansible-基础概念)
- [2. 安装与配置](#2-安装与配置)
- [3. Inventory 主机清单](#3-inventory-主机清单)
- [4. Ad-Hoc 临时命令](#4-ad-hoc-临时命令)
- [5. 常用模块](#5-常用模块)
- [6. Playbook 基础](#6-playbook-基础)
- [7. 变量](#7-变量)
- [8. Register 与 Facts](#8-register-与-facts)
- [9. Playbook 流程控制](#9-playbook-流程控制)
- [10. Jinja2 模板](#10-jinja2-模板)
- [11. Delegate 任务委派](#11-delegate-任务委派)
- [12. Vault 加密](#12-vault-加密)
- [13. Roles 角色](#13-roles-角色)
- [14. 综合实践案例](#14-综合实践案例)
- [15. 学习建议与排错清单](#15-学习建议与排错清单)

## 1. Ansible 基础概念

### 1.1 什么是 Ansible

Ansible 是一款自动化运维工具，用于批量管理服务器。它通过 SSH 连接被控端，不需要在被控端安装 Agent。

应用场景：

- 批量执行命令。
- 批量安装软件。
- 自动化部署服务。
- 管理配置文件。
- 编排多台服务器的发布流程。

### 1.2 Ansible 的特点

- 无 Agent：默认通过 SSH 管理远程主机。
- 幂等性：多数模块重复执行不会造成重复变更。
- 声明式：通过 Playbook 描述目标状态。
- 易读性：Playbook 使用 YAML 编写。
- 可扩展：支持模块、模板、变量、Roles。

应用场景：

- 管理几十台到几千台 Linux 主机。
- 标准化环境初始化。
- 替代大量重复 Shell 脚本。

### 1.3 核心组成

| 组件 | 作用 |
| --- | --- |
| Control Node | 控制端，运行 Ansible 的机器 |
| Managed Node | 被控端，被管理的服务器 |
| Inventory | 主机清单，定义被控主机 |
| Module | 模块，实际执行任务 |
| Ad-Hoc | 临时命令 |
| Playbook | 自动化剧本 |
| Role | 可复用的任务组织方式 |

## 2. 安装与配置

### 2.1 安装 Ansible

RPM/YUM 安装：

```bash
yum install -y epel-release
yum install -y ansible
```

Python pip 安装：

```bash
python3 -m pip install ansible
```

确认安装：

```bash
ansible --version
```

应用场景：

- CentOS/RHEL 环境通常使用 `yum` 安装。
- Python 虚拟环境或需要指定版本时可使用 `pip`。

### 2.2 配置文件优先级

Ansible 配置文件常见位置：

```text
ANSIBLE_CONFIG 环境变量指定的文件
当前目录 ./ansible.cfg
用户家目录 ~/.ansible.cfg
系统配置 /etc/ansible/ansible.cfg
```

优先级从上到下递减。

项目级配置示例：

```ini
[defaults]
inventory = ./hosts
host_key_checking = False
forks = 10
remote_user = root
timeout = 30

[privilege_escalation]
become = True
become_method = sudo
become_user = root
```

应用场景：

- 每个自动化项目使用自己的 `ansible.cfg`。
- 关闭首次 SSH 指纹确认，减少批量执行阻塞。
- 设置默认 Inventory 和远程用户。

### 2.3 普通用户提权

Inventory 示例：

```ini
[web]
web01 ansible_host=10.0.0.11 ansible_user=deploy ansible_become=true
```

命令示例：

```bash
ansible web -m command -a "id" -b
```

Playbook 示例：

```yaml
- hosts: web
  become: true
  tasks:
    - name: install nginx
      yum:
        name: nginx
        state: present
```

应用场景：

- 禁止 root 远程登录时，用普通用户 SSH 连接后 sudo 提权。
- 生产环境权限分离。

## 3. Inventory 主机清单

### 3.1 Inventory 基础格式

INI 格式：

```ini
[web]
web01 ansible_host=10.0.0.11
web02 ansible_host=10.0.0.12

[db]
db01 ansible_host=10.0.0.21

[all:vars]
ansible_user=root
ansible_port=22
```

YAML 格式：

```yaml
all:
  children:
    web:
      hosts:
        web01:
          ansible_host: 10.0.0.11
        web02:
          ansible_host: 10.0.0.12
    db:
      hosts:
        db01:
          ansible_host: 10.0.0.21
  vars:
    ansible_user: root
```

应用场景：

- 按业务分组管理主机。
- 给主机或组设置连接参数和业务变量。

### 3.2 密码连接方式

```ini
[web]
web01 ansible_host=10.0.0.11 ansible_user=root ansible_password='123456'
```

测试：

```bash
ansible web -m ping
```

应用场景：

- 实验环境快速验证。
- 临时管理没有配置 SSH key 的机器。

注意：生产环境不建议明文保存密码，推荐 SSH key 或 Vault。

### 3.3 SSH Key 连接方式

生成密钥：

```bash
ssh-keygen -t rsa -b 4096
ssh-copy-id root@10.0.0.11
```

Inventory：

```ini
[web]
web01 ansible_host=10.0.0.11 ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_rsa
```

应用场景：

- 生产环境批量管理。
- CI/CD 自动化部署。

### 3.4 主机匹配

```bash
ansible all -m ping
ansible web -m ping
ansible 'web:&prod' -m ping
ansible 'web:!web02' -m ping
ansible '10.0.0.*' -m ping
```

应用场景：

- 只对某个组执行任务。
- 排除指定主机。
- 组合多个环境或业务分组。

## 4. Ad-Hoc 临时命令

### 4.1 Ad-Hoc 是什么

Ad-Hoc 是一次性临时命令，适合快速检查、批量执行简单任务。

基础格式：

```bash
ansible 主机模式 -m 模块名 -a "模块参数"
```

示例：

```bash
ansible web -m ping
ansible web -m command -a "uptime"
ansible web -m shell -a "df -h | grep /$"
```

应用场景：

- 批量查看负载、磁盘、内存。
- 临时重启服务。
- 验证主机连通性。

### 4.2 执行状态

常见状态：

| 状态 | 含义 |
| --- | --- |
| `SUCCESS` | 执行成功 |
| `CHANGED` | 执行成功且目标发生变化 |
| `FAILED` | 执行失败 |
| `UNREACHABLE` | 主机不可达 |
| `SKIPPED` | 任务被跳过 |

应用场景：

- 根据状态判断批量任务是否真正变更。
- 排查网络、认证、命令错误。

## 5. 常用模块

### 5.1 command 模块

默认模块，执行普通命令，不支持管道、重定向、变量展开。

```bash
ansible web -m command -a "uptime"
ansible web -m command -a "creates=/tmp/installed touch /tmp/installed"
```

应用场景：

- 执行简单、安全的系统命令。
- 不需要 Shell 特性的命令优先用 `command`。

### 5.2 shell 模块

通过远程 Shell 执行命令，支持管道、重定向、变量。

```bash
ansible web -m shell -a "ps aux | grep nginx | grep -v grep"
ansible web -m shell -a "echo hello > /tmp/hello.txt"
```

应用场景：

- 需要管道、重定向、通配符时使用。
- 临时日志分析或复杂命令。

### 5.3 script 模块

把本地脚本复制到远程执行。

```bash
ansible web -m script -a "./check_system.sh"
```

应用场景：

- 已有 Shell 脚本需要批量执行。
- 临时巡检脚本。

### 5.4 yum 模块

安装、卸载软件包。

```bash
ansible web -m yum -a "name=nginx state=present"
ansible web -m yum -a "name=nginx state=absent"
```

Playbook：

```yaml
- name: install packages
  yum:
    name:
      - nginx
      - vim
    state: present
```

应用场景：

- 批量安装基础软件。
- 部署服务依赖。

### 5.5 copy 模块

复制本地文件到远程。

```bash
ansible web -m copy -a "src=./nginx.conf dest=/etc/nginx/nginx.conf owner=root group=root mode=0644"
```

Playbook：

```yaml
- name: copy nginx config
  copy:
    src: nginx.conf
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: "0644"
```

应用场景：

- 分发配置文件。
- 分发脚本、证书、公钥。

### 5.6 file 模块

管理文件、目录、软链接、权限。

```yaml
- name: create app directory
  file:
    path: /opt/app
    state: directory
    owner: deploy
    group: deploy
    mode: "0755"

- name: create symlink
  file:
    src: /opt/app/releases/current
    dest: /opt/app/current
    state: link
```

应用场景：

- 创建部署目录。
- 修改权限。
- 创建软链接。

### 5.7 lineinfile 模块

按行修改文件，适合修改单个配置项。

```yaml
- name: disable selinux
  lineinfile:
    path: /etc/selinux/config
    regexp: '^SELINUX='
    line: 'SELINUX=disabled'
```

应用场景：

- 修改 SSH、SELinux、系统配置中的单行配置。
- 确保某一行存在或被替换。

### 5.8 get_url 模块

下载远程文件。

```yaml
- name: download package
  get_url:
    url: https://example.com/app.tar.gz
    dest: /tmp/app.tar.gz
    mode: "0644"
```

应用场景：

- 下载安装包。
- 下载配置模板或静态资源。

### 5.9 systemd 模块

管理服务。

```yaml
- name: start nginx
  systemd:
    name: nginx
    state: started
    enabled: true
```

应用场景：

- 启动、停止、重启服务。
- 设置开机自启。
- 配置变更后 reload。

### 5.10 user 与 group 模块

```yaml
- name: create group
  group:
    name: app
    state: present

- name: create user
  user:
    name: app
    group: app
    shell: /sbin/nologin
    create_home: false
```

应用场景：

- 创建服务运行用户。
- 批量创建系统账号。

### 5.11 cron 模块

```yaml
- name: add backup cron
  cron:
    name: backup etc
    minute: "0"
    hour: "2"
    job: "/usr/local/bin/backup_etc.sh"
```

应用场景：

- 管理备份任务。
- 管理巡检、日志清理定时任务。

### 5.12 mount 模块

```yaml
- name: mount nfs
  mount:
    path: /data
    src: 10.0.0.31:/data
    fstype: nfs
    opts: defaults
    state: mounted
```

应用场景：

- 配置 NFS 挂载。
- 管理 `/etc/fstab`。

### 5.13 archive 与 unarchive 模块

```yaml
- name: archive logs
  archive:
    path: /var/log/nginx
    dest: /tmp/nginx-log.tar.gz
    format: gz

- name: unarchive app
  unarchive:
    src: /tmp/app.tar.gz
    dest: /opt/app
    remote_src: true
```

应用场景：

- 打包日志。
- 解压应用发布包。

### 5.14 selinux、firewalld、iptables

```yaml
- name: disable selinux
  selinux:
    state: disabled

- name: open http service
  firewalld:
    service: http
    permanent: true
    state: enabled
    immediate: true
```

应用场景：

- 初始化系统安全策略。
- 开放服务端口。

## 6. Playbook 基础

### 6.1 Playbook 是什么

Playbook 是 Ansible 的剧本文件，用 YAML 描述一组自动化任务。

基础结构：

```yaml
- hosts: web
  become: true
  tasks:
    - name: install nginx
      yum:
        name: nginx
        state: present

    - name: start nginx
      systemd:
        name: nginx
        state: started
        enabled: true
```

执行：

```bash
ansible-playbook site.yml
```

应用场景：

- 自动部署服务。
- 标准化系统初始化。
- 多步骤任务编排。

### 6.2 Playbook 与 Ad-Hoc 的区别

| 类型 | 适合场景 |
| --- | --- |
| Ad-Hoc | 临时、简单、一次性任务 |
| Playbook | 可重复、可维护、多步骤任务 |

应用场景：

- 查看磁盘：用 Ad-Hoc。
- 部署 Nginx：用 Playbook。

### 6.3 YAML 书写规则

- 使用空格缩进，不使用 Tab。
- 同级元素缩进一致。
- 字符串可以加引号，涉及特殊字符时建议加引号。
- 列表用 `-`。
- 字典用 `key: value`。

语法检查：

```bash
ansible-playbook --syntax-check site.yml
```

模拟执行：

```bash
ansible-playbook site.yml --check
```

查看会影响哪些主机：

```bash
ansible-playbook site.yml --list-hosts
```

## 7. 变量

### 7.1 变量定义方式

常见变量来源：

- Playbook 中的 `vars`。
- 外部变量文件 `vars_files`。
- Inventory 主机变量。
- `host_vars`。
- `group_vars`。
- 命令行 `-e`。
- Facts 自动采集变量。

应用场景：

- 不同环境使用不同端口、路径、软件版本。
- 不同主机生成不同配置。

### 7.2 vars 定义变量

```yaml
- hosts: web
  vars:
    package_name: nginx
    service_name: nginx
  tasks:
    - name: install package
      yum:
        name: "{{ package_name }}"
        state: present

    - name: start service
      systemd:
        name: "{{ service_name }}"
        state: started
        enabled: true
```

应用场景：

- 小型 Playbook 内部直接定义变量。
- 同一个任务模板替换服务名、包名。

### 7.3 vars_files 定义变量

变量文件 `vars/nginx.yml`：

```yaml
package_name: nginx
service_name: nginx
listen_port: 80
```

Playbook：

```yaml
- hosts: web
  vars_files:
    - vars/nginx.yml
  tasks:
    - name: show port
      debug:
        msg: "nginx listen port is {{ listen_port }}"
```

应用场景：

- 变量较多时独立管理。
- 按环境拆分 `dev.yml`、`prod.yml`。

### 7.4 Inventory 变量

```ini
[web]
web01 ansible_host=10.0.0.11 http_port=80
web02 ansible_host=10.0.0.12 http_port=8080

[web:vars]
worker_processes=2
```

调用：

```yaml
- hosts: web
  tasks:
    - debug:
        msg: "{{ inventory_hostname }} port={{ http_port }} workers={{ worker_processes }}"
```

应用场景：

- 不同主机端口不同。
- 同一组主机共享变量。

### 7.5 host_vars 与 group_vars

目录结构：

```text
project/
├── ansible.cfg
├── hosts
├── host_vars/
│   └── web01.yml
├── group_vars/
│   └── web.yml
└── site.yml
```

`group_vars/web.yml`：

```yaml
package_name: nginx
service_name: nginx
```

`host_vars/web01.yml`：

```yaml
listen_port: 80
```

应用场景：

- 大项目变量分层管理。
- 主机级变量覆盖组级变量。

### 7.6 命令行传参

Playbook：

```yaml
- hosts: web
  tasks:
    - debug:
        msg: "deploy version is {{ version }}"
```

执行：

```bash
ansible-playbook deploy.yml -e "version=1.2.3"
```

应用场景：

- 发布时传入版本号。
- 临时覆盖变量。

## 8. Register 与 Facts

### 8.1 register 保存任务结果

`register` 用于保存一个任务的执行结果，后续任务可以引用。

```yaml
- hosts: web
  tasks:
    - name: check nginx
      shell: systemctl is-active nginx
      register: nginx_status
      ignore_errors: true

    - name: show nginx status
      debug:
        var: nginx_status.stdout

    - name: start nginx when stopped
      systemd:
        name: nginx
        state: started
      when: nginx_status.stdout != "active"
```

应用场景：

- 根据上一条命令结果决定后续动作。
- 保存命令输出用于调试或模板渲染。

### 8.2 Facts 是什么

Facts 是 Ansible 自动采集的主机信息，如 IP、CPU、内存、主机名、系统版本。

查看 Facts：

```bash
ansible web -m setup
ansible web -m setup -a "filter=ansible_default_ipv4"
```

Playbook 使用：

```yaml
- hosts: web
  tasks:
    - debug:
        msg: "hostname={{ ansible_hostname }} ip={{ ansible_default_ipv4.address }}"
```

应用场景：

- 根据系统版本安装不同包。
- 根据 IP 生成配置。
- 根据 CPU 核数调整服务参数。

### 8.3 根据 Facts 生成配置

```yaml
- hosts: web
  tasks:
    - name: create nginx worker config
      copy:
        dest: /etc/nginx/conf.d/worker.conf
        content: |
          worker_processes {{ ansible_processor_vcpus }};
      notify: reload nginx

  handlers:
    - name: reload nginx
      systemd:
        name: nginx
        state: reloaded
```

应用场景：

- 不同机器使用不同配置。
- CPU、内存、IP、主机名驱动配置生成。

### 8.4 Facts 性能优化

关闭 Facts：

```yaml
- hosts: web
  gather_facts: false
  tasks:
    - ping:
```

应用场景：

- Playbook 不需要系统信息时加速执行。
- 大批量主机执行简单任务。

## 9. Playbook 流程控制

### 9.1 when 条件判断

```yaml
- hosts: all
  tasks:
    - name: install nginx on RedHat
      yum:
        name: nginx
        state: present
      when: ansible_os_family == "RedHat"

    - name: install nginx on Debian
      apt:
        name: nginx
        state: present
      when: ansible_os_family == "Debian"
```

应用场景：

- 根据系统版本执行不同任务。
- 只对特定主机、特定环境执行任务。

### 9.2 loop 循环

批量安装软件：

```yaml
- hosts: web
  tasks:
    - name: install packages
      yum:
        name: "{{ item }}"
        state: present
      loop:
        - nginx
        - vim
        - unzip
```

批量创建用户：

```yaml
- hosts: all
  tasks:
    - name: create users
      user:
        name: "{{ item.name }}"
        groups: "{{ item.groups }}"
        state: present
      loop:
        - { name: "alice", groups: "wheel" }
        - { name: "bob", groups: "dev" }
```

应用场景：

- 批量软件安装。
- 批量用户、目录、配置项创建。

### 9.3 handlers 与 notify

Handlers 只有在被通知且任务发生变更时才执行。

```yaml
- hosts: web
  tasks:
    - name: update nginx config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: restart nginx

  handlers:
    - name: restart nginx
      systemd:
        name: nginx
        state: restarted
```

应用场景：

- 配置文件变更后重启服务。
- 避免每次执行 Playbook 都无意义重启。

### 9.4 tags 标签

```yaml
- hosts: web
  tasks:
    - name: install nginx
      yum:
        name: nginx
        state: present
      tags: install

    - name: configure nginx
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      tags: config
```

只执行指定标签：

```bash
ansible-playbook site.yml --tags config
```

跳过指定标签：

```bash
ansible-playbook site.yml --skip-tags install
```

应用场景：

- 只更新配置。
- 只执行部署中的某个阶段。

### 9.5 include/import 任务复用

`tasks/install.yml`：

```yaml
- name: install nginx
  yum:
    name: nginx
    state: present
```

主 Playbook：

```yaml
- hosts: web
  tasks:
    - import_tasks: tasks/install.yml
      tags: install
```

应用场景：

- 多个项目复用相同任务。
- 把大型 Playbook 拆成多个小文件。

### 9.6 异常处理

忽略错误：

```yaml
- name: check process
  shell: pgrep nginx
  register: check_nginx
  ignore_errors: true
```

控制 changed 状态：

```yaml
- name: check nginx status
  shell: systemctl is-active nginx
  register: nginx_status
  changed_when: false
```

控制失败条件：

```yaml
- name: check http code
  uri:
    url: http://127.0.0.1
    status_code: 200
  register: http_check
  failed_when: http_check.status != 200
```

应用场景：

- 检查类任务不希望显示 changed。
- 某些非 0 返回码不代表真正失败。
- 自定义服务健康检查失败条件。

## 10. Jinja2 模板

### 10.1 Jinja2 是什么

Jinja2 是模板引擎，Ansible 用它根据变量生成配置文件。

模板文件 `nginx.conf.j2`：

```jinja2
worker_processes {{ worker_processes }};

events {
    worker_connections {{ worker_connections }};
}

http {
    server {
        listen {{ listen_port }};
        server_name {{ server_name }};
    }
}
```

Playbook：

```yaml
- hosts: web
  vars:
    worker_processes: 2
    worker_connections: 1024
    listen_port: 80
    server_name: www.example.com
  tasks:
    - name: render nginx config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
```

应用场景：

- 不同主机生成不同配置。
- 配置文件参数化。

### 10.2 Jinja2 判断

```jinja2
{% if ansible_processor_vcpus >= 4 %}
worker_processes 4;
{% else %}
worker_processes 1;
{% endif %}
```

应用场景：

- 根据 CPU、内存、环境类型输出不同配置。
- 主备节点生成不同 Keepalived 配置。

### 10.3 Jinja2 循环

变量：

```yaml
upstreams:
  - 10.0.0.11:80
  - 10.0.0.12:80
```

模板：

```jinja2
upstream backend {
{% for server in upstreams %}
    server {{ server }};
{% endfor %}
}
```

应用场景：

- 生成 Nginx upstream。
- 生成防火墙规则。
- 生成集群节点配置。

### 10.4 Keepalived 模板示例

`host_vars/lb01.yml`：

```yaml
keepalived_state: MASTER
keepalived_priority: 100
```

`host_vars/lb02.yml`：

```yaml
keepalived_state: BACKUP
keepalived_priority: 90
```

模板：

```jinja2
vrrp_instance VI_1 {
    state {{ keepalived_state }}
    interface eth0
    virtual_router_id 51
    priority {{ keepalived_priority }}
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.0.0.100
    }
}
```

应用场景：

- 根据主机变量生成主备不同配置。
- 避免为每台机器维护一份配置文件。

## 11. Delegate 任务委派

### 11.1 什么是任务委派

`delegate_to` 可以让某个任务不在当前目标主机上执行，而是委派给其他主机执行。

```yaml
- hosts: web
  tasks:
    - name: add web node to load balancer config
      lineinfile:
        path: /etc/haproxy/haproxy.cfg
        line: "server {{ inventory_hostname }} {{ ansible_host }}:80 check"
      delegate_to: lb01
```

应用场景：

- 部署 Web 节点时，自动更新负载均衡配置。
- 在控制端生成文件。
- 在监控服务器上注册新节点。

### 11.2 local_action

```yaml
- hosts: web
  tasks:
    - name: write deploy record on control node
      local_action:
        module: copy
        content: "{{ inventory_hostname }} deployed\n"
        dest: "/tmp/deploy-{{ inventory_hostname }}.log"
```

应用场景：

- 在控制端记录发布结果。
- 从控制端调用外部 API 或脚本。

### 11.3 run_once 配合委派

```yaml
- hosts: web
  tasks:
    - name: reload haproxy once
      systemd:
        name: haproxy
        state: reloaded
      delegate_to: lb01
      run_once: true
```

应用场景：

- 多台 Web 部署完成后，只重载一次负载均衡。
- 避免重复执行全局任务。

## 12. Vault 加密

### 12.1 Vault 是什么

Ansible Vault 用于加密敏感数据，如密码、Token、私钥。

加密文件：

```bash
ansible-vault encrypt secrets.yml
```

编辑加密文件：

```bash
ansible-vault edit secrets.yml
```

解密文件：

```bash
ansible-vault decrypt secrets.yml
```

执行 Playbook：

```bash
ansible-playbook site.yml --ask-vault-pass
```

应用场景：

- 保护数据库密码。
- 保护 SSH 密码、API Token。
- 避免敏感信息明文提交到仓库。

### 12.2 Vault 变量示例

`secrets.yml`：

```yaml
db_password: "StrongPassword"
```

Playbook：

```yaml
- hosts: db
  vars_files:
    - secrets.yml
  tasks:
    - debug:
        msg: "password length is {{ db_password | length }}"
```

应用场景：

- 敏感变量独立加密。
- 普通变量和敏感变量分文件管理。

## 13. Roles 角色

### 13.1 Roles 是什么

Role 是 Ansible 推荐的项目组织方式，用固定目录结构管理任务、变量、模板、文件、handlers。

常见结构：

```text
roles/
└── nginx/
    ├── defaults/
    │   └── main.yml
    ├── files/
    ├── handlers/
    │   └── main.yml
    ├── meta/
    │   └── main.yml
    ├── tasks/
    │   └── main.yml
    ├── templates/
    │   └── nginx.conf.j2
    └── vars/
        └── main.yml
```

应用场景：

- 部署逻辑复杂时拆分结构。
- 多项目复用同一个服务部署逻辑。
- 标准化团队自动化代码。

### 13.2 创建 Role

```bash
ansible-galaxy init roles/nginx
```

主 Playbook：

```yaml
- hosts: web
  become: true
  roles:
    - nginx
```

应用场景：

- 创建 Nginx、PHP、MySQL、NFS、Rsync 等可复用角色。

### 13.3 Role tasks 示例

`roles/nginx/tasks/main.yml`：

```yaml
- name: install nginx
  yum:
    name: nginx
    state: present

- name: render nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: restart nginx

- name: start nginx
  systemd:
    name: nginx
    state: started
    enabled: true
```

`roles/nginx/handlers/main.yml`：

```yaml
- name: restart nginx
  systemd:
    name: nginx
    state: restarted
```

应用场景：

- 服务安装、配置、启动流程固定化。
- 配置变更自动触发重启。

### 13.4 Role 依赖

`roles/web/meta/main.yml`：

```yaml
dependencies:
  - role: nginx
  - role: php-fpm
```

应用场景：

- Web 应用角色依赖 Nginx 和 PHP。
- 业务 Role 自动拉起基础服务 Role。

## 14. 综合实践案例

### 14.1 部署 NFS 服务

知识点：Inventory、yum、file、copy、systemd、handlers。

```yaml
- hosts: nfs
  become: true
  tasks:
    - name: install nfs
      yum:
        name: nfs-utils
        state: present

    - name: create share directory
      file:
        path: /data
        state: directory
        owner: nfsnobody
        group: nfsnobody
        mode: "0755"

    - name: configure exports
      copy:
        dest: /etc/exports
        content: "/data 10.0.0.0/24(rw,sync,all_squash)\n"
      notify: reload nfs

    - name: start nfs
      systemd:
        name: nfs-server
        state: started
        enabled: true

  handlers:
    - name: reload nfs
      systemd:
        name: nfs-server
        state: reloaded
```

应用场景：

- 部署共享存储。
- 给 Web 集群共享上传目录。

### 14.2 部署 Httpd 服务

```yaml
- hosts: web
  become: true
  tasks:
    - name: install httpd
      yum:
        name: httpd
        state: present

    - name: create index page
      copy:
        dest: /var/www/html/index.html
        content: "hello ansible\n"

    - name: start httpd
      systemd:
        name: httpd
        state: started
        enabled: true
```

应用场景：

- 快速部署静态站点。
- 验证 Playbook 基础流程。

### 14.3 LAMP 部署思路

```yaml
- hosts: lamp
  become: true
  tasks:
    - name: install lamp packages
      yum:
        name:
          - httpd
          - mariadb-server
          - php
          - php-mysqlnd
        state: present

    - name: start services
      systemd:
        name: "{{ item }}"
        state: started
        enabled: true
      loop:
        - httpd
        - mariadb

    - name: deploy php test page
      copy:
        dest: /var/www/html/index.php
        content: "<?php phpinfo(); ?>\n"
```

应用场景：

- 部署 PHP 应用运行环境。
- 学习多软件、多服务编排。

### 14.4 使用 Facts 批量修改主机名

Inventory：

```ini
[web]
web01 ansible_host=10.0.0.11
web02 ansible_host=10.0.0.12
```

Playbook：

```yaml
- hosts: web
  become: true
  tasks:
    - name: set hostname from inventory name
      hostname:
        name: "{{ inventory_hostname }}"
```

应用场景：

- 初始化新服务器。
- 让系统主机名和 Inventory 保持一致。

### 14.5 Nginx 负载均衡配置生成

变量：

```yaml
upstream_servers:
  - 10.0.0.11:80
  - 10.0.0.12:80
```

模板 `lb.conf.j2`：

```jinja2
upstream backend {
{% for server in upstream_servers %}
    server {{ server }};
{% endfor %}
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
    }
}
```

Playbook：

```yaml
- hosts: lb
  become: true
  vars_files:
    - vars/lb.yml
  tasks:
    - name: render load balance config
      template:
        src: lb.conf.j2
        dest: /etc/nginx/conf.d/lb.conf
      notify: reload nginx

  handlers:
    - name: reload nginx
      systemd:
        name: nginx
        state: reloaded
```

应用场景：

- 集群架构中自动生成 upstream。
- 扩容 Web 节点后统一更新负载均衡配置。

## 15. 学习建议与排错清单

### 15.1 推荐学习顺序

1. 先学 Inventory、SSH 连接、Ad-Hoc。
2. 掌握常用模块：`yum`、`copy`、`file`、`template`、`systemd`、`user`。
3. 学 Playbook YAML 结构和语法检查。
4. 学变量、Facts、register、when、loop、handlers。
5. 学 Jinja2 模板，把配置文件参数化。
6. 学 Vault 管理敏感信息。
7. 学 Roles，把服务部署拆成可复用结构。

### 15.2 常用排错命令

```bash
# 查看版本和配置路径
ansible --version

# 测试连通性
ansible all -m ping

# 查看 Inventory 解析结果
ansible-inventory --list

# Playbook 语法检查
ansible-playbook site.yml --syntax-check

# 模拟执行
ansible-playbook site.yml --check

# 显示详细日志
ansible-playbook site.yml -vvv
```

### 15.3 常见问题

| 问题 | 常见原因 | 处理方式 |
| --- | --- | --- |
| `UNREACHABLE` | SSH 不通、端口不对、认证失败 | 检查 `ansible_host`、用户、密钥、端口 |
| `Permission denied` | 用户权限不足 | 使用 `become: true` 或修复 sudo 权限 |
| YAML 语法错误 | 缩进错误、冒号后缺空格 | 用 `--syntax-check` 检查 |
| 变量未定义 | 变量文件未加载或变量名写错 | 用 `debug` 输出变量 |
| 模板渲染失败 | Jinja2 变量不存在 | 检查 `host_vars/group_vars` |
| 每次都 changed | 使用了 shell/command 检查任务 | 设置 `changed_when: false` |

### 15.4 生产 Playbook 建议

- 尽量使用模块，不要滥用 `shell`。
- 配置文件使用 `template`，不要硬编码多份。
- 服务重启放到 handlers。
- 敏感信息使用 Vault。
- 变量按环境、主机组、主机分层管理。
- 大项目使用 Roles 组织。
- 上线前执行 `--syntax-check` 和 `--check`。

