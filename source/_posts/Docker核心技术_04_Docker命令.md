---
title: 04 | Docker命令
date: 2019-07-03 17:40:42
tags: [Docker]
categories :
- 学习笔记
- Docker
- Docker核心技术
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">帮助命令</font></center>
```
docker version
docker info
docker --help
```

#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">镜像命令</font></center>
**1、列出本机的镜像**
```
docker images [OPTIONS]
```
<font color = "#CD5555">表头信息：</font>
- **REPOSITORY**：镜像的仓库源。              
- **TAG**：镜像的标签（同一仓库源可以有多个TAG）。 
- **IMAGE ID**：镜像ID。                 
- **CREATED**：镜像创建时间。              
- **SIZE**：镜像大小。  


<font color = "#CD5555">OPTIONS：</font>
- **-a**：列出本地所有的镜像（含中间映像层 ）。
- **-q**：只显示镜像ID。            
- **--digests**：显示镜像的摘要信息。         
- **--no-trunc**：显示完整的镜像信息。         

<font color = "#CD5555">镜像就相当于一个千层饼</font>。


**2、Docker镜像查找**
```
docker search [IMAGE_NAME] #可模糊查找，即使配了阿里云镜像加速，也会到DockerHub查。
```
<font color = "#CD5555">OPTIONS：</font>
- **--no-trunc**：显示完整的镜像描述。
- **-s**：列出收藏数不小于指定值的镜像。
- **--automated**：只列出automated build类型的镜像。



**3、拉Docker镜像**
```
docker pull [IMAGE_NAME]
```

**4、删除镜像**
```
docker rmi -f [IMAGE_ID]                            #删除单个
docker rmi -f [IMAGE_NAME] [IMAGE_NAME]    #删除多个
docker rmi -f $(docker images -qa)             #删除全部
```

**5、查看镜像构建历史**
```
docker history [IMAGE] 
```

**6、给镜像命名**
```
docker tag [IMAGE_ID] [REPOSITORY]:[TAG]
```


#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">容器命令</font></center>
**1、新建并启动容器**
```
docker run [OPTIONS] [IMAGE] [COMMAND][ARG..]
```
<font color = "#CD5555">OPTIONS：</font>
- **--name**：为容器指定一个名称。
- **-d**：后台运行容器，并返回容器ID，也即启动守护式容器。
- **-i**：以交互模式运行容器，通常与-t同时使用。
- **-t**：为容器重新分配一个伪输入终端，通常与-i同时使用。
- **-e**：设置环境的变量。
- **-P**：随机端口映射。
- **-p**：指定端口映射，有以下四种格式：
  - ip:hostPort:containerPort 
  - ip:containerPort 
  - hostPort:containerPort 
  - containerPort


<font color = "#CD5555">**启动并进入交互式容器**</font>
```
docker run -it [IMAGE] = docker run -it [IMAGE] /bin/bash
```


<font color = "#CD5555">**启动守护式容器**</font>(以后台进程方式运行)
```
docker run -d [IMAGE]
```
Docker容器运行，需要有一个前台进程。容器运行的命令如果不是那些一直挂起的命令（比如运行top，tail)，是会自动退出的。
可以启动容器并循环打印消息到前台（如下），这样容器就不会自动退出。（自动退出检测的时间间隔是多久？）
```
docker run-d [IMAGE] /bin/sh -c "while true;do echo hello zzyy;sleep 2;done"（可以通过容器日志看到这些输出。）
```
<font color = "#CD5555">**启动时指定端口映射**</font>
```
docker run -it -p 8080(容器端口):8080(宿主机端口) [IMAGE]

#宿主机端口随机分配
docker run -it -P [IMAGE]  
```

**2、列出当前所有正在运行的容器**
```
docker ps [OPTIONS]
```
<font color = "#CD5555">OPTIONS：</font>
- **-a**：列出当前所有正在运行的容器+历史上运行过的
- **-l** ：显示最近创建的容器。
- **-n**：显示最近n个创建的一个容器。
- **-q**：静默模式，只显示容器编号。
- **--no-trunc**：不截断输出。

**3、退出容器**
- **exit**: 容器停止退出。
- **ctrl+P+Q**: 容器不停止退出。


**4、启动容器**
```
docker start [CONTAINER]
```

**5、重启容器**
```
docker restart [CONTAINER]
```

**6、停止容器**
```
docker stop [CONTAINER]
```

**7、强制停止容器**
```
docker kill [CONTAINER]
```

**8、删除已停止的容器**
```
docker rm [CONTAINER_ID]
docker rm -f [CONTAINER_ID](强制删除，可以删除正在运行的容器)
```

**9、一次性删除多个容器**
```
docker rm -f $(docker ps -a -q)
docker ps -a -q | xargs docker rm
```

**10、查看容器日志**
```
docker logs -f -t --tail [CONTAINER_ID]
```
- -t: 是加入时间戳。
- -f: 跟随最新的日志打印。
- --tail: 数字显示最后多少条。

**11、查看容器内运行的进程**
```
docker top [CONTAINER_ID]
```

**12、查看容器内部细节**
```
docker inspect [CONTAINER_ID]
```

**13、进入正在运行的容器并以命令行交互**
```
docker exec -it [CONTAINER_ID] [BASH_SHELL]
```
exec是在容器中打开新的终端，并且可以启动新的进程。（不用进入到容器内进行交互，可以直接执行bashShell）
```
docker attach [CONTAINER_ID] = docker exec-t [CONTAINER_ID] /bin/bash
```
attach直接进入容器启动命令的终端，不会启动新的进程
attach进入容器后，exit会退出并关闭容器，exec进入容器（用/bin/bash这种方式），exit会退出但并不关闭容器。


**13、从容器内拷贝文件到主机上**
```
docker cp [CONTAINER_ID]:[容器内路径] [目的主机路径]
```



#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">导入与导出</font></center>
**1、导出镜像 > 导入镜像**
```
#导出
docker save -o [目标文件] [IMAGE]
docker save > [目标文件] [IMAGE]
#导出
docker load < [目标文件]
docker load -i [目标文件]
```

**2、导出容器 > 导入镜像**
```
#导出
docker export -o [目标文件] [CONTAINER]
#导入
docker import [目标文件] [REPOSITORY]:[TAG]
cat [目标文件] | docker import - [REPOSITORY]:[TAG]
```

**3、这两者的区别**
- save的来源是镜像，export的来源是容器。
- export的文件会小一点。
- load会保存镜像的所有历史记录。import丢失所有元数据和历史记录，仅保存容器当时的状态。











