# linux源码搭建lnmp过程

#### 安装nginx

下载安装编译器和依赖库

```bash
yum install -y gcc gcc-c++ pcre pcre-devel zlib zlib-devel openssl openssl-devel libxml2 libxslt
```

下载上传nginx源码解压http://nginx.org/en/download.html

```bash
tar -zxvf nginx-*.tar.gz
```

创建安装文件夹

```bash
mkdir /lnmp/nginx
```

添加用户

```bash
groupadd -r nginx 
useradd nginx -M -g nginx
```

切换到解压目录并编译

```bash
./configure --prefix=/lnmp/nginx --user=nginx --group=nginx --withhttp_ssl_module --with-http_realip_module --with-http_addition_module --withhttp_gzip_static_module
```

安装

```bash
make && make install
```

使用配置文件启动

```bash
cd /lnmp/nginx/sbin
./nginx -c conf/nginx.conf
```

创建systemctl控制文件

```bash
vi /usr/lib/systemd/system/nginx.service
```

```service
[Unit] 
Description=nginx - my nginx is best web server in world After=Static server compiled with its own source code 

[Service] 
Type=forking ExecStart=/lnmp/nginx/sbin/nginx ExecReload=/lnmp/nginx/sbin/nginx -s reload ExecStop=/lnmp/nginx/sbin/nginx -s stop 

[Install] 
WantedBy=multi-user.target
```

nginx安装完成

#### 安装php

上传解压php源码https://www.php.net/downloads.php

```bash
tar -zxvf php-*.tar.gz
```

创建安装文件夹

```bash
mkdir /lnmp/php
```

安装下载依赖库

```bash
yum install libxml2 libxml2-devel openssl openssl-devel bzip2 bzip2-devel libcurl libcurl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel gmp gmp-devel libmcrypt libmcrypt-devel readline readline-devel libxslt libxslt-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel ncurses curl gdbm-devel db4-devel libXpm-devel libX11-devel gd-devel gmp-devel expat-devel xmlrpc-c xmlrpc-c-devel libicu-devel libmcrypt-devel libmemcacheddevel libsqlite3x-devel oniguruma-devel
```

安装过程中 当yum提示没有找到软件包 那么软件包可能在扩展仓库里使用以下语句安装

```bash
yum install -y epel-release
```

编译配置

```bash
./configure --prefix=/lnmp/php --with-config-file-path=/etc --with-fpmuser=nginx --with-fpm-group=nginx --with-curl --with-freetype-dir --enable-gd - -with-gettext --with-iconv-dir --with-kerberos --with-libdir=lib64 --withlibxml-dir --with-mysqli --with-openssl --with-pcre-regex --with-pdo-mysql --with-pdo-sqlite --with-pear --with-png-dir --with-jpeg-dir --with-xmlrpc --with-xsl --with-zlib --with-bz2 --with-mhash --enable-fpm --enablebcmath --enable-libxml --enable-inline-optimization --enable-mbregex -- enable-mbstring --enable-opcache --enable-pcntl --enable-shmop --enable-soap --enable-sockets --enable-sysvsem --enable-sysvshm --enable-xml --enablezip --enable-fpm
```

完成编译后安装

```bash
make && make install
```

添加环境变量控制php

```bash
vi /etc/profile
```

添加到最后

```bash
PATH=$PATH:/usr/local/php/bin
export PATH
```

更新环境变量

```bash
source /etc/profile
```

查看版本

```bash
php -v
```

切换到源码解压包目录配置php进程

```bash
cp php.ini-production /etc/php.ini
cp /lnmp/php/etc/php-fpm.conf.default /lnmp/php/etc/php-fpm.conf
cp /lnmp/php/etc/php-fpm.d/www.conf.default /lnmp/php/etc/php-fpm.d/www.conf
cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
chmod +x /etc/init.d/php-fpm
```

启动php-fpm

```bash
/etc/init.d/php-fpm start
或者
service php-fpm start
```

在nginx中启动php

```bash
vi /lnmp/nginx/conf/nginx.conf
```

将# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000#下面的内容修改成这样

```bash
location ~ \.php$ {
    root html;
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}
```

php安装完成

#### 安装mysql

进入官网下载rpm文件 https://dev.mysql.com/downloads/repo/

点击yum 进入后下载下面这一项

| Red Hat Enterprise Linux 7 / Oracle Linux 7 (Architecture Independent), RPM Package |      | 25.4K | [Download](https://dev.mysql.com/downloads/file/?id=484922) |
| ------------------------------------------------------------ | ---- | ----- | ----------------------------------------------------------- |
| (mysql80-community-release-el7-3.noarch.rpm)                 |      |       |                                                             |

安装教程 https://dev.mysql.com/doc/refman/8.0/en/linux-installation.html

安装下载下来的yum源

yum localinstall mysql80-community-release-el7-3.noarch.rpm

**查看版本信息以及调整版本**

查看文件/etc/yum.repos.d/mysql-community.repo

文件中enabled=1为启用的版本;多个启用版本优先最新版 手动调整后开始安装

**安装mysql**

```bash
yum install mysql-community-server
```

启动mysql

利用官方yum源安装的mysql默认添加了service控制使用以下语句控制启动

```bash
service mysqld start
```

**查看初始化密码**

```bash
grep 'temporary password' /var/log/mysqld.log
```

利用初始密码登录mysql并修改初始密码

```mysql
mysql -uroot -p
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY
'password';
#mysql8.0
ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY
'password';
```

> 默认情况下安装。实现的默认密码策略要求密码至少包含一个大写字母、一个小写字母、一个数 字和一个特殊字符，并且密码总长度至少为 8 个字符。 validate_password 如果安装了mysql8.0 密码验证方式必须要使用caching_sha2_password

**mysql允许远程**

查询用户表

```mysql
select User,authentication_string,Host from user;
```

更新root的主机为允许所有%连接

```mysql
update user set host = '%' where User = 'root';
```

使用权限语句更新数据信息 权限更新会新增root远程用户 当前mysq表中就会有localhost本地以及一 条%所有主机的数据 他们使用了不同的哈希密码凭证

```mysql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'enterpass';
```

更新数据库缓存信息 因为启动数据库时数据库会将使用信息弹入缓存

```mysql
flush privileges;
```

#### centos搭建的lnmp环境所遇到的问题

搭建完lnmp环境后使用thinkphp来查看环境使用情况发现

1 STORAGE_WRITE_ERROR

数据不能写入改写权限到引用目录 例如thinkphp5中

```bash
chown -R nginx:nginx Application
```

2 SQLSTATE[HY000] [2002] No such file or directory

php找不到套接文件字 mysqld.sock 或者 mysql.sock

建立与mysql的通信有2种方式 一种是使用tcp/ip给出服务器IP地址使用账号密码连接 第二是使用套接 字协议进行连接 使用套接字协议能有规定的套接字寻找服务器地址例如localhost mysql套接字文件在 配置文件中规定 开启服务生成

查找sock路径 在mysql配置文件my.cnf中sock配置项给出 在php配置文件中指定mysql套接字路径开启 pdo_mysql.default_socket指向sock文件路径

第二种就是修改实际项目中数据库连接配置文件中的主机 使用127.0.0.1

3开启路由重写

不能使用mvc模型 在location ~ .php$ {}中给出开启pathinfo

```nginx
fastcgi_param PATH_INFO $fastcgi_path_info
fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info
fastcgi_split_path_info ^((?U).+\.php)(/?.+)$;
```

4 隐藏入口文件 在location / {}中给出

```nginx
if (!-e $request_filename) 
{ 
	rewrite ^(.*)$ /index.php?s=/$1 last; break; 
}
```

5 php开启pathinfo模式 支持tp3 tp5等框架的pathinfo路由模式

修改php.ini配置文件中cgi.fix_pathinfo配置项cgi.fix_pathinfo=1

6 nginx 网站配置文件举例

```nginx
location / {
        if (!-e $request_filename){
            rewrite ^/(.*)$ /index.php/$1 last;
        }
}
location ~ \.php($|/){
set $script $uri;
set $path_info “”;
if ($uri ~ “^(.+?\.php)(/.+)$”) {
set $script $1;
set $path_info $2;
}
try_files $uri =404;
fastcgi_pass unix:/tmp/php-cgi.sock;
fastcgi_index index.php;
include fastcgi.conf;
fastcgi_param script_FILENAME $document_root$script;
fastcgi_param script_NAME $script;
fastcgi_param PATH_INFO $path_info;
}
```

最后附上一些linux常用的网站 linux命令查询网站 https://man.linuxde.net/ https://ipcmen.com/ 编程教程网站 https://www.yiibai.com/ https://www.w3cschool.cn/ 程序员集成工具站 https://tool.lu/