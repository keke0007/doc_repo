# 1.for循环

## 目录

  - [1.1 什么是循环](#1.1-什么是循环)
  - [1.2 什么是for循环](#1.2-什么是for循环)
  - [1.3 for循环基础语法](#1.3-for循环基础语法)
  - [1.4 for循环通过文件创建用户脚本](#1.4-for循环通过文件创建用户脚本)
  - [1.6 for循环输出整数升序降序脚本](#1.6-for循环输出整数升序降序脚本)
  - [1.7 for循环计算10以内整除3脚本](#1.7-for循环计算10以内整除3脚本)
  - [1.8 for循环计算1+2+3+n的值脚本](#1.8-for循环计算123n的值脚本)
- [2.for循环案例](#2.for循环案例)
  - [2.1 探测主机存活性脚本](#2.1-探测主机存活性脚本)
  - [2.2 探测主机开放端口脚本](#2.2-探测主机开放端口脚本)
  - [2.3 获取系统普通用户脚本](#2.3-获取系统普通用户脚本)
  - [2.4 获取普通用户对应的组脚本](#2.4-获取普通用户对应的组脚本)
  - [2.5 批量下载镜像市场软件脚本](#2.5-批量下载镜像市场软件脚本)
  - [2.6 对MySQL数据库进行备份](#2.6-对mysql数据库进行备份)
  - [2.7 实现九九乘法表脚本](#2.7-实现九九乘法表脚本)
  - [2.8 随机点名脚本](#2.8-随机点名脚本)
  - [2.9 模拟恢复误删除文件](#2.9-模拟恢复误删除文件)
- [3.while循环](#3.while循环)
  - [3.1 什么是while](#3.1-什么是while)
  - [3.2 while与for如何选](#3.2-while与for如何选)
  - [3.3 while循环基础语法](#3.3-while循环基础语法)
- [1.while循环语法](#1.while循环语法)
  - [3.4 while嵌套整数比对](#3.4-while嵌套整数比对)
  - [3.5 while嵌套文件比对](#3.5-while嵌套文件比对)
  - [3.6 while嵌套字符比对](#3.6-while嵌套字符比对)
  - [3.7 while循环控制语句](#3.7-while循环控制语句)
    - [3.7.1 exit方法](#3.7.1-exit方法)
    - [3.7.2 break方法](#3.7.2-break方法)
    - [3.7.3 continue方法](#3.7.3-continue方法)
    - [3.7.4 while嵌套continue](#3.7.4-while嵌套continue)
    - [3.7.5 while嵌套break](#3.7.5-while嵌套break)
- [4.while循环案例](#4.while循环案例)
  - [4.1 猜测随机数字脚本](#4.1-猜测随机数字脚本)
  - [4.2 破解随机字符串脚本](#4.2-破解随机字符串脚本)
  - [4.3 通过文本创建用户脚本](#4.3-通过文本创建用户脚本)
  - [4.5 防止ssh暴力破解脚本](#4.5-防止ssh暴力破解脚本)

```
while
```
5.总结： 徐亮伟, 江湖人称标杆徐。多年互联网运维工作经 验，曾负责过大规模集群架构自动化运维管理工作。 擅长Web集群架构与自动化运维，曾负责国内某大型 电商运维工作。 个人博客"徐亮伟架构师之路"累计受益数万人。 笔者Q:552408925、572891887 架构师群:471443208

# 1.for循环

### 1.1 什么是循环

脚本在执行任务的时候，总会遇到需要循环执行的时 候。 场景：批量创建100个用户，我们就需要使用循环来 实现。

### 1.2 什么是for循环

很多人把for循环叫做条件循环； 因为for循环的次数和给予的条件是成正比的，也就是 你给5个条件，那么他就循环5次；

### 1.3 for循环基础语法

```
for 循环基础语法示例 #for循环语法                    for循环示例（默 认以空格做分割,给了三个条件） for 变量名  in [ 取值列表 ]          for var in file1 file2 file3 do                                  do 循环体
echo the text is $var
done for循环使用示例
[root@oldxu for]# cat for-1.sh
[root@oldxu for]# cat for-5.sh
#!/usr/bin/bash
#!/bin/bash
for i in ${1..100}
for i
in `cat /etc/hosts`
do
    useradd test-$i
echo "$i"
echo test-$i is ok
done
```
#c语言风格:

```
for ((i=1;i<=10;i++))
    echo $i

for 循环默认使用空格为分隔符，可以使用 IFS 来 自定义分隔符 以冒号做分隔符                    IFS=: 以换行符做为字段分隔符            IFS=$'\n'

[root@manager for]# cat  for-2.sh
IFS=$'\n'
for i in $(cat /etc/hosts)
    echo $i

sleep 0.5
done
### 1.4 for循环通过文件创建用户脚本

通过读入文件中的用户与密码文件，进行批量添加用 户。 文件中的格式: user:passwd 分析：循环 1.先读入文件； 2.进行字段分割； 3.完成创建；

[root@manager for]# cat user-file-pass.sh
#!/bin/bash
for user in $(cat user.txt)
    us=$(echo $user|awk -F ":" '{print
$1}')
    pw=$(echo $user|awk -F ":" '{print
$2}')

    id $us &>/dev/null
    if [ $? -eq 0 ];then
```
continue
```
else
        useradd $us
        echo "$pw" | passwd --stdin $us
&>/dev/null
        echo "$us is create ok......"
```
### 1.6 for循环输出整数升序降序脚本

需求：同时输出1-9的升序和降序

```
[root@oldxu for]# sh  for-9.sh
```
num is 1 9 num is 2 8 num is 3 7 num is 4 6 num is 5 5 num is 6 4 num is 7 3 num is 8 2 num is 9 1 # 脚本内容：

```
[root@oldxu for]# cat for-10.sh
#!/bin/bash
a=0
b=10
for i in {1..9}

let a++;

    let b--;
  echo num is $a - $b
```
### 1.7 for循环计算10以内整除3脚本

10以内能整除3的数值求和脚本 借助：取余数 1 % 3 = 不是0    为0，说明这个数值是可以除以3

```
[root@oldxu for]# cat for-sum.sh
for i in {1..10}
    num=$[ $i % 3 ]
    if [ $num -eq 0 ];then
        sum=$[ $sum + $i ]

        echo $sum
```
### 1.8 for循环计算1+2+3+n的值脚本

计算1+2+3+...+n求和，如n=8769 （笔试题） 可以先写1+2+3+10=

```
[root@manager for]# echo {1..8769} | xargs
| sed 's# #+#g' | bc
[root@oldxu for]# cat for-sum.sh
#!/usr/bin/bash
num=0
for i in {1..8769}

    num=$[ $i + $num ]
    echo $num

## 2.for循环案例

### 2.1 探测主机存活性脚本

需求：批量探测某个网段的主机存活状态； 1.通过for循环遍历出所有的IP地址； 2.将所有的IP地址写入到一个文本文件中； 10.0.0.1 ~ 10.0.0.254

[root@manager for]# seq 254 | sed -r 's#
(.*)#10.0.0.\1#g' > ip_new.txt
[root@manager for]# cat for-9.sh
for ip in $(cat ip_new.txt)
   {
    ping -c1 -W1 $ip &>/dev/null
    if [ $? -eq 0 ];then
        echo "$ip ok" >> ip_ok.txt
        echo "$ip ok"

    }&
```
wait echo "done.." 需求：批量探测某个网段的主机存活状态；要求判断 三次，如果三次失败则失败； 1.判断是否存活； 存活，则直接输出； 不存活，在探测三次；

```
[root@manager for]# cat for-10.sh
```
外侧循环，取IP地址

```
for i in {1..254}
    ip=10.0.0.$i
    ping -c1 -W2 $ip &>/dev/null
    if [ $? -ne 0 ];then

# 内循环  10.0.0.9

        for j in {1..3}
            ping -c1 -W2 $ip &>/dev/null
            if [ $? -eq 0 ];then

echo "$ip 探测第 $j 次 才 通.." break
else
echo "$ip 探测第 $j 次 butong.."
fi
done
else

        echo "$ip is tong..."
```
### 2.2 探测主机开放端口脚本

需求： 1.有一个ip.txt的文件，里面有很多IP地址。 2.有一个port.txt的文件，里面有很多端口号。

3.现在希望对ip.txt的每个IP地址进行 port.txt文件中的端口号进行挨个探测。 4.最后将开放的端口和IP保存到一个ok.txt文 件。

```
#ip.txt                         port.txt
```

192.168.10.1

172.16.1.6

```
[root@oldxu for]# cat for-13.sh
#!/bin/bash
```
#遍历文件中的IP地址

```
for ip in $(cat ip.txt)
```
#第二次循环,遍历文件中的端口号

```
    for port in $(cat port.txt)
```
#探测IP与端口的存活状态

```
        nc -z -w 1 $ip $port &>/dev/null
        if [ $? -eq 0 ];then
            echo "$ip $port is ok"

done

[root@manager for]# cat for-11.sh
```
#外层循环

```
for ip in $(cat ip.txt)
```
# 内循环

```
    for port in $(cat port.txt)
```
# 探测ip+pprt

```
        nc -z $ip $port &>/dev/null
        if [ $? -eq 0 ];then
            if [ $port == "80" ];then
                echo -e "\e[31m $ip 的 $port
```
开放了，比较危险..\e[0m" else

```
                echo -e "\e[32m $ip 的 $port
```
开放了.. \e[0m"
```
fi
fi
done
done
```
### 2.3 获取系统普通用户脚本

需求：获取系统的所有用户并输出。效果如下:

```
This is 1 user: root
This is 2 user: bin
This is 3 user: daemon
This is 4 user: adm
```
..............

```
[root@oldxu for]# cat for-14.sh
#!/bin/bash
i=1
for user in $(awk -F ":" '{print $1}'
/etc/passwd)
    echo This is ${i} $user
```
    #let i++
```
    i=$[ $i + 1 ]

### 2.4 获取普通用户对应的组脚本

需求：获取已存在的普通用户对应的基本组以及附加 组 用户名称: u1, 基本组: 1002(u1), 附加组:

1001(grp1) 1007(oldxu)
```
用户名称: u2, 基本组: 1003(u2), 附加组:

```
1001(grp1)
```
用户名称: oldxu, 基本组: 1007(oldxu), 附加组: Null

```
[root@oldxu ~]# cat get_user_groups.sh
#!/usr/bin/bash
LANG=en
pass_file=/etc/passwd
```
#1.获取用户名称

```
users=$(awk -F ':' '$3>=1000 {print $1}'
${pass_file})
for i in $users
```
#3.获取用户的基本组

```
    group=$(id ${i} | xargs -n1 | grep
"groups" | awk -F '=' '{print $2}' | tr ","
"\n" | awk 'NR==1')
```
#4.获取附加组

```
    groups=$(id ${i} | xargs -n1 | grep
"groups" | awk -F '=' '{print $2}' | tr ","
"\n" | awk 'NR>1' | xargs)
    if [ -z "$groups" ];then
echo "用户名称: ${i}, 基本组: ${group}, 附加组: Null" else

echo "用户名称: ${i}, 基本组:
${group}, 附加组: ${groups}"
```
### 2.5 批量下载镜像市场软件脚本

需求：有一个软件包仓库，它的地址是

```
https://mirror.tuna.tsinghua.edu.cn/zabbix/
zabbix/5.0/rhel/7/x86_64/
```
需要批量下载中所有的文件，然后找出其中大于5M的 文件存储至/tmp目录；

```
for i in  $(curl
https://mirror.tuna.tsinghua.edu.cn/zabbix/
zabbix/5.0/rhel/7/x86_64/)
```
判断大小，大于则拷贝至/tmp目录

```
    pkg_size=$(du -s  $i | awk '{print
$1}')
    if [ $pkg_size -ge 5120];then
        wget
```
### 2.6 对MySQL数据库进行备份

场景1：备份MySQL数据库，将每个库都备份一个sql文 件，存储至/backup/mysql/2021-09-23/xx.sql；

```
mysql -uroot -e "show databases;" | sed 1d
| egrep -v "*_schema"
mysqldump -uroot -B wordpress >
/tmp/wordpress.sql
[root@db01 ~]# cat mysql_database_backup.sh
#!/usr/bin/bash
. /etc/init.d/functions
Db_Name=$(mysql -uroot -e "show databases;"
| sed 1d | egrep -v "*_schema")
Date=$(date +%F)
DB_Dir=/backup/mysql/${Date}

# 确保备份的目录是存在的

if [ ! -d ${DB_Dir} ];then
    mkdir -p ${DB_Dir}
```
# 备份业务逻辑

```
for database in ${Db_Name}
    mysqldump -uroot -B ${database} >
${DB_Dir}/${database}.sql

判断文件是否有内容

    if [ -s ${DB_Dir}/${database}.sql
];then
        action "${DB_Dir}/${database}.sql 备
份成功" /bin/true else
        action "${DB_Dir}/${database}.sql 备
```
份失败" /bin/true fi done # 场景2：对MySQL数据库进行分库分表备份，存储

```
至/backup/mysql/2021-09-
23/database/tables.sql；
[root@db01 ~]# cat mysql_database_tables.sh
#!/usr/bin/bash
. /etc/init.d/functions
```
定义变量 （用户名称、密码、IP地址；

```
mysql_command）
DB_Name=$(mysql -uroot -e "show databases;"
| sed 1d | egrep -v "*_schema")
Date=$(date +%F)
DB_Dir=/backup/mysql/${Date}
```
外循环,提取库的名称

```
for database in ${DB_Name}
```
创建库对应的备份目录

```
    if [ ! -d $DB_Dir/$database ];then
        mkdir -p "$DB_Dir/$database"
```
提取表名称

```
    TB_Name=$(mysql -e "use
${database};show tables;" | sed 1d)
```
内循环，基于库名称然后取获取表名称

```
    for table in ${TB_Name}
        mysqldump -uroot ${database}
${table} > $DB_Dir/${database}/${table}.sql
        if [ -s
$DB_Dir/${database}/${table}.sql ];then

action

"$DB_Dir/${database}/${table}.sql 备份成功"
/bin/true
else action
"$DB_Dir/${database}/${table}.sql 备份失败"
/bin/false
done
```
### 2.7 实现九九乘法表脚本

```
1 * 1 =

2 * 1 = 2  2 * 2 =

3 * 1 = 3  3 * 2 = 6  3 * 3 =

4 * 1 = 4  4 * 2 = 8  4 * 3 = 12  4 * 4 = 16 5 * 1 = 5  5 * 2 = 10  5 * 3 = 15  5 * 4 = 20  5 * 5 = 25 6 * 1 = 6  6 * 2 = 12  6 * 3 = 18  6 * 4 = 24  6 * 5 =

6 * 6 =

7 * 1 = 7  7 * 2 = 14  7 * 3 = 21  7 * 4 = 28  7 * 5 =

7 * 6 = 42  7 * 7 =

8 * 1 = 8  8 * 2 = 16  8 * 3 = 24  8 * 4 = 32  8 * 5 =

8 * 6 = 48  8 * 7 = 56  8 * 8 =

9 * 1 = 9  9 * 2 = 18  9 * 3 = 27  9 * 4 = 36  9 * 5 = 45 9 * 6 = 54  9 * 7 = 63  9 * 8 = 72  9 * 9 =
[root@oldxu while]# cat for_99.sh
#!/bin/bash
```
# 外出循环1-9

```
for i in {1..9}
```
# 内循环1-9

```
    for j in {1..9}
        echo -n "$i * $j = $[ $i * $j ]  "

i变量等于几则让j变量循环几次

        if [ $i -eq $j ];then
echo "" break

done
```
### 2.8 随机点名脚本

需求： 1.执行脚本拿到一位同学的名字； 2.被点过的同学，不会在出现第二次； 3.该点名脚本将所有的同学都点到过之后，文件就空 了，空了就提示请重置；

```
[root@manager for]# cat dm.sh
#!/usr/bin/bash
```
#重置操作

```
if [ ! -s user.txt ];then
read -p "所有的同学都被点到过了,是否需要重置
[ yes | no ]:" Action
    if [ ${Action:=yes}  == "yes" ];then
        cat user_new.txt > user.txt
        rm -f user_new.txt
echo "重置成功，可以继续" exit else exit fi fi # 定义的变量；
file=user.txt
RANDOm=$(echo $RANDOM)
file_line=$(cat ${file}|wc -l)
sj=$[ ${RANDOm} % ${file_line} ]

# 避免随机数为0，因为为0，sed会报错；

if [ ${sj} -eq 0 ];then
    sj=$[ $sj + 1 ]
```
# 提取随机名单

```
Full_Name=$(sed -n "${sj}p" user.txt)
echo "此次回答问题的是: ${Full_Name}" #备份，然后删除名单
sed -n "${sj}p" user.txt >> user_new.txt
sed -i "${sj}d" user.txt
```
### 2.9 模拟恢复误删除文件

# 获取文件描述符

```
ll /proc/23416/fd | grep "deleted" |awk
'{print $9}'
```
# 获取文件描述符对应的文件名称

```
ll /proc/23416/fd | grep "deleted" |awk
'{print $11}
[root@web03 opt]# cat reset.sh
#!/usr/bin/bash
. /etc/init.d/functions
pid=23416
thread_id=$(pstree -p ${pid}| egrep -o "[0-
9]+")

thread_id 拿到很多线程的id号码

for iid in ${thread_id}
```
线程的一个ID

```
    fd=$(ls -l /proc/${iid}/fd | grep
"deleted" |awk '{print $9}')
    for id in ${fd}
```
基于文件描述符，提取对应的文件名称

```
        fd_file=$(ls -l
/proc/${pid}/fd/${id} | awk '{print $11}')
```
恢复操作

```
        cat /proc/${pid}/fd/${id} >
$fd_file
```
######################################
```

# 恢复后的文件获取md5值

        hf_file_md5=$(md5sum "${fd_file}"
2>/dev/null | awk '{print $1}')
```
# 源文件

```
        src_file_md5=$(md5sum
"${fd_file/opt/soft}" 2>/dev/null | awk
'{print $1}')
        if [ "${hf_file_md5}" ==
"${src_file_md5:=xxx}" ];then
            action " $fd_file 成功"
/bin/true

else

            action " $fd_file 失败"
/bin/false
```
######################################
```
done
## 3.while循环

### 3.1 什么是while

while 在 shell 中也是负责循环的语句，和 for 一 样。
### 3.2 while与for如何选

因为功能一样，很多人在学习和工作中的脚本遇到循 环到底该使用for还是while呢 很多人不知道，就会出现有人一遇循环就使用for、 有人一遇循环就使用while 到底选for好还是while好： 1.知道循环次数的使用for，比如一天循环24次；

2.如果不知道要循环多少次，那就用while，比如 猜数字游戏，每个人猜对的次数是未知的

### 3.3 while循环基础语法

## 1.while循环语法

#当条件测试成立（条件测试为真），执行循环体   if

[];then
while 条件测试 do 循环体 done 2.while循环基本使用示例，降序输出10到1的数字
[root@oldxu for]# cat while-1.sh
#!/bin/bash
var=10
while [ $var -gt 0 ]
    echo $var
    var=$[$var-1]

3.while循环基本使用示例，输出如下图，两数相乘 # 打印如下内容

9 * 9 = 81
8 * 8 = 64
7 * 7 = 49
6 * 6 = 36
5 * 5 = 25
4 * 4 = 16
3 * 3 = 9
2 * 2 = 4
1 * 1 = 1

#自增

[root@oldxu for]# cat while-2.sh
#!/usr/bin/bash
num=9
while [ $num -ge 1 ]
    sum=$(( $num * $num ))
    echo "$num * $num = $sum"
    num=$[$num-1]

#自减

[root@oldxu for]# cat while-3.sh
#!/usr/bin/bash
num=1
while [ $num -le 9 ]
    sum=$(( $num * $num ))
    echo "$num * $num = $sum"
    num=$[$num+1]

### 3.4 while嵌套整数比对

循环嵌套整数比对，判断用户输入的数值是否大于 0，如果大于0，则三秒输出一次大于。

[root@oldxu ~]# cat while_number.sh
#!/usr/bin/bash
read -p "请输入数字: " num
while [ $num -ge 0 ]
echo "大于" sleep 3
done
```
### 3.5 while嵌套文件比对

循环嵌套文件比较，判断 /tmp/oldxu 文件是否存 在， 如果不存在则 3s 输出一次 not found 如果存在自动退出

```
[root@oldxu ~]# cat  while_file.sh
#!/usr/bin/bash
while [ ! -d /tmp/oldxu ]
    echo "not found /tmp/oldxu"

sleep 3
done
### 3.6 while嵌套字符比对

循环嵌套字符比较，判断用户输入的用户名，如果不 是 root 则一直让其输入

[root@oldxu ~]# cat while_string.sh
#!/usr/bin/bash
read -p "Login: " account
while [ $account != 'root' ]
    read -p "Login: " account

### 3.7 while循环控制语句

在使用循环语句进行循环的过程中，有时候需要在未 达到循环结束条件时强制跳出循环； 那么 Shell 给我们提供了内置方法来实现该功能： exit、break、continue

#### 3.7.1 exit方法

exit，退出整个程序。 当脚本碰到 exit 时，直接退出，剩余不管有多少代 码都不执行。

[root@oldxu shell]# cat for_exit.sh
#!/usr/bin/bash
for i in {1..3}
echo "123" exit
echo "456"
done
echo "Done....." #执行后的结果
[root@Shell ~]# sh for_exit.sh
```
#### 3.7.2 break方法

break，结束当前循环 当脚本碰到 break 时，会结束当前循环，但会执行 循环之后的所有的代码。

```
[root@oldxu ~]# cat for_break.sh
#!/usr/bin/bash
for i in {1..3}
echo "123" break
echo "456"
done
echo "Done....." #执行后的结果
[root@ansible-node30 ~]# sh for_break.sh
```
123 Done.....

#### 3.7.3 continue方法

continue 忽略本次循环剩余的所有代码， 当脚本碰到 continue 时，直接进入下一次循环，直 到循环结束，然后继续执行循环之后的代码。

```
[root@oldxu ~]# cat for_continue.sh
#!/usr/bin/bash
for i in {1..3}
echo "123" continue
echo "456"
done
echo "Done....." #执行后的结果
[root@oldxu shell]# sh for_continue.sh
```
123 123 123 Done..... #continue示例

```
[root@oldxu ~]# cat continue.sh
#!/bin/sh
for i in `seq 10`
do
    useradd te$i &>/dev/null
    if [ $? -eq 0 ];then
    echo "Create User $i Success"
else
    echo "Create User $i already exists"
```
continue
```
fi
done
```
#### 3.7.4 while嵌套continue

需求：循环嵌套 continue，打印1-9当数值为5则跳 过本次循环，继续下一次循环。请分别使用for和 while实现

## 1234 6789

```
#for
[root@oldxu ~]# cat continue.sh
for i in {1..9}
```
#本地循环到此结束，可以开始下一次循环

```
    if [ $i -eq 5 ];then
```
continue
```
else
        echo $i
```
#while
```
[root@oldxu ~]# cat while_continue.sh
#!/usr/bin/bash
i=0
while [ $i -lt 10 ]

    i=$[ $i + 1 ]
    if [ $i -eq 5 ];then
```
continue
```
fi
    echo $i
```
#### 3.7.5 while嵌套break

需求：循环嵌套break，打印1-9当数值为5则停止。 请分别使用for和while实现。

```
#for
[root@oldxu ~]# cat break.sh
for i in `seq 1 9`
do
    echo $i
    if [ $i -eq 5 ];then
```
break
```
fi
done
#while
[root@oldxu ~]# cat while_break.sh
#!/usr/bin/bash
i=1
while [ $i -lt 10 ]
    if [ $i -eq 5 ];then

break
fi

    echo $i
    i=$[ $i + 1 ]
```
## 4.while循环案例

### 4.1 猜测随机数字脚本

猜数字游戏

1)随机输出一个1-100的数字

2)要求用户输入的必须是数字（数字处加判断）

3)如果比随机数小则提示比随机数小了 大则提示比

随机数大了

4)正确则退出 错误则继续死循环

5)最后统计猜了多少次（猜对了多少次，失败多少 次)

```
[root@manager while]# cat  while-10.sh
#!
sj=$(echo $[ $RANDOM%100+1 ])
echo $sj
index=0

while true
do
read -p "请输入数字: " Action

    if [[ ! $Action =~ ^[0-9]+$ ]];then
```
continue
```
fi
```
# 计数

```
    index=$[ $index +1 ]
    if [ $Action -eq ${sj} ];then
echo "恭喜你,你既然才花了 ${index} 次就 猜对了" exit fi
    if [ $Action -gt ${sj} ];then
echo "你输入的数字太大.." else echo "你输入的数字太小.." fi done
```
### 4.2 破解随机字符串脚本

字符串 efbaf275cd、4be9c40b8b、44b2395c46 是通过对随机数变量RANDOM随机执行命令： echo $RANDOM|md5sum|cut -c1-10 后的结果 请破解这些字符串对应的RANDOM值

```
[root@manager while]# cat while-11.sh
#!/usr/bin/bash
for i in efbaf275cd 4be9c40b8b 44b2395c46
do # i加密后的结果
        num=0
while true
do
         md5_sum=$(echo ${num}|md5sum|cut -
c1-10)
         if [ $md5_sum == "$i" ];then
echo "$num 加密后的结果是 $i" break else
            num=$[ $num + 1 ]
```
continue
```
fi
done
done
```
### 4.3 通过文本创建用户脚本

通过读入文件中的用户与密码文件，进行批量添加用 户。 文件中的格式: user:passwd

```
[root@oldxu while]# cat while-5.sh
#!/usr/bin/bash
while
read line
do
    user=$(echo $line|awk '{print $1}')
    pass=$(echo $line|awk '{print $2}')
    id $user &>/dev/null
    if [ $? -eq 0 ];then
    echo "useradd $user is already exists"
else
    useradd $user &&\
        echo "$pass"|passwd --stdin $user
&>/dev/null
        echo "useradd  $user is Created And
Passwd Is Ok."
fi
done<user2.tt
```
### 4.5 防止ssh暴力破解脚本

如果使用 oldxu 用户登录系统，则立即将其踢出， 然后将其拉入黑名单，以防止该用户继续使用该IP地 址进行登录。 将 sshd:username ip地址 内容 至

```
/etc/hosts.deny 则可拒绝用户登陆系统。
[root@oldxu tmp]# cat deny_user.sh
#!/bin/bash
while true do #获取用户的名称
deny_user=$(who | grep "oldxu" | awk
'{print $1}' | uniq)
```
#获取用户的来源IP

```
deny_ip=$(who | grep "oldxu" |sed -r
's#^.*\((.*)\)#\1#g' | uniq)
```
#判断哪用户变量是有值，有值说明该用户已登陆系统

```
if [ ! -z $deny_user ];then
```
#让用户下线

```
pkill -9 -U oldxu &&
echo "${deny_user}
user is killed"
```
#将用户加入黑名单，防止再次登陆系统。

```
    echo "sshd:${deny_user} ${deny_ip}" >>
/etc/hosts.deny
sleep 10 done 5.总结： 1.for循环； 2.嵌套for循环； 3.for脚本示例； （批量创建用户，删除用户的脚本，） 一个脚本，实现两个功能；  sh user del oldxu;   sh user add oldxu; 4.break,continue,exit;
```
