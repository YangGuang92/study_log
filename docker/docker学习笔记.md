# docker学习笔记

### 1. docker概述

##### 1.1 docker为什么出现

在以前经常会遇到这些问题

- 在我的电脑上能够运行，为什么在你的电脑上不行了！
- 在多机器上部署环境，费时费力

docker就是为了解决这些问题

docker的思想就来自于集装箱，**docker的核心思想就是隔离**，打包装箱，每一个箱子都是隔离的，docker通过隔离机制，可以将服务器利用到极致

##### 1.2 docker的历史

2010年，几个搞IT的年轻人，成立了一家`dotCloud`的公司，做一些PaaS的云计算服务，LXC相关的容器技术,他们将自己的技术（容器化技术）命名为docker。

docker刚诞生的时候，没有引起行业的注意，为了让公司活下去，2013年，docker开源，后来越来越多的人发现docker的优点，刚开始的几个月几乎每个月都会更新一个版本

2014年9月， docker1.0发布！

docker为什么火起来，**因为docker十分轻巧**，运行一个虚拟机可能需要几个G或者几十G空间，而且启动时间会有几分钟，而docker容器的启动往往只需要几十兆或者几百兆，几秒钟就能启动

官网地址： https://www.docker.com/ 

官方文档地址： https://docs.docker.com/ 文档内容超级详细！

仓库地址： https://hub.docker.com/ 

##### 1.3 docker能干什么

虚拟机技术的缺点：

- 资源占用十分多
- 冗余步骤多
- 启动很慢

容器化技术：**容器化技术不是模拟一个完整的操作系统**

比较docker和虚拟机技术的不同：

- 传统虚拟机，虚拟出一套硬件，运行一个完整的操作系统，然后在这个系统上安装和运行软件，容器内的应用是直接运行在宿主机的内核中的，容器没有自己的内核，也没有虚拟硬件，所以就很轻便
- 每个容器内是互相隔离的，每个容器内都有自己的文件系统，互不影响

docker的优势：

- **应用更快速的交付和部署**
- **更便捷的升级和扩缩容**
- **更简单的系统运维**
- **更高效的计算资源利用**

##### 1.4 docker的名词概念

- 镜像（image）: 镜像就好比一个模板，通过这个模板来创建容器服务，镜像是不能运行的，类似于一个类
- 容器（containers）:  Docker 的容器可以简单理解为提供了系统硬件环境，它是真正跑项目程序、消耗机器资源、提供服务的东西。 
- 仓库（repository）： Docker 的仓库用于存放镜像。这一点，和 Git 非常类似。 

 ![img](https://user-gold-cdn.xitu.io/2019/4/9/16a02cdab4a554d0?imageslim) 

##### 1.5 docker相关命令

```shell
#查看docker 版本
docker version

#查看docker 镜像
docker images
```



### 2.docker安装

##### 2.1 系统检查

```shell
# 系统内核是3.10以上的
[root@izbp1a01zic0bgbib738aaz etc]# uname -r
3.10.0-693.2.2.el7.x86_64
#查看系统版本 必须是centos7以上
[root@izbp1a01zic0bgbib738aaz etc]# cat /etc/os-release 
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"
```

##### 2.2 安装

```shell
#1.卸载旧版本
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
#2.需要的安装包
sudo yum install -y yum-utils

#3.设置镜像仓库 这里使用阿里云的镜像
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

#更新yum软件包索引
yum makecache fast

#4.安装docker  docker-ce：社区版 docker-ee:企业版
sudo yum install docker-ce docker-ce-cli containerd.io

#5. 启动docker
sudo systemctl start docker

#6. 查看docker版本
docker version

#7. 启动hello-world
docker run hello-world
```

了解：卸载docker

```shell
#1. 卸载依赖
sudo yum remove docker-ce docker-ce-cli containerd.io
#2. 删除资源
sudo rm -rf /var/lib/docker
```

##### 2.3 阿里云镜像加速

登录阿里云-->选择左上角`产品与服务`-->选择`弹性计算`下的`容器镜像服务`-->选择`镜像加速器`,然后根据自己的系统选择不同的镜像加速命令

### 3. docker流程、原理和常用命令

##### 3.1 docker run流程

docker会在本机寻找镜像，找到的话直接运行镜像，如果没有就去DockerHub上面下载，如果DockerHub也没有就返回错误，下载镜像并执行

##### 3.2 底层原理

docker是一个Client-Server结构的系统，Docker的守护进程运行在主机上，通过Socket从客户端访问，docker server 接收到 client的指令，就可以执行这个命令

docker比VM快的原因：

- docker有着比虚拟机更少的抽象层
- docker利用了宿主机的内核，vm需要Guest OS

##### 3.3 docker的常用命令(重要！)

###### 1. 帮助命令

```shell
docker version   # 显示docker的完整信息
docker info      # 显示docker的系统信息，包括镜像和容器的数量
docker help      # 帮助命令
docker stats     # 查看docker的cpu、内存等消耗
docker volume --help    # 查看数据卷相关命令
```

帮助文档地址： https://docs.docker.com/reference/ 

###### 2. 镜像命令

**docker images** ：查看所有本地的镜像

```shell
[root@izbp1a01zic0bgbib738aaz ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              18.04               ccc6e87d482b        5 months ago        64.2MB
python              latest              1f88553e8143        5 months ago        933MB
hello-world         latest              bf756fb1ae65        5 months ago        13.3kB
training/webapp     latest              6fae60ef3446        5 years ago         349MB

#常用的可选项
-a, -all  # 列出所有的镜像
-q, -quiet # 只显示镜像的id
```

|    名称    |      解释      |
| :--------: | :------------: |
| REPOSITORY |  镜像的仓库源  |
|    TAG     |   镜像的标签   |
|  IMAGE ID  |     镜像id     |
|  CREATED   | 镜像的创建时间 |
|    SIZE    |   镜像的大小   |

**docker search**： 搜索镜像

```shell
[root@izbp1a01zic0bgbib738aaz ~]# docker search mysql
NAME                              DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   9626                [OK]                
mariadb                           MariaDB is a community-developed fork of MyS…   3496                [OK]                
mysql/mysql-server                Optimized MySQL Server Docker images. Create…   702                                     [OK]

#常用的可选项
--filter=STARS=3000   # 搜索STARS大于3000的镜像
```

**docker pull**：下载镜像（下载镜像一般都需要加版本号，不版本号就默认下最新版本）

```shell
#下载mysql5.7版本(下载的版本必须是dockerhub有的版本)
[root@izbp1a01zic0bgbib738aaz ~]# docker pull mysql:5.7
5.7: Pulling from library/mysql
8559a31e96f4: Pull complete   #分层下载，docker镜像的核心
d51ce1c2e575: Pull complete 
c2344adc4858: Pull complete 
fcf3ceff18fc: Pull complete 
16da0c38dc5b: Pull complete 
b905d1797e97: Pull complete 
4b50d1c6b05c: Pull complete 
d85174a87144: Pull complete 
a4ad33703fa8: Pull complete 
f7a5433ce20d: Pull complete 
3dcd2a278b4a: Pull complete 
Digest: sha256:32f9d9a069f7a735e28fd44ea944d53c61f990ba71460c5c183e610854ca4854 # 签名
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7 #下载的真实地址
```

**docker rmi**：删除镜像

```shell
docker rmi -f 容器id     #根据容器id删除指定镜像
docker rmi -f 容器id 容器id 容器id 容器id   # 删除多个镜像
docker rmi -f $(docker images -aq)   # 删除所有镜像
```

###### 3. 容器命令

我们有了镜像才能创建容器，我们来下载一个cento镜像来测试学习

```shell
#下载一个centos镜像
docker pull centos
```

**新建容器并运行**

```shell
docker run [可选参数] image

# 可选参数说明
--name="name"   容器名称 mysql1 mysql2 用来区分容器
--rm            用完就删除 一般用来测试
--restart=always docker重启总是运行
--volumes-from 容器id或者容器名称   多容器数据卷共享 
--link 容器名称  在容器中可以直接ping通 这个link的容器（被绑的容器不能ping通这个容器）
--net            使用的网络，默认是bridge
-d              后台运行
-it 			以交互方式运行，进入容器查看内容 docker run -it centos /bin/bash
-p              指定容器的端口 -p 8000:8000
	-p ip:主机端口:容器端口
	-p 主机端口:容器端口（常用）
	-p 容器端口
-P               随机指定端口
-v               挂载目录,冒号前为宿主机目录，必须为绝对路径，冒号后为镜像内挂载的路径。

#测试进入容器
[root@izbp1a01zic0bgbib738aaz ~]# docker run -it centos /bin/bash
[root@dea39ffa6777 /]# ls          #查看容器内centos 基础版本 很多命令都不完善
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@dea39ffa6777 /]# exit        #退出容器
exit
```

**列出容器列表**

```shell
docker ps [可选参数]  #显示正在运行的容器

#可选参数说明
-a        # 列出所有容器
-n=?      # 显示最近创建的几个容器容器 -n=1
-q        # 只显示容器id
```

**删除容器**

```shell
docker rm 容器id           #删除指定容器，不能删除正在运行的容器
docker rm  -f $(docker ps -aq) #删除所有容器
```

**启动和停止容器的操作**

```shell
docker start 容器id        #启动容器
docker restart 容器id	     #重启容器
docker stop 容器id         #停止容器
docker kill 容器id         #强制停止容器
```

###### 4. 常用的其他命令

**后台启动容器**

```shell
docker run -d -it 镜像名称 /bash/bash
#！！！！！注意！！！！
#如果只是 docker run -d 镜像名称 /bash/sh 会发现容器运行后又自动停止了，这是因为docker容器使用后台运行，就必须要有一个前台进程，如果没有前台进程执行，容器认为空闲，就会自行退出
```

**查看日志**

```shell
docker logs [可选参数] 容器id

#可选参数说明
-tf    #显示日志
--tail number   #要显示日志条数

docker logs -tf --tail 10 26636    #显示这个容器的最近10条日志
```

**查看容器中的进程信息**

```shell
docker top 容器id
```

**查看容器元数据**

```shell
docker inspect 容器id   #会展示该容器的各种信息
```

**进入当前正在运行的容器**

```shell
docker exec -it 容器id /bin/bash

docker attach 容器id 

#两个命令的区别
#docker exec 进入容器后开启一个新的终端，可以在里面操作（常用）
#docker attach 进入容器正在运行的终端，不会启动新的进程
```

**从容器内拷贝文件到主机上**

```shell
docker cp 容器id:容器内路径 主机的路径
#例子
docker cp 26636:/home/test.php /home
```

###### 5. 练习

- 安装nginx

  ```shell
  #搜索nginx 最好去官网看下详细极少
  docker search nginx
  #下载镜像
  docker pull nginx:版本号
  #运行镜像 指定3344端口关联容器的80端口
  docker run -d --name nginx01 -p 3344:80 nginx:版本号
  #测试
  curl localhost:3344
  ```

- 部署es+kibana

  ```shell
  #问题：es消耗内存很多 ，暴露的端口多，需要挂在数据卷
  
  #启动了 单核2G的机器就会卡，因为es很耗内存，所以要增加内存的限制 -e ES_JAVA_OPTS="-Xms64m -Xmx512m" 限制内存从64m-512m
  docker run -d --name elasticsearch01  -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx512m" elasticsearch:7.6.2
  ```

###### 6. 可视化

- portainer(先用这个),平时不会用，可以自己玩一玩

  ```shell
  docker run -d -p 8088:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
  ```

- Rancher(CI/CD再用)

### 4. docker镜像理解

##### 4.1 镜像是什么

 **镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件所需的所有内容，包括代码、运行时、库、环境变量和配置文件** 

##### 4.2 docker镜像加载原理

1.  **UnionFS(联合文件系统)** 

   UnionFS（联合文件系统）：Union文件系统是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下，**Union文件系统是Dokcer镜像的基础**。**镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的镜像。**

   特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统加载起来，这样最终的文件系统会包含所有的底层文件和目录

2.  **Docker镜像加载原理** 

   docker的镜像实际上是由一层一层的文件系统构成，这种层级的文件系统UnionFS。

    **bootfs（boot file system）:**主要包含bootloader和kernel，bootloader主要是引导加载kernel，Linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层是bootfs。这一层与我们典型的linux/unix系统是一样的，包含boot加载器内核。当boot加载完之后整个内核就都在内存中了，此时内存的使用权已经由bootfs交给内核了，此时系统也会卸载bootfs

   **rootfs(root file system):** 在bootfs之上，包含的就是点性的Linux系统中的/dev,/proc,/bin,/etc等标准目录和文件，rootfs就是各种不同的操作系统发行版，比如Ubuntu,Centos等

   **平时我们安装进虚拟机的CentOS都是好几个G，为什么docker这里才200M**

    对以一个精简的OS，rootfs可以很小，只需要包括最基本的命令、工具和程序库就行，因为底层直接用host和kernel，自己只需要提供rootfs就行。由此可见对于不同的Linux发行版，bootfs基本是一致的，rootfs会有差别，因此不同的发行版可以公用bootfs。 

##### 4.3 分层的镜像

我们在下载过程，可以看到镜像是一层一层下载的，有时会出现，前几层已经存在的情况，这是因为某些镜像的底层肯能是共用的，其最大的好处就是**共享资源**

**特点**

docker镜像都是只读的

当容器启动时，一个新的可写层被加载到镜像的顶部。

这一层通常被称作“容器层”，“容器层”之下都叫“镜像层”

##### 4.4 commit 镜像

```shell
docker commit  提交容器成为一个新的镜像

#命令和git类似
docker commit -m="提交的描述信息" -a="作者" 容器id 目标镜像名:[tag]
```

其目的是为了对原有镜像做一些个性化的修改，然后提交，下次就可以直接run这个镜像

### 5. 容器数据卷

##### 5.1 为什么需要容器数据卷

 当我们启动容器之后，所有的数据就会存在容器内部，如果 容器删了，数据就会丢失，比如启动一个mysql容器，我们肯定希望数据能够存储在本地，这就需要有一个数据共享技术，容器中产生的数据，同步到本地

**这就是卷技术，实际就是目录的挂在，将容器内的目录挂载到宿主机的目录下**

在容器或者宿主机的绑定目录下的修改，都会互相同步，即便容器stop之后，对宿主机的目录下修改，也会对容器内进行同步

##### 5.2 使用数据卷

1. 方式一：直接使用命令挂载 `-v`

   ```shell
   # -v 宿主目录:容器目录
   #查看挂在信息可以使用 docker inspect 容器id 命令， 在显示的信息中有一个Mounts数组，显示的就是挂载目录信息
   ```

2. 方式二：使用dockerfile进行挂载，详情请看 6.1

##### 5.3 实战：安装MySQL

```shell
#下载容器
docker pull mysql:5.7
#运行容器，注意需要挂在数据卷，同时还要设置mysql密码
#绑定本地3307接口
#挂在配置文件目录，挂载数据目录
#设置root密码
docker run -d -p 3307:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7
```

##### 5.4 匿名挂载和具名挂载

```shell
#匿名挂载：就是 -v 后直接跟容器内的路径，如下面
docker run -d -P --name nginx01 -v /etc/nginx nginx

#具名挂载 就是 -v 后面跟具体的名称(没有/):容器内的路径,如下
docker run -d -P --name nginx01 -v nginx-volume:/etc/nginx nginx

#查看数据卷命令  一长串字符串的是匿名挂载，有具体名称的是具名挂载
[root@izbp1a01zic0bgbib738aaz conf]# docker volume ls
DRIVER              VOLUME NAME
local               a2417529cc46ce4a544579b80f8f927fe423c424ed921571d8194b6d73388842 #匿名挂载
local               c6ad31ddbd819aeabf77ede255ef7fd798a74ebbcff89b02b81c9fdfc6fb858e
local               c9b4eba66e5290b1c414acf70d32e569d34fc9910b96f20ac25f3561a69b0ef4
local               cca8f93e715f756990abffeb2e30e3d2c2c2a302ef6addf3fa20dc1ba48972b2
local               my-vol   #具名挂载
local               my-volume
```

**所有的docker容器内的卷，如果没有指定具体的目录，都在`/var/lib/docker/volumes/xxxx/_data`下，我们大多数情况下都是要使用具名挂载**

拓展：

```shell
#有时 设置挂载目录的时候会有:ro或者:rw命令,如
docker run -d -P --name nginx01 -v nginx-volume:/etc/nginx:ro nginx
docker run -d -P --name nginx01 -v nginx-volume:/etc/nginx:rw nginx

ro read only #只读权限
rw read write #读写权限
#ro 表示这个路径只能在宿主机上进行操作，不能容器内进行操作
```

##### 5.5 多容器数据卷共享

**多个容器内时间数据共享**

```shell
 #运行容器的时候 加上参数 --volumes-from 容器id或者容器名称 如：
 docker run -d -it --name docker02 --volumes-from e7cc71bdefc5 db8b2a32a84e

#即使删掉父数据卷所在的容器，其他容器的数据卷也不会受到影响
```

练习：多个mysql实现数据共享

```shell
docker run -d -p 3307:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7

docker run -d -p 3308:3306 --volumes-from mysql01 -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7
```

**结论**：

容器之间配置信息的传递，数据卷容器的生命周期一直持续到没有容器使用为止

但是一旦你持久化到本地，这个时候，本地的数据也是不会删除的

### 6. Dockerfile

Dockerfile就是用来构建docker镜像的构建文件，是命令脚本

**构建步骤**：

1. 编写dockerfile文件
2. docker build 构建成一个镜像 命令`docker build -f dockerfile文件路径 -t 镜像名称:版本号  .`
3. docker run 运行镜像
4. docker push 发布镜像(dockerhub、阿里云镜像仓库)

**基础知识：**

1. 每个保留关键字(指令)都必须是大写字母
2. 执行是从上到下顺序执行的
3. #是注释
4. 每个指令都会创建提交一个新的镜像层，并提交

##### 6.1 初识Dockerfile

先体验一下！通过这个脚本可以生成镜像，镜像是一层一层的，脚本也是一个一个的命令，每个命令都是一层

```shell
#创建一个dockerfile文件，名字随意，建议用dockerfile
#文件中的内容，指令都是大写的
FROM centos

VOLUME ["volume01","volume02"]

CMD echo "-----end-----"

CMD /bin/bash

#然后构建一下镜像
docker build -f /home/docker-file-test/docker_file_1 -t yangguang/centos:1.0 .
Sending build context to Docker daemon  2.048kB
Step 1/4 : FROM centos
 ---> 470671670cac
Step 2/4 : VOLUME ["volume01","volume02"]
 ---> Running in decefa801fd1
Removing intermediate container decefa801fd1
 ---> 19ebcd12340d
Step 3/4 : CMD echo "-----end-----"
 ---> Running in ae78f2cd44b8
Removing intermediate container ae78f2cd44b8
 ---> 12bff3e2d520
Step 4/4 : CMD /bin/bash
 ---> Running in 43ae9dcd9333
Removing intermediate container 43ae9dcd9333
 ---> db8b2a32a84e
Successfully built db8b2a32a84e
Successfully tagged yangguang/centos:1.0
#启动自己写的容器
docker run -d -it db8b2a32a84e /bin/bash
#进入容器，就可以看到两个目录volume01 volumeo2,这两个也是挂载目录，可以通过 docker inspect 容器id 来查看挂载的路径
```

这种方式未来会使用的十分多，因为我们通常会构建自己的镜像

如果dockerfile中没有挂载卷，就需要再运行容器的时候通过`-v`来挂载卷

##### 6.2 dockerfile指令说明

```shell
FROM				#基础镜像，一切从这里开始构建
MAINTAINER			#镜像是谁写的，姓名+邮箱
RUN					#镜像构建的时候需要的命令
ADD 				#步骤，比如需要加tomcat镜像，ADD里面就需要加tomcat压缩包
WORKDIR				#镜像的工作目录
VOLUME				#挂载的目录
EXPOSE				#暴露的端口配置
CMD					#指定这个容器启动的时候运行的命令,只有最后一个会生效，可被替代
ENTRYPOINT			#指定这个容器启动的时候运行的命令,可以追加命令
ONBUILD				#当构建一个被继承 dockerfile ，这个时候就会运行ONBUILD指令，是被动指令
COPY				#类似ADD,将我们的文件拷贝到镜像中
ENV					#构建的时候设置环境变量
```

**CMD和ENTRYPOINT的区别**

- cmd:指定这个容器启动的时候运行的命令,只有最后一个会生效，可被替代,如下：

  ```shell
  #编写一个dockerfile文件
  FROM centos
  CMD ["ls","-a"]
  #构建镜像
  ...
  #run运行,这时候可以看到显示的根目录下的所有文件和文件加
  docker run db8b2a32a84e
  #但是如果再run命令之后增加一个 -l，就会报错,如果增加的是 ls -al 就会执行，这是因为CMD命令不能追加命令，-l会以一个单独的命令去执行，但是-l并不是一个正常的命令，所以报错
  ```

- entrypoint:定这个容器启动的时候运行的命令,可以追加命令

  ```shell
  #编写一个dockerfile文件
  FROM centos
  ENTRYPOINT ["ls","-a"]
  #构建镜像
  ...
  #run运行,这时候可以看到显示的根目录下的所有文件和文件加
  docker run db8b2a32a84e
  #但是如果再run命令之后增加一个 -l 这时就不会报错，因为ENTRYPOINT可以追加命令，那么真实的命令就是 ls -a -l 所以就能正常显示
  ```

dockerfile中很多命令都十分的相似，我们需要了解区别，就需要自己去测试效果

##### 6.3 实战测试

DockerHub中99%的镜像都是从这个`FROM scratch`基础镜像过来的，然后配置需要的软件和配置来进行构建

**构建自己的centos**:

```shell
# 编写dockerfile文件，文件内容如下
FROM centos
MAINTAINER yangguang<1122334455@qq.com>
#设置环境变量
ENV MYPATH /usr/local
# 进入容器后 直接进入工作目录
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo "-----end-----"
CMD /bin/bash
#构建镜像
docker build -f dockerfile文件路径 -t 镜像名称:版本号  .
#运行镜像，生成容器
```

镜像制作成功后，可以通过 `docker history 镜像id`来查看构建构成

 **构建tomcat镜像**

1. 准备镜像文件，tomcat压缩包，jdk压缩包

2. 编写dockerfile文件，官方命名`Dockerfile`,build会自动寻找这个文件，就可以不用`-f`

   ```shell
   FROM centos
   MAINTAINER yangguang<1122334455@qq.com>
   
   COPY readme.txt /usr/local/readme.txt
   
   ADD jdk-Bull-linux-x64.tar.gz /usr/local/
   ADD apache-tomcat-9.0.tar.gz /usr/local/
   
   #设置环境变量
   ENV MYPATH /usr/local
   # 进入容器后 直接进入工作目录
   WORKDIR $MYPATH
   
   RUN yum -y install vim
   RUN yum -y install net-tools
   
   ENV JAVA_HOME /usr/local/jdk1.8.0_11
   ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
   ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.22
   ENV CATALINA_BASH /usr/local/apache-tomcat-9.0.22
   ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALIN_HOME/bin
   
   EXPOSE 8080
   
   CMD /usr/local/apache-tomcat-9.0.22/bin/starup.sh && tail -F /usr/local/apache-tomcat-9.0.22/bin/logs/catalina.out
   ```

3. build镜像

   ```shell
   docker build -t diytomcat:0.1 .
   ```

4. 运行镜像

   ```shell
   docker run -d -p 9090:8080 --name ygtomcat -v /home/tomcat/test:/usr/local/apache-tomcat-9.0.22/webapps/test -v /home/tomcat/tomcatlogs/:/usr/local/apache-tomcat-9.0.22/logs diytomcat:0.1
   ```

##### 6.4 发布镜像

- 发布到dockerhub

  ```shell
  #登录dockerhub
  docker login -u xxxx
  
  #push到dockerhub 如果没有tag 会发布不上去，所以如果再制作镜像的时候没有加tag的，要有个docker tag命令 加tag
  docker push 镜像名称:tag
  ```

- 发布到阿里云镜像服务上

  ```shell
  # 登录阿里云
  
  #找到容器镜像服务，并创建命名空间(一个账号可以创建3个命名空间)，创建仓库
  
  #按照操作指南推送到镜像仓库
  ```

### 7. docker网络

##### 7.1 理解docker0

```shell
#查看ip地址
[root@izbp1a01zic0bgbib738aaz home]# ip addr
#本机地址
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
#阿里云内网地址
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:16:3f:00:d6:f9 brd ff:ff:ff:ff:ff:ff
    inet 172.16.250.32/20 brd 172.16.255.255 scope global dynamic eth0
       valid_lft 289230763sec preferred_lft 289230763sec
#docker0地址
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN 
    link/ether 02:42:62:ff:24:92 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
22: br-88cbc37136fc: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN 
    link/ether 02:42:dd:0b:2a:09 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global br-88cbc37136fc
       valid_lft forever preferred_lft forever

#这是运行一个tomcat
docker run -d -P --name tomcat01 tomcat
#查看容器内的ip addr
[root@izbp1a01zic0bgbib738aaz home]# docker exec -it tomcat01 ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
281: eth0@if282: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
#然后ping  容器内第二个网卡的ip 发现可以ping的通  
```

**理解：**

1. 我们在服务器上安装了docker ，就会分配一个网卡`docker0`,这个网卡使用的时桥接模式，使用的技术是veth-pair技术，我们每启动一个docker容器，docker就会给容器分配一个ip，

2. 我们再启动一个容器，就会发现又多了**一对网卡**(比如容器内的是`281: eth0@if282`，那么容器外的就是`282: vethea82353@if281`)

   ```shell
   #veth-pair 就是一对的虚拟设备接口，他们都是成对出现的，一端连着协议，一端彼此相连
   #正因为有这个特性，veth-pair 充当一个桥梁，连接各种虚拟网络设备
   #OpenStac,Docker容器之间的连接，OVS的连接，都是使用veth-pair技术
   ```

3. 当通过第二个容器ping第一个容器的时候发现是可以ping通的

   ```shell
   [root@izbp1a01zic0bgbib738aaz home]# docker exec -it tomcat02 ping 172.17.0.2
   PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
   64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.100 ms
   64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.088 ms
   ^C
   --- 172.17.0.2 ping statistics ---
   2 packets transmitted, 2 received, 0% packet loss, time 999ms
   
   ```

**结论**:

​	容器1和容器2能够ping通，他们是通过docker0来通信的，可以把docker0理解为路由器，所有容器在不指定网络的情况下，都是通过docker0路由的，docker会给容器分配一个默认可用的ip, 容器删除后，对应的一对网卡就没有了

##### 7.2 --link

	>思考一个场景，我们编写了一个微服务，database url =ip, 项目不重启，数据库ip换掉了，数据库就连接不通了，所以我们希望能够通过容器名称来访问容器

```shell
#这样是ping不通的
[root@izbp1a01zic0bgbib738aaz home]# docker exec -it tomcat02 ping tomcat01
ping: tomcat01: Name or service not known


#如果在运行容器的时候，加上--link参数,这样在tomcat03中就能ping通tomcat01
docker run -d -P --link tomcat01 --name tomcat03 tomcat
root@izbp1a01zic0bgbib738aaz home]# docker exec -it tomcat03 ping tomcat01
PING tomcat01 (172.17.0.2) 56(84) bytes of data.
64 bytes from tomcat01 (172.17.0.2): icmp_seq=1 ttl=64 time=0.139 ms
64 bytes from tomcat01 (172.17.0.2): icmp_seq=2 ttl=64 time=0.066 ms
^C
--- tomcat01 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
```

其本质是：在容器内的/etc/hosts文件中添加了一个`172.17.0.2	tomcat01 3962e894c964`

现在已经不建议使用--link

##### 7.3 自定义网络(docker network create)

解决容器间互相访问的问题

> 查看所以docker网络

```shell
[root@izbp1a01zic0bgbib738aaz home]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
5d5844c15c15        bridge              bridge              local
88cbc37136fc        docker_es_default   bridge              local
aaa91651296f        host                host                local
d6a3572f9162        none                null                local
```

**网络模式**:

- bridge: 桥接模式(默认)
- none: 不配置网络
- host: 和宿主机共享网络
- container: 容器网络联通(用的少，局限性大)

```shell
#当我run一个镜像的时候，有一个默认设置是-net bridge 而这个就是我们的docker0

#docker0 的特点是：默认，域名不能访问，--link可以打通，但是不建议用

#我们可以自定义一个网络
#--driver bridge 桥接模式
#--subnet 192.168.0.0/16 子网掩码 192.168.0.2 - 192.168.255.255
#--gateway 192.168.0.1   网关
docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 ygnet
```

这时我们就自定义了一个网络

当我们run一个镜像的时候，如果使用的是自己创建的网络，那么就能解决docker0的问题(域名不能ping通)

```shell
#使用自己自定义的网络ygnet
[root@izbp1a01zic0bgbib738aaz home]# docker run -d -P --name tomcat01 --net ygnet tomcat
6547cdf4f5ef065fc91bc03d275c61753c8733029c7dcb7b942c966cc14bd525
[root@izbp1a01zic0bgbib738aaz home]# docker run -d -P --name tomcat02 --net ygnet tomcat
2e0e825623121972c773115b969543cb6fc76134e8ce1f433efd135c66bc8033

#在tomcat01中ping tomcat02 能够ping通
#在tomcat02中ping tomcat01 也能够ping通
[root@izbp1a01zic0bgbib738aaz home]# docker exec -it tomcat01 ping tomcat02
PING tomcat02 (192.168.0.3) 56(84) bytes of data.
64 bytes from tomcat02.ygnet (192.168.0.3): icmp_seq=1 ttl=64 time=0.103 ms
64 bytes from tomcat02.ygnet (192.168.0.3): icmp_seq=2 ttl=64 time=0.072 ms
64 bytes from tomcat02.ygnet (192.168.0.3): icmp_seq=3 ttl=64 time=0.074 ms
64 bytes from tomcat02.ygnet (192.168.0.3): icmp_seq=4 ttl=64 time=0.073 ms
^C
--- tomcat02 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3ms
rtt min/avg/max/mdev = 0.072/0.080/0.103/0.015 ms
```

**推荐我们平时这样使用网络**

这样的好处：不同的集群使用各自的网络，保证集群的安全和健康

##### 7.4 网络联通(docker network connect)

解决各网络间互相访问的问题，比如docker0网络下有两个容器1，2，自定义网络`ygnet`下有两个容器3,4，容器1是不能ping通容器3或者4的

**使用的是`docker network connect 网络  容器`命令**

```shell
#将容器tomcat03加入ygnet网络中
docker network connect ygnet tomcat03

#查看ygnet网络详情
docker network inspect ygnet
#可以看到tomcat03已经加进来了，
#这样就是一个容器有了两个不同的ip !
"Containers": {
            "2e0e825623121972c773115b969543cb6fc76134e8ce1f433efd135c66bc8033": {
                "Name": "tomcat02",
                "EndpointID": "decc0a1ea76d82dad20467f3ef0e8f39980b74ea97de08220df241fc449f9950",
                "MacAddress": "02:42:c0:a8:00:03",
                "IPv4Address": "192.168.0.3/16",
                "IPv6Address": ""
            },
            "6547cdf4f5ef065fc91bc03d275c61753c8733029c7dcb7b942c966cc14bd525": {
                "Name": "tomcat01",
                "EndpointID": "76222f9c93a056776ea3ef61a03be0682048833d4864b19531f4079964eb54eb",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            },
            "c142994b633e7df42d27fd763af831601389056102c547fb6de1ca0f0697fa90": {
                "Name": "tomcat03",
                "EndpointID": "69fa9020f1269191808a023b197ddb3dda0a570fd32df4b7b77bc29464f83103",
                "MacAddress": "02:42:c0:a8:00:04",
                "IPv4Address": "192.168.0.4/16",
                "IPv6Address": ""
            }
        },

#这样 tomcat03 就能ping通tomcat01或者02了
```

##### 7.5 部署redis集群

