## mysql cluster example

#### 使用手工方式创建集群

```
###创建集群使用的网络
docker network create cluster --subnet=192.168.0.0/16

###启动management节点
docker run -d --net=cluster --name=management1 --ip=192.168.0.2 mysql/mysql-cluster ndb_mgmd

###启动2个data节点
docker run -d --net=cluster --name=ndb1 --ip=192.168.0.3 mysql/mysql-cluster ndbd
docker run -d --net=cluster --name=ndb2 --ip=192.168.0.4 mysql/mysql-cluster ndbd

###启动mysql服务节点，直接设置密码
docker run -d --net=cluster --name=mysql1 --ip=192.168.0.10 -e MYSQL_RROOT_PASSWORD=123456 mysql/mysql-cluster mysql

###启动mysql服务节点，使用随机密码
docker run -d --net=cluster --name=mysql1 --ip=192.168.0.10 -e MYSQL_RANDOM_ROOT_PASSWORD=true mysql/mysql-cluster mysqld

###查看随机生成的密码
docker logs mysql1 | grep PASSWORD 

###连接到mysql集群服务，可以执行相关sql
docker exec -it mysql1 mysql -uroot -p

###启动一个带有交互式管理客户端的容器，以验证集群是否已启动。
docker run -it --net=cluster mysql/mysql-cluster ndb_mgm
可以选择执行的命令：show


###关闭所有节点，然后移除集群使用的网络
docker container stop mysql/mysql-cluster
docker container stop 8e8c86cf4468
docker container stop 7908634a2ca6 ccf163a41553 4365e6d4a7c8

docker network ls 
docker network rm cluster
docker network ls 

```

#### 使用compose启动mysql-cluster集群

```
###启动服务
docker-compose -f docker-compose-2.yaml up 

###服务启动后，查看创建的network： 
docker network ls

显示：mysql-cluster-sample_cluster


###检查数据节点连接状态：
docker run -it --net=mysql-cluster-sample_cluster mysql/mysql-cluster ndb_mgm

###连接mysql服务-mysql_1
docker exec -it mysql_1 mysql -uroot -p

###修改登录密码(原始密码为：'defaultpass'，在docker-compose-2.yaml中配置)
ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass';

###设置远程登录访问（不配置的话，无法从本地连接）
GRANT ALL on *.* to 'root'@'%' identified by "MyNewPass";
FLUSH privileges;

####本地测试mysql连接
mysql -h 127.0.0.1 -P 3306  -u root -p

显示内容：
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 156
Server version: 5.7.25-ndb-7.6.9-cluster-gpl MySQL Cluster Community Server (GPL)

or 

ERROR 1130 (HY000): Host '192.168.0.1' is not allowed to connect to this MySQL server

问题：

连接mysql_1, mysql_2, 分别需要单独连接配置远程登录权限，为什么？

####创建数据库及相关表
create database world;

use world;

create table city(
id int not null auto_increment,
name varchar(120) not null default '',
primary key (id)
) engine=ndbcluster default charset=utf8mb4;

insert into city values (1,'beijing'),(2,'shanghai');
select * from city;




```

