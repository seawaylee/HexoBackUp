---
title: Oracle数据库运维、备份常用指令 
date: 2017-03-09 22:04
tags: [Oracle]
---


## 1. Oracle数据泵备份导出

### 1.1 准备工作
**在linux系统下创建导出结果存放的文件夹**，切记要切换到oracle用户创建，否则会出现权限问题。

<!--more-->

```commandline
su - oracle
mkdir /home/oracle/dwcr_dumps
```
如果使用root创建了文件夹，可是使用命令来修改文件权限。

```commandline
chown /home/oracle/dumps oralce:oinstall
```
---
**连接oracle，创建导出映射地址、用户授权**
先查看当前实例是不是你需要的

```commandline
echo $ORACLE_SID
```
如果不是，修改一下

```commandline
export ORACLE_SID=dwcr
```

进入oracle命令环境,使用管理员权限登陆，不需要密码.

```commandline
sqlplus / as sysdba
```

Oracle参数举例:

| Key | Value |
| --- | --- |
| 用户名 | dwcr |
| 密码 | password |
| 表空间名 | dwcr_tablespace |
| 实例名 | dwcrsid |
| 导出路径 | /home/oracle/dwcr_dumps |
| 数据库IP | 192.168.1.1100 |


**创建导出地址的映射**（内部别名->Linux文件系统）

```sql
create or replace directory DWCRDUMP as '/home/oracle/dwcr_dumps';
```
**给用户授权**

```sql
grant read,write on directory DWCRDUMP to dwcr;
```

**数据泵导出**
退出oracle命令环境后
```commandline
expdp  dwcr/dwcr@192.168.1.1100:1521/dwcrsid  DIRECTORY=DWCRDUMP DUMPFILE=dwcr_20161206_1446.dmp  LOGFILE=dwcr_20161206_1446.log   SCHEMAS=dwcr 
```
假如想从11g的数据库导出，并导入到10g的数据库，需要在结尾加一个参数

```commandline
version=10.2.0.1.0
```
等待导出**正常**结束后，就可以去linux绝对路径下拷贝dmp文件了。
如果导出期间出现异常，根据ORA的错误提示，自行百度，无外乎就是那几种常见错误。


**数据泵导入**
**注意**:也需要创建文件映射、对用户授权，需要先将dmp文件放到指定的linux路径中供Oracle读取。
```commandline
impdp dwcr/dwcr@192.168.1.1200:1521/dwcrsid directory=DWCRDUMP schemas=dwcr dumpfile=dwcr_20161206_1446.dmp  
```




## 2. 使用Python脚本导出/定时备份
### 2.1 安装Python

1、在官方网站下载python安装包，这里注意python.org/download路径被屏蔽，需要使用http://www.python.org/页面上的中文“下载”链接进行下载。
这里下载了python最新的3.2.2版本：Python-3.2.2.tgz
下载后，文件目录在/home/python/下，这也是我python的安装目录

2、解压：

```commandline
[oot@www python]# tar zxvf    Python-3.2.2.tgz

3、打开安装目录，执行：
[root@www python]# cd Python-3.2.2
[root@www Python-3.2.2]#./configure
[root@www Python-3.2.2]# make
[root@www Python-3.2.2]# makeinstall
值此，安装完成。

4、但此时输入”python”命令，仍然显示是旧版本的，这就需要创建软连接：
[root@www bin]# cd /usr/bin
[root@www bin]# ll |grep python
[root@www bin]# rm -rf python
[root@www bin]# ln -s /home/python/Python-3.2.2/python python
[root@www bin]# python
Python 3.2.2 (default, Oct 26 2011, 23:40:16)
[GCC 4.1.2 20071124 (Red Hat 4.1.2-42)] on linux2
Type "help", "copyright", "credits" or"license" for more information.
```

**看一下 如果出现了以上提示信息就说明安装成功了 按ctrl+z退出来就好了。**


### 2.2 批量导出数据库脚本的编写
前阵子有个需求，需要一个人对10+台Oracle进行备份，每个数据库的信息又不相同，因此编写了一个简单的python导出脚本来完成这个工作。呵呵，酸爽。


```python
import os
import time
import sched

ISOTIMEFORMAT = '%Y%m%d%'
IP_ADDR = [
    '192.168.1.169', '192.168.1.172', '192.168.1.173',
    '192.168.1.167', '192.168.1.168', '192.168.1.168',
    '192.168.1.170', '192.168.1.170', '192.168.1.170', '192.168.1.170',
    '192.168.1.171', '10.161.8192.168.16.71', '192.168.1.171']
SID = [
    'dwcr', 'dwcrdci', 'dwcrweb',
    'ISRC2', 'cpcc', 'cpcc',
    'orcl', 'orcl', 'orcl2', 'orcl2',
    'classcas', 'copyrigh', 'custody']
USERNAME = [
    'dwcr1', 'dwcrdci', 'dwcrweb',
    'isrc2', 'cpccfs_develop', 'cpccfs_opus',
    'cac_develop', 'cac_opus', 'cac_develop', 'cac_opus',
    'lab1107', 'lab1107', 'custody2']
PASSWORD = [
    'dwcr1', 'dwcrdci', 'dwcrweb',
    'isrc2', 'cpccfs', 'cpccfs',
    'cac', 'cac', 'cac_develop_7112', 'cac_opus_7112',
    '237133', '237133', 'custody2']
TABLESPACE_NAME = [
    'dwcr_dumps', 'dwcrdci_dumps', 'dwcrweb_dumps',
    'isrc2', 'cpccfs_develop', 'cpccfs_opus',
    'softnew', 'opusnew', 'softold', 'opusold',
    'classcas', 'copyrigh', 'custody2']
DUMP_FILENAME = [
    'DWCRDUMP', 'DWCRDCIDUMP', 'DWCRWEBDUMP',
    'DP_ISRC2', 'CPCCFS_BACKUP', 'CPCCFS_BACKUP',
    'DAILI_NEW', 'DAILI_NEW', 'DAILI_OLD', 'DAILI_OLD',
    'THREEDUMP', 'THREEDUMP', 'THREEDUMP']

if __name__ == "__main__":
    now_time = time.strftime("%Y%m%d%H%M")
    print("当前系统时间:", now_time)

    for i in range(IP_ADDR.__len__()):
        ip_addr = IP_ADDR[i]
        sid = SID[i]
        username = USERNAME[i]
        password = PASSWORD[i]
        tablespace_name = TABLESPACE_NAME[i]
        dump_filename = DUMP_FILENAME[i]

        # DWCR Keep 10 days file

        if sid == 'dwcr':
            dir_path = "/home/oracle/dwcr_dumps"
            all_dmpfiles = os.listdir(dir_path)
            all_dmpfiles.sort()
            print(all_dmpfiles)
            if all_dmpfiles.__len__() > 20:
                del_file_num = all_dmpfiles.__len__() - 20
                print("即将删除10天前的文件:", del_file_num, " 个")
                for file in all_dmpfiles[:del_file_num]:
                    print('rm -f %s' % dir_path + "/" + file)
                    os.system('rm %s' % dir_path + "/" + file)
        # Export the new dmp file
        # expdp  dwcr1/dwcr1@192.168.1.1105:1521/dwcr  DIRECTORY=DWCRDUMP DUMPFILE=dwcr1_20161127.dmp  LOGFILE=dwcr1_20161127.log   SCHEMAS=dwcr1
        print("即将泵出", ip_addr, sid, username, tablespace_name, dump_filename)
        excmd = 'expdp %s/%s@%s:1521/%s  DIRECTORY=%s DUMPFILE=%s_%s.dmp  LOGFILE=%s_%s.log   SCHEMAS=%s' % \
                (username, password, ip_addr, sid, dump_filename, tablespace_name, now_time, tablespace_name, now_time,
                 username)
        print(excmd)
        result = os.system(excmd)
        print("执行结果:", result)
        os.system('exit')
    print("计划内导出数据库文件的服务器IP列表为:")
    for i in range(IP_ADDR.__len__()):
        print(i + 1, "->", IP_ADDR[i],"->","/home/oralce/"+TABLESPACE_NAME[i])
    print("请分别到各数据库服务器拷贝文件:", '*' + now_time + "*\n")

    print('''计划外导出数据库文件的服务器IP列表为:
    IP:192.168.1.166 -> /home/oracle/autobackup/(当天夜里3点的文件)
    IP:192.168.1.13  -> navicat/sqlyog连接 root/1Q2w3e4r@ 导出sql文件
    ''')



    print('===========结束==========')

```
其中一下部分代码可以删除，因为有的数据库文件特别大，我每次又都懒得手动删除，因此写了这一段代码，将导出的文件按时间排序，删除超过n天的导出文件 

```python
if sid == 'dwcr':
            dir_path = "/home/oracle/dwcr_dumps"
            all_dmpfiles = os.listdir(dir_path)
            all_dmpfiles.sort()
            print(all_dmpfiles)
            if all_dmpfiles.__len__() > 20:
                del_file_num = all_dmpfiles.__len__() - 20
                print("即将删除10天前的文件:", del_file_num, " 个")
                for file in all_dmpfiles[:del_file_num]:
                    print('rm -f %s' % dir_path + "/" + file)
                    os.system('rm %s' % dir_path + "/" + file)
```
将脚本拷贝到/home/oracle下
执行方法：

```commandline
su - oracle
cd /home/oracle/
python _export.py
```
**Lift is short , I need python.**
喝杯咖啡等着去吧~ 

### 2.3 全自动定时数据库备份
什么？你还不满意？不想天天去机房备份？
**yo~满足你**

定时器版python备份脚本

```python
import os
import time
import sched

ISOTIMEFORMAT = '%Y%m%d%'
IP_ADDR = ['192.168.1.169:1521', '192.168.1.172:1521', '192.168.1.173:1521']
SID = ['dwcr', 'dwcrdci', 'dwcrweb']
USERNAME = ['dwcr1', 'dwcrdci', 'dwcrweb']
TABLESPACE_NAME = ['dwcr1', 'dwcrdci', 'dwcrweb']
DUMP_FILENAME = ['DWCRDUMP', 'DWCRDCIDUMP', 'DWCRWEBDUMP']
DIR_PATH = ['dwcr_dumps', 'dwcrdci_dumps', 'dwcrweb_dumps']

SLEEP_TIME = 60 * 60 * 24


def export_Oracle():
    now_time = time.strftime("%Y%m%d%H%M")
    print("当前系统时间:", now_time)
    for i in range(3):
        ip_addr = IP_ADDR[i]
        sid = SID[i]
        username = USERNAME[i]
        tablespace_name = TABLESPACE_NAME[i]
        dump_filename = DUMP_FILENAME[i]

        # Keep 10 days file
        dir_path = "/home/oracle/" + DIR_PATH[i]
        all_dmpfiles = os.listdir(dir_path)
        all_dmpfiles.sort()
        print(all_dmpfiles)

        if all_dmpfiles.__len__() > 20:
            del_file_num = all_dmpfiles.__len__() - 20
            print("即将删除10天前的文件:", del_file_num, " 个")
            for file in all_dmpfiles[:del_file_num]:
                print('rm -f %s' % dir_path + "/" + file)
                os.system('rm %s' % dir_path + "/" + file)
        # Export the new dmp file
        # expdp  dwcr1/dwcr1@192.168.1.105:1521/dwcr  DIRECTORY=DWCRDUMP DUMPFILE=dwcr1_20161127.dmp  LOGFILE=dwcr1_20161127.log   SCHEMAS=dwcr1
        print("即将泵出", ip_addr, sid, username, tablespace_name, dump_filename)
        excmd = 'expdp %s/%s@%s/%s  DIRECTORY=%s DUMPFILE=%s_%s.dmp  LOGFILE=%s_%s.log   SCHEMAS=%s' % \
                (username, username, ip_addr, sid, dump_filename, tablespace_name, now_time, tablespace_name, now_time,
                 username)
        print(excmd)
        result = os.system(excmd)
        print("执行结果:", result)
    os.system('exit')
    print('===========结束==========')


def run_function():
    s = sched.scheduler(time.time, time.sleep)
    # 设置一个调度,因为time.sleep()的时间是一秒,所以timer的间隔时间就是sleep的时间,加上enter的第一个参数
    s.enter(SLEEP_TIME, 2, export_Oracle)
    s.run()


def timer():
    while True:
        run_function()


if __name__ == "__main__":
    timer()

```

其中以下这部分代码用来控制自动备份间隔时间，假如你今天早晨8点执行了脚本，备份会立即执行一次，然后下次执行就是明天早晨的8点了。自己按需配置吧。脚本几乎和之前的一样，只是套了一层定时器而已。

```commandline
SLEEP_TIME = 60 * 60 * 24
```
执行方法：
其中 --fork 是将该进程设置为守护进程，防止系统误杀。
```commandline
python /home/oracle/time_export.py --fork
```

## 3 常用Oracle语句
首先进入Oralce命令环境

```commandline
sqlplus / as sysdba
```
### 3.1 创建表空间

```sql
create tablespace dwcr1
logging
datafile '/DWCR_DBV2/dwcr/dwcr1_01.dbf'
size 800m
autoextend on
next 100m
maxsize unlimited
extent management local;
```
### 3.2 创建用户、给用户分配表空间

```sql
create user dwcrdci identified by dwcrdci
default tablespace dwcrdci;
```
### 3.3 给用户授权

```sql
grant dba,resource,connect to dwcrdci;
```

## 4 常用Oracle命令

```sql
# 查看dump文件映射
select * from dba_directories;
# 查看所有用户
select * from dba_users;
# 查看所有表空间
select * from dba_tablespaces;
# 查看用户的默认表空间
select   username,   DEFAULT_TABLESPACE     from   dba_users where username='dwcr1';
# 查看表空间所有文件路径
select * from dba_data_files where tablespace_name='dwcr1';
# 查看表在哪个表空间下
select tablespace_name,table_name from user_talbes where table_name='td_product';
# 查看表空间使用情况
SELECT tbs 表空间名,                                    
    sum(totalM) 总共大小M,                                    
    sum(usedM) 已使用空间M,                                    
    sum(remainedM) 剩余空间M,                                    
    sum(usedM)/sum(totalM)*100 已使用百分比,                            
    sum(remainedM)/sum(totalM)*100 剩余百分比                            
    FROM(                                            
     SELECT b.file_id ID,                                    
     b.tablespace_name tbs,                                    
     b.file_name name,                                    
     b.bytes/1024/1024 totalM,                                    
     (b.bytes-sum(nvl(a.bytes,0)))/1024/1024 usedM,                        
     sum(nvl(a.bytes,0)/1024/1024) remainedM,                            
     sum(nvl(a.bytes,0)/(b.bytes)*100),                                
     (100 - (sum(nvl(a.bytes,0))/(b.bytes)*100))                            
     FROM dba_free_space a,dba_data_files b                            
     WHERE a.file_id = b.file_id                                
     GROUP BY b.tablespace_name,b.file_name,b.file_id,b.bytes                    
     ORDER BY b.tablespace_name                                
    )                                            
    GROUP BY tbs                                        
# 重新调整表空间大小
alter database datafile 'D:\Oracle\PRODUCT\ORADATA\TEST\USERS01.DBF' resize 1000m;
# 设置自动扩展大小
alter database datafile 'D:\ORACLE\PRODUCT\ORADATA\TEST\USERS01.DBF' autoextend on next 200m maxsize unlimited;
# 增加数据文件
alter tablespace yourtablespacename add datafile 'd:\newtablespacefile_02.dbf' size 800m;

# 查看和修改用户默认表空间
alter user dwcr1 default tablespace dwcr1 temporary tablespace temp;
alter user dwcr1 TEMPORARY TABLESPACE DWCR_TEMP;
# 修改数据库默认表空间
alter database default temporary tablespace temp;
# 查看数据库最大连接数
select value from v$parameter where name ='processes' 
# 查看当前连接数
select count(*) from v$process;
# 修改最大连接数
alter system set processes = 300 scope =spfile;
# 查看当前并发量
select count(*) from v$session where status='ACTIVE'
# 查看参数
show parameter processes ;
# 查看当前session连接数
select count(*) from v$session
# 查看哪些用户正在使用数据库
SELECT osuser, a.username,cpu_time/executions/1000000||'s',b.sql_text,machine
from v$session a, v$sqlarea b
where a.sql_address =b.address order by cpu_time/executions desc;
# 批量杀掉连接
SELECT 'alter system kill session ' || '''' ||t.sid ||','||t.SERIAL#|| '''' FROM v$session t WHERE t.USERNAME='DWCR1';
# 删除用户
drop user dwcr cascade;
# 彻底删除表空间(包括内容,文件)
drop tablespace DWCR including contents and datafiles;
```

**转载请注明出处，谢谢。**

