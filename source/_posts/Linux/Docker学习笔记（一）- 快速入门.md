---
title: Docker学习笔记（一）- 快速入门
date: 2017-06-22 01:54:57
tags: [Linux,Docker]
---
[TOC]

本文是我通过Docker官方文档学习时做的一些笔记。能够快速理清Docker中的各种概念并进行实践。

<!--more-->

# 1 安装、运行

## 1.1 下载
https://download.docker.com/mac/stable/Docker.dmg
下载完成后安装运行


## 1.2 测试是否启动成功

**docker [?]**

- version 
- info
- images
- ps [-a]
- run hello-world
- run -it ubuntu bash
- run -d -p 80:80 --name webserver nginx
- stop webserver
- rm -f webserver
- pull [image_name]
- start [image_name]

# 2 Get-Started
## 2.1 概览

### 2.1.1 什么是image

image是一个轻量的、独立的、可执行的包，其中包含了运行某一软件所需要的全部资源（代码、运行时环境、库、环境变量、配置文件）

### 2.1.2 什么是container

container是image的一个运行实例--image在内存中被执行后的结果。
container的运行完全与宿主隔离（默认），仅会根据配置去访问宿主机的文件、端口。
containers在宿主机的内核中运行各种app。与虚拟机相比的优点是：这些containers比虚拟机更有特质。
每一个container都运行于不同的进程中，不占用用其他程序的内存。

### 2.1.3 VM与Container的对比

**VM**

![](14980137567058.png)


**Container**

![](14980137685830.png)

### 2.1.4 Docker的层级

* Stack: 一组相互关联的服务，共享依赖关系，并且可以协调一致。
* Services: 对containers的管理与配置，构建出一个服务
* Container: app真实运行的环境，

## 2.2 Containers

将应用的依赖、运行时环境打包并运行在一个image中。
使用Dockerfile定义一个container。

### 2.2.1 创建Dockerfile
Dockerfile是container的配置文件
`mkdir -p docker_envs/firstapp && cd docker_envs/firstapp && vim Dockerfile`
**Dockerfile**

```
# Use an official Python runtime as a base image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

### 2.2.2 创建其他配置文件

从Dockerfile的配置中可以看到，我们还需要 app.py requirements.txt 两个文件

**requirements.txt**


```txt
Flask
Redis
```

**app.py**

```python
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
	app.run(host='0.0.0.0', port=80)
```

当前 firstapp文件夹下有以下三个文件
`Dockerfile       app.py           requirements.txt`

### 2.2.3 构建APP

* 构建app `docker build friendlyhello`
* 查看镜像 `docker images`
* 运行app `docker run -p 4000:80 friendlyhello`
* 访问app localhost:4000 即可访问container中80端口启动的服务

![](14980238387578.jpg)


* 后台运行app `docker run -d -p 4000:80 friendlyhello`
* 查看container进程 `docker ps`
* 关闭container `docker stop [CONTAINER ID]`

### 2.2.4 共享image

#### 2.2.4.1 注册一个docker账号

https://cloud.docker.com/

#### 2.2.4.2 登陆

`docker login`
输入用户名密码，若登陆成功则显示`Login Succeeded`

#### 2.2.4.3 将本地image注册到docker云
打标签的格式 `docker tag image username/repository:tag`
举例 `docker tag friendlyhello NikoBelic/get-started:part1`
上传 `docker push NikoBelic/get-started:part1`
到docker云上查看
![](14980260054158.jpg)


远程获取并运行 `docker run -p 4000:80 username/repository:tag

### 2.2.5 最常用命令

```commandline
docker build -t friendlyname .  # Create image using this directory's Dockerfile
docker run -p 4000:80 friendlyname  # Run "friendlyname" mapping port 4000 to 80
docker run -d -p 4000:80 friendlyname         # Same thing, but in detached mode
docker ps                                 # See a list of all running containers
docker stop <hash>                     # Gracefully stop the specified container
docker ps -a           # See a list of all containers, even the ones not running
docker kill <hash>                   # Force shutdown of the specified container
docker rm <hash>              # Remove the specified container from this machine
docker rm $(docker ps -a -q)           # Remove all containers from this machine
docker images -a                               # Show all images on this machine
docker rmi <imagename>            # Remove the specified image from this machine
docker rmi $(docker images -q)             # Remove all images from this machine
docker login             # Log in this CLI session using your Docker credentials
docker tag <image> username/repository:tag  # Tag <image> for upload to registry
docker push username/repository:tag            # Upload tagged image to registry
docker run username/repository:tag                   # Run image from a registry
```

## 2.3 Services

一个分布式应用程序，其不同的部分被称为 服务。
一般情况下服务的扩展非常困难，需要修改程序的实例数量、分配更多的计算资源等等。
而在docker平台上，使用docker-compose.yml文件即可完成。

### 2.3.1 第一个 docker-compose.yml文件

- 创建文件 docker-compose.yml 于第二2节中相同的文件夹下


```json
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: username/repository:tag
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "80:80"
    networks:
      - webnet
networks:
  webnet:
```

- 初始化 docker集群   `docker swarm init`
- 部署  `docker stack deploy -c docker-compose.yml getstartedlab`
- 查看实例 `docker stack ps getstartedlab`


```
ID            NAME                 IMAGE                      NODE  DESIRED STATE  CURRENT STATE           ERROR  PORTS
z7tku2lor2y9  getstartedlab_web.1  lixiwei/get-started:part1  moby  Running        Running 14 seconds ago
wpc8euz8h3lt  getstartedlab_web.2  lixiwei/get-started:part1  moby  Running        Running 14 seconds ago
vda14cmy3yxh  getstartedlab_web.3  lixiwei/get-started:part1  moby  Running        Running 15 seconds ago
0pcmypr87bba  getstartedlab_web.4  lixiwei/get-started:part1  moby  Running        Running 14 seconds ago
l9adv4uufeer  getstartedlab_web.5  lixiwei/get-started:part1  moby  Running        Running 15 seconds ago
```

- 多次访问 `localhost:80` 发现 CONTAINER ID 不相同，原因是load-balance采用了轮询机制
- 修改replicas后再次部署 `docker stack deploy -c docker-compose.yml getstartedlab`
- 关闭应用  `docker stack rm getstartedlab`
- 关闭应用后仍然会有一个单节点集群在运行(`docker node ls` 查看) 使用 `docker swarm leave --foce` 关闭

**本小节使用命令**

```commandline
docker stack ls              # List all running applications on this Docker host
docker stack deploy -c <composefile> <appname>  # Run the specified Compose file
docker stack services <appname>       # List the services associated with an app
docker stack ps <appname>   # List the running containers associated with an app
docker stack rm <appname>                             # Tear down an application
```

## 2.4 Swarms

### 2.4.1 概念

Swarm 是一个运行着docker的集群可以向SwarmManager发送docker指令。

 - SwarmManager    在其环境上运行commands，像单机模式一样。
 - Workers    提供程序运行空间，但不能给其他worker发送指令。

### 2.4.2 配置Swarm

Swarm由多个节点组成，节点可以是真实物理机也可以使虚拟机。
在SwarmManager使用 `docker swarm init` 来激活swarm模式。
在Worker使用 `docker swarm join` 来将机器加入到集群中。

### 2.4.3 创建集群

- 首先需要安装virturalbox虚拟机软件 `brew cask install virtualbox`
- 创建虚拟机 
    - `docker-machine create --driver virtualbox myvm1`
    
    ```
    Running pre-create checks...
    (myvm1) No default Boot2Docker ISO found locally, downloading the latest release...
    (myvm1) Latest release for github.com/boot2docker/boot2docker is v17.05.0-ce
    (myvm1) Downloading /Users/lixiwei-mac/.docker/machine/cache/boot2docker.iso from https://github.com/boot2docker/boot2docker/releases/download/v17.05.0-ce/boot2docker.iso...
    (myvm1) 0%....10%....20%....30%....40%....50%....60%....70%....80%....90%....100%
    Creating machine...
    (myvm1) Copying /Users/lixiwei-mac/.docker/machine/cache/boot2docker.iso to /Users/lixiwei-mac/.docker/machine/machines/myvm1/boot2docker.iso...
    (myvm1) Creating VirtualBox VM...
    (myvm1) Creating SSH key...
    (myvm1) Starting the VM...
    (myvm1) Check network to re-create if needed...
    (myvm1) Found a new host-only adapter: "vboxnet0"
    (myvm1) Waiting for an IP...
    Waiting for machine to be running, this may take a few minutes...
    Detecting operating system of created instance...
    Waiting for SSH to be available...
    Detecting the provisioner...
    Provisioning with boot2docker...
    Copying certs to the local machine directory...
    Copying certs to the remote machine...
    Setting Docker configuration on the remote daemon...
    Checking connection to Docker...
    Docker is up and running!
    To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env myvm1
    ```

    - `docker-machine create --driver virtualbox myvm2`
- 查看虚拟机  `docker-machine ls`

```
NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
myvm1   -        virtualbox   Running   tcp://192.168.99.100:2376           v17.05.0-ce
myvm2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.05.0-ce
```

- 向虚拟机发送命令
    - `docker-machine ssh myvm1 "docker swarm init"`(指定myvm1为manager)
        
        - 若出现错误类似 --advertise-addr
        
            ```
            Error response from daemon: could not choose an IP address to advertise since this system has multiple addresses on different interfaces (10.0.2.15 on eth0 and 192.168.99.100 on eth1) - specify one with --advertise-addr
            exit status 1
            ```
            则使用 `docker-machine ssh myvm1 "docker swarm init --advertise-addr 192.168.99.100:2377"`
            设置成功
        
            ```
            Swarm initialized: current node (k650z1bprqz6qkvqhgpzvyk4k) is now a manager.
            
            To add a worker to this swarm, run the following command:
            
                docker swarm join \
                --token SWMTKN-1-5s83l6iknt9impn1pwdkog4oak3jktyjpy6de384juxsbcms1e-0hknwqjug1slnw1ehe7kgkzjc \
                192.168.99.100:2377
            
            To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
            ```
    - 指定myvm2加入swarm `docker-machine ssh myvm2 "docker swarm join --token SWMTKN-1-5s83l6iknt9impn1pwdkog4oak3jktyjpy6de384juxsbcms1e-0hknwqjug1slnw1ehe7kgkzjc 192.168.99.100:2377"`
        若成功，则显示  `This node joined a swarm as a worker.`
     
    - 进入myvm2的终端 `docker-machine ssh myvm2`
        
        ```
                                ##         .
                          ## ## ##        ==
                       ## ## ## ## ##    ===
                   /"""""""""""""""""\___/ ===
              ~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ /  ===- ~~~
                   \______ o           __/
                     \    \         __/
                      \____\_______/
         _                 _   ____     _            _
        | |__   ___   ___ | |_|___ \ __| | ___   ___| | _____ _ __
        | '_ \ / _ \ / _ \| __| __) / _` |/ _ \ / __| |/ / _ \ '__|
        | |_) | (_) | (_) | |_ / __/ (_| | (_) | (__|   <  __/ |
        |_.__/ \___/ \___/ \__|_____\__,_|\___/ \___|_|\_\___|_|
        Boot2Docker version 17.05.0-ce, build HEAD : 5ed2840 - Fri May  5 21:04:09 UTC 2017
        Docker version 17.05.0-ce, build 89658be
        docker@myvm2:~$
        ```
    - 查看集群节点  `docker-machine ssh myvm1`  `docker node ls`
    
    
    
    ```
    ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
    k650z1bprqz6qkvqhgpzvyk4k *   myvm1               Ready               Active              Leader
    pono9aqai63bx5iekdhy9x8if     myvm2               Ready               Active
    ```


### 2.4.4 在集群中部署app

最困难的步骤已经完成了，现在只需要将上一节中的程序部署到新的swarm中即可。
一定要记住，只有向myvm1这样的manager才能执行docker命令行，workers只用来工作。

- 复制docker-compose.yml到myvm1的home目录下
`docker-machine scp docker-compose.yml myvm1:~`

- 使用myvm1部署app `docker-machine ssh myvm1 "docker stack deploy -c docker-compose.yml getstartedlab"`
- 在上一节中的所有命令都可以在此节点中使用，例如

`docker-machine ssh myvm1 "docker stack ps getstartedlab"`

```
ID                  NAME                   IMAGE                       NODE                DESIRED STATE       CURRENT STATE              ERROR               PORTS
allz0w2fabv7        getstartedlab_web.1    lixiwei/get-started:part1   myvm1               Running             Preparing 37 seconds ago
l7w76v6ldure        getstartedlab_web.2    lixiwei/get-started:part1   myvm1               Running             Preparing 37 seconds ago
jei5oc8wuvqr        getstartedlab_web.3    lixiwei/get-started:part1   myvm1               Running             Preparing 37 seconds ago
xt6vr4dud1le        getstartedlab_web.4    lixiwei/get-started:part1   myvm1               Running             Preparing 37 seconds ago
j95zpui1u926        getstartedlab_web.5    lixiwei/get-started:part1   myvm2               Running             Preparing 37 seconds ago
v0lqwn40q6ui        getstartedlab_web.6    lixiwei/get-started:part1   myvm2               Running             Preparing 37 seconds ago
z5nkfsixu6uc        getstartedlab_web.7    lixiwei/get-started:part1   myvm2               Running             Preparing 37 seconds ago
gsn6xv4jenoo        getstartedlab_web.8    lixiwei/get-started:part1   myvm1               Running             Preparing 37 seconds ago
ottug1z7sy17        getstartedlab_web.9    lixiwei/get-started:part1   myvm2               Running             Preparing 37 seconds ago
x0zeq340z8i2        getstartedlab_web.10   lixiwei/get-started:part1   myvm2               Running             Preparing 37 seconds ago
```

    

### 2.4.5 访问集群

- 查看所有虚拟机  `docker-machine ls`


```
NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
myvm1   -        virtualbox   Running   tcp://192.168.99.100:2376           v17.05.0-ce
myvm2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.05.0-ce
```

可以通过myvm1或者myvm2来访问app。刚才创建的app在它们之间被共享并且负载均衡。
两个IP都能正常访问的原因是，swarm集群中的节点参与了入口的网络路由。

下图描述了swarm集群中节点的路由方式

![](14980567079650.png)

### 2.4.6 配置APP

- 现在你可以使用2.3节中的方式，修改docker-compose.yml文件来扩展你的app。
- 修改代码以改变app的的功能。
- 使用 `docker stack deploy` 来重新部署改变的内容
- 向swarm集群中添加新的机器后，需要执行 `docker stack deploy`来让app在新的节点中运行

### 2.4.7 关闭app

`docker-machine ssh myvm1 "docker stack rm getstartedlab"`

### 2.4.8 移除swarm节点

- 移除worker  `docker-machine ssh myvm2 "docker swarm leave"`
- 移除manager `docker-machine ssh myvm1 "docker swarm leave --force"`



### 2.4.9 小节总结

本节中，我们学习了如何指定swarm集群中的节点作为manager或者worker，并在集群中部署应用。
我们发现本节所使用的核心命令几乎与2.3节完全一致，只是命令作用于集群之上了。
我们也看到了docker网络系统的强大，对containers的负载均衡策略，即使container位于不同的机器也可以被负载均衡所调用。

以下是本节所用命令总结


```
docker-machine create --driver virtualbox myvm1 # Create a VM (Mac, Win7, Linux)
docker-machine create -d hyperv --hyperv-virtual-switch "myswitch" myvm1 # Win10
docker-machine env myvm1                # View basic information about your node
docker-machine ssh myvm1 "docker node ls"         # List the nodes in your swarm
docker-machine ssh myvm1 "docker node inspect <node ID>"        # Inspect a node
docker-machine ssh myvm1 "docker swarm join-token -q worker"   # View join token
docker-machine ssh myvm1   # Open an SSH session with the VM; type "exit" to end
docker-machine ssh myvm2 "docker swarm leave"  # Make the worker leave the swarm
docker-machine ssh myvm1 "docker swarm leave -f" # Make master leave, kill swarm
docker-machine start myvm1            # Start a VM that is currently not running
docker-machine stop $(docker-machine ls -q)               # Stop all running VMs
docker-machine rm $(docker-machine ls -q) # Delete all VMs and their disk images
docker-machine scp docker-compose.yml myvm1:~     # Copy file to node's home dir
docker-machine ssh myvm1 "docker stack deploy -c <file> <app>"   # Deploy an app
```

## 2.5 Stack

在2.4节中，我们学会了如何建立一个swarm集群，并在集群中的多个机器上部署我们的app。

本节将学习Stack。

- Stack是一组相互关联的服务，共享依赖关系，并且可以协调一致。
- 单个Stack能够定义和协调整个应用程序的功能（尽管非常复杂的应用程序可能希望使用多个Stack）。

我们在2.3节中其实已经使用过Stack，例如创建compose_file之后，使用`docker stack deploy`。但那仅是单个Stack运行于host上，在生产环境中这并不是常用的方式。


### 2.5.1 创建新的Service并部署

在docker-compose.yml添加一个新的Service很容易。

- 编辑 docker-compose.yml


```
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: username/repo:tag
    deploy:
      replicas: 5
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
    ports:
      - "80:80"
    networks:
      - webnet
  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      - webnet
networks:
  webnet:

```
与之前相比唯一新出现的就是一个对等的web服务--visualizer（可视化）。
volumes: 允许visualizer访问宿主机的socket文件
placement: 确保这个服务只在swarm的manager节点上运行


- 复制service配置文件到manager
`docker-machine scp docket-compose.yml myvm1:~`

- 重新部署
`docker-machine ssh myvm1 "docker stack deploy -c docker-compose.yml getstartedlab"`

更新services成功

```
Updating service getstartedlab_web (id: 43lo6a1be3tr03bxkekt32kw5)
Creating service getstartedlab_visualizer
```

- 查看可视化
![](14980638673223.jpg)

正如我们之前配置的，visualizer只有一个实例，而web则在swarm集群中有很多实例。
我们可以使用 `docket-machine ssh myvm1 "docker stack ps getstartedlab"`来证实其可视化结果

```
ID                  NAME                         IMAGE                             NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
fm0v360ndv24        getstartedlab_visualizer.1   dockersamples/visualizer:stable   myvm1               Running             Running 6 minutes ago
l7w76v6ldure        getstartedlab_web.2          lixiwei/get-started:part1         myvm1               Running             Running 6 hours ago
jei5oc8wuvqr        getstartedlab_web.3          lixiwei/get-started:part1         myvm1               Running             Running 6 hours ago
z5nkfsixu6uc        getstartedlab_web.7          lixiwei/get-started:part1         myvm2               Running             Running 6 hours ago
gsn6xv4jenoo        getstartedlab_web.8          lixiwei/get-started:part1         myvm1               Running             Running 6 hours ago
x0zeq340z8i2        getstartedlab_web.10         lixiwei/get-started:part1         myvm2               Running             Running 6 hours ago
```

下面我们创建一个需要有依赖的service：redis service 用来提供访问计数

### 2.5.2 持久化数据

- 继续修改 docker-compose.yml 添加redis服务


```
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: username/repo:tag
    deploy:
      replicas: 5
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
    ports:
      - "80:80"
    networks:
      - webnet
  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      - webnet
  redis:
    image: redis
    ports:
      - "6379:6379"
    volumes:
      - ./data:/data
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      - webnet
networks:
  webnet:
```

Redis在docker中有官方的image，所以不需要提供username/repo。
redis的端口6379已经在container中被预设，并且映射到了host（此时host不是我们所用的主机，而是myvm1这台虚拟机）的6379端口。
更重要的是，在stack中部署和使用redis进行持久化数据时

- redis只能在manager中运行，所以它永远使用同一个文件系统
- redis在宿主机内任意的文件夹中所访问的文件，都将存储在/data

综上，redis将创建 “source of truth” 在你的host文件系统中用来存储data。若没有这个东西，redis将会把数据存储在container所在文件系统的/data中，也就是说如果container被重新部署了，那么redis存储的data将会被擦除。

“source of truth”由两部分构成：

- 确保Redis服务永远使用同一个host
- 创建的容器将 ./data(在host上) 作为 /data(redis container内部)访问。当containers创建或销毁，存储在主机上 ./data上的文件将被持久化，从而实现连续性。

下面准备部署使用Redis的Stack

- 在manager中创建 ./data目录
`docker-machine ssh myvm1 "mkdir ./data"`

- 上传新的docker-compose.yml文件
`docker-machine scp docker-compose.yml myvm1:~`

- 再次部署服务
`docker-machine ssh myvm1 "docker stack deploy -c docker-compose.yml getstartedlab"`

- 查看结果

![](14980660196683.jpg)

![](14980659880173.jpg)


## 2.6 [部署APP](https://docs.docker.com/get-started/part6/)



