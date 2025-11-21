## 一、设置ini文件
---------
```
#######################################################################
# File name: my.ini l?#EOhoA/5m:
# Developed By: The Uniform Server Development Team
# Web: https://www.uniformserver.com
######################################################################## 

[mysql]
default-character-set=utf8mb4

# SERVER SECTION The following options will be read by the MySQL Server.
[mysqld]
# 设置 MySQL 安装目录
basedir=C:/app/mysql-8.4.7-winx64
# 设置 MySQL 数据目录
datadir=D:/DatabaseData/mysql
# 设置端口
port=3306
#Do not delete next line. Used for setting port when run as service
#{service_port}
bind-address=127.0.0.1
# server-id = 1  Comment Prevents error Cannot open table mysql/slave_master_info
server-id = 0
pid-file=mysql.pid
explicit_defaults_for_timestamp
#default-storage-engine=MYISAM
#authentication_policy = caching_sha2_password
# 日志设置
log-error=D:/DatabaseData/logs/mysql.err
general_log=1
general_log_file=D:/DatabaseData/logs/mysql.log

skip-external-locking
key_buffer_size = 32M
max_allowed_packet = 512M
table_open_cache = 1024
sort_buffer_size = 8M
read_buffer_size = 8M
read_rnd_buffer_size = 32M
thread_stack = 256K

# 字符集设置
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci

# federated
skip-federated


# Uncomment the following if you are using InnoDB tables
innodb_file_per_table = 1
innodb_data_file_path = ibdata1:10M:autoextend
innodb_buffer_pool_size = 256M
innodb_redo_log_capacity = 128M
innodb_flush_log_at_trx_commit = 1
innodb_lock_wait_timeout = 800
innodb_write_io_threads = 8 
innodb_read_io_threads = 8 
innodb_thread_concurrency = 16 


[mysqldump]
quick
max_allowed_packet = 512M

[mysql]
# no-auto-rehash
# Remove the next comment character if you are not familiar with SQL
#safe-updates

[myisamchk]
read_buffer = 8M 
write_buffer = 8M 

#[mysqlhotcopy]
#interactive-timeout

# [mysqld]
#disable_log_bin

[client]
default-character-set=utf8mb4
port=3306

```

## 2、初始化数据库
初始化
```
 .\bin\mysqld.exe --defaults-file=my.ini --initialize-insecure --console

```
注意生成的密码，如果提示：empty则说明没有生成密码，空密码直接登陆即可
启动服务
```
.\bin\mysqld.exe --defaults-file=my.ini --console
```
登陆
```
.\bin\mysql -u root
```

修改密码
```
-- MySQL 8.0+ 版本
ALTER USER 'root'@'localhost' IDENTIFIED BY '你的强密码';
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';

-- MySQL 5.7 版本
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('你的强密码');

-- 1. 创建远程root用户
CREATE USER 'root'@'192.168.46.113' IDENTIFIED BY 'root';

-- 2. 授予完整权限
GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.46.113' WITH GRANT OPTION;

-- 创建允许192.168.46网段任意IP登录的用户
CREATE USER 'xiao'@'192.168.46.%' IDENTIFIED BY 'xiao';

-- 授予权限（根据需要调整）
GRANT ALL PRIVILEGES ON *.* TO 'xiao'@'192.168.46.%' WITH GRANT OPTION;

-- 刷新权限
FLUSH PRIVILEGES;
-- 刷新权限
FLUSH PRIVILEGES;

-- 退出
EXIT;
```