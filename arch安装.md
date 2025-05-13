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
- intel
- amg
- nvida

####  添加一个新用户
- 用于登录

#### archinstall最后步骤记得保存配置


#### wayland桌面
> https://a-wing.top/linux/2022/01/03/translate_wayland

#### hyprland桌面
> https://cascade.moe/posts/hyprland-configure/
> https://blog.soulter.top/posts/arch-linux-hyprland.html

#### arch安装阶段忘记添加用户

#在登录界面

Ctrl + Alt +  F2-F6 切换到 TTY 终端

用 root 登录：

#### 创建新用户

```shell
useradd -m -G wheel -s /bin/bash 用户名  # 例如 useradd -m -G wheel -s /bin/bash archuser
passwd 用户名  # 设置该用户的密码（如 passwd archuser）
```

#### 允许 wheel 组使用 sudo

```shell
EDITOR=nano visudo

找到 # %wheel ALL=(ALL:ALL) ALL，去掉 # 取消注释，保存退出（Ctrl+O → Enter → Ctrl+X）
```

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

#### 重启进入 Arch ISO（安装U盘）
#### 挂载根分区：
```shell
mount /dev/nvme0n1p2 /mnt  # 替换为你的根分区
arch-chroot /mnt
passwd root  # 修改 root 密码
```

#### 默认终端
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
```
```shell
#主字体
pacman -S noto-fonts-cjk  # Google Noto 字体（包含简繁日韩）
#备字体
sudo pacman -S wqy-microhei    # 文泉驿微米黑
#GNOME/KDE支持中文
sudo pacman -S gnome-control-center  # GNOME 设置中心（已包含语言包）
sudo pacman -S plasma-desktop        # KDE 桌面中文包
#dolphin文件管理器中文，直接在文件管理器设置中只保留zh_cn,就会显示中文
#最后重启浏览器
```
