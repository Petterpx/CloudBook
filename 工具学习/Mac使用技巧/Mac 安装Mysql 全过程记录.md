# Mac 安装Mysql 全过程记录

在Mac上安装软件是一件比较轻松的事，但是总有一些奇怪的问题，鉴于网上的教程都太过于古老和分散，这里记录了安装的全过程。

## brew安装

```shell
brew install mysql
```

安装完成后如图所示：

![image-20211110120642775](https://tva1.sinaimg.cn/large/008i3skNly1gw9xc1pisrj30wm0cq766.jpg)

## 初始化配置

先启动mysql

```sql
mysql.server start
```

再执行初始化程序

```sql
mysql_secure_installation
```

## 异常情况

如果启动mysql时遇到提示mysqlID错误，按照下面步骤，先kill掉mysql的进程,再重新操作

**查找mysqlId**

```
ps -ef|grep mysqld
```

![image-20211110121416648](https://tva1.sinaimg.cn/large/008i3skNly1gw9xjwwwo8j30xw08i40v.jpg)

**kill掉指定进程**

```shell
sudo kill -9 进程id
```

## 常用命令

- mysql.server start
- Mysql.server restart
- mysql.server stop

## 一些其他问题

### 授权远程连接

Mysql8.0必须创建新用户才可以，具体如下

```sql
create user 'brady'@'%' identified by 'brady';

GRANT ALL PRIVILEGES ON *.* TO 'brady'@'%';

FLUSH PRIVILEGES;

ALTER USER 'brady'@'%' IDENTIFIED WITH mysql_native_password BY 'brady';

flush privileges;
```

对应过程如下：

```sql
<br>mysql> create user 'brady'@'%' identified by 'brady';
Query OK, 0 rows affected (0.02 sec)
mysql> GRANT ALL PRIVILEGES ON *.* TO 'brady'@'%';
Query OK, 0 rows affected (0.04 sec)
 
mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)
 
mysql> ALTER USER 'brady'@'%' IDENTIFIED WITH mysql_native_password BY 'brady';
Query OK, 0 rows affected (0.02 sec)
 
mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)
```

### 无法连接数据库

**java.sql.SQLException: Connections could not be acquired from the underlying database**

检查当前mysql版本与项目里的mysql驱动jar(mysql-connector-java)包是否匹配。