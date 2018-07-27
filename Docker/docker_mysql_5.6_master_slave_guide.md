# 基于 Docker 的 MySQL 5.6 主从配置

![](imgs/logo.jpeg)

## 通过 Docker 安装并运行 MySQL 5.6

运行 docker 命令``docker pull mysql:5.6``拉取 MySQL 5.6 版本。  

拉取完成后使用命令``docker images |grep mysql``查看 docker 镜像。  

创建两个数据库所用到的目录（数据和配置目录）。 

````
mkdir mysql1
cd mysql1
mkdir -p data logs conf

cd ..

mkdir mysql2
cd mysql2
mkdir -p data logs conf
````

这样可以得到如下结构的目录：

````
~
+
+ mysql1
+   +---- data
+   +---- logs
+   +---- conf
+
+ mysql2
+   +---- data
+   +---- logs
+   +---- conf
 
````

进入文件夹 mysql1 中，执行命令 ``docker run -p 3301:3306 --name mysql1 -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6``。  
进入文件夹 mysql2 中，执行命令 ``docker run -p 3302:3306 --name mysql2 -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6``。  

这下我们得到了两个容器，也就有了两个运转良好的 MySQL 服务。  

````
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                    PORTS                    NAMES
0aa17668528c        mysql:5.6           "docker-entrypoint.s…"   3 hours ago         Up About an hour          0.0.0.0:3301->3306/tcp   mysql1
c438ac36ca04        mysql:5.6           "docker-entrypoint.s…"   3 hours ago         Up 30 minutes             0.0.0.0:3302->3306/tcp   mysql2
````

通过命令查看两台服务的 IP 地址。  
mysql1 服务 IP 为 172.17.0.2，我们把它作为主服务；  
mysql2 服务 IP 为 172.17.0.3，我们把它作为从服务。  

````
$ docker inspect --format='{{.NetworkSettings.IPAddress}}' 0aa17668528c
172.17.0.2
$ docker inspect --format='{{.NetworkSettings.IPAddress}}' c438ac36ca04
172.17.0.3
````

## 分别配置两台服务器

### 配置主服务器（Master）
首先，进入 __Master__ 服务器命令行``docker exec -it 0aa17668528c /bin/bash``。  
进入后，查看 MySQL 服务状态

````
# service mysql status
[info] MySQL Community Server 5.6.40 is running.
````

修改 __Master__ 配置，进入目录``/etc/mysql/conf.d``。在 docker 容器中由于是最小安装环境，系统中并没有安装 vi 或 vim 等工具，我们在宿主机中完成配置文件的添加与修改。  
因为在创建和运行容器时，使用了``-v``参数指定 MySQL 的三个目录到宿主机的本地目录中，现在我们可以直接在 conf 文件夹中添加文件，为了区分主从配置，给文件取名``my-m.cnf``，文件内容如下：  

````
[mysqld]
## 设置server_id，一般设置为IP，同一局域网内注意要唯一
server_id=100  
## 复制过滤：也就是指定哪个数据库不用同步（mysql库一般不同步）
binlog-ignore-db=mysql  
## 开启二进制日志功能，可以随便取，最好有含义（关键就是这里了）
log-bin=edu-mysql-bin  
## 为每个session 分配的内存，在事务过程中用来存储二进制日志的缓存
binlog_cache_size=1M  
## 主从复制的格式（mixed,statement,row，默认格式是statement）
binlog_format=mixed  
## 二进制日志自动删除/过期的天数。默认值为0，表示不自动删除。
expire_logs_days=7  
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062
````

添加配置后重启 MySQL 服务，``service mysql restart``；在执行重启服务命令后，容器会自动停止，我们需要再执行启动命令，``docker start 0aa17668528c``。  

启动容器后再次进入容器的命令行，进入 MySQL 命令行添加数据同步用户并授权。

````
$ docker exec -it 0aa17668528c /bin/bash

# mysql -p
Enter password:

mysql> CREATE USER 'slave'@'%' IDENTIFIED BY '123456';
mysql> GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';

````

### 配置从服务器（Slave）

与 __Master__ 配置方法类似，这次我们在 mysql2 中的 conf 目录创建文件``my-s.cnf``，文件内容如下：

````
[mysqld]
## 设置server_id，一般设置为IP,注意要唯一
server_id=105
## 复制过滤：也就是指定哪个数据库不用同步（mysql库一般不同步）
binlog-ignore-db=mysql  
## 开启二进制日志功能，以备Slave作为其它Slave的Master时使用
log-bin=edu-mysql-slave1-bin  
## 为每个session 分配的内存，在事务过程中用来存储二进制日志的缓存
binlog_cache_size=1M  
## 主从复制的格式（mixed,statement,row，默认格式是statement）
binlog_format=mixed  
## 二进制日志自动删除/过期的天数。默认值为0，表示不自动删除。
expire_logs_days=7  
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062
## log_slave_updates表示slave将复制事件写进自己的二进制日志
log_slave_updates=1  
## 防止改变数据(除了特殊的线程)
read_only=1
````

文件添加后与 __Master__ 一样重启服务。  

## 连接 __Master__ 和  __Slave__ 

__* 注意：在做 Master 和 Slave 连接前，一定要确保 Master 和 Slave 除了不同步的数据库，其他数据库完全一致。__

进入 __Master__ 命令行，并进入 mysql 命令行，执行命令。  

````
mysql> show master status;
+----------------------+----------+--------------+------------------+-------------------+
| File                 | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+----------------------+----------+--------------+------------------+-------------------+
| edu-mysql-bin.000002 |      899 |              | mysql            |                   |
+----------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
````

以上信息记录 File 和 Position；再到 __Slave__ 命令行，并进入 mysql 命令行，执行命令。

````
mysql> change master to master_host='172.17.0.2', master_user='slave', master_password='123456', master_port=3306, master_log_file='edu-mysql-bin.000002', master_log_pos= 899, master_connect_retry=30;
Query OK, 0 rows affected, 2 warnings (0.03 sec)
````

查看主从同步状态，执行命令。

````
mysql> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State:
                  Master_Host: 172.17.0.2
                  Master_User: slave
                  Master_Port: 3306
                Connect_Retry: 30
              Master_Log_File: edu-mysql-bin.000002
          Read_Master_Log_Pos: 899
               Relay_Log_File: mysqld-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: edu-mysql-bin.000002
             Slave_IO_Running: No
            Slave_SQL_Running: No
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 899
              Relay_Log_Space: 120
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 100
                  Master_UUID: ad59c6e4-8b17-11e8-a885-0242ac110002
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State:
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
1 row in set (0.00 sec)

ERROR:
No query specified
````

重点在这里(下面)，Slave_IO_State为空，Slave_IO_Running 和 Slave_SQL_Running 都是 No，这是我们还没有启动主从同步。  

````
               Slave_IO_State:
                  Master_Host: 172.17.0.2
                  Master_User: slave
                  Master_Port: 3306
                Connect_Retry: 30
              Master_Log_File: edu-mysql-bin.000002
          Read_Master_Log_Pos: 899
               Relay_Log_File: mysqld-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: edu-mysql-bin.000002
             Slave_IO_Running: No
            Slave_SQL_Running: No
````

执行以下命令，开始开启主从同步：

````
start slave;
````

大功告成！

如果未能同步，通过上面的``show slave status \G;``命令可以查看状态和错误信息。

## 关于重置 MySQL 主从同步

1. 首先停掉 __Slave__ 的同步服务，``stop slave;``。
2. 对 __Master__ 数据库加锁，``flush tables with read lock;``。
3. 备份 __Master__ 。
4. 重置 __Master__ 服务，``reset master;``。
5. 解锁 __Master__ 数据库，``unlock tables;``。
6. 将备份的数据导入 __Slave__ 。
7. 重置 __Slave__ 服务，``reset slave;``。
8. 开启 __Slave__ 服务，``start slave;``。
9. 查看同步状态，``show slave status \G;``。

## 参考
本文中并未对所执行的命令进行说明，请对应《参考》查看。

> [基于 Docker 的 MySQL 主从复制搭建][1]  
> [重置mysql主从同步（MySQL Reset Master-Slave Replication）][2]  

[1]: https://www.jianshu.com/p/ab20e835a73f "基于 Docker 的 MySQL 主从复制搭建"
[2]: https://www.cnblogs.com/sunyuxun/archive/2012/09/13/2683338.html "重置mysql主从同步（MySQL Reset Master-Slave Replication）"