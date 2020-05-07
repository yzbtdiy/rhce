# 第5天

## mariadb 数据库
### <font color=red>基础操作</font>
#### 安装 mariadb
```
[root@server ~]# yum -y install mariadb mariadb-server
```
#### 启动 mariadb 服务
```
[root@server ~]# systemctl restart mariadb
```
#### 初始化
**设置 root 密码，移除匿名用户，关闭远程访问等一系列安全操作**

```
[root@server ~]# mysql_secure_installation
```
#### 用数据库的root用户登录，密码为redhat
```
[root@server ~]# mysql -uroot -predhat
```
#### 查看数据库
```
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
```
#### 创建数据库
```
MariaDB [(none)]> create database testsql;
```
#### 使用数据库
```
MariaDB [(none)]> use testsql;
MariaDB [testsql]>
```
#### 查看数据表
```
MariaDB [testsql]> show tables;
```
#### 创建表
```
MariaDB [testsql]> create table user(id int(3),name varchar(30));
```
#### 插入数据
```
MariaDB [testsql]> insert into user values(1,'zhangsan');

MariaDB [testsql]> insert into user values(2,'lisi');

MariaDB [testsql]> insert into user values(3,'wangwu');
```
#### 查看表中数据
```
MariaDB [testsql]> select * from user;
+------+----------+
| id   | name     |
+------+----------+
| 1    | zhangsan |
| 2    | lisi     |
| 3    | wangwu   |
+------+----------+
```
#### 删除id为1的数据
```
MariaDB [testsql]> delete from user where id=1;

MariaDB [testsql]> select * from user;
+------+--------+
| id   | name   |
+------+--------+
| 2    | lisi   |
| 3    | wangwu |
+------+--------+
```
#### 更新id为3的数据name字段为zhaoliu
```
MariaDB [testsql]> update user set name="zhaoliu" where id=3;

MariaDB [testsql]> select * from user;
+------+---------+
| id   | name    |
+------+---------+
| 2    | lisi    |
| 3    | zhaoliu |
+------+---------+
```
#### 删除数据表
```
MariaDB [testsql]> drop table user;
```
#### 删除testsql数据库
```
MariaDB [testsql]> drop database testsql;
```
#### 创建数据库用户
```
MariaDB [(none)]> create user testuser@localhost identified by '1234';
```
#### 赋予用户权限
```
MariaDB [(none)]> grant select,insert,update,delete on testsql.* to testuser@localhost;
```
#### 撤销用户权限
```
MariaDB [(none)]> revoke select,insert,update,delete on testsql.* from testuser@localhost;
```
#### 删除用户
```
MariaDB [(none)]> delete from mysql.user where user='testuser';
```
#### 刷新权限
```
MariaDB [(none)]> flush privileges;
```
### <font color=red>重置 mariadb root 密码</font>
#### 编辑 my.cnf 文件
```
[root@desktop ~]# vim /etc/my.cnf
[mysqld]
# 添加skip-grant-tables即可，修改完密码后删除此行
skip-grant-tables
```
#### 重启服务，重新加载配置文件
```
[root@desktop ~]# systemctl restart mariadb
```
#### 直接无密码登录
```
[root@desktop ~]# mysql
MariaDB [(none)]>
```
#### 修改root密码
```
MariaDB [(none)]> update mysql.user set password=password('root') where user='root';
```
#### 刷新权限
```
MariaDB [(none)]> flush privileges;
```
#### 用新密码登录
**修改完密码记得删除 my.cnf 文件里的 skip-grant-tables**

```
[root@desktop ~]# mysql -uroot -proot
MariaDB [(none)]>
```

### <font color=red>mariadb 主从复制</font>
### master
#### 开启二进制日志
```
[root@server ~]# vim /etc/my.cnf
[mysqld]
... ...
server-id=11
log-bin=/var/lib/mysql/master_bin_log
binlog_format=mixed
[root@server ~]# systemctl restart mariadb
```
#### 创建同步账号
```
[root@server ~]# mysql -uroot -predhat
MariaDB [(none)]> grant replication slave on *.* to 'slave'@'%' identified by 'redhat';
MariaDB [(none)]> flush privileges;
```
#### 备份数据库，查看 log 文件及位置
```
MariaDB [(none)]> flush tables with read lock;
[root@server ~]# mysqldump -uroot -predhat --all-databases > bakdb.sql
MariaDB [(none)]> show master status;
+----------------------+----------+--------------+------------------+
| File                 | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+----------------------+----------+--------------+------------------+
| master_bin_log.000001 |      328 |              |                  |
+----------------------+----------+--------------+------------------+
1 row in set (0.00 sec)
MariaDB [(none)]> unlock tables;
```

### slave
#### 开启二进制日志
```
[root@desktop ~]# vim /etc/my.cnf
[mysqld]
... ...
server-id=10
relay-log=/var/lib/mysql/slave_bin_log
[root@desktop ~]# systemctl restart mariadb
```
#### 导入备份数据
```
[root@desktop ~]# scp root@192.168.3.11:/root/bakdb.sql .
[root@desktop ~]# mysql -uroot -predhat < bakdb.sql
```
#### 开启 slave
```
MariaDB [(none)]> change master to master_host='192.168.3.11',master_user='slave',master_password='redhat',master_log_file='master_bin_log.000001',master_log_pos=328;
MariaDB [(none)]> start slave;
MariaDB [(none)]> show slave status\G
... ...
            Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
... ...
```

