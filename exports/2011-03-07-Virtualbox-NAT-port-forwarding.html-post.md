在VirtualBox NAT网络模式下面，Guest系统对Host系统和局域网内其他的机器都是不可见的，所以Guest上面的任何服务都不能被外界访问到。这样很多情况下，是不能接受的，比如想在Guest系统为Ubuntu的虚拟机上面开启SSH服务，怎么办呢？两个方法，1、把网络连接方式改成Bridge模式，这样虚拟机的Guest系统就有了自己的IP地址，相当于局域网内的一台主机，这样Host系统和局域网内的其他机器都可以访问他了。2、在NAT网络模式下，开启端口映射。需要什么服务就映射什么端口数据。今天我以VirtualBox的guest系统Ubuntu系统开启SSH服务为例来介绍怎么配置端口映射。

 首先介绍一下NAT网络模式下端口映射的优点，首先节省一个IP地址（有些情况下，IP地址资源比较宝贵）；其次，这样可以避免Server暴露过多的接口，提供什么服务暴露什么接口，这样保证Server安全性。当然这种方式也有一定的局限性，服务的端口必须是固定的，假如服务的端口是动态的，那么这种方式就没有办法了，比如NFS服务就不能用端口映射来实现。

 下面介绍怎样在VirtualBox的Guest系统Ubuntu中启用SSH服务，Host系统为Windows。SSH服务的端口是22端口，理论上可以把Host系统Windows的22端口映射到Ubuntu的22端口。这样做不好，假如有一天Host系统Windows也要在22端口上提供服务就没有办法了，所以我们准备用Host系统的2222端口，映射到Ubuntu的22端口。

#####1. 首先从命令行进入到VirtualBox安装目录。

#####2. 在安装目录下面，目录下面有VBoxManage.exe程序。如果你的VirtualBox版本是3.2之前的版本，请执行如下的命令：

```
VBoxManage setextradata "Linux Guest" "VBoxInternal/Devices/pcnet/0/LUN#0/Config/guestssh/Protocol" TCP

VBoxManage setextradata "Linux Guest" "VBoxInternal/Devices/pcnet/0/LUN#0/Config/guestssh/GuestPort" 22

VBoxManage setextradata "Linux Guest" "VBoxInternal/Devices/pcnet/0/LUN#0/Config/guestssh/HostPort" 2222
```
其中“Linux Guest”是你虚拟机Ubuntu系统的名称，pcnet表示Ubuntu选的是PCNet的网卡，0表示第一块网卡，guestssh是自定义的名字。TCP表示只映射TCP协议的数据，当然也可以设置成UDP协议数据映射的。

通过以上命令就可以完成Host Windows系统2222端口到Guest Ubuntu系统的端口映射了。当然需要三个命令来完成端口映射挺麻烦的，Oracle公司也这样认为。在最新的3.2版本中，VirtualBox简化了端口映射的步骤，只要一条命令就可以了。
``VBoxManage modifyvm "VM name" --natpf1 "guestssh,tcp,,2222,,22"``
“VM name”就是Guest系统的名字，guestssh还是用户自定义的名字。你可能发现第三个参数和第五个参数空白了，是什么意思呢？当Host系统有多块网卡的时候，通过第三个参数指定那款网卡的2222端口映射；如果Host系统有多块网卡时，通过第五个参数指定那个网卡的22端口接收数据。
``VBoxManage modifyvm "VM name" --natpf1 "guestssh,tcp,,2222,10.0.2.19,22"``
以上的命令式将Host系统2222端口的数据映射到Guest系统的10.0.2.10网卡的22端口。

#####3. 已经设置好了端口映射，现在登录到Ubuntu中，安装SSH server。

sudo apt-get install openssh-server4、在Windows平台下面用ssh工具，通过2222端口登录到Ubuntu就可以了。也可以从局域网内的机器上，用SSH工具，通过Host机器的2222端口登录到Ubuntu的ssh服务。

至此，Ubuntu的SSH服务已经暴露在局域网了。通过上面的描述可以看到在VirtualBox3.2版本中端口映射已经做得比较简单，并且非常完善了。

如有错误欢迎指正。