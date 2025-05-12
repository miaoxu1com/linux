## 安装手册/指导书

https://wiki.gentoo.org/wiki/Handbook:AMD64/Full/Installation/zh-cn

虚拟机安装  可以跳过引导盘制作、镜像校验、定制启动、用户添加（默认都是用root更方便）、启动参数相当于window安全启动或者高级启动

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

```
ip route add default via 192.168.0.1
```

查看ip地址和vmnat8网段一致

ip addr 显示ip地址

ip link 查看网卡接口，不显示ip地址，只显示网络接口

```
ping  114.114.114.114 
```

有时ping不同可以多一个223.5.5.5

```
ping 223.5.5.5
```



然后配置nameserver 223.5.5.5  
/etc/resolv.conf

系统安装的大致过程都一样

启动到live系统 live是带gui的 桌面系统  minalios 是最小化的在线安装

root登录,live系统都会有提示语

修改root密码

```
passwd root
```

开启ssh

修改sshd配置

```
PermitRootLogin yes
```

启动ssh服务

```
rc-service sshd start
```

分区并格式化文件系统,分区按照官方分区方案进行分区

常用分区工具

cfdisk 最简单

fdisk 纯命令行

常用的分区方案

EFI 系统分区文件系统

EFI 系统分区（/dev/sda1）必须是 FAT32 格式：


swap+efi+root

格式化分区

```
mkfs.xfs  /dev/sda3 root
mkfs.vfat -F 32 /dev/sda1
mkswap /dev/sda2  
```

//格式化不需要挂载 激活即可

为分区应用文件系统
附注
安装结束后，请确保在重新启动之前，为之后在手册中选择的文件系统 emerge 相应的用户空间组件软件包。在接近安装尾声时您将看到另一个提醒。
在一个分区或卷上创建一个文件系统，这里有用于每一个可能的分区的工具。 单击下表中的文件系统名称，了解每个文件系统的更多信息：

文件系统	创建命令	是否包含在live环境中？	软件包
btrfs	mkfs.btrfs	 是	sys-fs/btrfs-progs
ext4	mkfs.ext4	 是	sys-fs/e2fsprogs
f2fs	mkfs.f2fs	 是	sys-fs/f2fs-tools
xfs	mkfs.xfs	 是	sys-fs/xfsprogs
vfat	mkfs.vfat	 是	sys-fs/dosfstools
NTFS	mkfs.ntfs	 是	sys-fs/ntfs3g

挂载系统
创建目标系统目录

```
mkdir --parents /mnt/gentoo  
```

根目录

```
mkdir --parents /mnt/gentoo/efi
```

在live系统中挂载系统到目标 系统目录 sda3 是根目录分区 sda1是efi分区 挂载到对应的目录 

```
mount /mnt/gentoo /dev/sda3
```

```
mount /mnt/gentoo/efi /dev/sda1
```

安装初始化程序stage3到目标系统

```
cd /mnt/gentoo
tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
```

使用默认配置安装

切换到目标系统编译安装内核, 进行预安装工作
open-rc 和 system  是用来管理服务
重启服务

```
systemctl  restart sshd
rc-service sshd start
```

习惯那里一个就用那个

```
cd /mnt/gentoo 
```

从https://www.gentoo.org/downloads/#other-arches 获取sate下载连接wget 下载连接进行下载
跳过stage文件校验
stage 文件就是完整的系统文件，在gentoo下解压缩后
stage配置编译选项默认即可


### portage配置make.config：

```shell
  COMMON_FLAGS="-march=native -O2 -pipe"
  MAKEOPTS="-j12"
  GENTOO_MIRRORS="https://mirrors.ustc.edu.cn/gentoo/"
  USE="-gtk -gnome qt6 qt5 gtk4 gtk3 gtk2 kde alsa X wayland vulkan fcitx dist-kernel dbus"
  VIDEO_CARDS="amdgpu radeonsi"
  ACCEPT_LICENSE="@FREE @BINARY-REDISTRIBUTABLE @EULA"
  LINGUAS="en en_US zh zh_CN"
  L10N="en en-US zh zh-Hans zh-Hans-CN zh-CN"
  GRUB_PLATFORMS="efi-64"
  Appending getbinpkg to the list of values within the FEATURES variable
  FEATURES="${FEATURES} getbinpkg"
  Require signatures
  FEATURES="${FEATURES} binpkg-request-signature"


  INPUT_METHOD=fcitx5
  XIM=fcitx5
  XIM_PROGRAM=fcitx5
  GTK_IM_MODULE=fcitx5
  QT_IM_MODULE=fcitx5
  XMODIFIERS=@im=fcitx5
  SDL_IM_MODULE=fcitx5
```

```shell
# /usr/share/portage/config/make.conf.example

# GCC
CFLAGS="-march=haswell -O2 -pipe"
CXXFLAGS="${CFLAGS}"
CHOST="x86_64-pc-linux-gnu"
CPU_FLAGS_X86="aes avx avx2 fma3 mmx mmxext pclmul popcnt sse sse2 sse3 sse4_1 sse4_2 ssse3"
MAKEOPTS="-j5"

# USE
SUPPORT="pulseaudio btrfs mtp git chromium"
DESKTOP="infinality emoji cjk"
FUCK="-bindist -grub -plymouth -systemd consolekit -modemmanager -gnome-shell -gnome -gnome-keyring -nautilus -modules"
ELSE="client icu sudo python"

USE="${SUPPORT} ${DESKTOP} ${FUCK} ${ELSE}"

# Portage
PORTDIR="/usr/portage"
DISTDIR="${PORTDIR}/distfiles"
PKGDIR="${PORTDIR}/packages"
# GENTOO_MIRRORS="https://mirrors.tuna.tsinghua.edu.cn/gentoo/"
GENTOO_MIRRORS="https://mirrors.ustc.edu.cn/gentoo/"
EMERGE_DEFAULT_OPTS="--ask --verbose=y --keep-going --with-bdeps=y --load-average"
# FEATURES="${FEATURES} -userpriv -usersandbox -sandbox"
PORTAGE_REPO_DUPLICATE_WARN="0"
# PORTAGE_TMPDIR="/var/tmp/notmpfs"

ACCEPT_KEYWORDS="~amd64"
ACCEPT_LICENSE="*"

# Language
L10N="en-US zh-CN en zh"
LINGUAS="en_US zh_CN en zh"

# Else
VIDEO_CARDS="intel i965 nvidia"

RUBY_TARGETS="ruby24 ruby25"

LLVM_TARGETS="X86"

QEMU_SOFTMMU_TARGETS="alpha aarch64 arm i386 mips mips64 mips64el mipsel ppc ppc64 s390x sh4 sh4eb sparc sparc64 x86_64"
QEMU_USER_TARGETS="alpha aarch64 arm armeb i386 mips mipsel ppc ppc64 ppc64abi32 s390x sh4 sh4eb sparc sparc32plus sparc64"
# ABI_X86="64 32"
```

### 镜像工具安装

```shell
  emerge --ask --verbose --oneshot app-portage/mirrorselect
  mirrorselect -i -o >> /etc/portage/make.conf

  tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
  cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
```

#### 如果使用官方Gentoo install镜像，这一步可以被简化为 arch-chroot /mnt/gentoo，相当于官网进入新环境  chroot /mnt/gentoo /bin/bash 这一步

```shell
source /etc/profile
export PS1="(chroot) ${PS1}"
```

##### UEFI 系统

```
mkdir /efi
mount /dev/sda1 /efi
```

配置 Portage 包管理器

```
emerge-webrsync
emerge --sync
```

跳过阅读新闻条目这个就是更新公告没啥用

```
eselect profile list
```

一般默认就是准确的

```
eselect profile set 22
eselect profile list 
```

再次查看设置后的配置

打开https://mirrors.ustc.edu.cn/gentoo/releases 选择和eselect 一样版本的 路径https://mirrors.ustc.edu.cn/gentoo/releases/amd64/binpackages/23.0/x86-64
nano /etc/portage/binrepos.conf/gentoobinhost.conf sync-uri值替换为 https://mirrors.ustc.edu.cn/gentoo/releases/amd64/binpackages/23.0/x86-64
nano /etc/portage/make.conf 修改配置文件 安装程序默认优先使用二进制包  也可以使用命令行 传递--getbinpkg 参数 安装是指定从二进制安装

getuto 生成密钥环 portage USE变量配置不想配置默认即可
nano /etc/portage/make.conf  中配置  当通过编译方式安装软件时编译项 USE中排出了桌面环境 因为用的是无桌面安装的
USE="-X -gtk -gnome -qt5 -kde"
配置portage CPU_FLAGS_* 默认也可以

```
emerge --ask --oneshot app-portage/cpuid2cpuflags
cpuid2cpuflags
echo "*/* $(cpuid2cpuflags)" > /etc/portage/package.use/00cpu-flags
```

配置安装许可协议
先查看当前协议

```
portageq envvar ACCEPT_LICENSE
```

如何在系统范围接受 ACCEPT_LICENSE 许可证示例

```
nano /etc/portage/make.conf
```

添加 ACCEPT_LICENSE="-* @FREE @BINARY-REDISTRIBUTABLE"
更新@world集合

```
emerge --ask --verbose --update --deep --newuse --getbinpkg @world
```

清理包数据库过时的软件包元信息

```
emerge --ask --pretend --depclean
emerge --ask --depclean
```

查看可用时区

```
ls -l /usr/share/zoneinfo/Asia
```

设置时区为上海

```
ln -sf ../usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

查看可用的区域设置

```
cat  /usr/share/i18n/SUPPORTED
```

设置可用区域

```
nano /etc/locale.gen
```

添加中文区域

```
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_CN.GBK GBK
```

生成区域文件

```
locale-gen
```

设定系统级别的区域设置

```
eselect locale list
```

列出可用区域文件

```
eselect locale list
```

设置本地区域文件

```
eselect locale set 6
```

也可以手动编辑，在systemd stage中是/etc/locale.conf

```
cat /etc/locale.conf
```

重新加载环境更新区域

```
env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
```

大多数无线网卡和 GPU 需要固件才能运行，固件安装

```
emerge --ask sys-kernel/linux-firmware
```

添加引导配置

```
echo "sys-kernel/installkernel dracut grub efistub" >> /etc/portage/package.use/installkernel
tee -a /etc/portage/package.use/systemd <<EOF
sys-apps/systemd boot
sys-kernel/installkernel systemd-boot
EOF
tee -a /etc/portage/package.accept_keywords/installkernel <<EOF
sys-kernel/installkernel
sys-boot/uefi-mkconfig
app-emulation/virt-firmware
EOF

echo "quiet splash" >> /etc/kernel/cmdline

emerge --ask sys-apps/systemd sys-kernel/installkernel
```

统一内核镜像需要 stub loader。目前，唯一可用的是 systemd-stub。要启用它,uki是统一内核缩写，启用统一内核

```
echo "sys-apps/systemd boot" >> /etc/portage/package.use/uki
echo "" >> /etc/portage/package.use/uki

tee -a /etc/portage/package.use/uki <<EOF
sys-apps/systemd boot
sys-kernel/installkernel -dracut -ukify -ugrd uki
sys-kernel/gentoo-kernel-bin generic-uki
EOF
mkdir -p /etc/dracut.conf.d
tee -a /etc/dracut.conf.d/uki.conf <<EOF
uefi="yes"
kernel_cmdline="some-kernel-command-line-arguments"
EOF
emerge --ask -v sys-kernel/installkernel
emerge --ask -v sys-apps/systemd
emerge --ask -v  sys-kernel/gentoo-kernel-bin 
```

安装genfstab 工具

```
emerge --ask sys-fs/genfstab
```

生成fstab

```
genfstab -U /  >> /etc/fstab
```

