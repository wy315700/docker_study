
# 上手指南

Docker允许在容器内运行一个程序，使用一个非常简单的命令: `docker run`.

## 下载镜像

Docker使用 [Docker Hub](https://hub.docker.com/)来管理镜像。

比如我们要下载Ubuntu 14.04可以使用以下命令

    $ sudo docker pull ubuntu:14.04

其中 `ubuntu:14.04` 是镜像的名字，ubuntu指的是类别，而14.04指的是标签，如果没有指定标签，则默认为 `latest`

Docker Hub还有很多好用的镜像，比如 `centos` , `redis` , `nginx` , `mysql` 等预装了服务的镜像。
可以使用 `search` 命令来搜索镜像 

    $ sudo docker search ubuntu

## 管理镜像 

使用 `images` 命令列出系统里已下载的所有镜像

    $ sudo docker images
    REPOSITORY          TAG     IMAGE ID       CREATED       VIRTUAL SIZE
    training/sinatra    latest  5bc342fa0b91   10 hours ago  446.7 MB
    ouruser/sinatra     v2      3c59e02ddd1a   10 hours ago  446.7 MB
    ouruser/sinatra     latest  5db5f8471261   10 hours ago  446.7 MB

使用 `Dockerfile` 可以在已有的镜像的基础上制作一个镜像。比如：

    # This is a comment
    FROM ubuntu:14.04
    MAINTAINER Kate Smith <ksmith@example.com>
    RUN apt-get update && apt-get install -y ruby ruby-dev
    RUN gem install sinatra

将以上内容保存为 `Dockerfile` ，然后运行以下命令制作

    $ sudo docker build -t="ouruser/sinatra:v2" .

注意最后那个.表示在当前目录寻找Dockerfile

可以使用 `docker save` 命令将一个镜像导出

    $ sudo docker save busybox > busybox.tar

然后使用 `docker import` 导入

    $ cat exampleimage.tgz | sudo docker import - exampleimagelocal:new


## 使用容器

我们可以很方便的启动一个容器并且进入终端

    $ sudo docker run -i -t ubuntu:14.04 /bin/bash

-t的意思是启动到一个终端，-i的意思是启动交互式模式。最后的/bin/bash指定了在容器内运行的程序。

使用 `docker ps` 可以列出正在运行中的容器

    $ sudo docker ps
    CONTAINER ID  IMAGE         COMMAND               CREATED        STATUS       PORTS NAMES
    1e5535038e28  ubuntu:14.04  /bin/sh -c 'while tr  2 minutes ago  Up 1 minute        insane_babbage

也可以在容器内运行一个守护程序，比如一个web应用

    $ sudo docker run -d -p 5000:5000 training/webapp python app.py

-d选项让容器在后台运行，-p 5000:5000 则将容器内的5000端口映射到主机上。如果使用-P，则让docker自行决定映射后的外部端口。

可以使用 `docker top` 查看容器内的进程

    $ sudo docker top nostalgic_morse
    PID                 USER                COMMAND
    854                 root                python app.py

最后的nostalgic_morse指的是 `docker ps` 查询到的容器的名称

想要关闭一个运行中的容器也非常的简单，使用 `docker stop` 命令：

    $ sudo docker stop nostalgic_morse
    nostalgic_morse

## 连接容器

当两个容器内的应用需要进行通讯的时候，比如一个web服务容器和一个db容器，可以选择将db服务的端口映射出来，web服务器使用主机上映射出来的端口进行连接。

然而，这个是很不安全的做法，因为这么做数据库地址也会暴露在外，所以docker提供了一个叫容器连接的方法来解决这个问题，将web容器和db容器连接起来，这样以后，web容器可以直接访问db容器里的端口。

首先我们使用 --name选项给容器命名

    $ sudo docker run -d --name db training/postgres

这里启动了一个training/postgres容器并且命名为db。
然后我们就可以将db容器连接到web容器里

    $ sudo docker run -d -P --name web --link db:db training/webapp python app.py

此时，使用 `docker ps` 可以看到，两个db容器已经和web容器连接起来了

    $ sudo docker ps
    CONTAINER ID  IMAGE                     COMMAND               CREATED             STATUS             PORTS                    NAMES
    349169744e49  training/postgres:latest  su postgres -c '/usr  About a minute ago  Up About a minute  5432/tcp                 db, web/db
    aed84ee21bde  training/webapp:latest    python app.py         16 hours ago        Up 2 minutes       0.0.0.0:49154->5000/tcp  web

那么web容器如何去连接db容器呢，如何知道db容器的ip地址呢。docker会自动的在web容器里添加hosts记录

    # cat /etc/hosts
    172.17.0.7  aed84ee21bde
    . . .
    172.17.0.5  db

以及环境变量

    DB_NAME=/web/db
    DB_PORT=tcp://172.17.0.5:5432
    DB_PORT_5432_TCP=tcp://172.17.0.5:5432
    DB_PORT_5432_TCP_PROTO=tcp
    DB_PORT_5432_TCP_PORT=5432
    DB_PORT_5432_TCP_ADDR=172.17.0.5

这里假设db容器内的数据库运行在5432端口。
当然只有第一个程序的端口才会被添加到环境变量里，不过一般情况下一个容器都是拿来运行一个程序的。

当然，docker还支持多主机上的容器相互连接。

## 容器文件管理

我们可以在执行 `docker run` 的时候加上 `-v` 选项，这样就可以在容器内再挂载一个磁盘

    $ sudo docker run -d -P --name web -v /webapp training/webapp python app.py

这条命令会在 `/webapp` 下挂载一个虚拟磁盘 

同样，使用 `-v` 选项也可以将主机的一个目录挂载到容器里

    $ sudo docker run -d -P --name web -v /src/webapp:/opt/webapp training/webapp python app.py

这条命令将主机的 `/src/webapp` 挂载到了容器里的 `/opt/webapp` 里。

也可以将目录挂在成只读的

    $ sudo docker run -d -P --name web -v /src/webapp:/opt/webapp:ro training/webapp python app.py

当然，也可以将一个单独的文件挂载进去

    $ sudo docker run --rm -it -v ~/.bash_history:/.bash_history ubuntu /bin/bash

我们也可以从其他容器里挂载目录到另一个容器，使用 `--volumes-from` 选项

    $ sudo docker run -d -v /dbdata --name dbdata training/postgres echo Data-only container for postgres

    $ sudo docker run -d --volumes-from dbdata --name db1 training/postgres

使用 `docker cp` 命令从容器内往外复制文件 

    Usage: docker cp CONTAINER:PATH HOSTPATH

    Copy files/folders from the PATH to the HOSTPATH


