---
layout: post
title: 使用 Docker 完成 MySQL 数据库主从配置
---

使用 docker 进行数据库主从配置，因为我有这个需求，而在网上搜索后发现没有满足我需求的相关实践文档，有的是一些零零碎碎的文档，而且在参照这些文档进行部署的时候我还踩了许多坑。

因此根据我自己部署成功的经验，我写了这个文档。

使用 docker 自然就需要有 docker 环境，当然 docker 的镜像在国内访问比较慢，建议使用国内的源。

## 构建 DockerFile

我们的工作是在 Mysql 镜像基础上进行的。

`docker pull mysql:5.7.20` 这条命令会下载最新的 mysql 镜像，当然也可以指定版本。

新建一个 DockerFile 文件：

```
FROM mysql:5.7.20

EXPOSE 3306

COPY my.cnf /etc/mysql/

CMD ["mysqld"]
```

在构建镜像前，我们需要 mysql 的配置文件 my.cnf ，可以先启动一个 mysql 的容器，然后进入容器复制它下来，并对其进行修改：

```
!includedir /etc/mysql/conf.d/
[mysqld]
pid-file=/var/run/mysqld/mysqld.pid
socket=/var/run/mysqld/mysqld.sock
datadir=/var/lib/mysql
#log-error=/var/log/mysql/error.log
# By default we only accept connections from localhost
#bind-address	= 127.0.0.1
log-bin=/var/log/mysql/mysql-bin.index
server-id=1
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
```

以上是我修改好的一个 my.conf 文件，我们主要修改的是以下这些选项：

```
#bind-address	= 127.0.0.1 # 注释这个选项可以让远程机器访问，或者可以修改为 0.0.0.0
log-bin=/var/log/mysql/mysql-bin.index # 开启 log-bin 日志，日志路径
server-id=1 # 服务器唯一ID，默认是1，一般取IP最后一段，这里看情况分配
```

这是我的主库的 `my.cnf`，从库的基本一样，只是 `servier-id` 不一样，从库为 2，将这个文件也放在文件夹里。

构建需要的文件结构就是以下这样：

```shell
master
├── Dockerfile
└── my.cnf
slave
├── Dockerfile
└── my.cnf
```

然后在master目录中使用 `docker build -t master/mysql:5.7.20 .` 命令构建镜像主库，从库构建命令 `docker build -t slave/mysql:5.7.20 .`。命令最后有个`.`，不要忘记，代表当前目录。-t 的意思是tag，是 --tag 的缩写，也就是命名这个镜像，如果不加docker会随机给这个镜像一个名字，建议格式为`name:tag`

构建完成后我们使用以下命令启动主库和从库：

```
docker run -p 3306 --name mysql-master -e MYSQL_ROOT_PASSWORD=root -d master/mysql:5.7.20

docker run -p 3306 --name mysql-slave --link mysql-master:master -e MYSQL_ROOT_PASSWORD=root -d slave/mysql:5.7.20
```

然后分别执行`docker exec -it mysql-master bash`和`docker exec -it slave-master bash`命令进入到容器内部。执行`mysql -uroot -proot`进入mysql环境，这时我们的环境搭建就已经完成了，下面正式配置主从连接。

当然此时docker会分配一个唯一的端口给容器，我们也可以用mysql客户端连接进行配置。使用`docker ps`查看端口号，就可以用客户端连接进行管理。

```shell
☁  mysql-master-slave  docker ps
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                     NAMES
6c7648e829e0        slave/mysql:5.7.20    "docker-entrypoint..."   4 minutes ago       Up 4 minutes        0.0.0.0:32769->3306/tcp   mysql-slave
483842c63235        master/mysql:5.7.20   "docker-entrypoint..."   5 minutes ago       Up 5 minutes        0.0.0.0:32768->3306/tcp   mysql-master
```

使用`GRANT REPLICATION SLAVE ON *.* to 'user'@'%' identified by 'mysql';` 或者 `GRANT REPLICATION SLAVE ON *.* TO 'user'@'192.168.1.200' IDENTIFIED BY 'mysql';`创建一个用户，上一个是所有ip都可以访问，下一个是指定ip才可以访问。

然后使用`GRANT SELECT,REPLICATION SLAVE ON *.* TO 'user'@'%';`给这个用户读取权限。


使用`show master status`查看mysql主容器的状态，打印如下信息：

```
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000003 |      154 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
```

使用`hostname`查看主容器的hostname，如我的主容器是`483842c63235`。

接下来配置从库mysql：

```
change master to
master_host='master',#要连接的主服务器的ip
master_user='user',#指定的用户名，最好不要用root
master_log_file='mysql-bin.000003',#主库记录的值
master_log_pos=154,#主库的pos值
master_port=3306,#主库3306映射的端口
master_password='mysql';#主库要连接的用户的密码了
```

使用`start slave`启动主从同步

使用命令查看`show slave status\G`

打印如下信息：

```
*************************** 1. row ***************************
               Slave_IO_State: Connecting to master
                  Master_Host: master //主服务器地址
                  Master_User: user //授权帐户名，尽量避免使用root
                  Master_Port: 3306 //数据库端口，部分版本没有此行
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000003 //同步主库的日志文件名
          Read_Master_Log_Pos: 154 //同步读取二进制日志的位置，大于等于
               Relay_Log_File: 6c7648e829e0-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mysql-bin.000003
             Slave_IO_Running: Yes //此状态必须YES
            Slave_SQL_Running: Yes //此状态必须YES
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 154
              Relay_Log_Space: 154
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
             Master_Server_Id: 0
                  Master_UUID:
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)
```


这样就是完成了一组主从mysql的配置。

### shell脚本

一下一个相应的shell脚本，可以在1分钟能启动一组mysql主从容器，运行这个脚本需要稍作修改，主要是mysql配置文件。

```shell
#!/bin/bash

MASTER_DIR=/var/lib/mysql/mysql-master
SLAVE_DIR=/var/lib/mysql/mysql-slave

## First we could rm the existed container
docker rm -f mysql-master
docker rm -f mysql-slave

## Rm the existed directory
rm -rf $MASTER_DIR
rm -rf $SLAVE_DIR

## Start instance
docker run -p 3306 --name mysql-master  -v /etc/master.cnf:/etc/mysql/my.cnf -v $MASTER_DIR:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root -d master/mysql:5.7.20
docker run -p 3306 --name mysql-slave  -v /etc/slave.cnf:/etc/mysql/my.cnf -v $MASTER_DIR:/var/lib/mysql --link mysql-master:master -e MYSQL_ROOT_PASSWORD=root -d slave/mysql:5.7.20

## Creating a User for Replication
docker stop mysql-master mysql-slave
docker start mysql-master mysql-slave

sleep 3

docker exec -it mysql-master mysql -S /var/lib/mysql/mysql.sock -e "CREATE USER 'users'@'127.0.0.1' IDENTIFIED BY 'mysql';GRANT REPLICATION SLAVE ON *.* TO 'users'@'127.0.0.1';"

## Obtaining the Replication Master Binary Log Coordinates
master_status=`docker exec -it master mysql -S /var/lib/mysql/mysql.sock -e "show master status\G"`
master_log_file=`echo "$master_status" | awk  'NR==2{print substr($2,1,length($2)-1)}'`
master_log_pos=`echo "$master_status" | awk 'NR==3{print $2}'`
master_log_file="'""$master_log_file""'"

## Setting Up Replication Slaves 
docker exec -it mysql-slave mysql -S /var/lib/mysql/mysql.sock -e "CHANGE MASTER TO MASTER_HOST='master',MASTER_PORT=3306,MASTER_USER='users',MASTER_PASSWORD='mysql',MASTER_LOG_FILE=$master_log_file,MASTER_LOG_POS=$master_log_pos;"
docker exec -it mysql-slave mysql -S /var/lib/mysql/mysql.sock -e "start slave;"
docker exec -it mysql-slave mysql -S /var/lib/mysql/mysql.sock -e "show slave status\G"

## Creates shortcuts
grep "alias mysql-master" /etc/profile
if [ $? -eq 1 ];then
    echo 'alias mysql="docker exec -it mysql-master mysql"' >> /etc/profile
    echo 'alias master="docker exec -it mysql-master mysql -h 127.0.0.1 -P3306"' >> /etc/profile
    echo 'alias slave="docker exec -it mysql-master mysql -h 127.0.0.1 -P3307"' >> /etc/profile
    source /etc/profile
fi
```


