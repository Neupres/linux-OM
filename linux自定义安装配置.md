### centos7最小化安装初始化配置

#### 网络配置

```bash
ip ad
```

网卡配置文件详解

```bash
配置文件位置 vi /etc/sysconfig/network-scripts/ifcfg-网卡名
配置文件添加标准：
TYPE=Ethernet //网络类型：Ethernet以太网
BOOTPROTO=none //引导协议：dhcp自动获取、static静态、none不指定
DEFROUTE=yes //启动默认路由
IPV4_FAILURE_FATAL=no //不启用IPV4错误检测功能
IPV6INIT=yes //启用IPV6协议
IPV6_AUTOCONF=yes //自动配置IPV6地址
IPV6_DEFROUTE=yes //启用IPV6默认路由
IPV6_FAILURE_FATAL=no //不启用IPV6错误检测功能
NAME=eno16777736 // 网卡设备的别名
UUID=90528772-9967-46da-b401-f82b64b4acbc //网卡设备的UUID唯一标识号
DEVICE=eno16777736 // 网卡的设备名称
ONBOOT=yes //开机自动激活网卡
DNS1=8.8.8.8 //DNS域名解析服务器的IP地址
PREFIX=24 //子网掩码位数
GATEWAY=192.168.1.1 //默认网关IP地址
IPADDR=192.168.2.2 #固定IP
NETMASK=255.255.255.0 #子网掩码，不需要修改；
GATEWAY=192.168.2.1 #这决定了你从那个网关上网
```

重启网络

```bash
service network restart
```

#### 软件包管理

查看软件包数量

```bash
rpm -qa | wc -l
```

查看已经安装的软件

```bash
rpm -qa | sort
```

查询是否安装了xxx软件

```bash
rpm -qa | grep xxx
```

#### 关闭selinux安全子系统

子系统是linux使用虚拟化技术模拟的系统 不会影响宿主机 

查看是否安装了selinux

```bash
getenforce
/usr/sbin/sestatus -v
```

临时关闭

```bash
setenforce 0 //下次重启系统selinux将会被启动
```

永久关闭

```bash
vi /etc/selinux/config
```

> 进入配置文件将SELINUX配置项值修改为disabled 重启系统生效

#### 配置软件镜像源

```bash
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

下载阿里镜像源文件

```bash
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos7.repo
```

可以选择安装扩展源

```bash
yum -y install epel-release
```

清除yum缓存并重新生成缓存

```bash
yum clean all && yum makecache
```

#### centos7 防火墙软件firewall配置

放通某个端口

```bash
firewall-cmd --permanent --zone=public --add-port=80/tcp
```

> 添加参数--permanent会使firewall永久生效 不添加重启系统或者防火墙将会失效

移除以上规则

```bash
firewall-cmd --permanent --zone=public --remove-port=80/tcp
```

放通某个端口段

```bash
firewall-cmd --permanent --zone=public --add-port=10000-20000/tcp
```

查看所有放通的端口

```bash
firewall-cmd --zone=public --list-ports
```

查看防火墙的配置

```bash
firewall-cmd --list-all
```

放通某个IP访问

```bash
firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address=192.168.1.169 accept'
```

移除以上规则

```bash
firewall-cmd --permanent --remove-rich-rule='rule family=ipv4 source address=192.168.1.169 accept'
```

放通某个IP段访问

```bash
firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address=192.168.2.0/24 accept'
```

禁止某个IP访问

```bash
firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address=192.168.1.169 drop'
```

放通某个IP访问某个端口

```bash
firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address=192.168.1.169 port protocol=tcp port=6379 accept'
```

重新加载防火墙配置

```bash
firewall-cmd --reload
```

#### 进程操作命令

查看正在执行的所有进程

```bash
ps [-aux] [| more]
```

> 其中，-a表示显示当前所有进程，-u表示以用户格式显示进程，-x表示显示进程运行的参数。 上 述三个 参数一般都是要使用的。如果想分页显示，后面加 | more。

查看xxx软件的进程号pid

```bash
ps -ef | grep xxx
```

> 返回 root 9630 1 0 14:55 ? 00:00:00 nginx: master process ./usr/sbin/nginx xxx 9631 9630 0 14:55 ? 00:00:00 nginx: worker process root 12113 8137 0 16:25 pts/0 00:00:00 grep -- color=auto nginx xxx的进程由root账户开启 pid为9631 父进程9630

根据命令查询特定的进程

```bash
ps -aux | grep instruction [| more]
```

> 其中，instruction为命令名称。例如，要查看正在执行的sshd进程，ps -aux | grep sshd。

查看进程及其父进程

```bash
ps -ef [| more]
```

> 该命令会以全格式显示当前所有进程，比上述命令多一行PPID，即父进程的id。 例如，要查询 sshd进 程的父进程，ps -ef | grep sshd

终止进程

```bash
kill [-9] pid
```

> 其中，pid为要终止的进程编号；-9表示强制终止，用于某些核心进程(例如终端bash)

批量终止进程

```bash
killall pname
```

> 其中，pname为进程名称，支持通配符，这在系统因执行多个同类型进程变慢时很有用

以树状结构显示当前进程 

```bash
pstree [-pu] 
```

其中，-p表示显示进程编号，-u表示显示进程所属的用户。如下图所示:

![image-20200401160705604](.\Typora\typora-user-images\image-20200401160705604.png)

动态监控进程

```bash
top [-i] [-d n] [-p pid]
```

> 交互操作：P(按占用cpu比例排序)、M(按占用内存排序)、N(按进程编号排序)、u(只监控某个用 户)、 k(结束进程)、q(退出)。 这个命令跟ps很相似，区别是可以定时(默认3秒)刷新，支持交互操 作。其中，-i表示只显示正在运行的 进程；-d用于指定刷新时间间隔(n秒)；-p用于指定进程编 号，只监控编号为pid的进程。

踢掉某个非法登录的用户

(1)查询正在执行的sshd进程：ps -aux | grep sshd。

![image-20200401160749252](.\Typora\typora-user-images\image-20200401160749252.png)

(2)将正在登录的root用户踢掉，即终止进程112920：kill 112920

(3)dubhlinn用户会被强制退出，并提示：Connection to xxx closed by remote host.

批量关闭已经打开的多个gedit编辑器

```bash
killall gedit
```

进程列表简介

通过ps -aux命令可以以列表形式显示当前进程的详细信息，那么这些列代表什么

![image-20200401160833782](.\Typora\typora-user-images\image-20200401160833782.png)

> USER：进程所属的用户名； PID：进程编号 %CPU：占用CPU的比例 %MEM：占用内存的比例 VSZ： 占用虚拟内存的大小 RSS：占用物理内存的大小 TTY：终端名称 STAT：进程状态，S为睡 眠，R为正在 执行，Z为僵死，D为短期等待，N表示优先级低于普通进程 START：进程的启动时 间 TIME：进程使用 CPU的总时间 COMMAND：进程的命令名称和参数

**端口操作命令**

centos7已经将netstat更改为ss

查看进程号pid所占用的端口

```bash
ss -lnp|grep pid
```

> 返回tcp LISTEN 0 128 :80 :* users:(("nginx",pid=9631,fd=6),("nginx",pid=9630,fd=6)) 当前端 口 为tcp连接 监听端口为80端口

#### 硬盘管理