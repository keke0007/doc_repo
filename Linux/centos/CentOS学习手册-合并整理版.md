# CentOS Linux 学习手册

> 本文档由 `centos` 目录下原有 Markdown 笔记归纳、去重、重排后整理而成。内容按学习路径组织：基础入门 -> 文件管理 -> 用户权限 -> 软件与任务 -> 磁盘进程服务 -> 网络。每个知识点都补充了常用命令示例和典型应用场景。

## 目录

- [1. Linux 与 CentOS 基础](#1-linux-与-centos-基础)
- [2. Bash Shell 基础](#2-bash-shell-基础)
- [3. 文件系统与目录结构](#3-文件系统与目录结构)
- [4. 文件与目录管理](#4-文件与目录管理)
- [5. Vim 文件编辑](#5-vim-文件编辑)
- [6. IO 重定向与管道](#6-io-重定向与管道)
- [7. 文件查找](#7-文件查找)
- [8. 压缩与打包](#8-压缩与打包)
- [9. 用户与用户组管理](#9-用户与用户组管理)
- [10. 权限管理](#10-权限管理)
- [11. 软件包管理](#11-软件包管理)
- [12. 计划任务](#12-计划任务)
- [13. 磁盘与文件系统管理](#13-磁盘与文件系统管理)
- [14. 进程管理](#14-进程管理)
- [15. 系统服务与 systemd](#15-系统服务与-systemd)
- [16. 网络管理](#16-网络管理)
- [17. 常见学习实验](#17-常见学习实验)

## 1. Linux 与 CentOS 基础

### 1.1 Linux 是什么

知识点：Linux 是一个开源操作系统内核，日常所说的 Linux 系统通常指“Linux 内核 + GNU 工具 + 软件包管理 + 系统服务”的完整发行版。CentOS 是企业服务器环境中常见的 Red Hat 系发行版。

应用场景：服务器部署、Web 服务、数据库、中间件、容器、自动化运维、网络服务、日志分析。

代码示例：

```bash
# 查看内核版本
uname -r

# 查看系统发行版信息
cat /etc/redhat-release
cat /etc/os-release

# 查看系统架构
uname -m
```

### 1.2 操作系统的作用

知识点：操作系统负责管理 CPU、内存、磁盘、网络、进程、用户和权限，让上层应用不用直接操作硬件。

应用场景：排查服务器性能问题时，需要理解 CPU、内存、磁盘 IO、网络和进程之间的关系。

代码示例：

```bash
# 查看 CPU 信息
lscpu

# 查看内存信息
free -h

# 查看磁盘信息
lsblk

# 查看系统运行时间和负载
uptime
```

### 1.3 虚拟机快照与克隆

知识点：快照用于保存当前系统状态，便于实验失败后回滚；克隆用于快速复制一台相同环境的机器。

应用场景：学习分区、权限、网络、系统服务前先做快照；搭建多台测试服务器时使用克隆。

代码示例：

```bash
# 克隆后通常需要检查主机名和网络配置
hostnamectl set-hostname node01
nmcli connection show
ip addr
```

## 2. Bash Shell 基础

### 2.1 Bash 是什么

知识点：Bash 是 Linux 常用命令解释器，用户输入命令后由 Shell 解析并交给系统执行。

应用场景：日常运维、批量处理文件、编写自动化脚本。

代码示例：

```bash
# 查看当前使用的 Shell
echo $SHELL

# 查看系统支持的 Shell
cat /etc/shells

# 执行一条简单命令
date
```

### 2.2 命令格式

知识点：Linux 命令通常由“命令 + 选项 + 参数”组成。

应用场景：掌握命令结构后，可以快速理解陌生命令的帮助文档。

代码示例：

```bash
# 命令：ls，选项：-l -h，参数：/var/log
ls -lh /var/log

# 查看命令帮助
ls --help
man ls
```

### 2.3 历史记录、补全与别名

知识点：Tab 补全减少输入错误；history 查看历史命令；alias 可以给常用长命令起短名。

应用场景：提高命令行操作效率，减少重复输入。

代码示例：

```bash
# 查看历史命令
history

# 执行历史中的第 20 条命令
!20

# 设置临时别名
alias ll='ls -lh --color=auto'

# 永久生效可写入当前用户配置
echo "alias ll='ls -lh --color=auto'" >> ~/.bashrc
source ~/.bashrc
```

## 3. 文件系统与目录结构

### 3.1 常见系统目录

知识点：Linux 使用树形目录结构，一切从根目录 `/` 开始。

应用场景：排查配置、日志、软件、设备文件时，需要知道文件大概在哪个目录。

| 目录 | 作用 | 应用场景 |
| --- | --- | --- |
| `/bin`、`/usr/bin` | 普通命令 | 查找 `ls`、`cp` 等命令 |
| `/sbin`、`/usr/sbin` | 管理命令 | 管理网络、磁盘、服务 |
| `/etc` | 配置文件 | 修改服务、用户、网络配置 |
| `/home` | 普通用户家目录 | 保存用户个人文件 |
| `/root` | root 用户家目录 | 管理员工作目录 |
| `/var` | 可变数据 | 日志、缓存、队列 |
| `/dev` | 设备文件 | 磁盘、终端、随机设备 |
| `/proc` | 内核与进程虚拟文件 | 查看 CPU、内存、进程信息 |
| `/boot` | 启动文件 | 内核、grub 配置 |
| `/tmp` | 临时文件 | 临时测试、程序临时数据 |

代码示例：

```bash
# 查看根目录结构
ls /

# 查看日志目录
ls -lh /var/log

# 查看 CPU 信息的虚拟文件
cat /proc/cpuinfo | head
```

### 3.2 绝对路径与相对路径

知识点：绝对路径从 `/` 开始；相对路径基于当前目录。

应用场景：脚本中建议使用绝对路径，减少因工作目录不同导致的错误。

代码示例：

```bash
pwd
cd /var/log
ls messages
ls /var/log/messages
cd ..
cd -
```

## 4. 文件与目录管理

### 4.1 创建文件和目录

知识点：`touch` 创建空文件或更新时间戳；`mkdir` 创建目录。

应用场景：创建配置备份、初始化项目目录、准备测试文件。

代码示例：

```bash
touch app.log
touch file{1..5}.txt

mkdir /tmp/demo
mkdir -p /tmp/project/{conf,logs,data}

tree /tmp/project
```

### 4.2 复制、移动、删除

知识点：`cp` 复制，`mv` 移动或重命名，`rm` 删除。

应用场景：备份配置、发布文件、清理临时目录。

代码示例：

```bash
# 备份配置文件
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

# 复制目录
cp -a /etc/nginx /tmp/nginx.bak

# 重命名文件
mv app.log app.log.20260510

# 删除文件和目录
rm -f old.log
rm -rf /tmp/project
```

注意：`rm -rf` 风险很高，生产环境执行前先用 `ls` 确认目标路径。

### 4.3 查看文件内容

知识点：`cat` 一次性输出，`less` 分页查看，`head` 看开头，`tail` 看结尾，`tail -f` 实时跟踪。

应用场景：查看配置、检查日志、实时观察服务运行情况。

代码示例：

```bash
cat /etc/hosts
less /var/log/messages
head -n 20 /etc/passwd
tail -n 100 /var/log/secure
tail -f /var/log/messages
```

### 4.4 grep 过滤文本

知识点：`grep` 用于按关键字或正则表达式过滤文本。

应用场景：从日志中查找错误、从配置中定位参数。

代码示例：

```bash
# 查找包含 error 的日志行
grep -i "error" /var/log/messages

# 显示行号
grep -n "PermitRootLogin" /etc/ssh/sshd_config

# 递归查找目录中的关键字
grep -R "Listen" /etc/httpd/

# 排除注释和空行
grep -Ev '^#|^$' /etc/ssh/sshd_config
```

### 4.5 文本处理命令

知识点：`sort` 排序，`uniq` 去重，`cut` 截取字段，`wc` 统计行数/字数/字节数。

应用场景：统计日志 IP、分析用户列表、处理命令输出。

代码示例：

```bash
# 按用户名排序
cut -d: -f1 /etc/passwd | sort

# 统计重复 IP
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head

# 统计文件行数
wc -l /etc/passwd

# 取 /etc/passwd 的用户名和 UID
cut -d: -f1,3 /etc/passwd
```

### 4.6 链接文件

知识点：软链接类似快捷方式，可跨文件系统；硬链接指向同一个 inode，不能跨文件系统，通常不用于目录。

应用场景：软件版本切换、配置路径兼容、发布目录切换。

代码示例：

```bash
# 创建软链接
ln -s /opt/app/releases/v1 /opt/app/current

# 查看 inode
ls -li file.txt

# 创建硬链接
ln file.txt file.hard
```

## 5. Vim 文件编辑

### 5.1 Vim 三种常用模式

知识点：普通模式用于移动、复制、删除；插入模式用于输入内容；末行模式用于保存、退出、搜索替换。

应用场景：远程服务器上编辑配置文件。

代码示例：

```bash
vim /etc/hosts
```

常用按键：

| 操作 | 命令 |
| --- | --- |
| 进入插入模式 | `i`、`a`、`o` |
| 退出插入模式 | `Esc` |
| 保存退出 | `:wq` |
| 强制退出不保存 | `:q!` |
| 删除一行 | `dd` |
| 复制一行 | `yy` |
| 粘贴 | `p` |
| 撤销 | `u` |
| 搜索 | `/keyword` |
| 全文替换 | `:%s/old/new/g` |

### 5.2 Vim 应用示例

应用场景：修改 SSH 配置后重启服务。

代码示例：

```bash
vim /etc/ssh/sshd_config
grep -n "Port" /etc/ssh/sshd_config
systemctl restart sshd
```

## 6. IO 重定向与管道

### 6.1 标准输入、输出和错误

知识点：Linux 进程默认有标准输入 stdin、标准输出 stdout、标准错误 stderr。

应用场景：保存命令输出、记录脚本日志、丢弃无用错误。

代码示例：

```bash
# 覆盖写入
date > /tmp/time.log

# 追加写入
date >> /tmp/time.log

# 错误输出重定向
ls /not-exist 2> /tmp/error.log

# 正确和错误都写入同一文件
ls /etc /not-exist > /tmp/result.log 2>&1

# 丢弃输出
curl -s http://127.0.0.1 > /dev/null 2>&1
```

### 6.2 输入重定向

知识点：输入重定向让命令从文件或多行文本读取输入。

应用场景：批量导入数据、脚本自动输入内容。

代码示例：

```bash
# 从文件读取
wc -l < /etc/passwd

# 多行输入
cat > /tmp/server.txt << EOF
web01 10.0.0.11
web02 10.0.0.12
EOF
```

### 6.3 管道

知识点：管道 `|` 将前一个命令的输出交给后一个命令处理。

应用场景：组合多个小工具完成复杂文本处理。

代码示例：

```bash
# 查看 sshd 进程
ps aux | grep sshd | grep -v grep

# 统计系统用户数量
cat /etc/passwd | wc -l

# 查看占用内存最高的 5 个进程
ps aux --sort=-%mem | head -n 6

# tee 同时输出到屏幕和文件
df -h | tee /tmp/df.log
```

## 7. 文件查找

### 7.1 find 基础

知识点：`find` 可以按名称、类型、大小、时间、用户、权限查找文件。

应用场景：定位日志、清理临时文件、查找异常权限文件。

代码示例：

```bash
# 按名称查找
find /etc -name "*.conf"

# 忽略大小写
find /var/log -iname "*.log"

# 按类型查找，f 文件，d 目录，l 链接
find /tmp -type f
find /tmp -type d

# 查找大于 100M 的文件
find / -type f -size +100M

# 查找 7 天前修改过的日志
find /var/log -type f -mtime +7 -name "*.log"
```

### 7.2 find 动作处理

知识点：`-exec`、`xargs` 可以对查找到的文件继续执行命令。

应用场景：批量删除过期日志、批量修改权限、批量搜索文件内容。

代码示例：

```bash
# 删除 30 天前的临时文件
find /tmp -type f -mtime +30 -delete

# 批量修改权限
find /var/www/html -type f -exec chmod 644 {} \;
find /var/www/html -type d -exec chmod 755 {} \;

# 查找包含 root 的配置文件
find /etc -type f -name "*.conf" | xargs grep -n "root"
```

## 8. 压缩与打包

### 8.1 gzip

知识点：`gzip` 适合压缩单个文件，压缩后默认删除原文件。

应用场景：压缩单个大日志文件。

代码示例：

```bash
gzip access.log
gunzip access.log.gz
gzip -c access.log > access.log.gz
```

### 8.2 zip/unzip

知识点：zip 格式跨平台友好，Windows 和 Linux 都容易处理。

应用场景：和 Windows 用户交换文件。

代码示例：

```bash
yum install -y zip unzip
zip file.zip file.txt
zip -r project.zip project/
unzip project.zip
unzip project.zip -d /opt/project
unzip -l project.zip
```

### 8.3 tar

知识点：`tar` 常用于目录打包，可配合 gzip 或 xz 压缩。

应用场景：备份配置目录、迁移项目目录、发布代码包。

代码示例：

```bash
# 打包并 gzip 压缩
tar -czf etc-backup.tar.gz /etc

# 查看压缩包内容
tar -tf etc-backup.tar.gz | head

# 解压到指定目录
tar -xzf etc-backup.tar.gz -C /tmp

# 排除日志目录
tar --exclude='*.log' -czf app.tar.gz /opt/app
```

## 9. 用户与用户组管理

### 9.1 用户基础

知识点：Linux 用 UID 标识用户，用 GID 标识用户组。root 的 UID 为 0，普通用户通常从 1000 开始。

应用场景：区分管理员、服务用户、普通用户权限。

代码示例：

```bash
id
id nginx
whoami
who
w

# 用户配置文件
head /etc/passwd
head /etc/shadow
head /etc/group
```

### 9.2 创建、修改、删除用户

知识点：`useradd` 创建用户，`usermod` 修改用户，`userdel` 删除用户，`passwd` 设置密码。

应用场景：为开发人员创建账号，为服务创建专用运行用户。

代码示例：

```bash
# 创建普通用户
useradd alice
passwd alice

# 创建不能登录的服务用户
useradd -r -s /sbin/nologin nginx

# 修改用户家目录和 shell
usermod -d /home/newalice -m alice
usermod -s /bin/bash alice

# 删除用户及家目录
userdel -r alice
```

### 9.3 用户组管理

知识点：组用于批量授权。一个用户有一个主组，也可以加入多个附加组。

应用场景：让多个用户共同管理同一个项目目录。

代码示例：

```bash
groupadd devops
usermod -aG devops alice
id alice

groupmod -n ops devops
groupdel ops
```

### 9.4 su 与 sudo

知识点：`su` 切换身份，`sudo` 临时提权执行命令。生产环境更推荐使用 sudo 留审计记录。

应用场景：普通用户执行特定管理命令，例如重启服务。

代码示例：

```bash
# 切换到 root
su -

# 用 sudo 执行命令
sudo systemctl restart nginx

# 编辑 sudo 权限
visudo
```

sudo 配置示例：

```text
alice ALL=(root) /bin/systemctl restart nginx, /bin/systemctl reload nginx
```

## 10. 权限管理

### 10.1 rwx 权限含义

知识点：文件权限分为属主、属组、其他人三组。`r` 读，`w` 写，`x` 执行。

应用场景：控制用户能否查看、修改、执行文件。

代码示例：

```bash
ls -l /etc/passwd

# 数字权限
chmod 644 file.txt
chmod 755 script.sh

# 符号权限
chmod u+x script.sh
chmod g+w data.txt
chmod o-r secret.txt
```

### 10.2 文件权限与目录权限区别

知识点：对文件，`r` 表示读内容，`w` 表示改内容，`x` 表示执行；对目录，`r` 表示列出文件名，`w` 表示增删改目录项，`x` 表示进入目录。

应用场景：Web 目录、共享目录、日志目录授权时必须区分文件和目录权限。

代码示例：

```bash
mkdir /data/share
touch /data/share/a.txt

# 目录可进入、可列出，文件可读写
chmod 755 /data/share
chmod 644 /data/share/a.txt
```

### 10.3 chown 与 chgrp

知识点：`chown` 修改属主或属组，`chgrp` 修改属组。

应用场景：部署 Web 文件后，把目录交给 nginx/apache 用户读取。

代码示例：

```bash
chown nginx:nginx /var/www/html/index.html
chown -R nginx:nginx /var/www/html
chgrp devops /data/share
```

### 10.4 特殊权限 SUID、SGID、SBIT

知识点：SUID 让可执行文件以文件属主身份运行；SGID 作用于目录时，新建文件继承目录属组；SBIT 常用于公共目录，用户只能删除自己的文件。

应用场景：`passwd` 命令使用 SUID 修改密码；项目共享目录用 SGID；`/tmp` 使用 SBIT。

代码示例：

```bash
# 查看 passwd 的 SUID
ls -l /usr/bin/passwd

# 给共享目录设置 SGID
mkdir /data/project
chgrp devops /data/project
chmod 2775 /data/project

# 给公共目录设置粘滞位
mkdir /data/public
chmod 1777 /data/public
```

### 10.5 特殊属性 chattr

知识点：`chattr +i` 让文件不可修改、删除、重命名；`chattr +a` 让文件只能追加。

应用场景：保护重要配置、防止日志被覆盖。

代码示例：

```bash
# 设置不可变
chattr +i /etc/passwd
lsattr /etc/passwd
chattr -i /etc/passwd

# 日志只能追加
chattr +a /var/log/secure
lsattr /var/log/secure
```

### 10.6 默认权限 umask

知识点：umask 决定新建文件和目录的默认权限。文件默认基准是 666，目录默认基准是 777。

应用场景：控制用户新建文件是否默认对组可写。

代码示例：

```bash
umask
umask 022
touch a.txt
mkdir a.dir
ls -ld a.txt a.dir
```

## 11. 软件包管理

### 11.1 RPM

知识点：RPM 是 Red Hat 系系统的底层软件包格式和管理工具，但不会自动解决复杂依赖。

应用场景：安装本地 rpm 包、查询某个文件属于哪个包。

代码示例：

```bash
# 安装、升级、卸载
rpm -ivh package.rpm
rpm -Uvh package.rpm
rpm -e package-name

# 查询
rpm -qa | grep nginx
rpm -qi bash
rpm -ql bash
rpm -qf /bin/bash
rpm -qc openssh-server
```

### 11.2 YUM/DNF

知识点：YUM 和 DNF 能从软件仓库安装软件并自动处理依赖。CentOS 7 常用 `yum`，CentOS 8 常用 `dnf`，多数命令写法相近。

应用场景：安装服务、升级软件、查询软件来源、回滚安装操作。

代码示例：

```bash
# 查询
yum list nginx
yum list installed
yum provides "*/semanage"

# 安装、重装、更新、删除
yum install -y nginx
yum reinstall -y nginx
yum update -y nginx
yum remove -y nginx

# 包组
yum groups list
yum groups install -y "Development Tools"

# 历史记录
yum history
yum history undo 10
```

### 11.3 YUM 源配置

知识点：YUM 源定义软件包下载地址和元数据位置。

应用场景：使用国内镜像源、搭建内网仓库、离线安装软件。

代码示例：

```bash
# 查看启用的仓库
yum repolist

# 添加本地源示例
cat > /etc/yum.repos.d/local.repo << EOF
[local]
name=local repo
baseurl=file:///mnt
enabled=1
gpgcheck=0
EOF

yum clean all
yum makecache
```

### 11.4 源码编译安装

知识点：源码安装通常经历下载、解压、配置、编译、安装。

应用场景：需要定制编译参数，或仓库没有需要的版本。

代码示例：

```bash
yum groups install -y "Development Tools"
yum install -y pcre-devel openssl-devel zlib-devel

tar -xzf nginx-1.24.0.tar.gz
cd nginx-1.24.0
./configure --prefix=/usr/local/nginx --with-http_ssl_module
make
make install
```

## 12. 计划任务

### 12.1 crond 基础

知识点：crond 用于周期性执行任务。用户级任务用 `crontab -e`，系统级任务可放入 `/etc/crontab` 或 `/etc/cron.d/`。

应用场景：定时备份、日志清理、定时巡检、定时同步。

代码示例：

```bash
systemctl status crond
systemctl enable --now crond

# 编辑当前用户计划任务
crontab -e

# 查看当前用户计划任务
crontab -l
```

### 12.2 crontab 时间格式

知识点：格式为“分钟 小时 日期 月份 星期 命令”。

应用场景：精确控制任务执行周期。

代码示例：

```cron
# 每天凌晨 2 点备份 /etc
0 2 * * * /usr/bin/tar -czf /backup/etc-$(date +\%F).tar.gz /etc

# 每 5 分钟检查一次 nginx
*/5 * * * * /usr/bin/systemctl is-active nginx >/dev/null || /usr/bin/systemctl restart nginx

# 每周日 3 点清理 30 天前日志
0 3 * * 0 /usr/bin/find /var/log -type f -name "*.log" -mtime +30 -delete
```

注意：cron 中 `%` 有特殊含义，命令里的日期格式需要写成 `\%F`。

### 12.3 计划任务排错

知识点：cron 环境变量很少，脚本中建议使用绝对路径，并把输出重定向到日志。

应用场景：任务手动执行成功，但定时执行失败。

代码示例：

```bash
# 查看 cron 日志
grep CRON /var/log/cron

# 脚本中设置 PATH
cat > /opt/scripts/backup.sh << 'EOF'
#!/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
tar -czf /backup/etc-$(date +%F).tar.gz /etc
EOF

chmod +x /opt/scripts/backup.sh
```

## 13. 磁盘与文件系统管理

### 13.1 查看磁盘

知识点：磁盘设备通常位于 `/dev`，常见命名如 `/dev/sda`、`/dev/vda`、`/dev/nvme0n1`。

应用场景：新加磁盘、扩容、排查空间不足。

代码示例：

```bash
lsblk
fdisk -l
df -h
du -sh /var/log/*
blkid
```

### 13.2 分区与格式化

知识点：分区后需要创建文件系统才能挂载使用。CentOS 常见文件系统为 XFS 和 EXT4。

应用场景：新磁盘挂载到 `/data`。

代码示例：

```bash
# 使用 fdisk 对 /dev/sdb 分区
fdisk /dev/sdb

# 格式化
mkfs.xfs /dev/sdb1

# 创建挂载点并挂载
mkdir /data
mount /dev/sdb1 /data
df -h /data
```

### 13.3 临时挂载与永久挂载

知识点：`mount` 是临时挂载，重启会失效；写入 `/etc/fstab` 可永久挂载。

应用场景：服务器重启后自动挂载数据盘。

代码示例：

```bash
# 获取 UUID
blkid /dev/sdb1

# 写入 /etc/fstab，建议用 UUID
echo "UUID=xxxx-xxxx /data xfs defaults 0 0" >> /etc/fstab

# 测试 fstab 是否正确
mount -a
df -h
```

### 13.4 SWAP

知识点：SWAP 是磁盘上的交换空间，内存不足时可临时缓冲，但性能远低于内存。

应用场景：小内存机器防止进程因内存不足立即退出。

代码示例：

```bash
# 创建 2G swap 文件
dd if=/dev/zero of=/swapfile bs=1M count=2048
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
swapon --show

# 永久生效
echo "/swapfile swap swap defaults 0 0" >> /etc/fstab
```

### 13.5 LVM

知识点：LVM 把物理磁盘抽象为 PV、VG、LV，便于在线扩容和灵活分配空间。

应用场景：业务数据目录需要后续扩容。

代码示例：

```bash
# 创建 PV、VG、LV
pvcreate /dev/sdb
vgcreate datavg /dev/sdb
lvcreate -n datalv -L 10G datavg

# 格式化并挂载
mkfs.xfs /dev/datavg/datalv
mkdir /data
mount /dev/datavg/datalv /data

# 扩容 LV 和 XFS 文件系统
lvextend -L +5G /dev/datavg/datalv
xfs_growfs /data
```

### 13.6 RAID

知识点：RAID 通过多块磁盘组合提升容量、性能或可靠性。常见有 RAID0、RAID1、RAID5、RAID10。

应用场景：RAID1 用于镜像冗余，RAID5 兼顾容量和容错，RAID10 兼顾性能和可靠性。

代码示例：

```bash
yum install -y mdadm

# 创建 RAID1
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
cat /proc/mdstat
mdadm --detail /dev/md0

mkfs.xfs /dev/md0
mkdir /raid
mount /dev/md0 /raid
```

## 14. 进程管理

### 14.1 程序、进程与线程

知识点：程序是静态文件；进程是运行中的程序；线程是进程内的执行单元。

应用场景：排查 CPU、内存、服务卡死、僵尸进程等问题。

代码示例：

```bash
ps aux
ps -ef
pstree -p
pgrep sshd
```

### 14.2 进程状态监控

知识点：`ps` 静态查看进程，`top` 动态查看进程，`pidstat` 可分析 CPU、内存、IO。

应用场景：找出占用资源最高的进程。

代码示例：

```bash
# CPU 占用最高
ps aux --sort=-%cpu | head

# 内存占用最高
ps aux --sort=-%mem | head

# 动态监控
top

# 每秒查看一次进程统计
pidstat 1
```

### 14.3 kill 与信号

知识点：`kill` 给进程发送信号，常用 `TERM` 优雅终止、`KILL` 强制终止、`HUP` 重载配置。

应用场景：停止异常进程，重载服务配置。

代码示例：

```bash
# 查看信号
kill -l

# 优雅停止
kill -15 PID

# 强制停止
kill -9 PID

# 按名称停止
pkill nginx
killall nginx

# 重载 nginx
kill -HUP $(cat /run/nginx.pid)
```

### 14.4 进程优先级 nice/renice

知识点：nice 值范围通常为 -20 到 19，值越小优先级越高。

应用场景：降低批处理任务优先级，避免影响线上服务。

代码示例：

```bash
# 低优先级启动任务
nice -n 10 tar -czf backup.tar.gz /data

# 修改已运行进程优先级
renice 10 -p PID

# 查看优先级
ps -o pid,ni,comm -p PID
```

### 14.5 后台任务

知识点：`&` 后台运行，`nohup` 断开终端后继续运行，`screen`/`tmux` 用于长期会话。

应用场景：远程执行耗时任务，避免 SSH 断开导致任务中断。

代码示例：

```bash
# 后台运行
sh long_task.sh &

# 断开终端继续运行
nohup sh long_task.sh > /tmp/task.log 2>&1 &

# 查看后台任务
jobs
fg %1
bg %1
```

### 14.6 平均负载

知识点：平均负载表示处于可运行和不可中断状态的任务平均数量。通常与 CPU 核数对比判断压力。

应用场景：判断系统是 CPU 忙、IO 慢，还是进程排队过多。

代码示例：

```bash
uptime
nproc
mpstat 1
iostat -x 1
pidstat -u 1
pidstat -d 1
```

## 15. 系统服务与 systemd

### 15.1 systemd 基础

知识点：systemd 是 CentOS 7 以后常用的系统和服务管理器，服务单元通常是 `.service` 文件。

应用场景：启动、停止、重启、开机自启服务。

代码示例：

```bash
systemctl status sshd
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl reload nginx
systemctl enable nginx
systemctl disable nginx
systemctl is-enabled nginx
```

### 15.2 运行级别与 target

知识点：systemd 使用 target 替代传统运行级别。常用 `multi-user.target` 和 `graphical.target`。

应用场景：服务器通常使用命令行多用户模式。

代码示例：

```bash
# 查看默认 target
systemctl get-default

# 设置命令行模式
systemctl set-default multi-user.target

# 设置图形模式
systemctl set-default graphical.target
```

### 15.3 自定义 systemd 服务

知识点：可以为源码安装的软件编写 service 文件，纳入 systemd 管理。

应用场景：让自编译 nginx、脚本服务支持开机自启和统一管理。

代码示例：

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My App Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/myapp
Restart=always
User=nobody

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
systemctl enable --now myapp
systemctl status myapp
```

### 15.4 系统优化常见项

知识点：服务器初始化常包含源配置、时间同步、防火墙策略、资源限制等。

应用场景：新服务器上线前标准化配置。

代码示例：

```bash
# 查看防火墙
systemctl status firewalld

# 查看 SELinux 状态
getenforce

# 查看资源限制
ulimit -a

# 临时调整最大打开文件数
ulimit -n 65535
```

## 16. 网络管理

### 16.1 网络基础

知识点：主机通信依赖 IP、子网掩码、网关、DNS、端口和协议。常见协议有 TCP、UDP、ICMP、HTTP、SSH。

应用场景：配置服务器 IP、排查无法访问、定位 DNS 或路由问题。

代码示例：

```bash
ip addr
ip link
ip route
ss -lntup
ping -c 4 223.5.5.5
curl -I http://example.com
```

### 16.2 TCP 与 UDP

知识点：TCP 面向连接，可靠传输，有三次握手和四次挥手；UDP 无连接，开销低但不保证可靠。

应用场景：Web、SSH、数据库多用 TCP；DNS、音视频、部分监控采集常用 UDP。

代码示例：

```bash
# 查看 TCP 监听端口
ss -lnt

# 查看 UDP 监听端口
ss -lnu

# 查看连接状态统计
ss -ant | awk 'NR>1 {print $1}' | sort | uniq -c
```

### 16.3 nmcli 管理网络

知识点：CentOS 常用 NetworkManager 管理网络，`nmcli` 是命令行工具。

应用场景：配置静态 IP、DNS、网关，激活或禁用网卡连接。

代码示例：

```bash
# 查看设备和连接
nmcli device status
nmcli connection show

# 配置静态 IP
nmcli connection modify eth0 ipv4.method manual ipv4.addresses 10.0.0.10/24 ipv4.gateway 10.0.0.2 ipv4.dns 223.5.5.5
nmcli connection up eth0

# 改回 DHCP
nmcli connection modify eth0 ipv4.method auto
nmcli connection up eth0
```

### 16.4 网卡配置文件

知识点：CentOS 7 常见网卡配置文件位于 `/etc/sysconfig/network-scripts/ifcfg-*`。

应用场景：批量配置网络、排查开机网络未自动启动。

代码示例：

```ini
# /etc/sysconfig/network-scripts/ifcfg-eth0
TYPE=Ethernet
BOOTPROTO=none
NAME=eth0
DEVICE=eth0
ONBOOT=yes
IPADDR=10.0.0.10
PREFIX=24
GATEWAY=10.0.0.2
DNS1=223.5.5.5
```

```bash
systemctl restart NetworkManager
nmcli connection reload
nmcli connection up eth0
```

### 16.5 路由

知识点：路由决定数据包下一跳。默认路由用于访问非本地网段。

应用场景：多网卡服务器、跨网段通信、内网专线路由。

代码示例：

```bash
# 查看路由
ip route

# 临时添加网络路由
ip route add 192.168.10.0/24 via 10.0.0.1 dev eth0

# 临时添加默认路由
ip route add default via 10.0.0.2

# 删除路由
ip route del 192.168.10.0/24
```

### 16.6 Bond 网卡绑定

知识点：Bond 可把多块网卡绑定成一个逻辑网卡。常见模式：`mode=0` 轮询负载均衡，`mode=1` 主备高可用。

应用场景：提升网络可靠性或带宽。

代码示例：

```bash
# 创建 bond0 主备模式
nmcli connection add type bond con-name bond0 ifname bond0 mode active-backup
nmcli connection add type ethernet con-name bond0-slave1 ifname eth0 master bond0
nmcli connection add type ethernet con-name bond0-slave2 ifname eth1 master bond0

nmcli connection modify bond0 ipv4.method manual ipv4.addresses 10.0.0.20/24 ipv4.gateway 10.0.0.2
nmcli connection up bond0

# 查看 bond 状态
cat /proc/net/bonding/bond0
```

### 16.7 网络内核参数

知识点：高并发场景可能需要调整端口范围、连接队列、TIME_WAIT 等参数。

应用场景：Web 服务器、代理服务器、负载均衡器调优。

代码示例：

```bash
# 查看参数
sysctl net.ipv4.ip_local_port_range
sysctl net.ipv4.tcp_tw_reuse
sysctl net.core.somaxconn

# 临时修改
sysctl -w net.ipv4.ip_local_port_range="10000 65000"
sysctl -w net.core.somaxconn=4096

# 永久配置
cat >> /etc/sysctl.conf << EOF
net.ipv4.ip_local_port_range = 10000 65000
net.core.somaxconn = 4096
EOF

sysctl -p
```

## 17. 常见学习实验

### 17.1 创建一个 Web 目录并正确授权

应用场景：部署静态网站。

代码示例：

```bash
yum install -y httpd
systemctl enable --now httpd

mkdir -p /var/www/html/demo
echo "hello centos" > /var/www/html/demo/index.html
chown -R apache:apache /var/www/html/demo
find /var/www/html/demo -type d -exec chmod 755 {} \;
find /var/www/html/demo -type f -exec chmod 644 {} \;

curl http://127.0.0.1/demo/
```

### 17.2 定时备份并保留 7 天

应用场景：备份重要配置文件。

代码示例：

```bash
mkdir -p /backup

cat > /opt/backup-etc.sh << 'EOF'
#!/bin/bash
set -e
BACKUP_DIR=/backup
tar -czf ${BACKUP_DIR}/etc-$(date +%F).tar.gz /etc
find ${BACKUP_DIR} -type f -name "etc-*.tar.gz" -mtime +7 -delete
EOF

chmod +x /opt/backup-etc.sh

# crontab -e 添加：
# 0 2 * * * /opt/backup-etc.sh >> /var/log/backup-etc.log 2>&1
```

### 17.3 查找并清理大日志

应用场景：磁盘空间告警。

代码示例：

```bash
df -h
du -sh /var/log/* | sort -h
find /var/log -type f -size +100M -name "*.log"

# 压缩历史日志
find /var/log -type f -name "*.log" -mtime +3 -exec gzip {} \;

# 删除 30 天前压缩日志
find /var/log -type f -name "*.gz" -mtime +30 -delete
```

### 17.4 排查服务无法访问

应用场景：用户反馈网站打不开。

代码示例：

```bash
# 1. 服务是否运行
systemctl status nginx

# 2. 端口是否监听
ss -lntup | grep ':80'

# 3. 本机访问是否正常
curl -I http://127.0.0.1

# 4. 防火墙是否放行
firewall-cmd --list-all

# 5. 日志是否有错误
tail -n 100 /var/log/nginx/error.log
```

### 17.5 新磁盘挂载到 /data

应用场景：给业务数据单独挂载磁盘。

代码示例：

```bash
lsblk
fdisk /dev/sdb
mkfs.xfs /dev/sdb1
mkdir /data
mount /dev/sdb1 /data
blkid /dev/sdb1

# 将 UUID 写入 /etc/fstab 后测试
mount -a
df -h /data
```

## 学习建议

1. 先熟悉目录结构、文件管理、Vim、重定向和管道，这是后续所有 Linux 操作的基础。
2. 用户、权限、软件包、计划任务是日常运维高频内容，建议每个命令都在虚拟机中练习。
3. 磁盘、进程、服务、网络属于服务器管理核心能力，学习时要结合真实故障场景。
4. 任何涉及删除、格式化、分区、权限收紧、网络变更的命令，先在测试机练习，再在生产环境执行。
