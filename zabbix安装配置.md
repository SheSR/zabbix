# 基于LNMP搭建zabbix #
系统环境：Centos64位


## zabbix 服务端安装 ##

首先安装依赖包

	yum install net-snmp-devel libxml2-devel libcurl-devel libevent-devel

下载解压

	tar -xzvf cd zabbix-3.4.9
	cd cd zabbix-3.4.9/

编译安装

	./configure --prefix=/usr/local/zabbix-2.2.2/ --enable-server \
	--enable-agent --with-mysql --with-net-snmp --with-libcurl --with-libxml2

	make
	make install

创建用户

	# groupadd zabbix
	# useradd -g zabbix zabbix


## 初始化数据库 ##

	#mysql -uroot -p

	mysql> create database zabbix default charset utf8;
	mysql> quit;

备注：创建数据库请别忘记加 default charset utf8，有可能会导致你出现中文乱码问题


在tar解压的源码包文件夹中导入sql

	[root@localhost / ]#cd /download/zabbix-3.4.9/
	[root@localhost zabbix-3.4.9]

导入sql（数据库为帐号root  密码root）

	# mysql -uroot -proot zabbix < database/mysql/schema.sql

如果你仅仅是初始化 proxy 的数据库，那么够了。如果初始化 server，那么接着导入下面两个 sql

	# mysql -uroot -proot zabbix < database/mysql/images.sql
	# mysql -uroot -proot zabbix < database/mysql/data.sql


## 配置zabbix ##

修改zabbix配置文件

	# mkdir /etc/zabbix
	# cp conf/zabbix_server.conf /etc/zabbix/
	# vim /etc/zabbix/zabbix_server.conf

	找到以下字段并修改：
	DBName=zabbix
	DBUser=root
	DBPassword=root
	DBPort=3306


运行服务

	/usr/local/zabbix/sbin/zabbix_server
默认端口10051


如果出现错误提示

/usr/local/zabbix/sbin/zabbix_server: error while loading shared libraries: libmariadb.so.3: cannot open shared object file: No such file or directory

那就执行

	yum install MariaDB-shared

安装该软件提示不存在的话，就配置一下仓库：

	cat > /etc/yum.repos.d/MariaDB.repo <<- "EOF" #Need mariadb 10.2
	[mariadb] 
	name = MariaDB 
	baseurl = http://yum.mariadb.org/10.2/rhel7-amd64 
	gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB 
	gpgcheck=1 
	EOF


**安装客户端和配置（略过，流程基本差不多）**

## zabbix管理网站配置 ##

拷贝前端文件

	# mkdir /data/logs/nginx
	# mkdir -p /data/site/monitor.ssr.com/zabbix
	# cp -rp frontends/php/* /data/site/monitor.ssr.com/zabbix


配置虚拟主机

	#mkdir /usr/local/nginx/conf/vhost
	# vim /usr/local/nginx/conf/vhost/monitor.ssr.com.conf

	server {
	listen 80;
	server_name monitor.ssr.com;
	access_log /data/logs/nginx/monitor.ssr.com.access.log main;
	index index.html index.php index.html;
	root /data/site/monitor.ssr.com;
	location /
	{
	try_files $uri $uri/ /index.php?$args;
	}
	location ~ ^(.+.php)(.*)$ {
	fastcgi_split_path_info ^(.+.php)(.*)$;
	include fastcgi.conf;
	fastcgi_pass 127.0.0.1:9000;
	fastcgi_index index.php;
	fastcgi_param PATH_INFO $fastcgi_path_info;
	}
	}


修改属组为nobody:nobody

	/data/site/monitor.ssr.com	

登录网页在线设置zabbix

	http://monitor.ssr.com/zabbix

无法访问在/etc/hosts里加入一行

	127.0.0.1  monitor.ssr.com










