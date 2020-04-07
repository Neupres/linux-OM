#### docker虚拟容器技术--centos7-docker

**虚拟化层次与机制**

现代虚拟技术层次及对例

| 层次               | 例子            |
| ------------------ | --------------- |
| 应用程序级         | JVM \| .Net CLR |
| API库级虚拟        | WINE            |
| 操作系统级         | Docker          |
| 硬件抽象级         | VMware \| VMbox |
| 硬件指令体系结构集 | Bochs           |

> JVM为java应用程序虚拟机，使java实现了一份源码多平台编译
> WINE例如linux模拟调用微软桌面API库
> VMware软件模拟硬件环境
> Bochs可以再类unix系统上x86平台指令集、I/O、内存、BIOS等模拟

**docker虚拟技术原理**

Docker虚拟容器在系统进程中开启一个进程，该进程被称之为**容器**，所有容器通过docker镜像创建，并由docker守护进程调配管理，容器模拟了一个任意伪系统环境，这样使得我们可以在任意系统中开启docker守护进程创建管理不同于该任意系统的其他系统容器，可以使用docker守护进程将容器搬移到任意地方中继续运行，只要任意地方能运行docker并开启docker守护进程。docker容器作为一个进程消耗的性能更低，功能虽然比不上硬件抽象级多，因为docker虚拟面向运维，docker还提供了多种 容器对容器及容器对宿主机的网络交互功能，容器之间除开通信完全隔离，所以docker容器面向的运维是即时，通用，快捷，灵活，互通，安全的。

docker创建容器使用的文件叫**镜像**，创建镜像的方式有：一拉取官方镜像创建容器，进入容器操作配置完成后使用docker守护进程命令将当前容器制作成镜像。二是创建dockerfile文件，使用docker命令执行dockerfile文件生成docker镜像。第一种创建的docker镜像没有创建步骤历史版本，不方便于创建后修改，而后者使用dockerfile创建的镜像具有。

docker线上镜像管理系统被称为**仓库**，docker官方仓库为[docker hub](https://hub.docker.com/)。

**centos7-docker虚拟技术命令**

centos7安装docker环境

查看镜像源是否有dockery ||安装docker

```bash
yum list docker || yum install docker-ce-版本号
```

docker帮助 || docker命令帮助

```bash
docker -h || docker command --help
```

**镜像操作**

搜索镜像

```bash
docker search [OPTIONS] 镜像名	#镜像名被正则匹配
```

> OPTIONS说明：--automated :只列出 automated build类型的镜像；-no-trunc :显示完整的镜像描述；-s :列出收藏数不小于指定值的镜像。

拉取镜像

在docker中  凡是指定了镜像的地方 docker将会在本地查找 找不到就会在hub上找    将我们上传的镜像通过tag后的地址拉取下来

```bash
docker pull registry.cn-hongkong.aliyuncs.com/neupres/镜像名:[镜像版本号]
```

**创建镜像的方法**

镜像创建方式有2种

一是从已有或则docker hub上的基础镜像container使用docker commit制作

使用基础镜像创建一个容器centos并进入

```bash
docker run -it centos
```

进入后进行操作退出  退出后就得到了我们更改过的镜像容器 使用

```bash
docker commit -a "镜像作者" -m "镜像描述" -c "dockerfile文件路径" -P 容器ID 创建后的镜像名:镜像版本
```

> 其中指定-c参数将基于指定容器开始使用dockerfile文件制作镜像  -P为在制作时将容器暂停

二是从dockerfile制作 创建一个dockerfile文件使用docker build -f /dockerfilepath命令制作

dockerfile文件格式为一行一个镜像版本 最开始使用FROM 指定基础镜像 每执行一行dockerfile就会生成一个镜像版本 最后镜像制作完成后 制作的镜像成品包含所有版本即镜像缓存 可以查看以及回退到指定镜像版本

**dockerfile指令大全**

```dockerfile
#为文件的注释   dockerfile文件的格式为 指令 指令值
#当一行指令太长可以使用 \ 转义换行符
FROM：指定基础镜像，必须为第一个命令
#格式FROM <image> FROM <image>:<tag> FROM <image>:<digest> 
MAINTAINER: 维护者信息
#格式 MAINTAINER <name>
RUN：构建镜像时执行的命令
# RUN <command>通过对应的操作系统shell命令工具执行<command> 直接跟命令
# RUN ["executable", "param1", "param2"] executable为可执行文件 param为参数
ADD：将本地文件添加到容器中，tar类型文件会自动解压(网络压缩资源不会被解压)，可以访问网络资源，类似wget
#格式 ADD <src>... <dest> 文件路径支持文件路径通配符
#ADD ["<src>",... "<dest>"]多个src文件或目录到dest目标位置 
COPY：将主机文件复制到容器中，但是是不会自动解压文件，也不能访问网络资源
#格式 同ADD
CMD：构建容器后调用，也就是在容器启动时才进行调用。
#格式 CMD ["executable","param1","param2"] (执行可执行文件，优先)
#    CMD ["param1","param2"] (设置了ENTRYPOINT，则直接调用ENTRYPOINT添加参数)
#    CMD command param1 param2 (执行shell内部命令)
ENTRYPOINT：配置容器，使其可执行化。配合CMD可省去"application"，只使用参数。
#格式 ENTRYPOINT ["executable", "param1", "param2"] (可执行文件, 优先)
#    ENTRYPOINT command param1 param2 (shell内部命令)
#    ENTRYPOINT与CMD非常类似，不同的是通过docker run执行的命令不会覆盖ENTRYPOINT，而docker run命令中指定的任何参数，都会被当做参数再次传递给ENTRYPOINT。Dockerfile中只允许有一个ENTRYPOINT命令，多指定时会覆盖前面的设置，而只执行最后的ENTRYPOINT指令。
LABEL：用于为镜像添加元数据
#格式 LABEL <key>=<value> <key>=<value> <key>=<value> ... eg:LABEL version="1.0" description="这是一个Web服务器" by="IT笔录"
ENV：设置环境变量
#格式 ENV <key> <value>  or  ENV <key>=<value> ...
EXPOSE：指定于外界交互的端口
#格式 EXPOSE <port> [<port>...]
VOLUME：用于指定持久化目录
#格式 VOLUME ["/path/to/dir"] 一个卷可以存在于一个或多个容器的指定目录，该目录可以绕过联合文件系统，并具有以下功能：
#1 卷可以容器间共享和重用 2 容器并不一定要和其它容器共享卷 3 修改卷后会立即生效 4 对卷的修改不会对镜像产生影响 5 卷会一直存在，直到没有任何容器在使用它
WORKDIR：工作目录，类似于cd命令
#格式 WORKDIR /path/to/workdir 设置当前工作位置
USER:指定运行容器时的用户名或 UID，后续的 RUN 等也会使用指定用户。使用USER指定用户时，可以使用用户名、UID或GID，或是两者的组合
#格式 USER user|USER user:group|USER uid|USER uid:gid|USER user:gid|USER uid:group
ARG：用于指定传递给构建运行时的变量
#格式 ARG <name>[=<default value>]
ONBUILD：用于设置镜像触发器 
#格式 ONBUILD [上述的dockerfile指令]  当镜像构建后 此镜像被再次FROM为其他基础镜像时会被钥触发
```

当文件创建完成后使用以下语句创建镜像

```bash
docker build [OPTIONS] 
```

> OPTIONS说明：--build-arg=[] :设置镜像创建时的变量；--cpu-shares :设置 cpu 使用权重；--cpu-period :限制 CPU CFS周期；--cpu-quota :限制 CPU CFS配额；--cpuset-cpus :指定使用的CPU id；--cpuset-mems :指定使用的内存 id；--disable-content-trust :忽略校验，默认开启；-f :指定要使用的Dockerfile路径；--force-rm :设置镜像过程中删除中间容器；--isolation :使用容器隔离技术；--label=[] :设置镜像使用的元数据；-m :设置内存最大值；--memory-swap :设置Swap的最大值为内存+swap，"-1"表示不限swap；--no-cache :创建镜像的过程不使用缓存；--pull :尝试去更新镜像的新版本；--quiet, -q :安静模式，成功后只输出镜像 ID；--rm :设置镜像成功后删除中间容器；--shm-size :设置/dev/shm的大小，默认值是64M；--ulimit :Ulimit配置。--tag, -t: 镜像的名字及标签，通常 name:tag 或者 name 格式；可以在一次构建中为一个镜像设置多个标签。--network: 默认 default。在构建期间设置RUN指令的网络模式

当镜像创建完成后我们可以使用下面语句查看镜像结构

```bash
docker history [OPTIONS] nginx:latest 
```

> OPTIONS说明：-H :以可读的格式打印镜像大小和日期，默认为true；--no-trunc :显示完整的提交记录；-q :仅列出提交记录ID。

查看当前本地的镜像

```bash
docker images [OPTIONS] [REPOSITORY[:TAG]]
```

> OPTIONS说明：-a :列出本地所有的镜像（含中间映像层，默认情况下，过滤掉中间映像层） --digests :显示镜像的摘要信息；-f :显示满足条件的镜像；--format :指定返回值的模板文件；--no-trunc :显示完整的镜像信息；-q :只显示镜像ID。

删除一个或多个镜像

```docker
docker rmi [OPTIONS] IMAGE [IMAGE...]
```

> OPTIONS说明：-f :强制删除；--no-prune :不移除该镜像的过程镜像，默认移除；

将指定镜像保存成 tar 归档文件

```bash
docker save [OPTIONS] IMAGE [IMAGE...]
```

> OPTIONS 说明：-o :输出到的文件。

从归档文件中创建镜像

```bash
docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
```

> OPTIONS说明：-c :应用docker 指令创建镜像；-m :提交时的说明文字；

**容器操作**

列出容器

```bash
docker ps [OPTIONS]
```

> OPTIONS说明：-a :显示所有的容器，包括未运行的。-f :根据条件过滤显示的内容。--format :指定返回值的模板文件。-l :显示最近创建的容器。-n :列出最近创建的n个容器。--no-trunc :不截断输出。-q :静默模式，只显示容器编号。-s :显示总的文件大小。

 获取容器/镜像的元数据

```bash
docker inspect [OPTIONS] NAME|ID [NAME|ID...]
```

> OPTIONS说明：-f :指定返回值的模板文件。-s :显示总的文件大小。--type :为指定类型返回JSON。

查看容器中运行的进程信息，添加 ps 命令参数

```bash
docker top [OPTIONS] CONTAINER [ps OPTIONS]
```

> 容器运行时不一定有/bin/bash终端来交互执行top命令，而且容器还不一定有top命令，可以使用docker top来实现查看container中正在运行的进程。

连接到正在运行中的容器

```bash
docker attach [OPTIONS] CONTAINER
```

从服务器获取实时事件

```bash
docker events [OPTIONS]
```

> OPTIONS说明：-f ：根据条件过滤事件；--since ：从指定的时间戳后显示所有事件;--until ：流水时间显示到指定的时间为止；

阻塞运行直到容器停止，然后打印出它的退出代码

```bash
docker wait [OPTIONS] CONTAINER [CONTAINER...]
```

将文件系统作为一个tar归档文件导出到STDOUT

```bash
docker export [OPTIONS] CONTAINER
```

> OPTIONS说明：-o :将输入内容写到文件。

列出指定的容器的端口映射，或者查找将PRIVATE_PORT NAT到面向公众的端口

```bash
docker port [OPTIONS] CONTAINER [PRIVATE_PORT[/PROTO]]
```

创建一个新的容器并运行一个命令

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

> OPTIONS说明：-a stdin: 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项；-d: 后台运行容器，并返回容器ID；-i: 以交互模式运行容器，通常与 -t 同时使用；-P: 随机端口映射，容器内部端口随机映射到主机的高端口-p: 指定端口映射，格式为：主机(宿主)端口:容器端口-t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用；--name="nginx-lb": 为容器指定一个名称；--dns 8.8.8.8: 指定容器使用的DNS服务器，默认和宿主一致；--dns-search example.com: 指定容器DNS搜索域名，默认和宿主一致；-h "mars": 指定容器的hostname；-e username="ritchie": 设置环境变量；--env-file=[]: 从指定文件读入环境变量；--cpuset="0-2" or --cpuset="0,1,2": 绑定容器到指定CPU运行；-m :设置容器使用内存最大值；--net="bridge": 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；--link=[]: 添加链接到另一个容器；--expose=[]: 开放一个端口或一组端口；--volume , -v: 绑定一个卷

容器控制 重启|停止|启动

```bash
docker restart|stop|start [OPTIONS] CONTAINER [CONTAINER...]
```

杀掉一个运行中的容器

```bash
docker kill [OPTIONS] CONTAINER [CONTAINER...]
```

> OPTIONS说明：-s :向容器发送一个信号 杀掉容器是发送-s可以让容器被杀前指定命令

删除一个或多个容器

```bash
docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

> OPTIONS说明：-f :通过 SIGKILL 信号强制删除一个运行中的容器。-l :移除容器间的网络连接，而非容器本身。-v :删除与容器关联的卷。

暂停|解除暂停容器进程

```
docker pause|unpause [OPTIONS] CONTAINER [CONTAINER...]
```

创建一个新的容器但不启动它

```bash
docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
```

> 语法同run

在运行的容器中执行命令

```bash
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

> OPTIONS说明：-d :分离模式: 在后台运行-i :即使没有附加也保持STDIN 打开-t :分配一个伪终端

容器与主机之间的数据拷贝

```bash
docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH
```

> OPTIONS说明 -L :保持源目标中的链接

检查容器里文件结构的更改

```bash
docker diff [OPTIONS] CONTAINER
```

**从阿里云仓库拉取镜像**

**使用阿里云docker镜像加速**

登录阿里云 开通容器镜像服务后点击镜像加速器通过引导写入daemon配置文件/etc/docker/daemon.json 的"registry-mirrors"选项中

**登录到例如阿里云的远程镜像库**

```bash
sudo docker login --username=w3237877566 registry.cn-hongkong.aliyuncs.com
```

用于登录的用户名为阿里云账号全名，密码为开通服务时设置的密码。

您可以在访问凭证页面修改凭证密码。

**从Registry中拉取镜像**

```bash
sudo docker pull registry.cn-hongkong.aliyuncs.com/neupres/镜像名:[镜像版本号]
```

镜像版本号不提供默认为latest最新版

**将镜像推送到阿里云容器镜像服务**

```bash
sudo docker login --username=w3237877566 registry.cn-hongkong.aliyuncs.com
sudo docker tag [ImageId] registry.cn-hongkong.aliyuncs.com/neupres/镜像名:[镜像版本号]
sudo docker push registry.cn-hongkong.aliyuncs.com/neupres/镜像名:[镜像版本号]
```

> docker tag是为了将镜像打上适合阿里云容器镜像仓库的标签 其中[ImageId]为本地镜像id  registry.cn-hongkong.aliyuncs.com指向了阿里云香港容器镜像服务  neupres为你创建的镜像空间   镜像名:[镜像版本号]上传后保存的镜像名于版本号  最后这个docker镜像内会多一个打了标签的镜像  再将此镜像push推送到阿里云

#### 实例：使用docker容器php安装composer搭建php  laravel框架

使用官方或者自己制作的php镜像创建并运行一个名叫php的容器

```bash
docker run -d -h php --name php -v /home/nginx/wwwroot:/usr/local/nginx/html php:latest 
```

> 挂载nginx的html网站根目录  可以创建一个nginx link到php也挂载网站根目录到宿主机  宿主机的wwwroot成为网站根目录 并映射到了2个容器中

进入创建的容器

```bash
docker exec -it php /bin/bash
```

创建php系统环境变量

```bash
PATH+=$PATH:/usr/local/php/bin
export PATH
php -v
```

根据官网安装composer 

```bash
php -r "copy('https://install.phpcomposer.com/installer', 'composer-setup.php');"
```

```bash
php composer-setup.php
```

```bash
php -r "unlink('composer-setup.php');"
```

将下载下来的composer.phar可执行php扩展更名移动到php可执行文件夹中 

```bash
mv composer.phar /usr/local/php/bin/composer
```

设置国内镜像

```bash
composer config -g repo.packagist composer https://packagist.phpcomposer.com
```

安装git

```bash
yum -y install git
```

根据laravel官网安装laravel

```bash
composer create-project --prefer-dist laravel/laravel "6.*"
```

当前composer将从github安装laravel  安装laravel中会要求输入github的token 可以在github个人设置中创建一个token并设置权限  如果当前laravel使用git开发  就要合适的设置token权限   最终/usr/local/nginx/html下就会生成laravel框架

配置nginx容器nginx服务配置文件运行laravel环境

```nginx
server {
        listen 80;
        server_name 127.0.0.1;
        root html/public;

        add_header X-Frame-Options "SAMEORIGIN";
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Content-Type-Options "nosniff";

        index index.html index.htm index.php;

        charset utf-8;

        location / {
           #rewrite  ^(.*)$  /index.php?s=/$1  last;
           try_files $uri $uri/ /index.php?$query_string;
        }

        location = /favicon.ico { access_log off; log_not_found off; }
        location = /robots.txt  { access_log off; log_not_found off; }

        error_page 404 /index.php;

        location ~ \.php$ {
                fastcgi_pass php-cgi:9000;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
                include fastcgi_params;
        }

        location ~ /\.(?!well-known).* {
            deny all;
        }



}


```

由于我们将网站根目录挂载到了宿主机 我们需要在宿主机进入挂载的网站根目录修改权限

```bash
chmod -R 777 storage
```

#### 实例：docker自建仓库(htpasswd 认证)

1，拉取docker registry 镜像

```bash
docker pull registry
```

 2，创建证书存放目录 

```bash
mkdir -p /home/registry/certs
```

3，生成CA证书 Edit your /etc/ssl/openssl.cnf on the logstash host - add subjectAltName = IP:内网的IP地址 in [v3_ca] section. 一般情况下，证书只支持域名访问，要使其支持IP地址访问，需要修改配置文件openssl.cnf。 在redhat7系统中，openssl.cnf文件所在位置是/etc/pki/tls/openssl.cnf。在其中的[ v3_ca]部分， 添加subjectAltName选项：

```vi
[ v3_ca ] 
subjectAltName = IP:外网的IP地址
```

生成证书

```bash
openssl req -newkey rsa:4096 -nodes -sha256 \
-keyout /home/registry/certs/domain.key -x509 \
-days 365 -out /home/registry/certs/domain.crt
```

注意Common Name最好写为registry的域名

修改权限，并将认证文件添加到(客户端) /etc/docker/certs.d/内网的IP地址:5000/

```bash
chcon -Rt svirt_sandbox_file_t /home/registry/certs mkdir -p /etc/docker/certs.d/IP地址:5000/ cp /home/registry/certs/domain.crt /etc/docker/certs.d/内网的IP地址:5000/ca.crt
```

 3，使用registry镜像生成用户名和密码文件

```bash
mkdir /home/registry/auth cd /home/registry/auth echo "user:账号 passwd:密码" >htpasswd //htpasswd可以修改但是后面语句上都要修改 docker run --entrypoint htpasswd registry -Bbn 账号 密码 > /home/registry/auth/htpasswd chcon -Rt svirt_sandbox_file_t /home/registry/
```

4，运行registry并指定参数。包括了用户密码文件和CA书位置。--restart=always 始终自动重启

```bash
docker run -d -p 5000:5000 --restart=always --name registry \ -v /home/registry/auth:/auth \ -e "REGISTRY_AUTH=htpasswd" \ -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \ -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \ -v /home/registry/certs:/certs \ -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \ -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \ registry
```

5，登陆和登出 ###vim /etc/hosts

```bash
docker login 内网的IP地址:5000 -u uesr -p password docker logout 内网的IP地址:5000
```

6，添加用户

```bash
docker run --entrypoint htpasswd registry -Bbn Dapeng 123456 >> /home/registry/auth/htpasswd 
docker run --entrypoint htpasswd registry -Bbn user123 passwd123 >> /home/registry/auth/htpasswd
```

无需执行 docker restart registry 

上传镜像到仓库并从局域网内其它docker连接下载 

vi /etc/docker/daemon.json

{ "registry-mirrors": [""], #镜像加速地址 

"insecure-registries": ["内网的IP地址","等等非https的仓库"], # Docker如果需要从非https源管理镜 像，这里加上。 }

#### 实例：使用docker-compose文件搭建dnmp 

步骤大概 ：： 制作nginx mysql php的镜像

制作完成后使用docker compose 的compose.yml文件为3个镜像创建容器 并挂载卷到宿主机 并与宿主 机相互连接 最终形成一个 docker + nginx + mysql + php-fpm的环境运行web程序 

相对于制作镜像再写compose.yml文件创建docker容器运行环境 ， docker 分布式架构的优点是能通 用 这是分布式架构的特点 如果没有特殊情况 使用互联网带来的compose.yml文件创建容器执行 dnmp环境是最佳快速方案

分布式架构使用docker容器构建了环境后 ，向其他机器部署环境直接使用镜像就好 也可以上传到之前 搭建的docker仓库中 

安装docker compose

```bash
sudo curl -L "https://get.daocloud.io/docker/compose/releases/download/1.24.1/dockercompose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

如有需要，修改上面 1.24.1 为指定版本号即可 

安装完后执行：

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

文件结构

下载对应的环境源码

```bash
wget http://nginx.org/download/nginx-1.16.1.tar.gz 
wget https://libzip.org/download/libzip-1.2.0.tar.gz
wget https://www.php.net/distributions/php-7.3.9.tar.gz
```

![image-20200401150204912](.\Typora\typora-user-images\image-20200401150204912.png)

创建上图中的目录结构

compose部署文件

```dockerfile
version: '3'
services:
    nginx:
        hostname: nginx
        build:
            context: ./nginx
            dockerfile: Dockerfile
        ports:
        	- "80:80"
        links:
        	- php:php-cgi
        volumes:
        	- ./wwwroot:/usr/local/nginx/html
    php:
        hostname: php
        build: ./php
        links:
        	- mysql:mysql-db
        volumes:
        	- ./wwwroot:/usr/local/nginx/html
    mysql:
        hostname: mysql
        image: mysql:5.7
        ports:
        	- "3306:3306"
        volumes:
            - ./mysql/conf:/etc/mysql/conf.d
            - ./mysql/data:/var/lib/mysql
        environment:
            MYSQL_ROOT_PASSWORD: 1qaz!QAZ
            MYSQL_DATABASE: wslin
            MYSQL_USER: wslin
            MYSQL_PASSWORD: 1qaz!QAZ
```

其中hostname为容器主机名 build为创建方式 port为映射端口 link为连接的其他容器 columes 为挂载容器的文件 image为创建容器的镜像 environment容器创建结束cmd的命令

 PS:links php:php-cgi意思是链接到服务名为php的服务，可以使用host名 php-cgi访问该容器，在启动 容器后进入容器ping

mysql/conf/my.cnf

```conf
[mysqld]
user=mysql
port=3306
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
pid-file=/var/run/mysqld/mysql.pid
log_error=/var/log/mysql/error.log
character_set_server = utf8
max_connections=3600
```

nginx/Dockerfile

```dockerfile
FROM centos:7
MAINTAINER wslin
RUN yum install -y gcc-c++ zlib-devel pcre-devel make
ADD nginx-1.16.1.tar.gz /tmp
RUN cd /tmp/nginx-1.16.1 && ./configure --prefix=/usr/local/nginx && make -j 2
&& make install
RUN rm -f /usr/local/nginx/conf/nginx.conf
COPY nginx.conf /usr/local/nginx/conf
RUN chmod -R 777 /usr/local/nginx/html
EXPOSE 80
CMD ["/usr/local/nginx/sbin/nginx", "-g", "daemon off;"]
```

nginx/nginx.conf

```nginx
user root;
worker_processes auto;
error_log logs/error.log info;
pid logs/nginx.pid;
events {
	use epoll;
}
http {
    include mime.types;
    default_type application/octet-stream;
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
    access_log logs/access.log main;
    sendfile on;
    keepalive_timeout 65;
    server {
        listen 80;
        server_name localhost;
        root html;
        index index.html index.php;
        location ~ \.php$ {
            root html;
            fastcgi_pass php-cgi:9000;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
        }
    }
}

```

php/Dockerfile

```dockerfile
FROM centos:7
MAINTAINER wslin
RUN yum -y install epel-release gcc gcc-c++
RUN yum install -y libxml2 libxml2-devel openssl openssl-devel bzip2 bzip2-devel
libcurl libcurl-devel libjpeg libjpeg-devel libpng libpng-devel freetype
freetype-devel gmp gmp-devel libmcrypt libmcrypt-devel readline readline-devel
libxslt libxslt-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel
ncurses curl gdbm-devel db4-devel libXpm-devel libX11-devel gd-devel gmp-devel
expat-devel xmlrpc-c xmlrpc-c-devel libicu-devel libmcrypt-devel
libmemcacheddevel libsqlite3x-devel oniguruma-devel make perl
ADD libzip-1.2.0.tar.gz /tmp
RUN cd /tmp/libzip-1.2.0 && \
./configure && \
make && \
make install
ADD php-7.3.9.tar.gz /tmp
RUN echo "/usr/local/lib64">>/etc/ld.so.conf && \
echo "/usr/local/lib">>/etc/ld.so.conf && \
echo "/usr/lib">>/etc/ld.so.conf && \
echo "/usr/lib64">>/etc/ld.so.conf && \
ldconfig -v
RUN cd /tmp/php-7.3.9 && \
./configure --prefix=/usr/local/php --with-config-filepath=/usr/local/php/etc --with-curl --with-freetype-dir --enable-gd --withgettext --with-iconv-dir --with-kerberos --with-libdir=lib64 --with-libxml-dir -
-with-mysqli --with-openssl --with-pcre-regex --with-pdo-mysql --with-pdo-sqlite
--with-pear --with-png-dir --with-jpeg-dir --with-xmlrpc --with-xsl --with-zlib
--with-bz2 --with-mhash --enable-fpm --enable-bcmath --enable-libxml --enableinline-optimization --enable-mbregex --enable-mbstring --enable-opcache --
enable-pcntl --enable-shmop --enable-soap --enable-sockets --enable-sysvsem --
enable-sysvshm --enable-xml --enable-zip --enable-fpm && \
cp /usr/local/lib/libzip/include/zipconf.h /usr/local/include/ && \
make -j 4 && make install && \
cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
&& \
cp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/phpfpm.d/www.conf && \
sed -i "s/127.0.0.1/0.0.0.0/g" /usr/local/php/etc/php-fpm.d/www.conf && \
cp ./sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm && \
chmod +x /etc/init.d/php-fpm
#COPY php.ini /usr/local/php/etc
EXPOSE 9000
CMD /etc/init.d/php-fpm start && tail -F /var/log/messages
```

PS：1，CentOS7自带libzip版本较低需要下载编译安装 

​        2，需要把默认配置文件www.conf.default重命名为www.conf并且修改配置把127.0.0.1改成 0.0.0.0否则会导致无法访问php页面，因为启动了不同的容器无法通过127.0.0.1访问 

​        3，配置文件php.ini可以不拷贝 因为centos7selinux安全子系统问题 将所有相关的挂载目录权限开启 添加selinux规则 按照上面挂载的文件夹位置为./wwwroot 安装脚本当前位置的wwwroot以及mysql的 conf和data文件夹

```bash
chcon -Rt svirt_sandbox_file_t 挂载的文件夹位置
```

运行docker compose

```bash
docker-compose up -d
```
#### docker问题 

**此docker问题为容器启动问题**

改变storage driver类型， 禁用容器的selinux

```bash
systemctl stop docker
```

清理镜像

```bash
rm -rf /var/lib/docker
```

修改存储类型

```bash
vi /etc/sysconfig/docker-storage
```

把空的DOCKER_STORAGE_OPTIONS参数改为overlay:

```vim
DOCKER_STORAGE_OPTIONS="--storage-driver overlay"
```

禁用selinux

```bash
vi /etc/sysconfig/docker
```

去掉option的–selinux-enabled 启动docker可以了

```bash
systemctl start docker
```
