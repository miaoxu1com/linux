安装手册/指导书
https://wiki.gentoo.org/wiki/Handbook:AMD64/Full/Installation/zh-cn
虚拟机安装  可以跳过引导盘制作、镜像校验、定制启动、启动参数相当于window安全启动或者高级启动
adsl  wep  wifi 网络配置也可以跳过  因为虚拟机一般都是DHCP自动获取、网络代理也无需配置

使用nat网络方式上网，需要把已经配置了外网的物理网卡共享给虚拟网卡vmnat8，注意不要把共享网络配反了，是把物理公网共享给虚拟网络，然后继续配置vmnat8 ip为vmnat8子网ip  
例如：
192.168.x.2
255.255.255.0
192.168.x.1 //这里是vmnat的网关地址
具体过程参考
https://blog.csdn.net/hhc550056259/article/details/123916999

然后参考官方文档手动配置网络
检查 ip route 默认网关是否和配置的vmnat8一致
不一致使用命令改为一致
ip route add default via 192.168.0.1

查看ip地址和vmnat8网段一致
ip addr 显示ip地址

ip link 查看网卡接口，不显示ip地址，只显示网络接口


ping  114.114.114.114 有时ping不同可以多一个223.5.5.5
ping 223.5.5.5

然后配置nameserver 223.5.5.5  
/etc/resolv.conf


系统安装的大致过程都一样

启动到live系统
root登录,live系统都会有提示语
修改root密码
passwd root
开启ssh
修改sshd配置
PermitRootLogin yes
启动ssh服务
rc-service sshd start

分区
常用分区工具
cfdisk 最简单
fdisk 纯命令行

常用的分区方案
swap+efi+root
格式化分区
mkfs.ext4  /dev/sda root
mkfs.vfat -F 32 /dev/sda  efi
mkswap /dev/sda

挂载系统
创建目标系统目录
mkdir --parents /mnt/gentoo  根目录
mkdir --parents /mnt/gentoo/efi
在live系统中挂载系统到目标 系统目录
mount /mnt/gentoo /dev/sda
mount /mnt/gentoo/efi /dev/sda
安装初始化程序stage3到目标系统
cd /mnt/gentoo
tar -xvf stage*.tar
使用默认配置安装

切换到目标系统编译安装内核, 进行预安装工作


