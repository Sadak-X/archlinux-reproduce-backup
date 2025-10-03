# ThinkPad W540 archlinux 重建备份

- [ThinkPad W540 archlinux 重建备份](#thinkpad-w540-archlinux-重建备份)
  - [安装snapper](#安装snapper)
  - [安装显卡驱动](#安装显卡驱动)
    - [关闭独显](#关闭独显)
  - [安装软件列表备份](#安装软件列表备份)
  - [中文配置](#中文配置)
    - [系统中文](#系统中文)
    - [sddm 界面的中文](#sddm-界面的中文)
  - [plyomuth配置](#plyomuth配置)
  - [中文输入法](#中文输入法)
  - [discover 应用商店](#discover-应用商店)
  - [指纹](#指纹)
  - [防火墙](#防火墙)
  - [休眠](#休眠)
  - [文件搜索](#文件搜索)
  - [图形化git](#图形化git)
  - [文件共享](#文件共享)
  - [光盘刻录](#光盘刻录)
  - [视频播放](#视频播放)
  - [省电配置](#省电配置)

## 安装snapper

```bash
yay -S snapper snap-pac btrfs-assistant
```

## 安装显卡驱动

**你那破显卡，能装上驱动， KDE 的 settings 能认出来就谢天谢地了，别想着拿你那个破显卡去搞独显输出了，搞不好的。**\
实在需要独显的，用 prime-run 解决。\
目前已知网络上提供的独显输出方案会造成:

1. 虽然咱们是 Wayland 会话，但是运行在 X11 模式下的 sddm 会黑屏，虽然能输入密码验证登录，但是很影响观感。
2. 应用画面撕裂，渲染异常，卡死无响应。

我分析问题应该是 W540 这个机器的内置显示器连接在了核显上，就算使用一些方法强制使用独显作为渲染后端，实际屏幕输出仍然是由核显进行的，因此导致上述问题2的出现。

使用独显进行画面输出需要 ThinkPad 底座扩展坞，寝室里摆不下，反正网上存货很多，以后再考虑。

```bash
# 安装基础工具
sudo pacman -S --needed linux-zen-headers && sudo pacman -S --needed base-devel git && git clone https://aur.archlinux.org/yay.git && cd yay && makepkg -si && yay -S nvidia-470xx-dkms  

# 编辑uki生成参数，设置模块早启动
sudo vim /etc/mkinitcpio.conf  
# modules中加入
nvidia nvidia_modeset nvidia_uvm nvidia_drm  

# 编辑内核参数
sudo vim /etc/kernel/cmdline  
# 直接加到后面
nvidia_drm.modeset=1 nvidia_drm.fbdev=1  

sudo mkinitcpio -P  
reboot
```

### 关闭独显

我们使用 envycontrol 来完成这一目标，实测下来非常好用，唯一的缺点就是每次使用的时候需要重建 initramfs，这个需要等待至少一分钟，然后还需要重启，但是我觉得已经很OK了，确实能做到独显和集显的切换，而且不会有什么黑屏等乱七八糟的问题。

```bash
# 第一条和第三条命令实测下来几乎没有什么不一样，这个机器不支持独显输出
sudo envycontrol -s nvidia
sudo envycontrol -s integrated
sudo envycontrol -s hybrid
# 使用 glxinfo | grep "OpenGL renderer string" 命令验证
```

已知存在以下问题：

1. 切换至集成显卡后（integrate），系统设置中“关于此系统”页面无法打开，会导致系统设置崩溃退出；

以上问题不影响正常使用，因此我们直接将其忽略。

## 安装软件列表备份

```text
# 系统字体中文支持
adobe-source-han-sans-cn-fonts 2.005-1
adobe-source-han-serif-cn-fonts 2.003-1
ttf-firacode-nerd 3.4.0-1

# plymouth相关 
plymouth 24.004.60-11
plymouth-kcm 6.4.5-1
breeze-plymouth 6.4.5-1

# btrfs工具以及快照工具
btrfs-assistant 2.2-2
snap-pac 3.0.1-3
snapper 0.13.0-1

# 光驱刻录工具 kde官方出品
k3b 1:25.08.1-1
cdrdao 1.2.5-7
cdrtools 3.02a09-6
dvd+rw-tools 7.1-12

# 编译器
clang 20.1.8-1

# 打印机
cups-pk-helper 0.2.7-2
system-config-printer 1.5.18-5

# rime fcitx5 等输入法
qt5-base 5.15.17+kde+r123-2
qt5-wayland 5.15.17+kde+r57-1
librime 1:1.12.0-4
fcitx5-configtool 5.1.10-1
fcitx5-gtk 5.1.4-1
fcitx5-rime 5.1.11-1
rime-ice-double-pinyin-flypy-git r829.fe40df5-1

# 磁盘工具 kde官方出品
partitionmanager 25.08.1-1
filelight 25.08.1-1

# discover应用商店后端
flatpak 1:1.16.1-1
flatpak-kcm 6.4.5-1

# 指纹功能
fprintd 1.94.5-1

# 固件
fwupd 2.0.16-1
wd719x-firmware 1-9
upd72020x-fw 1:1.0.0-3
linux-firmware 20250917-1
linux-firmware-qlogic 20250917-1

# 看图工具 kde官方出品
gwenview 25.08.1-1

# 摄像头 kde官方出品
kamoso 25.08.1-1

# 计算器 kde官方出品
kcalc 25.08.1-1

# dolphin 图形化 root 支持
kio-admin 25.08.1-1

# 日志查看工具 kde官方出品
ksystemlog 25.08.1-1

# 装显卡驱动以及其他要用 dkms 的软件使用
nvidia-470xx-dkms 470.256
nvidia-prime 1.0-5
linux-zen 6.16.8.zen3-1
linux-zen-headers 6.16.8.zen3-1

# 浏览器
microsoft-edge-stable-bin 140.0.3485.94-1

# 防火墙
ufw 0.36.2-5
ufw-extras 1.1.0-2

# U盘启动镜像刷写工具
ventoy-bin 1.1.07-1

# 开发
visual-studio-code-bin 1.104.2-1

# 下拉式终端
yakuake 25.08.1-1

# aur helper
yay 12.5.2-2
```

## 中文配置

### 系统中文

先在设置里面选中文，会提示无法生成 local-gen 之类的玩意，去命令行运行以下内容。

```bash
sudo vim /etc/locale.gen
# 取消zh_CN.UTF-8 UTF-8前面的注释

sudo locale-gen
reboot
```

### sddm 界面的中文

```bash
sudo vim /usr/lib/systemd/system/sddm.service

# 修改service小节的内容如下
[Service]
Environment=LANG=zh_CN.UTF-8

reboot
```

## plyomuth配置

将`plymouth.conf`文件复制到路径`/etc/systemd/system/display-manager.service.d`，这是自定义过的 spinner 主题配置\
将`spinner`文件夹覆盖到路径`/usr/share/plymouth/themes/`，这有助于 plymouth 显示界面和 sddm 的平滑切换

配置一下 initramfs

```bash
sudo vim /etc/mkinitcpio.conf
## 在HOOKS中的udev后面加上plymouth，保存

重新生成 initramfs
sudo mkinitcpio -P
```

为了防止内核日志的打印挤掉plymouth的界面，需要对内核启动参数做修改

```bash
sudo vim /etc/kernel/cmdline
# 加入以下内容
splash quiet loglevel=3 rd.udev.log_level=3 vt.global_cursor_default=0 systemd.show_status=no
```

kde 设置主题启动界面选择 spinner 主题，重启即可

## 中文输入法

中文输入法采用 fcitx5 + rime +雾凇+万象大模型的方案\
上面的软件安装完了以后，直接去 kde 设置界面点点点就能搞出来全拼了，记得在经典用户界面中把皮肤调成 kde plasma，然后把两份备份的配置文件放到 rime 配置文件目录下，分词模型文件也放到配置文件目录下，lua 脚本也放到对应的目录下，在状态栏点击重新部署即可。

**注意：** 万象大模型的分词文件`wanxiang-lts-zh-hans.gram`体积过大，不能直接提交Github。需要使用git lfs。

其实可以直接从[万象的 Github 发布页](https://github.com/amzxyz/RIME-LMDG/releases)下载模型，但是为了方便直接 clone 这个仓库就能开始重建，免去跑两趟下载的烦恼，我还是将其加入了这个仓库中。

```bash
# 使用 git lfs 跟踪大文件更改
git lfs track "*.gram"
```

在这之后正常使用 git 同步到Github就可以了

## discover 应用商店

这个东西一开始不好用，会各种报错，记得把软件源里面自带的 flathub 删除，然后重新添加 flathub 的软件源就可以用了。这个应用商店里面基本都是 flatpak 应用，运行时有沙箱机制，所以有些可能要重新设置权限。

## 指纹

指纹安装 fprintd 即可使用，录入指纹可以到设置里面的用户账户页面最底下添加。\
目前已知：

1. sddm 不能使用指纹登录，否则 kwallet 不能自动解锁；
2. su 不能使用指纹认证，出于安全的考虑，不建议给root用户添加生物识别信息。使用 su 时，会对 root 用户进行身份验证而不是当前使用的用户账户，因此当前用户账户的生物认证信息不能用于验证。同理，任何调用 su 进行认证的图形化软件，都不能使用指纹认证；

如果想让某个认证过程使用指纹，在`/etc/pam.d/`目录下的对应文件中加入

```text
auth sufficient pam_fprintd.so
```

这样就可以使用指纹认证了。

目前有需求的就是 sudo 和 polkit 身份认证代理程序这两个软件，只需要将备份的`sudo`和`polkit-1`文件复制到`/etc/pam.d/`目录下即可。

## 防火墙

防火墙使用 ufw，可以在 kde 的图形界面进行配置。如果需要使用rdp，使用以下命令。

``` bash
# 允许 3389 端口
sudo ufw allow 3389

# 拒绝 3389 端口
sudo ufw deny 3389
```

## 休眠

你好，咱们的机器经过我和[ChatGPT同学](https://chatgpt.com/c/68ddefbe-b9e4-8324-ab8c-0a5faac5aaa9)的一番折腾，终于支持休眠啦。然后您猜怎么着，休眠的速度慢的跟乌龟一样，都够我重启两三遍了，不建议在这个机器上使用休眠。

## 文件搜索

fsearch，装好就用

## 图形化git

github-desktop，装好就用

## 文件共享

文件共享使用`samba`作为后端，前端使用 kde 自带的右键菜单中的共享，即可实现 Windows 端和 Linux 端局域网中文件互传。

需要安装`samba`和`kdenetwork-filesharing`两个软件。

samba 的配置文件放到对应目录下即可。

这俩玩意装好并不是开箱即用的，`kdenetwork-filesharing`大概率会甩锅给`samba`说他没有配置好。这个时候跟随下面的步骤。

```bash
# 建立 usershares 目录，虽然不知道这个目录有什么用，但是如果没有 samba 就不会正常工作，因此咱们给他加上。
mkdir -p /var/lib/samba/usershares
groupadd sambashares
usermod -a -G sambashares sadak
chown root:sambashares /var/lib/samba/usershares
chmod 1770 /var/lib/samba/usershares

# 启动 samba 服务
systemctl restart smb
systemctl enable smb

smbpasswd -a sadak
# 输入希望为 samba 用户 sadak 设定的密码



# 最后把备份的配置文件放到指定路径下就可以了

# 配置文件内容解释
# usershare path = /var/lib/samba/usershares
# usershare allow guests = Yes
# usershare max shares = 10
# usershare owner only = False

# 第一条是你刚才的路径，第二条是说明如果你是否想让guest访问。第三条是你最多可以共享几个。第四条是设置共享的人是否一定是被共享目录的所有者。第一条是你刚才的路径，第二条是说明如果你是否想让guest访问。第三条是你最多可以共享几个。第四条是设置共享的人是否一定是被共享目录的所有者。
```

弄好了以后，在你希望共享的文件夹上面右键点击，选择共享，这个时候共享页面上应该没有任何报错，按需要设置即可。

`wsdd`这个软件包声称能解决Windows 端文件管理器中的“网络”界面不显示 Linux 主机的问题，但是我装了，没啥鸟用，还是得用`\\192.168.6.231\Share`这样的方式来访问。

## 光盘刻录

使用`k3b`进行光盘刻录，但是目前遇到的问题是，即使使用k3b推荐的 Linu /Unix+Windows 这样的兼容性光盘文件系统设置，刻录出来的 CD-R 仍然只能被 Linux 读取，Windows 上只能看到光盘有空间占用，但是点开没有任何文件。

目前仅在一台老 Windows 笔记本的自带光驱上进行了测试，需要更多测试以进一步排除问题。

## 视频播放

vlc，默认界面很丑，没有啥好用的，捏着鼻子用得了。

## 省电配置

使用`tlp`进行电源管理，不要安装`power-profile-daemon`，两者有冲突。

配置文件的位置在头部的注释里面，放到对应位置即可。

安装`tlpui`进行图形化管理，方便快捷。
