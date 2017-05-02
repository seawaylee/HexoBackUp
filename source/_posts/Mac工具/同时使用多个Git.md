---
title:  如何在同一台电脑上使用两个Git账户
date: 2017-03-10 00:18:37
tags: [Mac工具,Git]
---

如果你想在一台机器使用两个github账号（比如私人账号和工作用账号）。这个时候怎么指定push到哪个账号的test仓库上去呢

解决方案是两套key，再写个配置文件，

注意生成两个Key时，不要随便输入enter键就就不会覆盖掉老的两个key
（假设你已经拥有私有账号且已经OK，现在想使用另一个工作用账号）：

<!--more-->

```commandline
1：为工作账号生成SSH Key

$ ssh-keygen -t rsa -C "your-email-address"

#存储key的时候，不要覆盖现有的id_rsa，在生成两个Key时，不要随便输入enter键就就不会覆盖掉老的两个key ,使用一个新的名字，比如id_rsa_work 
2：把id_rsa_work.pub加到你的work账号上

3：把该key加到ssh agent上。由于不是使用默认的.ssh/id_rsa，所以你需要显示告诉ssh agent你的新key的位置

$ ssh-add ~/.ssh/id_rsa_work

# 可以通过ssh-add -l来确认结果 
4：配置.ssh/config

$ vi .ssh/config

# 加上以下内容
#default github
Host github.com
  HostName github.com
  IdentityFile ~/.ssh/id_rsa

Host github_work
  HostName github.com
  IdentityFile ~/.ssh/id_rsa_work
 

5：这样的话，你就可以通过使用github.com别名github_work来明确说你要是使用id_rsa_work的SSH key来连接github，即使用工作账号进行操作。



#本地建库
$ git init
$ git commit -am "first commit'
 
#push到github上去
$ git remote add origin git@github_work:xxxx/test.git
$ git push origin master

```