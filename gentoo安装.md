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


