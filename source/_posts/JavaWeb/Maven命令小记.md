---
title: Maven命令小记
date: 2017-03-09 20:21:06
tags: [JavaWeb]
---


## 1 基本命令
| 命令 |阶段| 功能 |
| --- | --- | --- |
|mvn clean|清理|清理输出的class文件|
|mvn compile|编译|将java代码编译成class文件|
|mvn test|测试|test下的单元测试代码一次性测试|
| mvn package | 打包 |java项目生成jar、web项目生成war |
|mvn install| 安装 | 将maven打包成jar或war发布到本地仓库|
| mvn tomcat:run | 部署 |以tomcat为容器启动项目 |

<!--more-->

执行后面阶段的命令，求安眠的命令会自动执行

## 2 仓库
**conf/settings.xml**

- 本地仓库
- 私服
- 中央仓库

 
## 3 创建maven项目
1.使用IDE创建MavenProject

- GroupId：公司名
- ArtifactId：项目名
- Version：版本号

2.依赖作用范围

    
| 依赖范围 | 含义 | 举例 |
| --- | --- | --- |
| compile | 编译范围，默认，在编译、测试、运行时都会被使用 | spring-core |
| provided | 只有当JDK或者容器已经提供该依赖之后才会使用 | servlet-api |
| runtime | 依赖子啊运行时和测试系统时需要，编译时不需要 | jdbc |
| test | 测试时需要的依赖，编译和运行时都不需要 | junit |
| system | 与provided类似，但是必须提供一个本地系统中jar文件的路径 | 不推荐使用 |





