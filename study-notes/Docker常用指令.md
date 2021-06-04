#### Docker是什么？

​	docker是管理容器的引擎

#### 镜像的基本概念：

> Docker镜像是一个只读的模板，可以用来创建Docker容器。例如：一个镜像可以包含一个完整的centos操作系统环境，里面仅安装了mysql或用户需要的其他应用程序。Docker提供了一个非常简单的机制来创建镜像或更新现有的镜像，用户甚至可以直接从其他人那里下载一个已经做好的镜像来直接使用。

#### 容器的基本概念

> 容器是从镜像创建的运行实例。它可以被启动，停止，删除。每个容器都是相互隔离的，保证安全平台。

#### Docker常用指令

> Centos常用指令
>
> 下载镜像：docker pull tomcat
>
> 运行镜像：docker run tomcat 前台运行，要后台运行，加参数：-d（docker run -d tomcat）
>
> 显示本地 已有的镜像：docker images
>
> 搜索镜像：docker search tomcat
>
> 查看容器中运行的服务：docker ps
>
> 停止容器中运行的服务：docker stop 容器ID/容器名称
>
> 进入Docker容器：docker exec -it 容器ID bash
>
> 删除镜像：docker rmi 镜像ID
>
> 查看所有容器：docker ps -a
>
> 查看容器更多的信息：docker inspect + 容器ID或容器名称
>
> 停止全部运行中的容器：docker stop ${docker ps -q}
>
> 删除全部容器：docker rm ${docker ps -aq}
>
> 一条命令实现停用并删除容器：docker stop ${docker ps -q} & docker rm ${docker ps -aq}

#### 客户机访问容器

​	从客户机上访问容器，需要有端口映射，docker容器默认采用桥接模式与宿主机通信，需要宿主机的ip端口映射到容器的ip端口上；

Docker的网络访问机制：需要将docker容器中的tomcat端口映射到linux上

> docker run -d -p 8080:8080 tomcat

#### Docker使用示例

![](F:\zookeeper\docker-1.png)

![](F:\zookeeper\docker-2.png.png)

![](F:\zookeeper\docker-3.png)

将linux的文件拷贝到docker容器的目录下

docker cp /root/test.html  **容器ID**:/usr/share/nginx/html

![](F:\zookeeper\docker-4.png)

![](F:\zookeeper\docker-5.png)

#### Docker 自定义镜像

![](F:\zookeeper\docker-6.png)

![](F:\zookeeper\docker-7.png)

#### Dockerfile指令

- **FROM**

  > 格式为  FROM <image>  或 FROM <image>:<tag>
  >
  > Dockerfile文件的第一条指令必须为FROM指令。并且如果在同一个Dockerfile中创建多个镜像时，可以使用多个FROM指令(每个镜像一次)

- **MAINTAINER**

  > 格式为 MAINTAINER <name>，指定维护者信息；

- **ENV**

  > 格式为 ENV <key> <value> , 指定一个环境变量，会被后续RUN指令使用，并在容器运行是保持；

- **ADD**

  > 格式为 ADD <src> <dest>;
  >
  > 复制指定的<src>到容器中的<dest>;

- **EXPOSE**

  > 格式为 EXPOSE <port> [<port>...]
  >
  > 告诉Docker服务端容器暴露的端口号，供互联系统使用，在启动容器时需要通过“-p”映射端口，Docker主机会自动分配一个端口转发到指定的端口；

- **RUN**

  > 格式为 RUN <command>
  >
  > RUN指令将在当前镜像基础上执行指定命令，并提交为新的镜像，当命令较长时，可以使用“\”来执行；

- **CMD**

  > 指定启动容器时执行的命令，每一个Dockerfile只能有一条CMD命令。如果指令了多条命令，只有最后一条会被执行。如果用户启动容器时指定了运行的命令，则会覆盖掉CMD指定的命令。

#### 自定义JDK镜像

> FROM centos:latest
>
> MAINTAINER zephyr
>
> ADD jdk-8u121-linux-x64.tar.gz /usr/local
>
> ENV JAVA_HOME /usr/local/jdk1.8.0_121
>
> ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
>
> ENV PATH $PATH:$JAVA_HOME/bin
>
> CMD java -version

构建镜像：docker build -t zephyr_jdk1.8.0_121 .

运行镜像：docker run -d  镜像ID

#### 自定义Tomcat镜像

![](F:\zookeeper\docker-9.png)

#### 自定义MYSQL镜像

![](F:\zookeeper\docker-10.png)

#### 自定义Redis镜像

![](F:\zookeeper\docker-11.png)

#### 镜像发布到阿里云仓库

![](F:\zookeeper\docker-12.png)

![](F:\zookeeper\docker-13.png)