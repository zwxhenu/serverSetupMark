##	安装lua环境和相关库
# lua模块官网地址：https://github.com/openresty/lua-nginx-module
##  安装LuaJIT
```shell
wget wget http://luajit.org/download/LuaJIT-2.0.5.tar.gz(安装版本低)
wget https://github.com/openresty/luajit2/archive/v2.1-20190912.tar.gz

tar -zxvf LuaJIT-2.0.5.tar.gz
cd LuaJIT-2.0.5
make install PREFIX=/usr/local/webservice/luajit2-2.1-20190912
#设置环境变量或添加到/etc/profile  重新编译profile文件（source /etc/profile）
export LUAJIT_LIB=/usr/local/webservice/luajit2-2.1-20190912/lib
export LUAJIT_INC=/usr/local/webservice/luajit2-2.1-20190912/include/luajit-2.1
```
##	安装ngx_devel_kit和lua-nginx-module
```shell
wget https://github.com/openresty/lua-nginx-module/archive/v0.10.15.tar.gz
wget https://github.com/simplresty/ngx_devel_kit/archive/v0.3.1.tar.gz
//解压到指定目录下（我这解压的目录 /usr/local/webservice/）
/*****解压lua-nginx-module-0.10.15****/
tar -zxvf v0.10.15.tar.gz -C /usr/local/webservice/
/****解压ngx_devel_kit-0.3.1***/
tar -zxvf v0.3.1.tar.gz -C /usr/local/webservice/
```
##	重新编译安装nginx（当前版本1.16.1）
```shell
./nginx -V #查看当前编译加载模块	
#下面是我个人配置的	
--prefix=/usr/local/webservice/nginx1.16/ --user=www --group=www --with-pcre --with-http_stub_status_module --with-http_ssl_module --conf-path=/etc/nginx/nginx.conf --add-module=/usr/local/webservice/fastdfs-nginx-module/src/	
#重新编译nginx，进入安装nginx目录执行	
./configure --prefix=/usr/local/webservice/nginx1.16/ --user=www --group=www --with-pcre --with-http_stub_status_module --with-http_ssl_module --conf-path=/etc/nginx/nginx.conf --add-module=/usr/local/webservice/fastdfs-nginx-module/src/ --add-module=/usr/local/webservice/ngx_devel_kit-0.3.1 --add-module=/usr/local/webservice/lua-nginx-module-0.10.15 --with-ld-opt="-Wl,-rpath,$LUAJIT_LIB"
# 如若出现：
# error: the HTTP gzip module requires the zlib library.
# yum install zlib zlib-devel 一下即可

make -j 4 && make install
```	
##	查看模块是否安装成功
```shell
./nginx -V
./nginx: error while loading shared libraries: libluajit-5.1.so.2: cannot open shared object file: No such file or directory
# 报未加载lua的库
# 加载lua库，加入到ld.so.conf文件
echo "/usr/local/webservice/luajit2-2.1-20190912/lib" >> /etc/ld.so.conf
# 然后执行如下命令：
ldconfig
# 在执行
./nginx -V
nginx version: nginx/1.16.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/usr/local/webservice/nginx1.16/ --user=www --group=www --with-pcre --with-http_stub_status_module --with-http_ssl_module --conf-path=/etc/nginx/nginx.conf --add-module=/usr/local/webservice/fastdfs-nginx-module/src/ --add-module=/usr/local/webservice/ngx_devel_kit-0.3.1 --add-module=/usr/local/webservice/lua-nginx-module-0.10.15
#模块已加载上。
```
##  使用lua
# 创建lua脚本文件目录/usr/local/webservice/lua_script
# 在nginx.conf配置文件中添加
```shell	
lua_package_path "/usr/local/webservice/lua_script/?.lua;;";
lua_load_resty_core off;
location /lua_content {
			default_type 'text/plain';
			content_by_lua_block {
					ngx.say('Hello, world!')
			}
}
#重启nginx
./nginx -s reload
#访问地址：http://192.168.222.128/lua_content
#访问不了，后来发现了如下错误
#错误：
 ./nginx -c /etc/nginx/nginx.conf
nginx: [error] lua_load_resty_core failed to load the resty.core module from https://github.com/openresty/lua-resty-core; ensure you are using an OpenResty release from https://openresty.org/en/download.html (rc: 2, reason: module 'resty.core' not found:
no field package.preload['resty.core']
no file '/usr/local/webservice/lua_script/resty/core.lua'
no file './resty/core.lua'
no file '/usr/local/webservice/luajit2-2.1-20190912/share/luajit-2.1.0-beta3/resty/core.lua'
no file '/usr/local/share/lua/5.1/resty/core.lua'
no file '/usr/local/share/lua/5.1/resty/core/init.lua'
no file '/usr/local/webservice/luajit2-2.1-20190912/share/lua/5.1/resty/core.lua'
no file '/usr/local/webservice/luajit2-2.1-20190912/share/lua/5.1/resty/core/init.lua'
no file './resty/core.so'
no file '/usr/local/lib/lua/5.1/resty/core.so'
no file '/usr/local/webservice/luajit2-2.1-20190912/lib/lua/5.1/resty/core.so'
no file '/usr/local/lib/lua/5.1/loadall.so'
no file './resty.so'
no file '/usr/local/lib/lua/5.1/resty.so'
no file '/usr/local/webservice/luajit2-2.1-20190912/lib/lua/5.1/resty.so'
no file '/usr/local/lib/lua/5.1/loadall.so')
#报以上错误的话，在nginx.conf中添加lua_load_resty_core off;屏蔽掉。
#问题解决：https://gitlab.alpinelinux.org/alpine/aports/issues/10478
```