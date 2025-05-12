## 安装手册/指导书

### vmware虚拟机安装一定要在高级中设置为uefi模式，创建虚拟机时一定要选择linux 其他对应内核版本  选择其他的无法修改uefi模式

https://wiki.gentoo.org/wiki/Handbook:AMD64/Full/Installation/zh-cn

虚拟机安装可以跳过引导盘制作、镜像校验、定制启动、用户添加（默认都是用root更方便）、启动参数相当于window安全启动或者高级启动

adslwepwifi 网络配置也可以跳过因为虚拟机一般都是DHCP自动获取、网络代理也无需配置

使用nat网络方式上网，需要把已经配置了外网的物理网卡共享给虚拟网卡vmnat8，注意不要把共享网络配反了，是把物理公网共享给虚拟网络，然后继续配置vmnat8 ip为vmnat8子网ip

例如：

192.168.x.2

255.255.255.0

192.168.x.1 //这里是vmnat的网关地址

具体过程参考
https://blog.csdn.net/hhc550056259/article/details/123916999

#### 修改dns

```shell
sed -i '$a nameserver 233.5.5.5' /etc/resolv.conf
```

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
ping 114.114.114.114 
```

有时ping不同可以多一个223.5.5.5

```
ping 223.5.5.5
```



然后配置nameserver 223.5.5.5
/etc/resolv.conf

系统安装的大致过程都一样

启动到live系统 live是带gui的 桌面系统minalios 是最小化的在线安装

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
mkfs.xfs /dev/sda3
mkfs.vfat -F 32 /dev/sda1
mkswap /dev/sda2
swapon /dev/sda2
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
在live系统中挂载系统到目标 系统目录 sda3 是根目录分区 sda1是efi分区 挂载到对应的目录 

```
mount /dev/sda3 /mnt/gentoo
```

创建efi目录并挂在分区

```
mkdir --parents /mnt/gentoo/efi
```
```
mount /dev/sda1 /mnt/gentoo/efi
```






使用默认配置安装

切换到目标系统编译安装内核, 进行预安装工作
open-rc 和 system是用来管理服务
重启服务

```
systemctlrestart sshd
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
#### 时间同步

```shell
chronyd -q
```

#### 下载stage3
```shell
links https://mirrors.ustc.edu.cn/gentoo/releases/amd64/autobuilds/current-stage3-amd64-systemd/
lynx https://mirrors.ustc.edu.cn/gentoo/releases/amd64/autobuilds/current-stage3-amd64-systemd/

```
#### 指定源同步
```shell
GENTOO_MIRRORS="https://mirrors.ustc.edu.cn/gentoo" emerge-webrsync
```

#### 安装初始化程序stage3到目标系统

```
cd /mnt/gentoo
tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
```

GCC编译配置 -O3代表优化级别,如果采用更高的-Ofast可能会导致部分软件包编译错误, -march=native代表为本机cpu进行编译,如果是交叉编译需要去掉

### 设置gcc编译选项 make.config：

```shell
nano /mnt/gentoo/etc/portage/make.conf
emerge -vj --getbinpkg --ask ccache aria2
mkdir -p /var/cache/ccache
chown root:portage /var/cache/ccache -R
chmod 2775 /var/cache/ccache -R
```
##### 方案1
```shell
COMMON_FLAGS="-march=native -O2 -pipe"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
FCFLAGS="${COMMON_FLAGS}"
FFLAGS="${COMMON_FLAGS}"
# This sets the language of build output to English.
# Please keep this setting intact when reporting bugs.
LC_MESSAGES=C.utf8
# NOTE: This stage was built with the bindist USE flag enabled
USE="dist-kernel"
PORTAGE_BINHOST="https://mirrors.ustc.edu.cn/gentoo/releases/amd64/binpackages/23.0/x86-64/"
GENTOO_MIRRORS="https://mirrors.ustc.edu.cn/gentoo/"
LINGUAS="en en_US zh zh_CN"
L10N="en en-US zh zh-Hans zh-Hans-CN zh-CN"
GRUB_PLATFORMS="efi-64"
FEATURES="${FEATURES} getbinpkg"
FEATURES="${FEATURES} binpkg-request-signature"
FEATURES="${FEATURES} ccache -test"
ACCEPT_KEYWORDS="~amd64"
ACCEPT_LICENSE="*"
AUTO_CLEAN="yes"
# Portage
PORTDIR="/usr/portage"
DISTDIR="${PORTDIR}/distfiles"
PKGDIR="${PORTDIR}/packages"
CCACHE_DIR="/var/cache/ccache"
FETCHCOMMAND="/usr/bin/aria2c -d \${DISTDIR} -o \${FILE} --allow-overwrite=true --max-tries=5 --max-file-not-found=2 --max-concurrent-downloads=5 --connect-timeout=5 --timeout=5 --split=5 --min-split-size=2M --lowest-speed-limit=20K --max-connection-per-server=9 --uri-selector=feedback \${URI}"
RESUMECOMMAND="${FETCHCOMMAND}"
EMERGE_DEFAULT_OPTS="--ask --verbose=y --keep-going --with-bdeps=y --load-average"
```


##### 方案2
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
FEATURES="${FEATURES} getbinpkg"
FEATURES="${FEATURES} binpkg-request-signature"
ACCEPT_KEYWORDS="~amd64"
ACCEPT_LICENSE="*"

INPUT_METHOD=fcitx5
XIM=fcitx5
XIM_PROGRAM=fcitx5
GTK_IM_MODULE=fcitx5
QT_IM_MODULE=fcitx5
XMODIFIERS=@im=fcitx5
SDL_IM_MODULE=fcitx5
```
##### 方案3
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
##### 方案4
```shell
# These settings were set by the catalyst build script that automatically
# built this stage.
# Please consult /usr/share/portage/config/make.conf.example for a more
# detailed example.
COMMON_FLAGS="-march=native -O2 -pipe -finline-functions -fomit-frame-pointer"（无论何种intel、amd的CPU，且无论何种新老CPU架构，均建议-march=native，CPU指令集自动识别全面）  #程序员的用户注意了，“-fomit-frame-pointer"这一项会导致你编译出来的程序无法debug；不做程序开发或debug的普通用户可以放心开启。
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
FCFLAGS="${COMMON_FLAGS}"
FFLAGS="${COMMON_FLAGS}"
LDFLAGS="${COMMON_FLAGS} -Wl,-O2 -Wl,--as-needed -Wl,--hash-style=gnu -Wl,--sort-common -Wl,--strip-all"
MAKEOPTS="-j6"（推荐值为CPU中核心/逻辑处理器的数量，可用lscpu命令查看，结果为“CPU(s):”后面的数字）(比如intel core i5 8400是6核心，i7是8核)
#CPU_FLAGS_X86="aes avx avx2 f16c fma3 mmx mmxext pclmul popcnt sse sse2 sse3 sse4_1 sse4_2 ssse3"（这是intel core i5 8400的参数，其它cpu参数用cpuid2cpuflags命令看，这里先不用管，暂时注释掉，后面再来配置）

CHOST="x86_64-pc-linux-gnu"（64位系统）
EMERGE_DEFAULT_OPTS="--with-bdeps=y --ask --verbose=y --load-average --keep-going --deep"
# NOTE: This stage was built with the bindist Use flag enabled
PORTDIR="/var/db/repos/gentoo"
DISTDIR="/var/cache/distfiles"
PKGDIR="/var/cache/binpkgs"
PORTAGE_TMPDIR="/tmp"    #如果你的内存足够大(8G、16G)，那么建议你把编译程序时存放临时中间文件的目录设置为内存的tmpfs(/tmp目录)，以减少编译时对硬盘的大量读写、延长硬盘使用寿命、并加快编译速度；但如果你的内存较小(<=4G)，那么建议你把此项注释掉，否则很多程序会因内存容量不足而导致编译失败

# This sets the language of build output to English.
# Please keep this setting intact when reporting bugs.
LC_MESSAGES=C

USE="-bindist -mdev -systemd -consolekit -doc -test -gnome-shell -gnome -gnome-keyring -handbook -gtk -dhcpcd -netifrc -pulseaudio -wayland -oss -gpm -iptables -bluetooth gdbm iwd udev gold icu nftables X staging fortran lto pgo graphite openmp minizip udev blkid efi hwdb smack acpi kde alsa sudo cjk ccache dbus policykit network elogind udisks aria2 dhclient networkmanager connection-sharing wifi ppp jack libsamplerate vdpau vaapi vulkan layers nvidia glamor http2"
ACCEPT_LICENSE="*"
ACCEPT_KEYWORDS="amd64"("amd64"是使用稳定版的较旧的软件，"~amd64"是使用不稳定版的更新的软件；建议用稳定版的，免得不稳定的软件包出了问题还要折腾；而且gentoo所谓“稳定版”的“旧”软件相比起debian、ubuntu、centos这些已经很新了，除非你是想像archlinux那样追新)
L10N="en-US zh-CN en zh"
LINGUAS="en-US zh-CN en zh"
AUTO_CLEAN="yes"

GRUB_PLATFORMS="efi-64"（UEFI 64位系统引导必须项）

VIDEO_CARDS="intel i965 iris nvidia"（Intel UHD630和nvidia双显卡）
ALSA_CARDS="hda-intel"（intel HD声卡）
INPUT_DEVICES="libinput synaptics"（笔记本电脑的触控板）
MICROCODE_SIGNATURES="-S"（如果想把CPU的microcode直接编译进内核，则需要设置为“-S”；否则注释掉）

LLVM_TARGETS="X86"

GENTOO_MIRRORS="https://mirrors.163.com/gentoo"

#安装完aria2之后去掉注释,具体参数参考aria2官方文档
#FETCHCOMMAND="/usr/bin/aria2c -d \${DISTDIR} -o \${FILE} --allow-overwrite=true --max-tries=5 --max-file-not-found=2 --max-concurrent-downloads=5 --connect-timeout=5 --timeout=5 --split=5 --min-split-size=2M --lowest-speed-limit=20K --max-connection-per-server=9 --uri-selector=feedback \${URI}"
#RESUMECOMMAND="${FETCHCOMMAND}"

#此处先注释掉,配置完ccache后再去掉注释
#FEATURES="ccache -test"
#CCACHE_DIR="/var/cache/ccache"
```
##### 方案5

> https://blog.zozx.top/2025/02/22/gentoo-installation-guide/
> https://zhuanlan.zhihu.com/p/122222365
> https://bitbili.net/gentoo-linux-installation-and-usage-tutorial.html#%E5%87%86%E5%A4%87%E5%B7%A5%E4%BD%9C
> https://gitzhangzhao.github.io/posts/linux/gentoo/gentoo/
> https://gtrush.com/2022/06/12/%E6%96%B0%E6%89%8BGentoo%E6%8A%98%E8%85%BE%E8%AE%B0%E5%BD%951-%E5%AE%89%E8%A3%85%E7%AF%87-%E4%BA%8C%E8%BF%9B%E5%88%B6kernel%E5%BF%AB%E9%80%9F%E5%AE%89%E8%A3%85%E6%96%B9%E6%B3%95/

```shell
# These settings were set by the catalyst build script that automatically
# built this stage.
# Please consult /usr/share/portage/config/make.conf.example for a more
# detailed example.
NTHREADS=12 # 线程数

COMMON_FLAGS="-march=skylake -O3 -pipe -fgraphite-identity -floop-nest-optimize -fno-stack-protector -fno-align-functions -fno-align-jumps -fno-align-loops -fno-align-labels"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
FFLAGS="${COMMON_FLAGS}"
FCFLAGS="${COMMON_FLAGS}"
LDFLAGS="-Wl,-O3 -Wl,--as-needed -Wl,--hash-style=gnu -Wl,--sort-common -Wl,--strip-all" # 该LDFLAGS会导致networkmanager装不了
RUSTFLAGS="-C opt-level=3 -C target-cpu=skylake"

# NOTE: This stage was built with the bindist Use flag enabled
PORTDIR="/var/db/repos/gentoo"
DISTDIR="/var/cache/distfiles"
PKGDIR="/var/cache/binpkgs"
PORTAGE_TMPDIR="/tmp"

# This sets the language of build output to English.
# Please keep this setting intact when reporting bugs.
LC_MESSAGES=C

MAKEOPTS="-j${NTHREADS} -l${NTHREADS}"
PORTAGE_NICENESS=15
PORTAGE_IONICE_COMMAND="ionice -c 3 -p \${PID}"
GENTOO_MIRRORS="https://mirrors.tuna.tsinghua.edu.cn/gentoo"
FETCHCOMMAND="/usr/bin/aria2c -d \${DISTDIR} -o \${FILE} --allow-overwrite=true --max-tries=8 --max-file-not-found=2 --max-concurrent-downloads=128 --connect-timeout=15 --timeout=15 --split=128 --min-split-size=2M --lowest-speed-limit=20K --max-connection-per-server=16 --uri-selector=feedback \${URI}" # 此处是用aria2代替wget
RESUMECOMMAND="${FETCHCOMMAND}"
USE="lto pgo graphite jemalloc ccache clang staging zsh-completion bluetooth pulseaudio pipewire screencast ffmpeg openssl network wifi iptables zstd lz4 7zip rar btrfs tpm gnome-keyring qemu wayland gles2 vdpau vaapi vulkan vkd3d d3d9 nvidia nvenc steamfonts trayicon systray -joystick -games -education -xinerama -firewall -networkmanager -ppp -kaccounts -webengine -kwallet -bittorrent -phonon -vlc -gtk2 -gnome -gnome-shell -gnome-online-accounts -bindist -ssp -doc -gtk-doc -handbook -spell -grub -oss -gpm"
ACCEPT_KEYWORDS="~amd64" # ~表示unstable，amd64表示架构是x86_64
ACCEPT_LICENSE="*" # 接受所有协议
EMERGE_DEFAULT_OPTS="--keep-going --with-bdeps=y --jobs=${NTHREADS} --load-average=${NTHREADS}"
L10N="en-US zh-CN en zh"
LINGUAS="en_US zh_CN en zh"
VIDEO_CARDS="nvidia intel"
ALSA_CARDS="hda-intel"
LLVM_TARGETS="X86 NVPTX"
PYTHON_TARGETS="python3_10"
PYTHON_SINGLE_TARGET="python3_10"
RUBY_TARGETS="ruby30 ruby31"
ABI_X86="64 32"
FEATURES="ccache"
CCACHE_DIR="/var/cache/ccache"
CPU_FLAGS_X86="aes avx avx2 f16c fma3 mmx mmxext pclmul popcnt rdrand sse sse2 sse3 sse4_1 sse4_2 ssse3" # 本行由app-portage/cpuid2cpuflags生成
CONFIG_PROTECT="/usr/share/sddm/scripts/Xsetup"
UNINSTALL_IGNORE="/bin /lib /lib64 /sbin /usr/sbin"
```
#### 查看配置的USE

```shell
emerge --info | grep ^USE
```

#### 配置软件镜像源

```shell
mkdir -p -v /mnt/gentoo/etc/portage/repos.conf
cp -v /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf

sed -i 's|^\(sync-uri\s*=\s*\).*|\1rsync://rsync.mirrors.ustc.edu.cn/gentoo-portage/|' /mnt/gentoo/etc/portage/repos.conf/gentoo.conf

nano -w /mnt/gentoo/etc/portage/repos.conf/gentoo.conf    #加入中国源
sync-uri = rsync://mirrors.163.com/gentoo-portage
#sync-uri = rsync://mirrors.tuna.tsinghua.edu.cn/gentoo-portage/
#sync-uri = rsync://rsync.mirrors.ustc.edu.cn/gentoo-portage/
#sync-uri = rsync://rsync.mirrors.ustc.edu.cn/gentoo-portage/
#sync-uri = rsync://mirrors.yun-idc.com/gentoo-portage/
```

#### 镜像工具安装

```shell
emerge --ask --verbose --oneshot app-portage/mirrorselect
mirrorselect -i -o >> /etc/portage/make.conf
sed -i '$a nameserver 233.5.5.5' /etc/resolv.conf
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
```

#### 如果使用官方Gentoo install镜像，这一步可以被简化为 arch-chroot /mnt/gentoo，相当于官网进入新环境chroot /mnt/gentoo /bin/bash 这一步

```shell
arch-chroot /mnt/gentoo
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
#### 配置二进制包的镜像源地址
```shell
sed -i 's|^\(sync-uri\s*=\s*\).*|\1https://mirrors.ustc.edu.cn/gentoo/releases/amd64/binpackages/23.0/x86-64|' /etc/portage/binrepos.conf/gentoobinhost.conf
```
打开https://mirrors.ustc.edu.cn/gentoo/releases 选择和eselect 一样版本的 路径https://mirrors.ustc.edu.cn/gentoo/releases/amd64/binpackages/23.0/x86-64
```shell
nano /etc/portage/binrepos.conf/gentoobinhost.conf sync-uri
```
值替换为 
https://mirrors.ustc.edu.cn/gentoo/releases/amd64/binpackages/23.0/x86-64
```shell
nano /etc/portage/make.conf
``` 
修改配置文件 安装程序默认优先使用二进制包也可以使用命令行 传递--getbinpkg 参数 安装是指定从二进制安装
getuto 生成密钥环 portage USE变量配置不想配置默认即可
nano /etc/portage/make.conf中配置当通过编译方式安装软件时编译项 USE中排出了桌面环境 因为用的是无桌面安装的
```shell
USE="-X -gtk -gnome -qt5 -kde"
```
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
> [!TIP]
提示
如果选择了桌面环境配置文件，则此过程可能大大增加安装过程所需的时间量。时间紧迫的人可以通过这个“经验法则”工作：配置文件名称越短，系统 @world 集 越不具体; @world 集越不具体，系统将需要的软件包越少。换一种说法：
选择 default/linux/amd64/23.0 将只有很少的包被重装或更新
选择 default/linux/amd64/23.0/desktop/gnome/systemd 将需要安装许多软件包，因为 init 系统要从 OpenRC 更改为 systemd，并且将安装 GNOME 桌面环境框架。

```
emerge --ask --verbose --update --deep --newuse --getbinpkg @world
# 简写
emerge -vuDN @world
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
cat/usr/share/i18n/SUPPORTED
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
---
==========================这里无须执行========================
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
emerge --ask -vsys-kernel/gentoo-kernel-bin 
```
=====================无须执行=================================
---

#### 编辑器和其他工具安装
```shell
emerge -vj --ask --getbinpkg  app-editors/vim  btrfs-progs neovim eselect-repository xfsprogs  dosfstools
```

#### 修改默认编辑起
```shell
eselect editor list
eselect editor set 「序号」
# 之后再运行一次
. /etc/profile
PS1=(chroot)$PS1
```

#### 磁盘文件系统安装
```shell
emerge -vj --getbinpkg --ask sys-fs/xfsprogs sys-fs/dosfstools
```

#### 为方便在安装二进制内核时安装 initramfs，需添加如下 USE 配置（什么是 USE 见下文 USE 标记 一节）
```shell
echo 'sys-kernel/installkernel dracut' >/etc/portage/package.use/installkernel
emerge -vj --getbinpkg --ask linux-firmware gentoo-kernel-bin grub
```


安装genfstab 工具

```
emerge -vj --getbinpkg --ask sys-fs/genfstab
```

生成fstab

```
genfstab -U />> /etc/fstab
```

#### 修改root密码
passwd
#### 修改hostsname
echo tux > /etc/hostname
#### 安装工具
emerge -vj --getbinpkg --ask sys-apps/mlocate net-misc/chrony app-shells/bash-completion net-misc/dhcpcd
#### 设置服务
systemctl enable dhcpcd
systemd-machine-id-setup
systemd-firstboot --prompt
systemctl preset-all --preset-mode=enable-only
systemctl preset-all
systemctl enable sshd


#### （可选）如果之前有分配交换分区，在这里可以执行如下命令以启用其休眠后唤醒的功能
sed -Ei "/GRUB_CMDLINE_LINUX_DEFAULT/s/^#*(GRUB.*DEFAULT=).*$/\1\"resume=UUID=$(blkid -o value /dev/sdX4 | head -1)\"/" /etc/default/grub
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=Gentoo --removable
grub-mkconfig -o /boot/grub/grub.cfg
#### 同步一下当前的文件系统
sync
exit
umount -l /mnt/gentoo/dev{/shm,/pts,}
umount -R /mnt/gentoo
reboot
