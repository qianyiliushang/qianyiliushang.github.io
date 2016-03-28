---
title: linux下使用nginx运行wordpress
date: 2016-03-21 11:43:10
tags: nginx, wordpress
categories: 运维相关
---

# 写在前面的一些
其实我只是一个测试工程师，最近公司在搞开放API平台，API文档管理问题居然落到了我身上，此处省略1W字。
既然任务来了，那就做吧，初步的方案无非两种，一是使用confluence做为wiki来管理，二是使用基于wordpress的cms系统管理，第一种方案，其实早在刚来公司的时候就已经把wiki建好了，到时只需要添加内容进去即可。对于第二种方案，只能从头开始了。
一开始打算找运维给搞个php的环境，把wordpress架上去就OK了，结果，运维同学扔了句，ngix现成的，自己去配置吧，心中再次跑过万只草泥马。结合之前的的工作经历，测试跟运维打交道其实还是挺多的，但是，结果只是，运维真的很难搞定，由于之前的工作都是java相关的，既然搞PHP，那就自己从新开始摸索吧。
<!-- more -->
# 软件版本
不知道是什么原因，对于PC或者server上的软件，我总是喜欢最新的
* php 7.0.4
* mysql 5.6.28(之前用过6.x，wiki各种坑，无奈换成5.6)
* nginx 1.8.1
以上软件官方均可下载
# 依赖安装
```
yum -y install gcc gcc-c++ autoconf automake libtool make cmake
yum -y install zlib zlib-devel openssl openssl-devel pcre-devel
```
zlib: 为nginx提供gzip模块，需要zlib库支持
openssl: 为nginx提供ssl功能
pcre: 为支持地址重写rewrite功能

```
yum -y install libxml2 libxml2-devel openssl openssl-devel curl-devel libjpeg-devel libpng-devel freetype-devel libmcrypt-devel

```
# 安装PHP
* 安装php依赖，`yum install php-fpm`,`yum install php-mysql`
* 解压php`tar zxvf php-7.0.4.tar.gz`
* 配置 php-fpm和php-mysql
```
cd php-7.0.4
./configure --prefix=/usr/local/php7 \
--with-config-file-path=/usr/local/php7/etc \
--with-config-file-scan-dir=/usr/local/php7/etc/php.d \
--with-mcrypt=/usr/include \
--enable-mysqlnd \
--with-mysqli \
--with-pdo-mysql \
--enable-fpm \
--with-fpm-user=nginx \
--with-fpm-group=nginx \
--with-gd \
--with-iconv \
--with-zlib \
--enable-xml \
--enable-shmop \
--enable-sysvsem \
--enable-inline-optimization \
--enable-mbregex \
--enable-mbstring \
--enable-ftp \
--enable-gd-native-ttf \
--with-openssl \
--enable-pcntl \
--enable-sockets \
--with-xmlrpc \
--enable-zip \
--enable-soap \
--without-pear \
--with-gettext \
--enable-session \
--with-curl \
--with-jpeg-dir \
--with-freetype-dir \
--enable-opcache
```


* 编译、安装 `make` `make install`
* 复制配置文件到安装目录
```
cp php.ini-production /usr/local/php7/etc/php.ini
cd /usr/local/php7/etc
mv php-fpm.conf.default php-fpm.conf
mv php-fpm.d/www.conf.default php-fpm.d/www.conf
```

* 修改php.ini配置
```
cd /usr/local/php/
vim php,ini
#找到这行配置 ;cgi.fix_pathinfo=1
#去掉分号并将1改为0
cgi.fix_pathinfo=0
```

将php-fpm添加到系统启动目录
```
$ cd /usr/src/php-7.0.0/sapi/fpm
$ ls
$ cp init.d.php-fpm /etc/init.d/php-fpm
$ chmod +x /etc/init.d/php-fpm
$ chkconfig --add php-fpm
$ chkconfig php-fpm on
```
启动php-fpm `/etc/init.d/php-fpm start`,然后查看一下端口使用`netstat -ntulp | grep 9000`
PHP的安装到这里就先告一段落
# 安装Nginx

* 下载并解压安装包 `tar zxvf nginx-1.8.1.tar.gz`
* 编译安装nginx
```
cd nginx-1.8.1
./configure --prefix=/app/nginx
#以下为配置完成后的输入
Configuration summary
  + using system PCRE library
  + OpenSSL library is not used
  + md5: using system crypto library
  + sha1: using system crypto library
  + using system zlib library

  nginx path prefix: "/app/nginx"
  nginx binary file: "/app/nginx/sbin/nginx"
  nginx configuration prefix: "/app/nginx/conf"
  nginx configuration file: "/app/nginx/conf/nginx.conf"
  nginx pid file: "/app/nginx/logs/nginx.pid"
  nginx error log file: "/app/nginx/logs/error.log"
  nginx http access log file: "/app/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"
#安装
make & make install
```
* 配置nginx以支持php
```
cd /app/nginx/conf/
vim nginx.conf
#修改默认的location使其支持php
location / {
            root   html;
            index  index.html index.htm index.php;
        }
#这一步配置来保证对于 .php 文件的请求将被传送到后端的 PHP-FPM 模块， 取消默认的 PHP 配置块的注释，并修改为下面的内容
location ~* \.php$ {
    fastcgi_index   index.php;
    fastcgi_pass    127.0.0.1:9000;
    include         fastcgi_params;
    fastcgi_param   SCRIPT_FILENAME    $document_root$fastcgi_script_name;
    fastcgi_param   SCRIPT_NAME        $fastcgi_script_name;
}
```
重启nginx `/app/nginx/sbin/nginx -s stop`,`/app/nginx/sbin/nginx`
删除默认的index.html`rm /app/nginx/html/index.html`
创建测试文件 `echo "<?php phpinfo(); ?>" >> /app/nginx/nginx/html/index.php`

测试安装成功![php-info](phpinfo.png)
# 部署配置wordpress
将wordpress安装包拷到`/app/nginx/html/`目录下，并解压
打开浏览器访问ip/wordpress就可以开始安装wordpress了
![wordpressStart](wordpressStart.png)
**记得先在DB里创建一个名为`wordpress`的数据库**
后面就是正常的配置数据库连接信息和用户名密码了，在此不再多述

# 总结
其实环境搭建这个事情，运维觉得太简单，而测试可能对有些操作不太熟悉，比如我一直用java的，现在让我搞php的环境，遇到各种坑都是在所难免的。
当然，这也是一个学习的过程。
