# Docker 学习笔记

![Docker Logo](imgs/logo.jpeg)

## 什么是 Docker

关于 Docker 请详情查看[《什么是 Docker》][2]，这里详细介绍了 Docker 的前世今生，以及容器与传统虚拟机之间相比的不同和优势。  

下面摘抄一段容器与传统虚拟机差异的描述：  
> 传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程；
> 而容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核，而且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。

## Docker 中的概念

Docker 包括三个基本概念

* [镜像（Image）][3]：  
理解为 Java 中的类；

* [容器（Container）][4]：  
理解为 Java 中的对象(实例)；  
内容摘抄：  
> 按照 Docker 最佳实践的要求，容器不应该向其存储层内写入任何数据，容器存储层要保持无状态化。所有的文件写入操作，都应该使用 数据卷（Volume）、或者绑定宿主目录，在这些位置的读写会跳过容器存储层，直接对宿主（或网络存储）发生读写，其性能和稳定性更高。  
> 
> 数据卷的生存周期独立于容器，容器消亡，数据卷不会消亡。因此，使用数据卷后，容器删除或者重新运行之后，数据却不会丢失。

* [仓库（Repository）][5]：  
有 Docker Registry 分为公开和私有。  
内容摘抄：  
> 我们可以通过 <仓库名>:<标签> 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 latest 作为默认标签。
> 
> 以 Ubuntu 镜像 为例，ubuntu 是仓库的名字，其内包含有不同的版本标签，如，14.04, 16.04。我们可以通过 ubuntu:14.04，或者 ubuntu:16.04 来具体指定所需哪个版本的镜像。如果忽略了标签，比如 ubuntu，那将视为 ubuntu:latest。

理解了这三个概念，就理解了 Docker 的整个生命周期。

## 安装 Docker

### 树莓派

基本操作安装[《树莓派卡片电脑安装 Docker CE》][6]文章中操作，但在进行到这一段出现了问题，
````
sudo add-apt-repository "deb [arch=armhf] https://mirrors.ustc.edu.cn/docker-ce/linux/raspbian $(lsb_release -cs) stable"
````
错误信息：  
````
Error: could not find a distribution template for Raspbian/stretch
````  
最终通过编辑``sudo vi /etc/apt/sources.list``解决，在文件中添加如下内容：  
````
deb https://download.docker.com/linux/raspbian stretch stable
````

## 镜像加速器
在开始操作之前，这里啰嗦几句。由于各个方面的原因，从国内方法 Docker Hub 的速度着实让人捉急；这里我们先添加一下国内的镜像加速服务器信息，这样会使之后的操作快那么一丢丢（ 其实也很有限 :-(  ）。  
详情看这里：[《镜像加速器》][8]。
> Docker 官方提供的中国 registry mirror https://registry.docker-cn.com  
> 七牛云加速器 https://reg-mirror.qiniu.com/  

````
$ sudo vi /etc/default/docker
````
在文件中加上这一行  

````
DOCKER_OPTS="--registry-mirror=https://registry.docker-cn.com"
````

修改完毕后保存，重新启动服务。

````
$ sudo service docker restart
````

## 操作

### 镜像（Image）

#### 获取镜像

````
$ docker pull --help

Usage:  docker pull [OPTIONS] NAME[:TAG|@DIGEST]

Pull an image or a repository from a registry

Options:
  -a, --all-tags                Download all tagged images in the repository
      --disable-content-trust   Skip image verification (default true)
````

下面的命令是获取一个Ubuntu镜像，版本为16.04。  

````
docker pull ubuntu:16.04
````

#### 查看镜像

上面安装的镜像，在哪里能看到呢？通过如下命令。

````
$ docker image ls

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              16.04               0d6bd8934d0f        4 weeks ago         91.1MB
````
在列表中可以看到所有顶级镜像信息，信息包括：仓库名、标签、镜像ID、镜像创建时间、镜像大小。  

#### 删除本地镜像
````
docker image rm <镜像ID>
````

### 容器（Container）

#### 创建并启动

* ``docker run nginx``这条命令似乎没有什么问题？的确，没有问题。可是做的是什么呢？
* ``docker run nginx ls``只使用一次的容器，之后再启动也是什么都不会发生。
* ``docker run -dit nginx``使用守护进程方式创建并运行容器，之后可以通过命令进入正在运行的容器。还是有个个问题，就是nginx运行起来以后，端口没有对外(宿主机)绑定。  
  选项说明：  
  ``-d``容器启动后会进入后台；  
  ``-t``让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上，  
  ``-i``让容器的标准输入保持打开。
* ``docker run -p 8080:80 -dit nginx``使用守护进程方式创建并运行容器，并将容器的``80``端口绑定到宿主机的``8080``端口上，这下就可以访问``http://localhost``来访问了。

#### 列表

如果上面4条命令都执行了，那已经创建了4个容器了，怎么查看呢？  

* ``docker container ls``只看到了两个容器，说好的4个呢？这是只列出了还活着的容器。
* ``docker container ls -a``加个``-a``选项，会看到创建的所有容器，死活都显示出来了。
* ``docker ps``这个简便的命名具有同样功效。

记好列表中``CONTAINER ID``的后面有大用。

#### 停止

``docker container stop [CONTAINER ID]``容器ID用上了，启动时加了``-d``的容器会在后台一直运行着，通过``stop``命令停止正在运行的容器。

#### 再启动

``docker container start [CONTAINER ID]``容器ID又用上了，启动时加了``-d``的容器停止后，通过``start``命令再次启动容器。

#### 删除

``docker container rm [CONTAINER ID]``容器ID又双用上了，已经停止的容器以后也不想用了，使用``rm``命令把它删掉。  
``docker container prune``清理所有处于终止状态的容器。  

#### 进入容器

``docker exec -it [CONTAINER ID] bash``容器ID又双叒用上了，正在后台运行的容器，通过``exec``进入，之后就可以像使用原生系统命令行一样操作容器内容了。  
只用``-i``参数时，由于没有分配伪终端，界面没有我们熟悉的 Linux 命令提示符，但命令执行结果仍然可以返回。  
当``-i`` ``-t``参数一起使用时，则可以看到我们熟悉的 Linux 命令提示符。

#### 导出/导入

* ``docker export [CONTAINER ID] > imp_test.tar``容器ID又双叒叕用上了，将指定的容器导出容器快照到 tar 文件。
* ``cat imp_test.tar | docker import - test/nginx:v1.0``将指定的容器快照文件导入为镜像。

### 仓库（Repository）

>仓库（Repository）是集中存放镜像的地方。  
>一个容易混淆的概念是注册服务器（Registry）。实际上注册服务器是管理仓库的具体服务器，每个服务器上可以有多个仓库，而每个仓库下面有多个镜像。

分为公共仓库和私有仓库。  
公共仓库包括 Docker Hub 等；Docker官方提供了私有仓库工具 docker-registry ，它也是以镜像的形式存在的，可以通过 docker 的 pull 或 run 来拉取和运行，但在树莓派上使用时提示：“ docker: no matching manifest for linux/arm in the manifest list entries. ”，看来目前是不支持ARM平台。  
所有这一部分先跳过，留下链接待以后学习[仓库（Repository）][5]。


## 参考
> * [Docker — 从入门到实践][1]
> * [免sudo使用docker命令][7]
 
[1]: https://yeasy.gitbooks.io/docker_practice/    "Docker — 从入门到实践"
[2]: https://yeasy.gitbooks.io/docker_practice/introduction/what.html  "什么是 Docker"
[3]: https://yeasy.gitbooks.io/docker_practice/basic_concept/image.html      "基本概念-镜像"
[4]: https://yeasy.gitbooks.io/docker_practice/basic_concept/container.html  "基本概念-容器"
[5]: https://yeasy.gitbooks.io/docker_practice/basic_concept/repository.html "基本概念-仓库"
[6]: https://yeasy.gitbooks.io/docker_practice/install/raspberry-pi.html "树莓派卡片电脑安装 Docker CE"
[7]: https://blog.csdn.net/baidu_36342103/article/details/69357438 "免sudo使用docker命令"
[8]: https://yeasy.gitbooks.io/docker_practice/install/mirror.html "镜像加速器"