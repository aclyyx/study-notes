# 基于 Docker 搭建 Tomcat 服务

![](imgs/logo.jpeg)

## 确认有没有 Tomcat 的镜像

通过命令``docker search tomcat``在 Docker Hub 中搜索``tomcat``。  

````
$ docker search tomcat
NAME                                  DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
tomcat                                Apache Tomcat is an open source implementati…   1970                [OK]
tomee                                 Apache TomEE is an all-Apache Java EE certif…   53                  [OK]
dordoka/tomcat                        Ubuntu 14.04, Oracle JDK 8 and Tomcat 8 base…   49                                      [OK]
...
...
...
trollin/tomcat                                                                        0
````

从搜索结果中可以看到第一条“STARTS”的最多，那个就是官方镜像，登录到 [Docker Hub][1] 搜索一下“tomcat”，我们也可以看到同样的结果。  
进入 [Docker Hub 的 Tomcat 页面][2] 可以看到目前有很多版本的镜像，包含了 Tomcat7、8、9 各个版本。本文中以 __Tomcat8__ 为例介绍安装配置过程。  

## 拉取 Tomcat 镜像

通过命令``docker pull tomcat:8``拉取镜像，拉取完成后，``docker images``检查拉取结果。  

## 运行 Tomcat 容器

通过执行以下命令运行一个我们想要的 Tomcat 容器，具体说明查看《命令说明》。  

````
# 【命令说明】
# [docker]：docker命令
# [run]： 创建并运行一个容器
# [-p 8080:8080]：映射宿主机 8080 端口指向容器的 8080 端口
# [--name wx_tomcat]：指定容器的名字叫 wx_tomcat
# [-v $PWD/webapps:/usr/local/tomcat/webapps]：指定容器中 tomcat 的 webapps 目录到宿主机的 $PWD/webapps 位置
# [-v $PWD/logs:/usr/local/tomcat/logs]：指定容器中 tomcat 的 logs 目录到宿主机的 $PWD/logs 位置
# [--privileged=true]：授予 Docker 挂载的权限
# [-d]：以后台守护进程的方式启动
# [tomcat:8]：指定容器源自哪个镜像
docker run -p 8080:8080 --name wx_tomcat -v $PWD/webapps:/usr/local/tomcat/webapps -v $PWD/logs:/usr/local/tomcat/logs --privileged=true -d tomcat:8
````

## 其他

### 进入 Tomcat 容器

上面命令是用``-d``后台守护进程的方式启动，可以使用``docker exec -it wx_tomcat /bin/sh``命令进入容器，进行查看 Tomcat 运行状态等操作。  

## 参考

> [Docker Hub][1]  
> [Docker Hub 的 Tomcat 页面][2]  
> [Docker实践 - 安装Docker并在容器里运行tomcat][3]  

[1]:https://hub.docker.com "Docker Hub"  
[2]:https://hub.docker.com/_/tomcat/ "Docker Hub 的 Tomcat 页面"  
[3]:https://blog.csdn.net/massivestars/article/details/54352484 "Docker实践 - 安装Docker并在容器里运行tomcat"  