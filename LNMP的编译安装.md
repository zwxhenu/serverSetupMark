scp *.tgz *.gz *.zip root@121.41.161.140:/usr/local/src/
◆修改ssh登录端口
vim /etc/ssh/sshd_config

◆配置防火墙

mkdir /usr/local/webserver
◆安装依赖包
yum -y install lrzsz vsftpd openssh gcc gcc-c++ autoconf libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel openssl openssl-devel pcre pcre-devel cmake libcurl-devel libjpeg-turbo-devel gd-devel libmcrypt-devel iptraf vim

◆安装nginx
nginx下载地址：http://nginx.org/en/download.html
cd /usr/local/src
wget http://nginx.org/download/nginx-1.10.1.tar.gz
groupadd nginx
useradd -r -g nginx nginx -s /sbin/nologin
tar -zxvf nginx-1.10.1.tar.gz
cd nginx-1.10.1
./configure --prefix=/usr/local/webserver/nginx --with-pcre --user=nginx --group=nginx --with-http_stub_status_module --with-http_ssl_module
make
make install
cd /usr/local/webserver/nginx/conf
mkdir conf.d

vim /usr/local/webserver/nginx/conf/nginx.conf

◆安装mysql
下载地址：http://dev.mysql.com/downloads/mysql/
安装说明：http://dev.mysql.com/doc/refman/5.7/en/source-installation.html

cd /usr/local/src
wget http://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.15.tar.gz
groupadd mysql
useradd -r -g mysql mysql -s /sbin/nologin
tar -xzvf mysql-5.7.15.tar.gz
cd mysql-5.7.15
cmake ./ -DCMAKE_INSTALL_PREFIX=/usr/local/webserver/mysql-5.7 -DSYSCONFDIR=/usr/local/webserice/mysql-5.7 -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_DEBUG=0 -DWITH_INNOBASE_STORAGE_ENGINE=1 -DMYSQL_USER=mysql -DDOWNLOAD_BOOST=1 -DWITH_BOOST=/usr/local/boost_1_59_0
make
make install
如果安装mysql出现这样的错误：Download failed, error: 28;"Timeout was reached"时添加-DOWNLOAD_BOOST_TIMEOUT=28800
boost1_59_0编译安装
wget https://iweb.dl.sourceforge.net/project/boost/boost/1.59.0/boost_1_59_0.tar.gz
tar zxvf boost_1_59_0.tar.gz
cd boost_1_59_0/
运行脚本boostrap.sh
./bootstrap.sh –with-libraries=all –with-toolset=gcc
--with-libraries 指定编译哪些boost库，all全部编译，只想编译部分库的话就把库的名称写上，用逗号分隔即可
--with-toolset 指定编译时使用哪种编译器，Linux下使用gcc即可，如果系统中安装了多个版本的gcc，在这里可以指定gcc的版本，比如–with-toolset=gcc-4.4
编译boost
./b2 toolset=gcc
安装boost
./b2 install
可以加–prefix参数:用来指定boost的安装目录，不加此参数的话默认的头文件在/usr/local/include/boost目录下，库文件在/usr/local/lib/目录下 
更新系统的动态链接库
ldconfig
安装mysql遇到的问题
默认加载/etc/my.cnf的配置注意要替换掉
mysql初始化
（1）./mysqld --no-defaults --initialize --user=mysql --basedir=/usr/local/webserice/mysql-5.7 --datadir=/usr/local/webserice/mysql-5.7/data
（2）./mysql_install_db --no-defaults --basedir=/usr/local/webserice/mysql-5.7 --datadir=/usr/local/webserice/mysql-5.7/data
basedir和datadir非默认的话加上--no-defaults
将mysql加入系统自动启动
cp support-files/mysql.server /etc/init.d/mysqld     
vim /etc/profile     
      PATH=/usr/local/mysql/bin:/usr/local/mysql/lib:$PATH    
      export PATH    
source /etc/profil
chkconfig --level 35 mysqld on
netstat -tulnp | grep 3306
◆安装PHP
下载地址：http://php.net/downloads.php
安装说明：http://php.net/manual/zh/install.php

cd /usr/local/src
wget http://cn2.php.net/distributions/php-7.0.10.tar.gz

tar -xzvf php-7.0.10.tar.gz
cd php-7.0.10
./configure --prefix=/usr/local/webserver/php-5.6 --with-config-file-path=/usr/local/webserver/php-5.6/etc --with-mysql=/usr/local/webserver/mysql-5.7 --with-mysqli=/usr/local/webserver/mysql-5.7/bin/mysql_config --with-pdo-mysql=/usr/local/webserver/mysql-5.7 --with-iconv --enable-mbstring --with-openssl --with-xmlrpc --enable-zip --enable-soap --with-curl --with-freetype-dir --with-jpeg-dir --with-png-dir --with-zlib --enable-xml --with-gd --enable-bcmath --enable-gd-native-ttf --with-zlib-dir=/usr/local --with-mcrypt --with-mhash --with-freetype-dir=/usr/local --enable-exif --enable-ftp --enable-pcntl --enable-sockets --enable-fpm --with-fpm-user=nginx --with-fpm-group=nginx

./configure --prefix=/usr/local/webserver/php-7 --with-config-file-path=/usr/local/webserver/php-7/etc  --with-mysqli=/usr/local/webserver/mysql-5.7/bin/mysql_config --with-pdo-mysql=/usr/local/webserver/mysql-5.7/ --with-iconv --enable-mbstring --with-openssl --with-xmlrpc --enable-zip --enable-soap --with-curl --with-freetype-dir --with-jpeg-dir --with-png-dir --with-zlib --enable-xml --with-gd --enable-bcmath --enable-gd-native-ttf --with-zlib-dir=/usr/local --with-mcrypt --with-mhash --with-freetype-dir=/usr/local --enable-exif --enable-ftp --enable-pcntl --enable-sockets --enable-fpm --with-fpm-user=nginx --with-fpm-group=nginx
make
make install
1、安装出现的问题：configure: error: off_t undefined; check your library configuration
解决方法：
vim /etc/ld.so.conf 
#添加如下几行
/usr/local/lib64
/usr/local/lib
/usr/lib
/usr/lib64 
#保存退出
:wq
ldconfig -v # 使之生效
2、安装zlib 低版本的用configure 高版本用cmake
tar -zxvf libzip-1.2.0.tar.gz
cd libzip-1.2.0
./configure
make && make install
wget https://libzip.org/download/libzip-1.5.2.tar.gz
tar -zxvf libzip-*
cd libzip*
mkdir build && cd build && cmake .. && make && make install
cmake会出现CMake 3.0.2 or higher is required
需要升级cmake的版本
wget https://github.com/Kitware/CMake/releases/download/v3.14.5/cmake-3.14.5-Linux-x86_64.tar.gz
移除低版本的 yum remove cmake
解压 tar -zxvf cmake-3.14.5-Linux-x86_64.tar.gz 到/usr/local/webserice/cmake
编辑 /etc/profile.d/cmake.sh 文件，加入以下内容：
export CMAKE_HOME=/usr/local/webserice/cmake
export PATH=$PATH:$CMAKE_HOME/bin
source /etc/profile  使配置生效

cp /usr/local/src/php-5.4.17/sapi/fpm/init.d.php-fpm /usr/local/webserver/php-5.4.17/sbin/
sudo chmod 755 /usr/local/webserver/php-5.4.17/sbin/init.d.php-fpm
cd /usr/local/webserver/php-5.4.17/etc
cp php-fpm.conf.default php-fpm.conf
修改php-fpm.conf
vim php-fpm.conf
log_level = error
daemonize = yes
user = nginx
group = nginx
pm.max_children = 100
pm.start_servers = 20
pm.min_spare_servers = 10
pm.max_spare_servers = 35
pm.max_requests = 1000

cp /usr/local/src/php-5.4.17/php.ini-production /usr/local/webserver/php-5.4.17/etc/php.ini
修改php.ini
expose_php = Off
error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT  & ~E_NOTICE
session.gc_maxlifetime = 14400
upload_max_filesize = 10M
cd /usr/local/src/

◆安装memcache
cd /usr/local/src
tar -xzvf libevent-2.0.21-stable.tar.gz
cd libevent-2.0.21-stable
./configure --prefix=/usr/local/webserver/libevent-2.0.21
make
make install

cd /usr/local/src
tar -xzvf memcached-1.4.15.tar.gz
cd memcached-1.4.15
./configure --prefix=/usr/local/webserver/memcached-1.4.15 --with-libevent=/usr/local/webserver/libevent-2.0.21
make
make install
cd /usr/local/webserver/memcached-1.4.15
touch test_memcached.sh
chmod +x test_memcached.sh

◆安装php memcache扩展
wget http://pecl.php.net/get/memcached-3.1.3.tgz
cd /usr/local/src
tar -xzvf memcache-2.2.7.tgz
cd memcache-2.2.7
/usr/local/webserver/php-5.4.17/bin/phpize
./configure --with-php-config=/usr/local/webserver/php-5.4.17/bin/php-config
make
make install

修改php.ini
vim /usr/local/webserver/php-5.4.17/etc/php.ini
extension_dir = "/usr/local/webserver/php-5.4.17/lib/php/extensions/no-debug-non-zts-20100525/"
extension=memcache.so

◆安装redis
cd /usr/local/src
tar -xzvf redis-2.4.16.tar.gz
cd redis-2.4.16
make PREFIX=/usr/local/webserver/redis install
cd /usr/local/webserver/redis
mkdir etc var
cp /usr/local/src/redis-2.4.16/redis.conf /usr/local/webserver/redis/etc/test_redis.conf
vim etc/test_redis.conf
/usr/local/webserver/redis/bin/redis-server /usr/local/webserver/redis/etc/test.conf

◆安装php redis扩展
http://pecl.php.net/get/redis-5.0.1.tgz
cd /usr/local/src
unzip phpredis.zip
cd phpredis-master
/usr/local/webserver/php-5.4.17/bin/phpize
./configure --with-php-config=/usr/local/webserver/php-5.4.17/bin/php-config
make
make install

修改php.ini
vim /usr/local/webserver/php-5.4.17/etc/php.ini
extension_dir = "/usr/local/webserver/php-5.4.17/lib/php/extensions/no-debug-non-zts-20100525/"
extension=redis.so


◆nfs
yum -y install nfs-utils rpcbind
service rpcbind start
service nfs start
showmount -e 10.153.203.183
exportfs -a  修改/etc/exports之后生效

nfs固定端口
vi /etc/sysconfig/nfs
RQUOTAD_PORT=20053		rquotad
LOCKD_TCPPORT=20052		nlockmgr
LOCKD_UDPPORT=20052		nlockmgr
MOUNTD_PORT=20051		mountd
STATD_PORT=20050
nfsd 端口 2049 udp/tcp
mountd	20048

rpcinfo -p localhost
系统 RPC服务在 nfs服务启动时默认会为 mountd动态选取一个随机端口（32768--65535）来进行通讯，我们可以通过编辑/etc/services 文件为 mountd指定一个固定端口：
# vi /etc/services
查看是否有mountd没有的话
在末尾添加 
mountd          20048/tcp               # NFS mount protocol
mountd          20048/udp               # NFS mount protocol

客户端在挂载的时候遇到的一个问题如下，可能是网络不太稳定，NFS默认是用UDP协议，换成TCP协议即可：

mount -t nfs 192.168.1.97:/opt/test /mnt -o proto=tcp -o nolock

fdisk -l
分区：
fdisk /dev/vdb
Command (m for help): n
Command action
e extended
p primary partition (1-4)
输入：e
Partition number (1-4): 1
First cylinder (1-9729, default 1):回车
Last cylinder or +size or +sizeM or +sizeK (1-9729, default 9729):回车
Command (m for help):w(保存退出)

格式化：

mkfs -t ext3 /dev/vdb


挂载:
mkdir /data
mount /dev/vdb /data


设置开机直接挂载
编辑/etc/fstab　文件
添加：/dev/vdb /data ext3 defaults 0 0