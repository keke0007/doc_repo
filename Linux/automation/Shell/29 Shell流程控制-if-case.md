# 1.if判断语句

## 目录

  - [1.1 什么是if](#1.1-什么是if)
  - [1.2 为何要使用if](#1.2-为何要使用if)
  - [1.3 if的基础语法](#1.3-if的基础语法)
    - [1.3.1 单分支结构](#1.3.1-单分支结构)
    - [1.3.2 双分支结构](#1.3.2-双分支结构)
    - [1.3.3 多分支结构](#1.3.3-多分支结构)
  - [1.4 if分支场景](#1.4-if分支场景)
    - [1.4.1 单分支脚本案例](#1.4.1-单分支脚本案例)
    - [1.4.2 双分支脚本案例](#1.4.2-双分支脚本案例)
    - [1.4.3 多分支脚本案例](#1.4.3-多分支脚本案例)
  - [1.5 if基于文件比较](#1.5-if基于文件比较)
    - [1.5.1 备份脚本案例-1](#1.5.1-备份脚本案例-1)
    - [1.5.2 备份脚本案例-2](#1.5.2-备份脚本案例-2)
    - [1.5.3 备份脚本案例-3](#1.5.3-备份脚本案例-3)
    - [1.5.4 为执行脚本添加锁](#1.5.4-为执行脚本添加锁)
  - [1.6 if基于整数比较](#1.6-if基于整数比较)
    - [1.6.1 检查服务状态脚本](#1.6.1-检查服务状态脚本)
    - [1.6.2 配置yum仓库脚本](#1.6.2-配置yum仓库脚本)
- [2.根据不同的系统配置不同yum源](#2.根据不同的系统配置不同yum源)
    - [1.6.3 获取进程详情脚本](#1.6.3-获取进程详情脚本)
- [3.还需要该服务的active (running)](#3.还需要该服务的active-running)
    - [1.6.4 数字排序脚本](#1.6.4-数字排序脚本)
  - [1.7 if基于字符比较](#1.7-if基于字符比较)
    - [1.7.1 检查执行身份是否为root](#1.7.1-检查执行身份是否为root)
    - [1.7.2 判断用户输入是否为空脚本](#1.7.2-判断用户输入是否为空脚本)
    - [1.7.2 检查selinux是否为disabled](#1.7.2-检查selinux是否为disabled)
  - [1.8 if基于正则比较](#1.8-if基于正则比较)
    - [1.8.1 控制用户输入必须为整数脚本](#1.8.1-控制用户输入必须为整数脚本)
    - [1.8.2 遍历单词筛选其中关键字脚本](#1.8.2-遍历单词筛选其中关键字脚本)
    - [1.8.3 批量创建用户脚本](#1.8.3-批量创建用户脚本)
- [2 if判断相关脚本案例](#2-if判断相关脚本案例)
  - [2.1 备份脚本场景](#2.1-备份脚本场景)
- [1.判断操作的用户身份](#1.判断操作的用户身份)
- [2.加锁](#2.加锁)
- [4.判断打包是否成功](#4.判断打包是否成功)
  - [2.2 创建随机密码脚本](#2.2-创建随机密码脚本)
- [3.case判断语句](#3.case判断语句)
  - [3.1 什么是case](#3.1-什么是case)
  - [3.2 case使用场景](#3.2-case使用场景)
  - [3.3 case基础语法](#3.3-case基础语法)
- [1|backup)](#1backup)
- [2|copy)](#2copy)
- [3|quit)](#3quit)
  - [3.4 case场景实践](#3.4-case场景实践)
    - [3.4.1 场景1-编写nginx启停脚本](#3.4.1-场景1-编写nginx启停脚本)
    - [3.4.2 场景2-编写rsync启停脚本](#3.4.2-场景2-编写rsync启停脚本)
    - [3.4.3 场景3-编写LVS启停脚本](#3.4.3-场景3-编写lvs启停脚本)

```
case
```
徐亮伟, 江湖人称标杆徐。多年互联网运维工作经 验，曾负责过大规模集群架构自动化运维管理工作。 擅长Web集群架构与自动化运维，曾负责国内某大型 电商运维工作。 个人博客"徐亮伟架构师之路"累计受益数万人。 笔者Q:552408925、572891887 架构师群:471443208

# 1.if判断语句

### 1.1 什么是if

if就是模仿人类的判断来进行的，但它没有人类那么 有情感，只有 True 和 False 这两种结果。

### 1.2 为何要使用if

当我们在写程序的时候，经常需要对上一步的执行结 果进行判断，那么判断就需要使用if语句来实现。 if语句在我们程序中主要就是用来做判断的； 不管大家以后学习什么语言，以后只要涉及到判断的 部分，大家就可以直接拿if来使用， 不同的语言之间的 if只是语法不同，原理是相同的。

### 1.3 if的基础语法

#### 1.3.1 单分支结构

单分支语法 单分支代码示例 if [ 如果你有房 ];then if [ $1 -eq $2

```
];then
```
我就嫁给你
```
echo "ok"
fi
fi
```
#### 1.3.2 双分支结构

双分支语法 双分支代码示例 if [ 如果你有房 ];then if [ $1 -eq $2

```
];then
```
我就嫁给你
```
echo "ok!"
else
else 再见
echo "error!"
fi
fi
```
#### 1.3.3 多分支结构

多分支结构 多分支代码示例 if [ 如果你有房 ];then 我就嫁给你 elif [ 你有车 ];then 我就嫁给你 elif [ 你有钱 ];then 我就嫁给你 else 再见 fi

### 1.4 if分支场景

#### 1.4.1 单分支脚本案例

需求：判断当前用户是不是root，如果不是那么返回 ERROR

```
[root@oldxu shell]# cat if-1.sh
if [ $USER != 'root' ];then
echo "ERROR" exit 1
fi
```
#### 1.4.2 双分支脚本案例

需求：判断当前登录用户是管理员还是普通用户 如果是管理员输出 hey admin 如果是普通用户输出hey guest

```
[root@oldxu shell]# cat if-2.sh
if [ $USER == 'root' ];then
echo "hey admin"
else
echo "hey guest"
fi
```
#### 1.4.3 多分支脚本案例

需求：通过脚本传入两个参数，进行整数关系比较。 比如： if.sh [ 1 2 | 2 2 |  2 3 ]，请使用双 分支和多分支两种方式实现。

#双分支，嵌套if方式实现

```
[root@web01 ~]# cat if-3.sh
#!/usr/bin/bash
if [ $1 -eq $2 ];then
    echo "$1 = $2"
    if [ $1 -gt $2 ];then
        echo "$1 > $2"

        echo "$1 < $2"
fi 多分支结构实现
[root@web01 ~]# cat if-4.sh
#!/usr/bin/bash
if [ $1 -eq $2 ];then
    echo "$1 = $2"
elif  [ $1 -gt $2 ];then
    echo "$1 > $2"
    echo "$1 < $2"

### 1.5 if基于文件比较

if语句中的文件比较 -e：如果文件或目录存在则为真

-s：如果文件存在且至少有一个字符则为真 -d：如果文件存在且为目录则为真 -f：如果文件存在且为普通文件则为真 -r：如果文件存在且可读则为真 -w：如果文件存在且可写则为真 -x：如果文件存在且可执行则为真

#### 1.5.1 备份脚本案例-1

需求1：备份文件至/backup/system/filename- 2019-10-29，如果该目录不存在则自动创建。 1.源文件，让用户手动输入； 2.目标位置：/backup/system/   判断， 判断该目录 是否存在，如果不存在则创建；

[root@manager if]# cat if-14.sh
#!/bin/bash
Dest=/backup/system
Date=$(date +%F)
read -p "请输入备份源: " Src
if [ ! -d $Dest ];then
    mkdir $Dest
cp -rpv $Src $Dest/filename-$Date

#### 1.5.2 备份脚本案例-2

需求2：继需求1，判断备份的文件是否存在， 如果备份文件存在则继续； 如果备份文件不存在则提示 No such file or directory，然后退出；

[root@manager if]# cat if-14.sh
#!/bin/bash
Dest=/backup/system
Date=$(date +%F)
read -p "请输入备份源: " Src # 判断用户输入的路径是否存在,是否是一个文件
if [ ! -f $Src ];then
    echo "$Src No such file or directory"
```
exit
```
fi
if [ ! -d $Dest ];then
    mkdir $Dest
cp -rpv $Src $Dest/filename-$Date

#### 1.5.3 备份脚本案例-3

需求3：继需求1、2，判断备份的文件是否为空， 如果备份文件不为空则继续；

如果备份文件为空则提示 "This is file empty"，然后退出；

[root@manager if]# cat if-14.sh
#!/bin/bash
Dest=/backup/system
Date=$(date +%F)
read -p "请输入备份源: " Src #1.判断用户输入的路径是否存在,是否是一个文件
if [ ! -f $Src ];then
    echo "$Src No such file or directory"
```
exit fi #2.判断文件为空,则报错

```
if [ ! -s $Src ];then
    echo "$Src This is file empty"
```
exit fi #3.备份源没有问题,则创建备份的目录

```
if [ ! -d $Dest ];then
    mkdir $Dest
cp -rpv $Src $Dest/filename-$Date

#### 1.5.4 为执行脚本添加锁

root --》1--》a.sh root--》2--》a.sh # 判断是否存在锁

if [ -f /tmp/test.lock ];then
echo "该脚本正在运行，请稍后..." exit fi # 加锁
touch /tmp/test.lock
```
#####业务逻辑 sleep 20 # 解锁

```
if [ -f /tmp/test.lock ];then
    rm -f /tmp/test.lock
```
### 1.6 if基于整数比较

if语句中的整数比较 [ 整数1 操作符 整数2 ] -eq：等于则条件为真，示例：[ 1 -eq 10 ] if [ 1 -le 1 ];then echo "成立" else echo "不成立" fi -ne：不等于则条件为真，示例：[ 1 -ne 10 ] -gt：大于则条件为真，示例：[ 1 -gt 10 ] -lt：小于则条件为真，示例：[ 1 -lt 10 ] -ge：大于等于则条件为真，示例：[ 1 -ge 10 ] -le：小于等于则条件为真，示例：[ 1 -le 10 ]

#### 1.6.1 检查服务状态脚本

需求：用户执行脚本 sh status.sh nginx 则检查 nginx 服务的运行状态。（仅支持传递一个参数） 1.控制能传递的参数； 2. 3：非运行； 3. 0：运行状态； 4. 4：暂未安装此服务；

5.

```
[root@oldxu if]# cat status.sh
#!/usr/bin/bash
if [ $# -ne 1 ];then
    echo "USAGE: $0 { nginx | httpd |
zabbix-agent | vsftpd | Service_Name }"
```
exit
```
fi
systemctl status $1 &>/dev/null
rc=$?
if [ $rc -eq 4 ];then
echo "$1 服务没有安装..." exit else
    if [ $rc -eq 3 ];then
echo "$1 服务没有启动..."
    elif [ $rc -eq 0 ];then
echo "$1 服务已经启动...." fi fi
```
#### 1.6.2 配置yum仓库脚本

需求：根据相同的系统不同版本，配置不同的yum源 1.获取相同系统，不同的版本

## 2.根据不同的系统配置不同yum源

```
[root@manager if]# cat if-23.sh
#!/bin/bash
system_status=$(cat /etc/redhat-release  |
awk '{print $(NF-1)}')
if [ ${system_status%%.*} -eq 7 ];then
echo "systemctl 7"
elif [ ${system_status%%.*} -eq 6 ];then
echo "systemctl 6"
fi
```
#### 1.6.3 获取进程详情脚本

需求：获取进程的详情 1.首先要传递参数，1个，服务的名称； 2.判断：是否存在该进程，如果不存在，则警告， 然后退出脚本； 2.获取进程的pid相关的信息； ps aux|grep $1

3.还需要该服务的active (running)

4.值需要提取运行的用户：pid，STAT， command命令；

```
[root@oldxu if]# cat if-07.sh
#!/bin/bash
if [ $# -ne 1 ];then
    echo "USAGE: $0 { nginx | httpd |
zabbix-agent | vsftpd | Service_Name }"
```
exit fi # 获取进程pid总数

```
get_service=$(pidof $1 | wc -l)
```
获取进程详情

```
get_service_pid=$(ps -ef | grep -v grep |
egrep "$(pidof $1| sed 's# #|#g')"  >
/tmp/$1_status.txt)
```
判断pid的数量是否大于等于1

```
if [ $get_service -ge 1 ];then
echo "打印当前 $1 进程的详情...." sleep 1 ; echo ""
    cat /tmp/$1_status.txt && rm -f
/tmp/$1_status.txt
echo "$1 服务没有进程详情....." fi
```
#### 1.6.4 数字排序脚本

需求：输入三个数并进行升序排序 1.控制只能输入三个参数； 2.将行，转为列的显示方式； 3.sort排序；

```
[root@manager if]# cat if-30.sh
#!/bin/bash
if [ $# -ne 3 ];then
echo "请传递三个参数" exit fi
echo "$1 $2 $3" | xargs -n1 | sort -n
```
### 1.7 if基于字符比较

if语句中的字符串比较 [ 整数1 操作符 整数2 ] ==：等于则条件为真，示例 [ "$a" == "$b" ] !=：不相等则条件为真，示例 [ "$a" != "$b"

```
]
```
-z：字符串的长度为零则为真（内容空则为真）， 示例 [ -z "$a" ] -n：字符串的长度不为空则为真（有内容则为 真），示例 [ -n "$a" ]

#### 1.7.1 检查执行身份是否为root

需求：判断用户是否为root超级管理员； 如果是则提示 hey admin 如果不是则提示 hey guest

```
[root@web01 shell-if]# cat if-03.sh
#!/bin/bash
if [ $USER == "root" ];then
echo "hey admin"
else
echo "hey guest"
fi # 不等于写法!=
[root@web01 shell-if]# cat if-03.sh
#!/bin/bash
if [ $USER != "root" ];then

echo "ERROR!" exit
fi
#### 1.7.2 判断用户输入是否为空脚本

#判断用户输入是否为空-z

[root@web01 shell-if]# cat if-03.sh
#!/bin/bash
read -p "请输入一个字符: " action
if [ -z $action ];then
echo "请不要直接回车..." exit fi echo "你输入的是: $action"
```
#### 1.7.2 检查selinux是否为disabled

1.检查是否为disabled状态， 是：则输出 该状态已经是disabled，不做修改； 不是：则修改状态为disabled，并返回一段提示，说 selinux已经修改为disabled；

```
[root@dns-master ~]# cat  selinux.sh
#!/bin/bash
Selinux_file=/etc/selinux/config
selinux_status=$(grep  "^SELINUX="
${Selinux_file} | awk -F '=' '{print $2}')
if [ "${selinux_status}" != "disabled"
];then
    sed -i  '/^SELINUX=/c SELINUX=disabled'
${Selinux_file}
echo "Selinux已修改为Disabled状态" else echo "Selinux已是disabled状态，无需修改" fi
```
### 1.8 if基于正则比较

if语句中的正则比较 [[ 变量 =~ 正则匹配的内容 ]] [[ "$USER" =~ ^r ]]：判断用户是否已r开 头； ro,rt,ra,root, [[ "$num" =~ ^[0-9]+$ ]]：判断用户输入的是 否为全数字； 注意：单中括号使用正则语法会报错，bash: [:

```
=~: binary operator expected
```
#### 1.8.1 控制用户输入必须为整数脚本

需求：通过正则方式控制用户输入的必须是数子。

```
[root@oldxu shell]# cat if-7.sh
#!/bin/bash
read -p "请输入一个数值: " num
if [[ ! "$num" =~ ^[0-9]+$ ]];then
echo " 你输入的不是数字，程序退出!!!" exit fi echo "你输入的数字是 $num"
```
#### 1.8.2 遍历单词筛选其中关键字脚本

需求：使用for循环打印一推单词，然后仅输出以r 开头的单词。 正则的匹配；

```
[root@oldxu shell]# cat if-5.sh
for var in ab ac rx bx rvv vt
do
    if [[ "$var" == r* ]];then
        echo "$var"
done
```
#### 1.8.3 批量创建用户脚本

需求：编写一个创建用户的脚本。 1.提示用户输入要创建用户的前缀，必须是英文。 test 2.提示用户输入后缀，必须是数字。  010203 3.如果前缀和后缀都没有问题，则进行用户创建。 test01 test02 4.并且密码是随机的；

```
[root@manager if]# cat if-18.sh
#!/bin/bash
read -p "请输入用户的前缀: " qz #判断用户输入的前缀
if [[ ! $qz =~ ^[a-Z]+$ ]];then
echo "你输入的不是纯英文....." exit 1 fi read -p "请输入用户的后缀: " hz #判断用户输入的后缀
if [[ ! $hz =~ ^[0-9]+$ ]];then
echo "你输入的不是纯数字...." exit 2 fi #开始拼接用户输入的前缀+后缀=user_name变量
user_name=$qz$hz
id $user_name &>/dev/null
if [ $? -eq 0 ];then
echo " $user_name 用户已存在" exit 3
else
    useradd $user_name
echo "$user_name 用户创建成功" fi
```
## 2 if判断相关脚本案例

### 2.1 备份脚本场景

需求：在每月第一天备份并压缩/etc目录的所有内 容，存放到/opt/bak目录，存放的形式 2019_04_10_etc.tar.gz，脚本名称为fileback， 存放在/root的家目录下。 1.借助定时任务； 2.判断/opt/bak是否存在； tar czf /opt/bak/时间_etc.tar.gz /etc

```
[root@manager if]# cat fileback.sh
#!/usr/bin/bash
Date=$(date +%F)
Dir=/opt/bak
Lock_File=/tmp/back.lock
Log_File=/var/log/${0}.log
Log_Format="$(date +%F_%T) $(hostname) $0:"
```
1.判断操作的用户身份

```
if [ ! $USER == "root" ];then
    Log_Format="$(date +%F_%T) $(hostname)
$0:"
echo "$Log_Format 请使用Root身份运行.." >>
${Log_File}
```
exit
```
else
    Log_Format="$(date +%F_%T) $(hostname)
$0:"
echo "$Log_Format Root开始执行备份脚本----
>" >> ${Log_File}
```
2.加锁

```
if [ -f $Lock_File ];then
echo "该脚本 $0 正在执行备份操作--->" exit fi # 第一次备份时，没有锁，所以需要添加
    touch $Lock_File
```
#2.判断备份的目录是否存在

```
if [ ! -d $dir ];then
    mkdir -p $dir
```
#3.执行tar打包

```
Log_Format="$(date +%F_%T) $(hostname) $0:"
echo "$Log_Format Root开始执行tar命令打包----
>" >> ${Log_File}
sleep 2 # 4.判断打包是否成功
cd / &&  tar czf $dir/$Date_etc.tar.gz etc
if [ -f $dir/$Date_etc.tar.gz ];then
    Log_Format="$(date +%F_%T) $(hostname)
$0:"
echo "$Log_Format 备份文件
${dir}/${Date}_etc.tar.gz 成功---->" >>
${Log_File}
    Log_Format="$(date +%F_%T) $(hostname)
$0:"

echo "$Log_Format 备份文件

${dir}/${Date}_etc.tar.gz 失败---->" >>
${Log_File}
```
程序执行成功后记得解锁

```
    rm -f  $Lock_File
```
#####
```
[root@manager if]# cat fileback.sh
#!/usr/bin/bash
Date=$(date +%F)
Dir=/opt/bak
Lock_File=/tmp/back.lock
Log_File=/var/log/${0}.log
Log_Format(){
    echo "$(date +%F_%T) $(hostname) $0:
$1" >> ${Log_File}
}

# 1.判断操作的用户身份

if [ ! $USER == "root" ];then
```
Log_Format "请使用Root身份运行.." exit else Log_Format "Root开始执行备份脚本---->" fi # 2.加锁

```
if [ -f $Lock_File ];then
echo "该脚本 $0 正在执行备份操作--->" exit fi # 第一次备份时，没有锁，所以需要添加
    touch $Lock_File
```
#2.判断备份的目录是否存在

```
if [ ! -d $dir ];then
    mkdir -p $dir
```
#3.执行tar打包 Log_Format "Root开始执行tar命令打包---->" sleep

4.判断打包是否成功

```
cd / &&  tar czf $dir/$Date_etc.tar.gz etc
if [ -f $dir/$Date_etc.tar.gz ];then
```
Log_Format "备份文件

```
${dir}/${Date}_etc.tar.gz 成功---->"
```
Log_Format "备份文件

```
${dir}/${Date}_etc.tar.gz 失败---->"
```
程序执行成功后记得解锁

```
    rm -f  $Lock_File
```
### 2.2 创建随机密码脚本

需求：根据用户输入密码位数，生成随机密码（包含 数字、大小写字母、特殊符号） 1.怎么生成随机数，mkpasswd -l 8 2.控制输入的长度，最少8位

```
[root@manager if]# cat if-31.sh
#!/bin/bash
read -p "请输入你想生成的随机数密码位数: " Action #控制回车 #控制必须是数字
if [ $Action -ge 7  -a $Action -lt 20
];then
    mkpasswd -l $Action
echo "复杂度密码必须7位以上" fi
```
## 3.case判断语句

### 3.1 什么是case

```
case 语句和 if 多分支判断语句类似，主要用来做 多条件判断； 只不过 case 在  Shell 脚本中比 if 多分支条件判 断更方便。
```
### 3.2 case使用场景

在生产环境中，我们会根据”一个问题“ 做多种预案， 然后根据用户选择来加载不同的预案。 比如服务的启停脚本，我们首先要写好启动、停止、 重启的预案，然后根据用户选择来加载不同的预案。

### 3.3 case基础语法

```
case 基础语法
case $1 in    start
```
条件 1) 执行代码块1

```
    ;;
```

条件 2) 执行代码块2

```
    ;;
start)
```
执行代码块3

```
     ;;
*)
```
无匹配后命令序列 esac case 示例:  (PS: 建议先使用 if 实现如下例子)

```
[root@bgx shell]# cat case.sh
#!/bin/bash
cat <<eof
**************** ** 1. backup  ** ** 2. copy    ** ** 3. quit    ** **************** eof
read -p "Input a choose: " OP
case $OP in
## 1|backup)
echo "BACKUP......"
    ;;
## 2|copy)
echo "COPY....."
    ;;
## 3|quit)
```

exit

```
    ;;
  *)
echo error!
esac
```
### 3.4 case场景实践

#### 3.4.1 场景1-编写nginx启停脚本

```
start)
```
1.判断pid是否存在

是：

1.1判断配置文件是否错误

是：尝试让其修改 否： 尝试启动nginx 检查是否正常启动成功 否：

```
stop)
status)
```
#### 3.4.2 场景2-编写rsync启停脚本

```
[root@manager if]# cat case-1.sh
#!/usr/bin/bash
. /etc/init.d/functions
Rsync_Pid_File=/var/run/rsyncd.pid
start_rsync() {
```
判断pid是否存在，如果存在说明服务已 经启动过了

```
            if [ -f $Rsync_Pid_File ];then
```
action "Rsync正在运行中,不用重 复启动.." /bin/false else

```
                /usr/bin/rsync --daemon
                Rsync_Status=$(netstat -
lntp |grep 873 | grep -w "^tcp" | wc -l)
                if [ $Rsync_Status -eq 1
];then

action "Rsync 启动成功"

/bin/true
```
action "Rsync 启动失败"

```
/bin/false
fi
}
stop_rsync() {

# 判断是否存活

            if [ -f  $Rsync_Pid_File ];then
```
#停止

```
                kill  $(cat
${Rsync_Pid_File})
sleep 0.5
                Rsync_Status=$(netstat -
lntp |grep 873 | grep -w "^tcp" | wc -l)
                if [ $Rsync_Status -eq 0
];then
```
action "Rsync 停止成功"

```
/bin/true
```
action "Rsync 停止失败"

```
/bin/false
else # 已经是停止状态 action "Rsync 已经是停止状态"
/bin/false
}
case $1 in
    start)
        start_rsync
```
    ;;
```
    stop)
        stop_rsync
    ;;
    restart)
        if [ -f $Rsync_Pid_File ];then
            stop_rsync
            start_rsync
            start_rsync

    ;;
    status)
        if [ -f $Rsync_Pid_File ];then
            rsync_process_id=$(cat
${Rsync_Pid_File})
echo ""
echo "****************Rsync Status ****************************"
            rsync_process_message=$(ps
aux|grep ${rsync_process_id} | grep -v
grep)
            echo "${rsync_process_message}"
echo "****************************************** *************"
else

action "Rsync 暂未启动.."
/bin/false
```
    ;;
```
    *)
        echo "Usage $0 [ start | status |
restart | stop ]"

exit
esac
#### 3.4.3 场景3-编写LVS启停脚本

lvs_ds.sh 脚本

[root@dns-master ~]# cat  lvs_ds.sh
#!/usr/bin/bash
echo 1 > /proc/sys/net/ipv4/ip_forward
VIP=172.16.1.100
RS1=172.16.1.5
RS2=172.16.1.6
PORT=80
SCHEDULER=rr
DEV=eth1:1
case $1 in
    start)
    cat  >/etc/sysconfig/network-
scripts/ifcfg-${DEV} <<-EOF
    TYPE=Ethernet
    BOOTPROTO=none
    DEFROUTE=yes
    NAME=${DEV}
    DEVICE=${DEV}
    ONBOOT=yes
    IPADDR=${VIP}
    PREFIX=24

EOF # 启动网卡

    ifup ${DEV}
```
配置LVS规则

```
    ipvsadm -C
    ipvsadm -A -t ${VIP}:${PORT} -s
${SCHEDULER}
    ipvsadm -a -t ${VIP}:${PORT} -r ${RS1}
-m
    ipvsadm -a -t ${VIP}:${PORT} -r ${RS2}
-m
    ;;
    stop)
```
停止网卡

```
        ifdown ${DEV}
        rm -f /etc/sysconfig/network-
scripts/ifcfg-${DEV}
```
删除规则

```
        ipvsadm -C
       ;;
    *)
        echo "Usage: sh $0 { start | stop
}"
    ;;
esac lvs_rs.sh 脚本
[root@dns-master ~]# cat lvs_rs.sh
#!/usr/bin/bash
VIP=172.16.1.100
DEV=lo:0
case $1 in
    start)
echo "1"
>/proc/sys/net/ipv4/conf/all/arp_ignore
echo "1"
>/proc/sys/net/ipv4/conf/default/arp_ignore
echo "1"
>/proc/sys/net/ipv4/conf/lo/arp_ignore
echo "2"
>/proc/sys/net/ipv4/conf/all/arp_announce
echo "2"
>/proc/sys/net/ipv4/conf/default/arp_announ
```
ce

```
echo "2"
>/proc/sys/net/ipv4/conf/lo/arp_announce
       cat  >/etc/sysconfig/network-
scripts/ifcfg-${DEV} <<-EOF
       DEVICE=lo:0
       IPADDR=${VIP}
       NETMASK=255.0.0.0
       ONBOOT=yes
       NAME=loopback
EOF ifup ${DEV} # 启动网卡
    ;;
    stop)
>/proc/sys/net/ipv4/conf/all/arp_ignore

>/proc/sys/net/ipv4/conf/default/arp_ignore
>/proc/sys/net/ipv4/conf/lo/arp_ignore

>/proc/sys/net/ipv4/conf/all/arp_announce
>/proc/sys/net/ipv4/conf/default/arp_announ

ce
echo "0"

>/proc/sys/net/ipv4/conf/lo/arp_announce
ifdown ${DEV}  # 停止网卡
        rm -f /etc/sysconfig/network-
scripts/ifcfg-${DEV}
        ;;
    *)
        echo "Usage: sh $0 { start | stop
}"
esac 作业： 获取Nginx|php-fpm状态脚本 明天： for; while; 嵌套循环； 9 * 9 = break,continue,exit; 周六： shell 函数； shell 属组；
```
