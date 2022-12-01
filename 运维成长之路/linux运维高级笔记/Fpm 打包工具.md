### Fpm 打包工具

安装

```shell
yum install ruby ruby-devel redhat-rpm-config rpm-build squashfs-tools -y
gem install fpm
```

创建打包目录

mkdir /data0/fpm/rpms -p





### nginx 编译

安装依赖

```shell
yum install -y gcc pcre-devel zlib-devel openssl-devel
```

下载源码包

wget http://nginx.org/download/nginx-1.22.0.tar.gz

tar zxvf nginx-1.22.0.tar.gz

cd nginx-1.22.0

编译nginx

```shell
./configure --prefix=/usr/local/webserver/nginx \
--user=www \
--group=www \
--with-http_stub_status_module \
--with-http_ssl_module  \
--with-http_gzip_static_module \
--with-http_realip_module \
--with-http_v2_module
```

make && make install

查询 依赖库

```shell
for so in  `find /usr/local/webserver/nginx -type f -exec ldd {} 2>/dev/null \;|grep -oP '/.+[0-9]\s'`;do rpm -qf $so;done|sort|uniq|sed 's/-[0-9].*//g'

输出如下：
glibc
libxcrypt
openssl-libs
pcre2
zlib
输出的库名与下面fpm 的-d 一一对应
```

打包成rpm包

```shell
#Almalinux 8
cd /data0/fpm

fpm --prefix=/usr/local/webserver/nginx \
-C /usr/local/webserver/nginx \
--config-files /usr/local/webserver/nginx/conf \
--iteration 1 \
--rpm-dist el8 \
-n nginx \
-v 1.22.0 \
-s dir \
-t rpm \
-p rpms \
-d glibc \
-d libxcrypt \
-d openssl-libs \
-d pcre2 \
-d zlib
```

查看rpm包



### php 编译

安装依赖包

```shell
yum install -y libxml2-devel bzip2-devel libcurl-devel libjpeg-turbo-devel libpng-devel libtool-ltdl-devel libXpm-devel freetype-devel gmp-devel openssl-devel sqlite-devel libmcrypt-devel oniguruma-devel
```

下载php源码包



编译php

```shell
./configure --prefix=/usr/local/webserver/php \
--with-fpm-user=www \
--with-fpm-group=www \
--with-bz2 \
--with-zlib \ 
--with-openssl \
--with-curl \
--with-pdo-mysql \
--with-mysqli \
--with-freetype \
--with-jpeg \
--with-pic \
--with-gettext \ 
--with-gmp \
--with-iconv \
--with-pear \
--with-layout=GNU \
--without-gdbm \
--enable-fpm \
--enable-opcache \
--enable-mbregex \
--enable-mbstring \
--enable-gd \
--enable-exif \
--enable-ftp \
--enable-sockets \
--enable-sysvsem \
--enable-sysvshm \
--enable-sysvmsg \
--enable-shmop \
--enable-xml \
--enable-soap \
--enable-bcmath \
--enable-pcntl \
--disable-debug \
--disable-rpath \

make && make install

```

查询php依赖库

```shell
for so in  `find /usr/local/webserver/php -type f -exec ldd {} 2>/dev/null \;|grep -oP '/.+[0-9]\s'`;do rpm -qf $so;done|sort|uniq|sed 's/-[0-9].*//g'
```



打包

```shell
cd /data0/fpm

fpm --prefix=/usr/local/webserver/php \
-C /usr/local/webserver/php \
--config-files /usr/local/webserver/php/etc \
--iteration 1 \
--rpm-dist el8 \
-n php \
-v 8.1.7 \
-s dir \
-t rpm \
-p rpms \
-d brotli \
-d bzip2-libs \
-d cyrus-sasl-lib \
...

```



编译问题：

找不到 oniguruma-devel 源

yum config-manager powertools --enable

yum install oniguruma-devel -y