---
title: Mysql快速重装与配置
date: 2017-09-04 19:12:41
tags: [Mysql]
---

# 1 彻底删除旧版

1. yum remove  mysql mysql-server mysql-libs mysql-server
2. find / -name mysql   将找到的相关东西delete掉；
3. rpm -qa|grep mysql  (查询出来的东东yum remove掉)

<!--more-->

# 2 安装、启动Mysql

1. yum install mariadb-server -y
2. systemctl start mariadb.service
3. systemctl enable mariadb.service
4. mysql_secure_installation    初始化配置
5. mysql

# 3 开启远程访问

- 方法一
    - 本地登入mysql，更改 "mysql" 数据库里的 "user" 表里的 "host" 项，将"localhost"改为"%"
    - mysql -u root -proot
    - mysql>use mysql;
    - mysql>update user set host = '%' where user = 'root';
    - mysql>select host, user from user;

- 方法二
    - 直接授权(推荐)
    - 从任何主机上使用root用户，密码：youpassword（你的root密码）连接到mysql服务器：
    - mysql -uroot -proot 
    - mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'youpassword' WITH GRANT OPTION;
