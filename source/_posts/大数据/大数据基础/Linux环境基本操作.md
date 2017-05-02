---
title: Linux基本操作
date: 2017-03-22 20:09:06
tags: [Linux,大数据]
---

配置三台Linux服务器，为Hadoop集群准备硬件资源。


## 1 设备条件

- 性能: 由一台32G的8Core服务器虚拟出3台(4G 4Core)CentOS7服务器,内存固定分配，Core根据使用状态动态调节 
    - 查看CPU信息 `cat /proc/cpuinfo`
    - 查看内存信息 `free -h`
- IP地址: 10.5.151.241 10.5.151.242 10.5.151.243 
- VM网络配置使用的桥接模式，使用真实的网络地址，Bridge适合远程操作；而NAT模式适合将虚拟机装在本机，使用虚拟网关，如果是在笔记本电脑中虚拟机器可以使用此方式。


## 2 操作

### 2.1 Vim基本操作

**一般模式**
- a  在光标后一位开始插入
- A   在该行的最后插入
- I   在该行的最前面插入
- gg   直接跳到文件的首行
- G    直接跳到文件的末行
- dd   删除行，如果  5dd   ，则一次性删除光标后的5行
- yy  复制当前行,  复制多行，则  3yy，则复制当前行附近的3行
- p   粘贴
- v  进入字符选择模式，选择完成后，按y复制，按p粘贴
- ctrl+v  进入块选择模式，选择完成后，按y复制，按p粘贴
- shift+v  进入行选择模式，选择完成后，按y复制，按p粘贴

<!--more-->

**命令模式**

查找并替换（在底行中输入）
- %s/sad/666     查找文件中所有sad，666
- /you       查找文件中出现的you，并定位到第一个找到的地方，按n可以定位到下一个匹配位置（按N定位到上一个）


### 2.2 日常操作

- pwd 显示当前目录
- date 查看当前时间
- who 查看当前登录到服务器的人
- last 查看最近的登陆历史记录

### 2.3 文件权限操作

- chmod g-rw haha.dat    表示将haha.dat对所属组的rw权限取消
- chmod o-rw haha.dat 	表示将haha.dat对其他人的rw权限取消
- chmod u+x haha.dat      表示将haha.dat对所属用户的权限增加x

**也可以用数字的方式来修改权限**

- chmod 664 haha.dat 就会修改成   rw-rw-r--   

**如果要将一个文件夹的所有内容权限统一修改，则可以-R参数**

- chmod -R 770 aaa/
- chown angela:angela aaa/    <只有root能执行>

>目录没有执行权限的时候普通用户不能进入
文件只有读写权限的时候普通用户是可以删除的(删除文件不是修改它,是操作父及目录),只要父级目录有执行和修改的权限

### 2.4 用户管理

- useradd hadoop   添加用户hadoop
- passwd hadoop   修改用户密码
- 为hadoo用户配置sudo的使用权限
    - vim /etc/sudoers
    - hadoop ALL=(ALL)  ALL
    
### 2.5 系统管理操作

- hostname 查看主机名
- hostname hadoop 修改主机名,重启后无效
- vim /etc/sysconfig/network 修改主机名，重启后永久生效
- ifconfig eth0 192.168.1.100 修改IP，重启后无效
- vim /etc/sysconfig/network-scripts/ifcfg-eth0 修改IP 重启网卡后永久生效
- 挂载外置磁盘
    - mkdir /mnt/udisk 创建一个目录用来挂载
    - mount -t iso1234 ro /dev/sd4 /mnt/udisk   将设备/dev/sd4挂在到/mnt/udisk中
    - umount /mnt/udisk 解除挂载
- du -sh /var 查看文件夹大小sh run
- df -h 查看磁盘空间
- halt 关机
- reboot 重启
- uname -a 查看系统信息


### 2.6 查看文件操作

- cat    somefile    一次性将文件内容全部输出（控制台）
- more   somefile     可以翻页查看, 下翻一页(空格)    上翻一页（b）   退出（q）
- less   somefile      可以翻页查看,下翻一页(空格)    上翻一页（b），上翻一行(↑)  下翻一行（↓）  可以搜索关键字（/keyword）
- tail -10  install.log   查看文件尾部的10行
- tail -f install.log    小f跟踪文件的唯一inode号，就算文件改名后，还是跟踪原来这个inode表示的文件
- tail -F install.log    大F按照文件名来跟踪
- head -10  install.log   查看文件头部的10行


### 2.7 后台服务管理

- service network status   查看指定服务的状态
- service network stop     停止指定服务
- service network start    启动指定服务
- service network restart  重启指定服务
- service --status-all  查看系统中所有的后台服务

### 2.8 系统启动级别管理

vi  /etc/inittab

### 2.9 设置后台服务的自启配置

- chkconfig   查看所有服务器自启配置
- chkconfig iptables off   关掉指定服务的自动启动
- chkconfig iptables on   开启指定服务的自动启动

```
# Default runlevel. The runlevels used are:
#   0 - halt (Do NOT set initdefault to this)
#   1 - Single user mode
#   2 - Multiuser, without NFS (The same as 3, if you do not have networking)
#   3 - Full multiuser mode
#   4 - unused
#   5 - X11
#   6 - reboot (Do NOT set initdefault to this)
#
id:3:initdefault:
```

### 2.10 压缩解压缩 打包解包

- gzip access.log 压缩文件
- gzip -d access.log.gz
- tar -cvf mytarball.tar aaa/ 打包aaa文件夹
- tar -xvf mytarball.tar 解包
- tar -zcvf my.tar.gz  aaa/ 压缩并打包
- tar -zxvf my.tar.gz 解包解压缩

### 2.11 wget 使用

wget --no-check-certificate https:xxxx/mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz  下载https路径的文件

### 2.12 rpm 使用
- rpm -qa | grep mysql 查看系统中安装的rpm包
- rpm -ivh perl*  安装perl依赖  《可能会提示有包冲突，解决： rpm -e 冲突包名 --nodeps 》
- rpm -ivh MySQL-server-5.5.48-1.linux2.6.x86_64.rpm 安装server
- rpm -ivh MySQL-client-5.5.48-1.linux2.6.x86_64.rpm 安装clent 

### 2.13 后台服务管理

- chkconfig   查看所有服务器自启配置
- chkconfig iptables off   关掉指定服务的自动启动
- chkconfig iptables on   开启指定服务的自动启动


## 3 配置

### 3.1 配置hostname
```commandline
vim /etc/hosts

10.5.151.241 hadoop1
10.5.151.242 hadoop2
10.5.151.241 hadoop3
```

### 3.2 配置SSH免密登陆

若 A 需要登录 B

**A上操作**

- ssh-keygen  生成密钥对
- ssh-copy-id B 将A的公钥复制并追加到B的授权列表文件 authorized_keys 中 

### 3.3 Clone Linux 遇到的问题 

1. 从一台配置好的linux克隆多个系统时，由于物理网卡地址重复，因此生成了一块新的网卡eth1，原来了eth0不见了
    
    - 直接修改ifcfg-eth0 、删除UUID和HWARE、配置静态地址
    - rm -rf /etc/udev/rules.d/70-persisitent-net.rules 或 将eth1的无礼地址复制给eth0
    - reboot 


## 4 安装Mysql

- 卸载自带mysql
    - rpm -qa | grep mysql*
    - rpm -qa | grep maria*
    - rpm -e 以上查询结果
    - yum list installed mysql* 
    - yum erase 以上查询结果
    - find / -name mysql  逐一删除

- 下载mysql
    - wget  --no-check-certificate https://cdn.mysql.com//Downloads/MySQL-5.6/MySQL-5.6.35-1.linux_glibc2.5.x86_64.rpm-bundle.tar 
    - tar -xvf 下载结果
    - rpm -ivh mysql-server 和 mysql-client

- 启动、连接 
    - systemctl start mysql 启动服务
    - cat /root/.mysql_secret  查找默认密码
    - mysql_secure_installation  修改默认密码
    - mysql -uroot -pxxxxxx 连接mysql
    - use mysql; select host,user from user; 查看用户
    - update user set host = '%' where user = 'root'; 设置允许远程登录
    - flush privileges; 刷新
     
## 5 iptables防火墙设置

- systemctl status/start/stop/restart firewalld 查看/启动/关闭/重启防火墙服务

**查看**
- iptables -L -n --line-numbers 查看所有规则
- iptables -L -n -t nat  列出iptables nat表规则（默认是filter表）




**防火墙规则**

- -A, --append chain	追加到规则的最后一条
- -D, --delete chain [rulenum]	Delete rule rulenum (1 = first) from chain
- -I, --insert chain [rulenum]	Insert in chain as rulenum (default 1=first) 添加到规则的第一条
- -p, --proto  proto	protocol: by number or name, eg. 'tcp',常用协议有tcp、udp、icmp、all
- -j, --jump target 常见的行为有ACCEPT、DROP和REJECT三种，但一般不用REJECT，会带来安全隐患

**修改**

- iptables -F    清除默认规则（注意默认是filter表，如果对nat表操作要加-t nat）
- service iptables save 保存配置

**实例**

- iptables -A INPUT -p tcp --dport 22 -j DROP   禁止ssh登陆（若果服务器在机房，一定要小心）
- iptables -D INPUT -p tcp --dport 22 -j DROP   删除规则

- iptables -A INPUT -p tcp -i eth0 -s 192.168.33.0 -j DROP  禁止192.168.33.0网段从eth0网卡接入
- iptables -A INPUT -p tcp --dport 22 -i eth0 -s 192.168.33.61  -j ACCEPT
- iptables -A INPUT ! -s 192.168.10.10 -j DROP   禁止ip地址非192.168.10.10的所有类型数据接入
- iptables -I INPUT -p icmp --icmp-type 8 -s 192.168.50.100 -j DROP   禁止ip地址非192.168.10.100的ping请求
- iptables -I INPUT -p tcp --dport 22:80 -j DROP   匹配端口范围
- iptables -I INPUT -p tcp -m multiport --dport 22,80,3306 -j ACCEPT   匹配多个端口
- iptables -I OUTPUT -p tcp --sport 80 -j DROP 不允许源端口为80的数据流出

**扩展匹配：1.隐式扩展 2.显示扩展**
**隐式扩展**
- -p tcp
    - --sport PORT 源端口
    - --dport PORT 目标端口

**显示扩展：使用额外的匹配规则**
- -m EXTENSTION --SUB-OPT
- -p tcp --dport 22 与 -p tcp -m tcp --dport 22功能相同

**state：状态扩展，接口ip_contrack追踪会话状态**
- NEW：新的连接请求
- ESTABLISHED：已建立的连接请求
- INVALID：非法连接
- RELATED：相关联的连接
	
	
	   
## 6 自动化shell脚本

### 6.1 自动配置免密登陆

```commandline

SERVERS="node-3.itcast.cn node-4.itcast.cn"
PASSWORD=123456
BASE_SERVER=172.16.203.100

auto_ssh_copy_id() {
    expect -c "set timeout -1;
        spawn ssh-copy-id $1;
        expect {
            *(yes/no)* {send -- yes\r;exp_continue;}
            *assword:* {send -- $2\r;exp_continue;}
            eof        {exit 0;}
        }";
}

ssh_copy_id_to_all() {
    for SERVER in $SERVERS
    do
        auto_ssh_copy_id $SERVER $PASSWORD
    done
}

ssh_copy_id_to_all


for SERVER in $SERVERS
do
    scp install.sh root@$SERVER:/root
    ssh root@$SERVER /root/install.sh
done

```

### 6.2 自动下载部署tomcat脚本

```commandline
BASE_SERVER=mini4
yum install -y wget
wget $BASE_SERVER/soft/jdk-7u45-linux-x64.tar.gz
tar -zxvf jdk-7u45-linux-x64.tar.gz -C /usr/local
cat >> /etc/profile << EOF
export JAVA_HOME=/usr/local/jdk1.7.0_45
export PATH=\$PATH:\$JAVA_HOME/bin
EOF
```