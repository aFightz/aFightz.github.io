---
title: 07 | DockerFile
date: 2019-07-08 20:14:14
tags: [Docker]
categories :
- 学习笔记
- Docker
- Docker核心技术
---

#### <center><font color = "#36648B">✎</font><br/><font color = "#36648B">DokcerFile构建步骤</font></center>
- 编写Dockerfile文件。
- docker build。
- docker run。

#### <center><font color = "#36648B">✎✎</font><br/><font color = "#36648B">DokcerFile基础知识</font></center>
- 每条保留字指令都必须为大写字母且后面要跟随至少一个参数。
- 指令按照从上到下，顺序执行。
- '#'表示注释。
- 每条指令都会创建一个新的镜像层，并对镜像进行提交。

#### <center><font color = "#36648B">✎✎✎</font><br/><font color = "#36648B">DockerFile执行流程</font></center>
- docker从基础镜像运行一个容器。
- 执行一条指令并对容器作出修改。
- 执行类似docker commit的操作提交一个新的镜像层。
- docker再基于刚提交的镜像运行一个新容器。
- 执行dockerfile中的下一条指令直到所有指令都执行完成。

#### <center><font color = "#36648B">✎✎✎✎</font><br/><font color = "#36648B">DokcerFile保留字指令</font></center>
- **FROM**：基础镜像，当前新镜像是基于哪个镜像的。
- **MAINTAINER**: 镜像维护者的姓名和邮箱地址。
- **RUN**: 容器构建时需要运行的命令。
- **EXPOSE**: 当前容器对外暴露出的端口。
- **WORKDIR**: 指定在创建容器后，终端默认登陆的进来工作目录，一个落脚点。
- **ENV**: 用来在构建镜像过程中设置环境变量。
> **问题**：设置的环境变量永久生效吗？
- **ADD**: 将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包。
- **COPY**: 类似ADD，拷贝文件和目录到镜像中。
  - `COPY src dest`
  - `COPY ["src","dest"]`
- **VOLUME**: 容器数据卷，用于数据保存和持久化工作。
- **CMD**: 指定一个容器启动时要运行的命令。Dockerfile中可以有多个CMD指令，但只有最后一个生效。CMD会被`docker run`之后的参数替换。
  - shell格式：`CMD<命令>`
  - exec 格式：`CMD[“可执行文件”，“参数1”，“参数2..]`
  - 参数列表格式：`CMD[“参数1”，“参数2”,...]`。在指定了`ENTRYPOINT`指令后，用CMD指定具体的参数。
- **ENTRYPOINT**: 指定一个容器启动时要运行的命令。ENTRYPOINT的目的和CMD一样，都是在指定容器启动程序及参数，ENTRYPOINT不会被dockerrun之后的参数替换，而是追加这个参数命令。
- **ONBUILD**: 当构建一个被继承的Dockerfile时运行命令，父镜像在被子继承后父镜像的onbuild被触发。
















