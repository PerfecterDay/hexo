---
title: mysql的安装及配置
date: 2018-05-11 11:02:00
tags: mysql
category: sql
---
# windwos 下 mysql 的安装及配置

## mysql 安装

1. 下载zip包，解压到某个目录
2. 解压目录下创建my.ini文件，配置basedir和datadir路径
3. 执行mysqld --initialize--console,初始化数据库，会在控制台打印出root账户的随机密码
4. 启动mysqld服务：mysqld
5. 停止mysqld服务：mysqladmin -u root shutdown，如果要密码加上-p
6. 执行mysql -hlocalhost -uroot -p连接数据库
7. 首次连接数据库后要更改初始化密码：ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';否则不能执行sql语句



## mysql 安装为服务
1. mysqld --install-manual安装为手动启动的服务，mysqld --install 安装位自动启动的服务
2. 启动：net start mysql，停止：net stop mysql
2. mysqld --remove卸载服务

# linux 下安装 mysql
    shell> groupadd mysql
    shell> useradd -r -g mysql -s /bin/false mysql
    shell> cd /usr/local
    shell> tar zxvf /path/to/mysql-VERSION-OS.tar.gz
    shell> ln -s full-path-to-mysql-VERSION-OS mysql
    shell> cd mysql
    shell> mkdir mysql-files
    shell> chown mysql:mysql mysql-files
    shell> chmod 750 mysql-files
    shell> bin/mysqld --initialize --user=mysql
    shell> bin/mysql_ssl_rsa_setup
    shell> bin/mysqld_safe --user=mysql &
    # Next command is optional
    shell> cp support-files/mysql.server /etc/init.d/mysql.server
