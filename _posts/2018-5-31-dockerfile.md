---
layout: post
title: dockerfile
description: dockerfile
category: docker
---

## Instruction(指令)

- FROM (指定基础镜像)
- MAINTAINER (指定维护者信息)
- RUN (执行镜像的操作指令)
- CMD (容器启动时执行的指令)    
- EXPOSE (导出端口)
- ENV (设置环境变量)
- ADD (复制本地文件夹到容器)
- COPY (使用本地文件夹到容器)
- ENTRYPOINT (容器启动后执行的命令)
- VOLUME (给容器创建挂载点)
- USER (指定运行容器时的用户名或UID)
- WORKDIR (指定工作目录)
- ONBUILD (作为基础镜像时执行的操作)
- LABEL (为镜像指定标签) 
- STOPSIGNAL (当容器退出时给系统发送什么样的指令)
- HEALTHCHECK (健康检查)

## FROM

DockerFile第一条必须为From指令。如果同一个DockerFile创建多个镜像时，可使用多个From指令（每个镜像一次）
```dockerfile
FROM <image> [AS <name>]
FROM <image>[:<tag>] [AS <name>]
FROM <image>[@<digest>] [AS <name>]
```

## MAINTAINER

```dockerfile
MAINTAINER docker_user  docker_user@mail.com
```

## RUN 

```dockerfile
RUN <linux命令>
RUN ["executable", "Param1", "param2"] 
```
前者在shell终端上运行，即/bin/sh -C，后者使用exec运行。例如：`RUN [“/bin/bash”, “-c”,”echo hello”]`
每条run指令在当前基础镜像执行，并且提交新镜像。当命令比较长时，可以使用“/”换行。

## CMD

```dockerfile
#使用exec执行，推荐
CMD ["executable", "Param1", "param2"] 

#在/bin/sh上执行
CMD command param1 param2

#提供给ENTRYPOINT做默认参数。 
CMD ["Param1", "param2"] 
```
每个容器只能执行一条CMD命令，多个CMD命令时，只最后一条被执行。

## EXPOSE
功能为暴漏容器运行时的监听端口给外部
但是EXPOSE并不会使容器访问主机的端口
如果想使得容器与主机的端口有映射关系，必须在容器启动的时候加上 -P参数
```dockerfile
EXPOSE [...] 
```
例如
```dockerfile
EXPOSE 22 80 8443
```

告诉Docker服务端容器暴露的端口号，供互联系统使用。在启动Docker时，可以通过-P,主机会自动分配一个端口号转发到指定的端口。使用-P，则可以具体指定哪个本地端口映射过来

##  ENV

```dockerfile
ENV <key> <value>
```
指定一个环境变量，会被后续 RUN 指令使用，并在容器运行时保持。

例如
```dockerfile
ENV PG_MAJOR 9.3
ENV PG_VERSION 9.3.4
RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgress && …
ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH
```

## ADD 
 一个复制命令，把文件复制到景象中。

如果把虚拟机与容器想象成两台linux服务器的话，那么这个命令就类似于scp，只是scp需要加用户名和密码的权限验证，而ADD不用。
```dockerfile
ADD <src>... <dest>
ADD ["<src>",... "<dest>"]
```
<dest>路径的填写可以是容器内的绝对路径，也可以是相对于工作目录的相对路径

<src>可以是一个本地文件或者是一个本地压缩文件，还可以是一个url`

该命令将复制指定的文件夹到容器中的。其中文件夹可以是Dockerfile所在目录的一个相对路径；也可以是一个URL；还可以是一个tar文件(自动解压为目录)。

例：
```dockerfile
ADD test relativeDir/ 
ADD test /relativeDir
ADD http://example.com/foobar /
```


## COPY

```dockerfile
COPY <src>... <dest>
COPY ["<src>",... "<dest>"]
```
复制本地主机的文件夹（为Dockerfile所在目录的相对路径）到容器中。
当使用本地目录为源目录时，推荐使用`COPY`。

## ENTRYPOINT

```dockerfile
ENTRYPOINT ["executable", "param1", "param2"]
# 在shell中执行
ENTRYPOINT command param1 param2 
```
配置容器启动后执行的命令，并且不可被`docker run`提供的参数覆盖。    
每个Dockerfile中只能有一个`ENTRYPOINT`，当指定多个时，只有最后一个起效。

## VOLUME

```dockerfile
VOLUME ["/data"] 
```
创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等。
例：
```dockerfile
VOLUME ["/var/log/"]
VOLUME /var/log
VOLUME /var/log /var/db
```
一般的使用场景为需要持久化存储数据时。
容器使用的是`AUFS`，这种文件系统不能持久化数据，当容器关闭后，所有的更改都会丢失。
所以当数据需要持久化时用这个命令。

## USER

```dockerfile
USER daemon
USER UID
```
指定运行容器时的用户名或UID，后续的 RUN 也会使用指定用户。
当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建所需要的用户，例如：`RUN groupadd -r postgres && useradd -r -g postgres postgres`。要临时获取管理员权限可以使用`gosu`，而不推荐`sudo`。

## WORKDIR

```dockerfile
WORKDIR /path/to/workdir
```
为后续的`RUN`、`CMD`、`ENTRYPOINT`指令配置工作目录。
可以使用多个`WORKDIR`指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径。
例如：
```dockerfile
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
#则最终路径为 /a/b/c 
```

## ONBUILD

```dockerfile
ONBUILD [INSTRUCTION]
```

配置当所创建的镜像作为其它新创建镜像的基础镜像时，所执行的操作指令。
例如，Dockerfile使用如下的内容创建了镜像 image-A。

```dockerfile
[...]
ONBUILD ADD . /app/src
ONBUILD RUN /usr/local/bin/python-build –dir /app/src
[...] 
```
如果基于A创建新的镜像时，新的Dockerfile中使用`FROM image-A`指定基础镜像时，会自动执行`ONBUILD`指令内容，等价于在后面添加了两条指令。

```dockerfile
FROM image-A

#Automatically run the following
ADD . /app/src
RUN /usr/local/bin/python-build --dir /app/src
#使用 ONBUILD 指令的镜像，推荐在标签中注明，例如 ruby:1.9-onbuild。
```

## LABEL

为镜像指定标签
```dockerfile
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```

一个Dockerfile种可以有多个LABEL，如下：
```dockerfile
LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
``` 
说明：LABEL会继承基础镜像种的LABEL，如遇到key相同，则值覆盖

## STOPSIGNAL
当容器退出时给系统发送什么样的指令
```dockerfile
STOPSIGNAL signal
```

## HEALTHCHECK

容器健康状况检查命令

语法有两种：
```dockerfile
HEALTHCHECK [OPTIONS] CMD command
HEALTHCHECK NONE
```


第一个的功能是在容器内部运行一个命令来检查容器的健康状况

第二个的功能是在基础镜像中取消健康检查命令

[OPTIONS]的选项支持以下三中选项：

    --interval=DURATION 两次检查默认的时间间隔为30秒

    --timeout=DURATION 健康检查命令运行超时时长，默认30秒

    --retries=N 当连续失败指定次数后，则容器被认为是不健康的，状态为unhealthy，默认次数是3

注意：

HEALTHCHECK命令只能出现一次，如果出现了多次，只有最后一个生效。

CMD后边的命令的返回值决定了本次健康检查是否成功，具体的返回值如下：

0: success - 表示容器是健康的

1: unhealthy - 表示容器已经不能工作了

2: reserved - 保留值

 

例子：
```dockerfile
HEALTHCHECK --interval=5m --timeout=3s \
CMD curl -f http://localhost/ || exit 1
```

健康检查命令是：curl -f http://localhost/ || exit 1

两次检查的间隔时间是5秒

命令超时时间为3秒

## create image
通过Docker Build 创建镜像。
命令读取指定路径下（包括子目录）所有的Dockefile，并且把目录下所有内容发送到服务端，由服务端创建镜像。另外可以通过创建.dockerignore文件（每一行添加一个匹配模式）让docker忽略指定目录或者文件。

格式为`Docker Build [选项] 路径`
需要制定标签信息，可以使用-t选项
例如：Dockerfile路径为`/tmp/docker_build/`，生成镜像的标签为`build_repo/my_images`
```shell
sudo docker build -t build_repo/my_images /tmp/docker_build/
```

## 配置实例
```dockerfile
FROM frolvlad/alpine-oraclejdk8:slim
VOLUME /tmp
ADD eureka-server-0.0.1-SNAPSHOT.jar app.jar
#RUN bash -c 'touch /app.jar'
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
EXPOSE 8761
```
