# Docker

docker 是一个开源的应用容器引擎，基于 Go 语言并遵从 Apache2.0 协议开源。

docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间没有任何接口，更重要的是容器性能开销极低。

docker 从 17.03 版本之后分为 CE（Community Edition: 社区版） 和 EE（Enterprise Edition: 企业版）。

# 开始使用

## 安装

1. 查看当前系统版本

   ```
   cat /etc/redhat-release
   ```

2. 卸载旧版本

   ```
   yum remove docker \
           docker-client \
           docker-client-latest \
           docker-common \
           docker-latest \
           docker-latest-logrotate \
           docker-logrotate \
           docker-selinux \
           docker-engine-selinux \
           docker-engine
   ```

2. 安装语言环境

   ```
   yum -y install gcc
   yum -y install gcc-c++
   ```

3. 首次安装 docker，需要先设置 docker 仓库地址，然后才能从仓库安装和更新 docker

   ```
   # 阿里云
   yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
   ```

   ```
   # 不推荐，官方地址
   yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
   ```

4. 更新 yum 软件包索引

   ```
   yum makecache fast

5. 安装 docker

   ```
   yum -y install docker-ce

## 卸载

1. 删除安装包

   ```
   yum -y remove docker-ce
   ```

2. 删除镜像、容器、配置文件等内容

   ```
   rm -rf /var/lib/docker
   ```

# 使用镜像

同一仓库源可以有多个 `tag`，代表这个仓库源的不同个版本。

1. 查看本地所有镜像

   ```
   docker images [options]
   
   -a：列出本地所有的镜像（含中间映像层）
   -q：只显示镜像ID
   --digests：显示镜像的摘要信息，类似备注
   --no-trunc：显示完整的镜像信息
   
   #####信息表头
   REPOSITORY：镜像仓库源
   TAG：镜像标签
   IMAGE ID：镜像ID
   CREATED：镜像创建时间
   SIZE：镜像大小
   
   仓库源可以有多个Tag，表示不同版本，使用Repository:Tag定义不同镜像，Tag默认为latest
   ```

2. 搜索镜像

   ```
   docker search [OPTIONS] 镜像名称
   ```

3. 从远程仓库拉取镜像（`tag` 默认为 `latest`）

   ```
   docker pull 镜像名称[:TAG]
   ```

4. 删除本地镜像

   ```
   docker rmi [-f] 镜像名称|ID [名称1 名称2 ...]
   
   -f：强制删除
   
   eg：删除本地全部镜像
   docker rmi -f $(docker images -qa)
   ```

# 使用容器

## Docker 客户端

1. 查看版本

   ```
   docker version
   ```

2. 查看详细信息

   ```
   docker info
   ```

3. 查看命令说明

   ```
   docker --help
   ```

## 创建启动容器

1. 普通创建

   ```
   docker run [-d] [--name] 镜像名称|ID <command> [arg...]
   
   --name：为新容器指定名称
   -d：指定容器在后台运行，并返回容器的ID
   command：容器启动后执行的命令
   arg：任意参数
   ```

   容器必须有一个前台进程才能持续运行，否则自动退出，可以执行一些一直挂起的命令来达到此目的。

   docker 尝试在本机获得镜像，有则构造该镜像的容器并运行，没有则从远程仓库下载并运行。

2. 交互式创建，进入容器操作其内部

   ```
   docker run -it 镜像名称|ID
   
   -i：允许与容器的标准输入进行交互
   -t：在新容器内指定一个终端
   
   eg：创建centos镜像的容器，并打开bash与容器进行交互
   docker run -it centos /bin/bash
   ```

3. 为新容器设置环境变量

   ```
   docker run -e "name=value" 镜像名称|ID
   ```

4. 端口映射

   ```
   docker run -p 母机端口:容器端口 镜像名称|ID
   ```

   使用大写 `-P` 还可以将容器内部使用的端口随机映射到母机

   ```
   docker run -P
   ```

   默认绑定 tcp，如果要绑定 udp 端口，可以在端口号后面加上 `/udp`。

   查看指定容器端口的绑定情况：

   ```
   docker port 镜像名称|ID 容器端口

## 管理容器

1. 启动停止的容器

   ```
   docker start 容器名称|ID
   ```

2. 停止正在运行的容器

   ```
   docker stop 容器名称|ID
   ```

3. 杀死容器进程

   ```
   docker kill 容器名称|ID
   ```

4. 重启容器

   ```
   docker restart 容器名称|ID
   ```

5. 删除容器（已经停止）

   ```
   docker rm [-f] 容器名称|ID
   
   -f：强制删除
   
   eg：删除所有容器
   docker rm -f $(docker ps -a -q)
   ```

## 查看容器

1. 查看容器

   ```
   docker ps [OPTIONS]
   
   默认显式当前正在运行的容器
   -a：所有容器
   -l：最近创建的容器
   -n m：最近创建的m个容器
   -q：静默模式，只显示容器编号
   --no-trunc：不截断输出
   ```

2. 查看容器进程

   ```
   docker top 容器名称|ID
   ```

3. 查看容器内部信息，返回记录容器配置和状态的 JSON 数据：

   ```
   docker inspect 容器名称|ID
   ```

4. 查看容器日志

   ```
   docker logs -f -t --tail 容器名称|ID
   
   -t：时间戳
   -f：追踪最新的日志打印
   --tail n：显示最近n条日志
   ```

## 其它操作

1. 拷贝容器内的文件到母机

   ```
   docker cp 容器名称|ID:容器路径 母机路径
   ```

2. 将容器提交为新的镜像

   ```
   docker commit -m="描述信息" -a="作者" 容器名称|ID 镜像名称[:标签名称]
   ```

3. 设置容器重启策略

   ```
   docker update --restart=always 容器名称|ID
   
   --restart：重启策略
   	no 默认，容器停止后不重启
   	on-failure 容器非正常退出（退出状态非0）才会重启
       on-failure:3 容器非正常退出重启最多3次
       unless-stopped 容器退出时总是重启，但不考虑启动时就已经停止的容器
       always 容器退出时总是重启
   ```

4. 在运行的容器内执行命令

   ```
   docker exec [OPTIONS] 容器名称|ID [COMMAND] [ARG...]
   
   eg：进入正在运行的容器，并打开一个bash终端，会在容器内单独启动一个bash进程
   docker exec -it centos1 /bin/bash
   ```

   ```
   # 进入正在运行的容器，进入该容器启动命令的终端，不开启新进程
   docker attach 容器名称|ID
   ```

5. 进入容器后，想要退出回到母机，有两种方式

   ```
   exit
   CTRL+P+Q
   ```

   使用 `attach` 命令进入容器，退出会导致容器停止，而 `exec -it` 命令不会。

# 镜像仓库

将本地镜像推送至（阿里云）镜像仓库

1. 创建阿里云本地镜像仓库

   https://cr.console.aliyun.com/cn-hangzhou/instances

2. 登录阿里云 Docker Registry（首次需要输入密码）

   ```
   docker login --username=poch**** registry.cn-hangzhou.aliyuncs.com
   ```

   登录的用户名为阿里云账号名称，密码为开通服务时设置的密码

   可以在访问凭证页面修改密码


3. 为镜像生成一个新标签

   ```
   docker tag <ImageId> registry.cn-hangzhou.aliyuncs.com/aliyun_pochaiyi/ali_docker:[镜像版本号]
   ```

4. 使用 `docker push` 命令将该镜像推送到 Registry：

   ```
   docker tag <ImageId> registry.cn-hangzhou.aliyuncs.com/aliyun_pochaiyi/ali_docker:[镜像版本号]
   ```

5. 从 Registry 拉取镜像

   ```
   docker pull registry.cn-hangzhou.aliyuncs.com/aliyun_pochaiyi/ali_docker:[镜像版本号]
   ```

# Docker 概念

## 成员

* 镜像 Image

  只读的模板，可用来创建容器，一个镜像可以创建任意个容器。

* 容器 Container

  容器是用镜像创建的运行实例，它可以被创建、启动、停止、删除和修改。容器之间相互隔离，以保证平台的安全性。可以把容器看做简易版 Linux 环境和运行在其内部的应用程序的结合。容器的定义和镜像几乎一样，但容器的最上面一层是支持读写的。

* 仓库 Repository

  仓库是集中存放镜像文件的场所。仓库和仓库注册服务器是不同的概念，仓库注册服务器用于存放多个仓库，仓库中包含多个镜像，每个镜像有不同的标签 `Tag`。

  仓库分为公共仓库（Public）和私有仓库（Private）两种。最大的公共仓库是 DockerHub，存放着大量镜像供用户使用，国内公共仓库有阿里云 、网易云等

## 优点

以前在程序开发中，程序测试后上传至生产机器运行，需要对生产机器配置与开发时相同的环境，如果机器较多，比如分布式系统，那么这将是一项繁杂且难以保证的工作。docker 可以将程序与它的运行环境一同打包，那么这个程序无论在哪台机器哪种系统，都可以轻松地成功运行。

### 虚拟机技术

在操作系统上运行另一个操作系统，比如在 Windows 上运行 Linux。虚拟机中的应用对此毫无知觉，对它而言虚拟机和真实的机器一模一样，而对于母机的底层系统，虚拟机只是一个普通文件，可以随意删掉，没有任何影响。这种技术可以完美运行另一套系统，使应用程序、操作系统、硬件三者之间的逻辑不变。 

缺点：资源占用多、冗余步骤多、启动慢。

### 容器虚拟化技术

Linux 容器不是模拟一个完整的操作系统，而是对进程进行隔离，容器技术将软件和它运行时需要的资源打包到一个隔离的环境。容器与虚拟机不同，它不需要捆绑一整套操作系统，只需要软件工作时需要的库资源和设置。因此，容器比虚拟机更轻量高效，还能保证软件始终如一地运行。

### 容器与虚拟机的不同

虚拟机技术是模拟一整套硬件，它是一个完整的操作系统，然后在该系统运行程序进程。

容器运行的应用进程是寄宿于母机的内核，它并没有自己的内容，也没有进行硬件的模拟。容器之间相互隔离，就像两个进程一样，互不影响，能区分计算资源。

### 容器为什么比虚拟机快

docker 有比虚拟机更少的抽象层，它不需要实现硬件资源的虚拟化，比如 CPU。运行在 docker 上的程序直接使用母机的硬件资源，所以 docker 在硬件资源的使用效率更高。

另外，docker 使用母机的内核，而不需要 Guest OS。因此，创建容器时，docker 不需要像虚拟机一样重新加载操作系统的内核，它跳过引寻、加载操作系统内核等费时费资源的过程。

## 镜像

镜像是一种轻量级、可执行的独立软件包，用来打包环境和软件，它包含运行某个软件需要的所有内容，包括代码、库、环境变量和配置文件等。

### 联合文件系统 UnionFileSystem

UnionFS 是一种分层、轻量级并且高性能的文件系统，它将对文件系统的一次修改作为一次提交，一层一层地叠加，构成一个完整的系统。同时，它可以将不同目录挂载到同一个虚拟文件系统。UioinFS 是 docker 镜像的基石，镜像可以通过分层来进行继承，基于基础镜像（没有父镜像的镜像），可以制作各种应用镜像。

UnionFS 支持联合加载文件系统，同时加载多个文件系统，但从外面看起来只是一个文件系统。联合加载会将这些层叠加起来，这样最终的文件系统仍然包含所有底层的文件和目录。

### 镜像加载原理

docker 镜像是由一层一层的 UnionFS 文件系统组成。

其中，最底层的 UnionFS 文件是 bootfs（boot file system）。它包含 bootloader 和 kernel，前者用于引导加载 kernel。这就类似于 Linux/Unix 系统的 boot 加载器和内核。当 boot 加载完后，整个内核就存在于内存中，此时内存的使用权将由 bootfs 转交给内核，系统也会即时卸载 bootfs。

roofs (root file system) 是在 bootfs 上面的 UnionFS，它包含 Linux 系统中的 /dev，/proc，/bin，/etc 等目录和文件。rootfs 其实就是各种不同操作系统的发行版，如 Ubuntu，CentOS 等。

rootfs 可以很小，只需要实现最基本的命令、工具和程序，因为底层是使用 host 的 kernel，自己只需要提供 rootfs。由此可见，对于不同的 Linux 发行版，bootfs 基本相同，rootfs 会有差别。

### 分层结构的好处

不同镜像可以共享一些底层相同的文件系统，同时再加上自己独有的改动层，大大提高了存储的效率。

### 镜像特点

docker 镜像的所有层都是只读的，当启动容器时，docker 会在容器的顶部加载一个可写层，这层通常被称为容器层，容器层之下都叫镜像层。

# 容器数据卷

## 数据卷概念

**数据卷解决的问题**

docker 将环境与应用打包 ，容器与母机是完全隔离的，有时想持久化应用产生的数据，或在容器间共享数据。这时，如果不将容器提交为新的镜像，那么容器删除后，想要获得的数据也会随之消失。

当然，可以使用 `docker cp` 拷贝容器的数据到母机，但这并不是一个好的解决方法。

为解决此问题，便提出数据卷的概念。

**数据卷的概念**

数据卷其实就是目录或文件，它存在于一个或多个容器中，由 docker 挂载到容器。但是，数据卷不属于 UnionFS，也是因此，它能绕过 UnionFS 提供持续化存储和共享数据的功能。

数据卷是专门用于容器数据的持久化和容器间数据的共享，它独立于任何容器的生命周期，即使删除挂载的容器，它也依然存在。

**数据卷的特点**

1. 数据卷可在容器间共享或重用
2. 数据卷的修改立即生效
3. 数据卷的修改不包含在镜像的更新中
4. 数据卷的生命周期一直持续到没有容器使用它为止，但这不是说它会消失，而是说它由数据卷变为普通的文件或目录

## 添加数据卷

### 方式1：Docker 命令

在 `run` 创建启动容器时，设置容器与母机共享的目录或文件

```
docker run -v /母机目录路径:/容器目录路径[:ro] 镜像名称|ID

:ro 是readOnly 的意思，表示容器只能读数据卷
```

不论是在容器内修改数据卷，或在母机修改数据卷，对方都能立即生效，这说明容器是直接引用这个数据卷，而不是拷贝的方式。

```
eg：创建启动容器centos2并设置共享目录映射
docker run -it --name centos2 -v /opt/docker/centos2:/opt/docker/centos2
```

可以通过 `docker inspcet` 名称查看容器的数据卷挂载情况，内容在 `"Mounts"` 结点下

```
"Mounts": [
    {
        "Type": "bind",
        # 母机目录
        "Source": "/opt/docker/centos2",
        # 容器目录
        "Destination": "/opt/docker/centos2",
        "Mode": "",
        "RW": true,
        "Propagation": "rprivate"
    }
],
```

### 方式2：Docker File

创建 dockerFile 文件，文件名任意，编辑 VOLUME 指令给镜像以设置数据卷

```
# 镜像基于centos分层
FROM centos
# 创建任意数据卷映射
VOLUME ["/opt/docker/container1","/opt/docker/container2"]
CMD echo "finished，DockerFile"
CMD /bin/bash
```

`build` 执行文件指令，创建镜像

```
docker build -f 执行文件 -t 镜像名称 .

-f 执行文件
-t repositoryName
```

```
eg：
docker build -f /opt/docker/dockerFile -t pochaiyi/centos .
```

创建该镜像的容器，然后查看容器的数据卷挂载情况。

## 数据卷容器

当前的容器已经挂载有数据卷，其它的容器可以**挂载这个容器**来实现数据卷的共享，挂载了数据卷的容器，被称为数据卷容器。

假如，容器 `centos1` 已经挂载有数据卷，现在创建 `centos2`

```
docker run -it --name centos2 --volumes-from centos1 Centos:5.8

--volumes-from centos1 表示当前创建的容器会挂载centos1容器挂载的数据卷
```

此时，`centos1`，`centos2` 和母机都共享数据卷，无论三者中的哪个修改了数据卷，都会同步生效其它两者。

如果此时，再创建一个容器 `centos3` 继承 `centos2` 的数据卷，那么就有 4 者共享该数据卷。

其实，可以把这里的数据卷挂载理解为容器对母机文件的引用，容器只有在运行时可以修改数据卷，容器的停止、卸载都不能影响数据卷。

## 挂载数据卷的几种情况

1. `本地目录:容器目录`，直接挂载
2. `本地无:容器目录`，本地创建挂载目录
3. `本地目录:容器无`，容器创建挂载目录
4. `本地无:容器无`，都创建创建目录
5. `本地文件:容器目录`，报错
6. `本地目录:容器文件`，报错

# DockerFile

## 什么是 DockerFile？

dockerFile 文件是 docker 镜像的构建文件，它是由一系列命令和参数组成的脚本。类比 Maven，dockerFile 就是 POM 文件，而镜像就如同构建的 Jar 包，容器如运行时。

## DockerFile 规范

1. dockerFile 的每条保留字指令都必须为大写字母，且后面要跟随至少一个参数
2. 指令从上到下按顺序执行
3. `#` 符号表示注释
4. 每条指令都会创建一个新的镜像层（其实就是 UnionFS 层），并对当前镜像进行提交

## 构建镜像的步骤

创建 dockerFile 文件 -> 执行 `docker build` 命令 -> `docker run` 创建容器

1. docker 从基础镜像运行一个容器，比如 Centos 镜像
2. 每执行一条指令就对容器作出一次修改
3. 执行类似 `docker commit` 的操作，提交一个新的镜像层
4. docker 再基于前面提交的镜像运行一个新容器
5. 重复执行 2，3，4 步，直到执行完所有指令

## 保留字指令

1. `FROM`：表示当前新镜像的基础镜像

2. `MAINTAINER`：镜像作者的姓名和邮箱

3. `RUN`：容器构建时执行的命令

4. `EXPOSE`：容器对外暴露的端口

5. `WORKDIR`：启动容器后终端的默认登陆目录

6. `ENV`：设置环境变量

7. `ADD`：将母机目录文件拷贝进镜像，且 `ADD` 命令会自动处理 `URL` 和解压 `TAR` 包

8. `COPY`：类似 `ADD`，拷贝文件和目录到镜像，但它可以控制拷贝的路径

   将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置

   ```
   # 格式1
   COPY src dest
   
   # 格式2
   COPY ["src", "dest"]
   ```

9. `VOLUME`：设置数据卷

10. `CMD`：容器启动时运行的命令。dockerFile 可以有多条 `CMD` 指令，但只有最后一条生效，而 `docker run` 的参数会替换 `CMD` 指令

    > 这里的 `docker run` 参数，是创建容器时指定容器执行的命令

11. `ENTRYPOINT`：容器启动时运行的命令

    与 `CMD` 不同，`docker run` 的参数会加入最后一条 `ENTRYPOINT` 指令

12. `ONBUILD`：被继承后触发执行指令

# 操作实践

## MySQL

1. 拉取镜像

   ```
   docker pull mysql:8.0
   ```

2. 创建启动容器 mysql

   环境变量 *MYSQL_ROOT_PASSWORD* 指定 root 用户的密码

   ```
   docker run -d -p 3306:3306 --name mysql \
   -v /docker/mysql/conf/my.cnf:/etc/mysql/my.cnf \
   -v /docker/mysql/logs:/logs \
   -v /docker/mysql/data:/var/lib/mysql \
   -e "MYSQL_ROOT_PASSWORD=root" \
   mysql:8.0
   ```

## Redis

Redis 容器通常都没有 `redis.conf`  文件，需要从其它地方获取并做文件挂载。

拉取镜像

```
docker pull redis:6.2.6
```

创建挂载目录

```
mkdir -p /docker/redis/data
```

把 `redis.conf` 文件放在母机的 `/data/docker/redis/conf` 目录。

修改配置属性，`daemonize` 属性必须设置为 `no`，因为容器必须有一个前台进程才能留存。

创建容器

```
docker run -d --name redis -p 6379:6379 \
-v /docker/redis/redis.conf:/docker/redis.conf \
-v /docker/redis/data:/data \
redis:6.2.6 \
redis-server /docker/redis.conf

#####说明
redis-server /etc/redis/redis.conf 表示启动容器后使用指定的配置文件启动Redis
--appendonly yes 开启持久化
如果母机没有创建redis.conf文件，那么docker会将redis.conf创建为空目录
```

启动 Redis 客户端

```
docker exec -it redis redis-cli
```

查看持久化文件

```
ls /data/docker/redis/data
```

## RocketMQ

拉取镜像

```
docker pull rocketmqinc/rocketmq:4.3.2
docker pull styletang/rocketmq-console-ng
```

`broker.conf` 配置

```
#集群名称
brokerClusterName = DefaultCluster
#broker名称
brokerName = broker-a
#0表示Master
brokerId = 0
#清除消息时间，默认凌晨4点
deleteWhen = 04
#消息保留时长，单位小时
fileReservedTime = 48
#同步规则：SYNC_MASTER、ASYNC_MASTER、SLAVE
brokerRole = ASYNC_MASTER
#刷盘策略：ASYNC_FLUSH、SYNC_FLUSH、SYNC_FLUSH
flushDiskType = ASYNC_FLUSH
# 磁盘使用达到95%之后,生产者再写入消息报错
disk full

brokerIP1 = brokerIP
namesrvAddr = nameSrvIP:Port
```

启动 NameServer

```
docker run -d --name namesrv -p 9876:9876 \
-e "MAX_POSSIBLE_HEAP=10000000" \
-e "autoCreateTopicEnable=true" \
rocketmqinc/rocketmq:4.3.2 sh mqnamesrv
```

启动 Broker，这里因为内存极容易启动失败

```
docker run -d --name broker-a --link namesrv:nameIP \
-p 10911:10911 -p 10909:10909 \
-v /docker/rmq/broker.conf:/opt/rocketmq-4.3.2/conf/broker.conf \
-e "NAMESRV_ADDR=nameIP:9876" -e "MAX_POSSIBLE_HEAP=40000000" \
rocketmqinc/rocketmq:4.3.2 sh mqbroker -c /opt/rocketmq-4.3.2/conf/broker.conf
```

启动控制台

```
docker run -d --name mq-console --link namesrv:nameIP \
-e "JAVA_OPTS=-Drocketmq.namesrv.addr=nameIP:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false" \
-p 9001:8080 \
styletang/rocketmq-console-ng
```
