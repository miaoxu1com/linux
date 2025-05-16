## GUI概念

从一开始接触linux桌面的时候，我就看到网上有很多资深的linux玩家各种夸奖宣传i3wm、dwm等平铺式桌面的好。看着他们分享出来的桌面截图，说实话真的很漂亮。于是，在习惯了KDE、Gnome、Xfce、Cinammon这些主流桌面后，我决定尝试一下新的不同设计风格的桌面。结果体验下来的感受是——平铺式桌面实在是太棒了！

准确来说，KDE、Gnome、Xfce、Cinammon这些应该叫“桌面环境(Desktop Enviroment，DE)”，同时也自带了“窗口管理器(Windows Manager，WM)”，其窗口排布设计是采用浮动式的，类似于windows或mac的体验，注重鼠标操作；而i3wm、dwm之流则直接舍弃了DE，只有一个WM，且采用平铺式的窗口排布，因而体验与windows/mac完全不同，主要是通过键盘操作(当然也可以用鼠标操作)，但是习惯后你会发现比起鼠标操作要便捷效率多了。

那还有一个显示管理器呢？显示管理器(Display Manager，DM)又叫做“登陆管理器”，如sddm、lightdm等，其作用仅仅只是在你开机后，让你输入用户名和密码登陆，然后引导进入桌面，至此任务完成，之后就交给kde或i3wm管理桌面了。你可以不需要DM，直接通过startx脚本命令进入桌面。

在i3wm和dwm是目前平铺式桌面中的佼佼者，它们自己本身就是一个窗口管理器WM，二者的体验相似，都可以做到个性化定制。我更喜欢dwm，它是由suckless这个开源组织开发的，特点是极其轻量(整个项目代码不到2000行，编译后程序只有88kb)、资源占用极低、高性能、配置简单、美观，且有着不逊于i3wm的体验。

WM虽好，但是各种GUI应用仍然依赖各种五花八门的库，最终还是少不了GTK和Qt

### 使用不是最新的liveos 引导镜像
```shell
#必须要执行禁用校验
sed -i 's/SigLevel.*/SigLevel = Never/' /etc/pacman.conf
```


### archinstall  安装
#### esc显示archinstall的帮助
#### space进行多选/取消多选
#### 启动live os
#### 启动ssh
```shell
systemctl start sshd
```
#### 修改root密码
```shell
passwd
```

#### archinstall安装
```shell
archinstall
```

#### 桌面环境组件
#省的给自己找很多麻烦，导致桌面环境组件缺少，桌面体验不完整，如果要用预发布的桌面选择带桌面环境的，如果要hepyland选择安装minimal，不带桌面环境的后续自己下载hepyland，自定义桌面环境

- ked
- gnome
- minimal




#### 安装时arch可以选择greeter：默认是sddm可以选择其他的  
- gdm是gonme系列
- kdm是kde系列登录管理器

#### 内核版本  
- Linux 默认内核
- lts 长期稳定
- zen 高性能版
- Hardened 稳定强化版  

#### 驱动
- 虚拟机版本
- intel 可以选择
- amg
- nvida

####  添加一个新用户
- 用于登录

#### 可选包和GUI安装引导中的用户自定义安装包一样

- 可以在命令行中/包名  进行匹配安装自己想要的包，然后回车就选中了自定义包，比如最小化安装没有vim 就选择安装vim



#### archinstall最后步骤记得保存配置

#### gnome桌面
> https://dustwind.ink/2024/04/06/ArchLinux+GNOME%E5%AE%89%E8%A3%85/

#### wayland桌面
> https://a-wing.top/linux/2022/01/03/translate_wayland
> https://blog.fangjiahui.me/posts/2024-02-20-arch-wayland-sway-install-fcitx5/
> https://zhuanlan.zhihu.com/p/462322143
#### hyprland桌面
> https://cascade.moe/posts/hyprland-configure/
> https://www.sqlsec.com/2024/09/hyprland.html

> https://blog.soulter.top/posts/arch-linux-hyprland.html
>
> https://www.bilibili.com/opus/778159722494689457
>
> https://www.cnblogs.com/wcisns/p/18706911
>
> https://book.bsdcn.org/di-4-zhang-zhuo-mian-huan-jing/di-4.14-jie-an-zhuang-hyprland

---

### 这里正常不需要执行

#### arch安装阶段忘记添加用户

#在登录界面
#虚拟机启动桌面会比较慢要耐心等待

Ctrl + Alt +  F2-F6 切换到 TTY 终端

用 root 登录：

#### 启动网络组件
```shell
sudo systemctl enable NetworkManager && sudo systemctl start NetworkManager
```


---
#### 创建新用户

```shell
useradd -m -G wheel -s /bin/bash 用户名  # 例如 useradd -m -G wheel -s /bin/bash archuser
passwd 用户名  # 设置该用户的密码（如 passwd archuser）
```

#### 允许 wheel 组使用 sudo

```shell
EDITOR=nano visudo

找到 # %wheel ALL=(ALL:ALL) ALL，去掉 # 取消注释，保存退出（Ctrl+O → Enter → Ctrl+X）
sudo sed -i '/# %wheel ALL=(ALL:ALL) ALL/s/^# //' /etc/sudoers
sudo grep '%wheel ALL=(ALL:ALL) ALL' /etc/sudoers
如果输出 %wheel ALL=(ALL:ALL) ALL，说明修改成功。
```
---
---
#### 重新启动显示管理器

如果使用 SDDM（Hyprland 默认）

```shell
systemctl enable --now sddm  # 确保 SDDM 已启用
systemctl restart sddm       # 重启 SDDM
```

在 TTY 终端用新用户登录后，手动启动 Hyprland：

```shell
startx /usr/bin/Hyprland
```

- 检查用户是否可登录
- 返回图形登录界面（如 SDDM），应该能看到新创建的用户。
- 如果仍然看不到用户，尝试：
```shell
chown -R 用户名:用户名 /home/用户名  # 确保家目录权限正确
```

❌ 登录后黑屏/闪退

检查显卡驱动是否安装（如 nvidia、mesa）。

查看 Hyprland 日志：

```shell
journalctl -u sddm -b  # 如果使用 SDDM
cat ~/.local/share/hyprland/hyprland.log  # Hyprland 日志
```
---


#### 重启进入 Arch ISO（安装U盘）
#### 挂载根分区：
```shell
mount /dev/nvme0n1p2 /mnt  # 替换为你的根分区
arch-chroot /mnt
passwd root  # 修改 root 密码
```

#### gnome无法修改默认终端
```shell
echo $TERMINAL
```

没有输出是没有设置默认终端

```shell
#追加到bashrc配置文件
sed -i '$a export TERMINAL=alacritty' ~/.bashrc
#确保不重复追加
grep -q "export TERMINAL=alacritty" ~/.bashrc || echo "export TERMINAL=alacritty" >> ~/.bashrc
#图形界面下执行$TERMINAL，打开终端配置默认终端成功
```

### 这里正常不需要执行



---



#### 中文语言包

- 方式1 修改文件
```shell
nano /etc/locale.gen
```
- 方式2 命令替换
```shell
sudo sed -i 's/^#zh_CN.UTF-8 UTF-8/zh_CN.UTF-8 UTF-8/' /etc/locale.gen
sudo locale-gen
#如果希望保持终端/命令行仍显示英文（便于排错），可改为
echo 'LANG=en_US.UTF-8' | sudo tee /etc/locale.conf
echo 'LC_ALL=zh_CN.UTF-8' | sudo tee -a /etc/locale.conf
#备字体 字体很小
sudo pacman -S wqy-microhei    # 文泉驿微米黑

```
---
### 这里的不需要执行
```shell
#主字体
pacman -S noto-fonts-cjk  # Google Noto 字体（包含简繁日韩）
#GNOME/KDE支持中文
sudo pacman -S gnome-control-center  # GNOME 设置中心（已包含语言包）
sudo pacman -S plasma-desktop        # KDE 桌面中文包
#dolphin文件管理器中文，直接在文件管理器设置中只保留zh_cn,就会显示中文
#最后重启系统
```
### 这里的不需要执行
---

#### 卸载不需要的文件
```shell
#卸载文件及不再需要的依赖项
sudo pacman -Rs kitty
#查询卸载成功
pacman -Q kitty
```
#### alacritty终端中文
```shell
mkdir -p ~/.config/alacritty && touch ~/.config/alacritty/alacritty.yml
nano ~/.config/alacritty/alacritty.yml
#复制进入alacritty.yml
cat << 'EOF' >> ~/.config/alacritty/alacritty.yml
font:
  normal:
    family: "WenQuanYi Micro Hei"
    style: Regular
  bold:
    family: "WenQuanYi Micro Hei"
    style: Bold
  italic:
    family: "WenQuanYi Micro Hei"
    style: Italic
  size: 11.0
  character_width: 1.0

charset:
  language: "zh_CN.UTF-8"
  renderer: gl

env:
  LC_ALL: "zh_CN.UTF-8"
  LANG: "zh_CN.UTF-8"
EOF
```
#### archinstall安装的gnome不带终端，从启动菜单启动命令行应用会报找不到gnome终端，安装终端后问题解决
```shell
sudo pacman -S  gnome-tweaks gnome-terminal dconf-editor
```
#### man命令
```shell
sudo pacman -S lsd exa

sudo pacman -S man-db man-pages texinfo yazi fastfetch fzf eza zoxide  bash-completion ripgrep ffmpegthumbnailer p7zip jq poppler fd imagemagick

```


#### 修改官方镜像源

#注意：官方archinstall脚本安装时选择中国源，会自动配置mirrorlist，此处无须手动进行



```bash
sudo cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak
sudo tee -a /etc/pacman.d/mirrorlist <<EOF
# 华为镜像站
Server = https://repo.huaweicloud.com/archlinux/\$repo/os/\$arch
# 阿里镜像站
Server = https://mirrors.aliyun.com/archlinux/\$repo/os/\$arch
# 清华镜像站
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/\$repo/os/\$arch
EOF
tail -n 10 /etc/pacman.d/mirrorlist
# **优化镜像顺序**
速度优化：
可以运行以下命令选择最快的镜像：
sudo pacman-mirrors -c China 
使用 `reflector` 自动选择最快镜像（推荐）：
sudo pacman -S reflector
sudo reflector --country China --protocol https --sort rate --save /etc/pacman.d/mirrorlist
或手动将最快的镜像移到文件顶部（Pacman 从上到下选择）
```

#### 修改**中文用户社区仓库**源

#注意：这里需要手动添加



```bash
sudo cp /etc/pacman.conf /etc/pacman.conf.bak
sudo tee -a /etc/pacman.conf <<EOF

[archlinuxcn]
# 阿里archlinuxcn源
Server = https://mirrors.aliyun.com/archlinuxcn/\$arch
# 清华archlinuxcn源（注释备用）
# Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/\$arch
EOF
tail -n 10 /etc/pacman.conf  # 查看最后 10 行
sudo pacman -Sy archlinuxcn-keyring
sudo pacman -Syu
```
---
```shell
> https://mirrors.aliyun.com/chaotic-aur/x86_64/?spm=a2c6h.25603864.0.0.70592e66dhN2RP
# 手动去地址验证 搜索firedragon
# Chaotic-AUR源，Chaotic-AUR打包后的是.pkg.tar.zst包是预编译好的，可以直接安装
# 方式1 在线导入  外网超时
> https://aur.chaotic.cx/docs
> https://github.com/chaotic-aur/keyring/tree/master
sudo pacman -Sy chaotic-keyring
# 可以手动从https://github.com/chaotic-aur/keyring/tree/master中找3056513887B78AEB的密码，搜索master找和3056513887B78AEB匹配的再去matser目录找和匹配的别名相同的密钥手动下载导入
sudo pacman-key --recv-key 3056513887B78AEB --keyserver keyserver.ubuntu.com
sudo pacman-key --lsign-key 3056513887B78AEB
sudo pacman -U 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-keyring.pkg.tar.zst'
sudo pacman -U 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-mirrorlist.pkg.tar.zst'
# 方式2 手动更新
curl -s https://mirrors.aliyun.com/chaotic-aur/chaotic-keyring.pkg.tar.zst -o chaotic-keyring.pkg.tar.zst
sudo pacman -U chaotic-keyring.pkg.tar.zst
sudo tee -a /etc/pacman.conf <<EOF
[chaotic-aur]
Include = /etc/pacman.d/chaotic-mirrorlist
EOF
tail -n 10 /etc/pacman.conf  # 查看最后 10 行
sudo touch /etc/pacman.d/chaotic-mirrorlist
sudo tee /etc/pacman.d/chaotic-mirrorlist<<EOF
Server = https://mirrors.aliyun.com/$repo/$arch
EOF
# 下载软件前要及时更新源否则可能出现用不了的情况
sudo pacman -S firefox-nightly
sudo pacman -Syu
sudo pacman -Syy

# 搜软件查看是否是 Chaotic-AUR 源
pacman -Ss firedragon
```
---
#### 开启pacman多线程

```bash
编辑 /etc/pacman.conf 文件，将对应位置前 # 删除即可
#UseSyslog
Color
#NoProgressBar
CheckSpace
#VerbosePkgLists
ParallelDownloads = 4
```

#### aur不稳定源，最常用的便是 **yay**

```bash
#安装yay需要从源码编译，源码是go需要配置代理
#启用 Go Modules 功能
go env -w GO111MODULE=on
# 配置 GOPROXY 环境变量，以下三选一# 1. 七牛 CDN
go env -w  GOPROXY=https://goproxy.cn,direct
# 2. 阿里云
go env -w GOPROXY=https://mirrors.aliyun.com/goproxy/,direct
# 3. 官方
go env -w  GOPROXY=https://goproxy.io,direct
# **yay** 可以直接通过之前配置的 archlinuxcn 源进行安装：**
sudo pacman -S yay
```

#### Rust 写的 AUR 工具 **paru**安装

```bash
sudo pacman -Suy --needed --noconfirm base-devel
git clone https://aur.archlinux.org/paru.git
mkdir -p ~/.cargo/ && touch ~/.cargo/config.toml

#方式1
sudo tee ~/.cargo/config.toml <<EOF
[source.crates-io]
replace-with = 'ustc'

[source.ustc]
registry = "sparse+https://mirrors.ustc.edu.cn/crates.io-index/"
EOF

#方式2
临时切换镜像（无需修改文件）
# 使用中科大镜像临时加速
export CARGO_REGISTRIES_CRATES_IO_PROTOCOL=sparse
export RUSTUP_DIST_SERVER="https://mirrors.ustc.edu.cn/rust-static"
export RUSTUP_UPDATE_ROOT="https://mirrors.ustc.edu.cn/rust-static/rustup"

#方式3
#在 ~/.cargo/config 中添加
sudo tee ~/.cargo/config.toml <<EOF
[net]
git-fetch-with-cli = true
EOF

cd paru
RUSTUP_UPDATE_ROOT=https://rsproxy.cn/rustup RUSTUP_DIST_SERVER=https://rsproxy.cn GOPROXY=https://goproxy.cn makepkg -si
```

#### 安装HypeLand配置文件

[end-4](https://github.com/end-4)/[dots-hyprland](https://github.com/end-4/dots-hyprland)

yay缓存放在了/home/mx/.cache/yay目录



arch-packages包缓存放在/home/mx/.cache/dots-hyprland/arch-packages目录



[prasanthrangan](https://github.com/prasanthrangan)/[hyprdots](https://github.com/prasanthrangan/hyprdots)

> https://bbs.archlinuxcn.org/viewtopic.php?id=12144

```shell
# 脚本文件下载文件的（fake_curl_makepkg.sh）
#!/bin/bash

# 参数检查
if [ $# -lt 2 ]; then
    echo "Usage: $0 <output_file> <url>"
    exit 1
fi

output_file="$1"
url="$2"

# 提取域名和路径
domain=$(echo "$url" | awk -F/ '{print $3}')
path_after_domain=$(echo "$url" | awk -F/ '{for(i=4;i<=NF;i++) printf "/%s", $i}')

case "$domain" in
    "github.com")
# https://github.proxy.class3.fun经常失效要注意进行替换
        mirror_url="https://github.proxy.class3.fun/https://github.com${path_after_domain}"
        echo "Downloading from GitHub mirror: $mirror_url"
        if ! /usr/bin/curl -gqb "" -fLC - --retry 3 --retry-delay 3 --connect-timeout 30 -m 600 -o "$output_file" "$mirror_url"; then
            echo "Mirror download failed, trying original URL"
            /usr/bin/axel -n 15 -a -o "$output_file" "$url"
        fi
        ;;
    *)
        echo "Downloading using axel: $url"
        /usr/bin/axel -n 15 -a -o "$output_file" "$url"
        ;;
esac

# 检查最终下载结果
if [ ! -f "$output_file" ]; then
    echo "Download failed"
    exit 1
fi
```

```shell
# 克隆项目的
#!/bin/bash

# Git Clone 镜像加速脚本
# 使用方法: ./git-clone-mirror.sh [原始git仓库URL]

if [ $# -lt 1 ]; then
    echo "Usage: $0 <git_repository_url>"
    exit 1
fi

original_url="$1"

# 提取域名和路径
domain=$(echo "$original_url" | awk -F/ '{print $3}')
repo_path=$(echo "$original_url" | awk -F/ '{for(i=4;i<=NF;i++) printf "/%s", $i}' | sed 's/\.git$//')
echo "$domain $repo_path"
case "$domain" in
    "github.com")
        mirror_url="https://github.proxy.class3.fun/https://github.com${repo_path}.git"
        echo "Cloning from GitHub mirror: $mirror_url"
        git clone --depth 1 "$mirror_url"
        if [ $? -ne 0 ]; then
            echo "Mirror clone failed, trying original URL"
            git clone --depth 1 "$original_url"
        fi
        ;;
    "gitlab.com")
        mirror_url="https://gitlab.com.cnpmjs.org${repo_path}.git"
        echo "Cloning from GitLab mirror: $mirror_url"
        git clone --depth 1 "$mirror_url"
        if [ $? -ne 0 ]; then
            echo "Mirror clone failed, trying original URL"
            git clone --depth 1 "$original_url"
        fi
        ;;
    *)
        echo "Cloning original repository: $original_url"
        git clone --depth 1 "$original_url"
        ;;
esac

# 检查克隆结果
if [ $? -ne 0 ]; then
    echo "Clone failed"
    exit 1
fi

echo "Clone completed successfully"
```

#### 安装依赖工具

```bash
###hyprland有二进制包
pacman -S hyperland-bin
#预置主题需要使用uv创建虚拟环境设置镜像源
export UV_PYTHON_INSTALL_MIRROR=https://gh-deno.mocn.top/https://github.com/astral-sh/python-build-standalone/releases/download
#临时设置pypi镜像源
export UV_DEFAULT_INDEX="https://mirrors.aliyun.com/pypi/simple"
mkdir ~/.config/uv/ && touch ~/.config/uv/uv.toml
sudo tee ~/.config/uv/uv.toml<<EOF
index-url="https://mirrors.cloud.tencent.com/pypi/simple/"
EOF
#git 加速 替换后可以手动git clone一个仓库看看速度 ，注意这里https://gh-deno.mocn.top会经常失效 去网上搜索 github加速下载 找有效地址替换
# 方式1
git config --global url."https://gh-deno.mocn.top/https://github.com".insteadOf "https://github.com"
# deb rpm 转arch系统安装包
# PKGBUILD 和你 wget 时使用的是 HTTPS 协议，你自己 git clone 的时候使用的是 ssh 协议
# 修改 PKGBUILD 里的 source 改成 git+ssh://git@github.com/xxxx 这样来使用 ssh 协议
# 方式2
yay -S --needed --noconfirm axel
cat /etc/makepkg.conf
touch ~/fake_curl_makepkg.sh
chmod +x fake_curl_makepkg.sh
copy -v fake_curl_makepkg.sh /usr/bin
# 修改文件中,此处是脚本路径
'https::/usr/bin/fake_curl_makepkg.sh %o %u'

git clone https://github.com/DreamMaoMao/maomaowm.git
#yay-bin预编译的二进制包，AUR helpers
paru -S rysnc  yay-bin less
#恢复urrpcurl值为官方
yay --aururl "https://aur.archlinux.org" --save
yay --aurrpcurl "https://aur.archlinux.org/rpc" --save
#查看yay修改的配置
yay -P -g
#手动修改 AUR 软件包的 PKGBUILD
#如果某些 AUR 软件包的源码托管在 GitHub，可以手动替换 PKGBUILD 中的 github.com 为国内镜像
git clone https://aur.archlinux.org/package-name.git
cd package-name
#这个方法是可行的，安装过程中根据安装过程查看是那个包需要从aur仓库下载，从网页aur搜索对应仓库，查看详情页的git克隆地址手动进行克隆，克隆后手动进行替换编译文件的github地址为镜像地址，比如：
illogical-impulse-bibata-modern-classic-bin
ttf-rubik-vf

git clone  https://aur.archlinux.org/packages/ttf-rubik-vf
cd ttf-rubik-vf
sed -i 's|https://github.com|https://gh-deno.mocn.top/https://github.com|g' PKGBUILD  # 替换为 FastGit 镜像
makepkg -si
```

#### 启动

```
nano ~/.bashrc
export XDG_RUNTIME_DIR=/run/user/$(id -u)
#重启系统，执行命令，不能用root用户启动
Hyprland
```

#### 配置开机启动Hyperland
配置完后一定要从虚拟机运行，不能从ssh连接启动

#### 远程桌面
#### 安装maomaodw
- 方式1
yay -S maomaowm-git
- 方式2 手动执行多个
yay -S wlroots-0.19-git
git clone https://github.com/DreamMaoMao/maomaowm.git
cd maomaowm
meson build -Dprefix=/usr
sudo ninja -C build install

- 会去下载gitlab仓库很慢https://gitlab.freedesktop.org/wlroots/wlroots.git/
- 可以去gitee搜速wlroots加速下载 https://gitee.com/DreamMaoMao/wlroots  


