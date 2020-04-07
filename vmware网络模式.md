## vmware workstation网络环境配置

**Bridged(桥接模式)**

![image-20200401161800654](https://github.com/Neupres/linux-OM/blob/master/typora-user-images/image-20200401160749252.png)

> 物理主机网卡将与vmware模拟的虚拟交换机桥接 增加了网络终端虚拟机群 此时虚拟机群与主机 网卡处在同一lan子网段

**Nat(网络地址转换模式)**

![image-20200401161834663](https://github.com/Neupres/linux-OM/blob/master/typora-user-images/image-20200401161834663.png)

> vmware虚拟机创建虚拟网络适配器-网卡 此适配器将模拟网关向虚拟机提供地址 由虚拟nat网络 虚拟DHCP服务 和当前交换网络组成了虚拟lan网 虚拟lan网就是VMnet0的子网 VMnet0本身是模 拟网关 地址由自己决定

**Host-Only主机模式**

![image-20200401161902839](https://github.com/Neupres/linux-OM/blob/master/typora-user-images/image-20200401161902839.png)

> 虚拟机创建模拟适配器 没有nat网络 模拟适配器将提供网关向lan网服务 此时的虚拟lan网将无法 上网

**主机上网方法(windows网卡共享)**

设置位置 --控制面板-网络和Internet-网络连接 

将主机网络共享给虚拟适配器 

右键以太网(也就是主机网卡)选择属性 切换共享选项卡 复选 允许其他网络用户通过此计算机的Internet连接 来连接 选择VMnet0网络适配器 

source by wslin
