#mysql的安装及配置

##mysql安装

1. 下载zip包，解压到某个目录
2. 解压目录下创建my.ini文件，配置basedir和datadir路径
3. 执行mysqld --initialize--console,初始化数据库，会在控制台打印出root账户的随机密码
4. 启动mysqld服务：mysqld
5. 停止mysqld服务：mysqladmin -u root shutdown，如果要密码加上-p
6. 执行mysql -hlocalhost -uroot -p连接数据库
7. 首次连接数据库后要更改初始化密码：ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';否则不能执行sql语句



##mysql安装为服务
1. mysqld --install-manual安装为手动启动的服务，mysqld --install 安装位自动启动的服务
2. 启动：net start mysql，停止：net stop mysql
2. mysqld --remove卸载服务