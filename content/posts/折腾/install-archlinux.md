+++
title = "记录安装 ArchLinux"
date = "2025-04-16T00:00:00+08:00"
categories = ["折腾", "ArchLinux"]
tags = ["安装ArchLinux"]
+++

## 目的

记录安装 archlinux 过程, 方便下次装机。

## 准备

如果没有网线, 则使用手机的 USB 网络共享功能即可。

## 安装系统

```sh
# ## Check
# UEFI OR BOIS
ls /sys/firmware/efi/efivars

# ## Time
timedatectl set-ntp true
timedatectl status

# ## Keymap
loadkeys us

# ## Network
dhcpcd

# ## Disk partion
fdisk /dev/sda

-   /boot: 1G
-   /: 50G
-   swap: 分区大小是的内存大小的 1 或 1.5 倍
-   /home: 剩余的容量
-   /opt: 如果是安装比较大的软件则根据需要划分一个分区再挂上即可。等根分区的容量不够再处理。

# boot
mkfs.fat -F32 /dev/sda1
# swap
mkswap /dev/sda2
swapon /dev/sda2
# root
mkfs.ext4 /dev/sda3
# home
mkfs.ext4 /dev/sda4

mount /dev/sda3 /mnt
mkdir boot
mount /dev/sda1 /mnt/boot
mkdir home
mount /dev/sda4 /mnt/home

# ## Pacman mirrorlist
#reflector -c China -a 10 --sort rate --save /etc/pacman.d/mirrorlist
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch

# ## Install System
pacstrap /mnt base linux linux-firmware
pacstrap /mnt dhcpcd neovim
# optional
pacstrap /mnt base-devel

# ## 生成 fstab 文件
genfstab -U /mnt >> /mnt/etc/fstab

# ## 进入到安装的系统
arch-chroot /mnt

# ## Grub
pacman -S grub efibootmgr
# （optional）如果是双系统且另一个系统是 windows
pacman -S os-prober ntfs-3g

grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg

# ## 系统的基本配置
# ### 时区
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
timedatectl set-ntp true
# ### hostname, hosts
# /etc/hostname
ArchLinux
# /etc/hosts
127.0.0.1	localhost
::1		localhost
127.0.0.1	ArchLinux.localdomain ArchLinux
# ### locale
# /etc/locale.gen
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8

locale-gen
# /etc/locale.conf
LANG=en_US.UTF-8

# ### 字体
pacman -S ttf-dejavu wqy-microhei
pacman -S ttf-jetbrains-mono-nerd       # optional. 安装附带有文字图形的字体

# Users
passwd root
useradd -m johan
passwd johan
usermod -aG wheel johan         # for sudo

# ## reboot
exit
umount -R /mnt
reboot

# ## Keymap
loadkeys us         # 临时
localectl set-keymap --no-convert us        # 永久

# ## Pacman
# /etc/pacman.conf
[archlinuxcn]
#Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch

pacman -S archlinuxcn-keyring
pacman -Syu
pacman -S paru
```

## 显卡

开源驱动:
-   [Nouveau - Arch Linux 中文维基](https://wiki.archlinuxcn.org/wiki/Nouveau)
-   [NVIDIA - Arch Linux 中文维基](https://wiki.archlinuxcn.org/wiki/NVIDIA)

### 查看进程在使用哪个显卡驱动 (检测是否安装成功)

```sh
# ## xf86-video-nouveau
lspci -k | grep -A 2 -E "(VGA|3D)"
ls -l /dev/dri/by-path
sudo lsof /dev/dri/card1        # 这些进程通常是与图形渲染和 GPU 加速相关的应用程序。
sudo lsof /dev/dri/renderD128   # 这些进程通常是与图形渲染任务和 GPU 渲染相关的应用程序。

# ## nvidia
nvidia-smi
```

### 显卡切换

See [nvidia-prime](https://wiki.archlinuxcn.org/wiki/PRIME)

```sh
paru -S nvidia-prime

xrandr --listproviders
xrandr --setprovideroffloadsink 0xce 0x43
prime-run glxinfo | grep "OpenGL renderer"          # 使用独显运行 glxinfo 程序
DRI_PRIME=1 glxinfo | grep "OpenGL renderer"        # 使用独显运行 glxinfo 程序
```

## 声音

安装 ALSA 和基础工具:

```sh
paru -S alsa-utils alsa-firmware alsa-lib alsa-plugins
```

安装 PipeWire:

```sh
paru -S pipewire pipewire-alsa pipewire-pulse
```

启动 pipewire 并开机自启:

```sh
systemctl --user enable --now pipewire pipewire-pulse
```

安装声音控制工具:

```sh
paru -S pavucontrol
```

运行 `pavucontrol` 并设默认的声卡即可。

### References

-   [ALSA](https://wiki.archlinuxcn.org/wiki/ALSA)
-   [PipeWire](https://wiki.archlinuxcn.org/wiki/PipeWire)

## 网络

```sh
paru -S linux-headers
paru -S broadcom-wl-dkms        # 要安装相应的 wifi 驱动
paru -S iwd                     # optional

# 连接网络（重启之后）
# systemctl start iwd
# systemctl start systemd-resolved

# ### networkmanager
# networkmanager 包含了 nmcli, nmtui（Text User Interface） 这两个 client。
paru -S networkmanager network-manager-applet
paru -S impala

networkmanager 使用 iwd 连接 wifi. See [ref](https://wiki.archlinuxcn.org/wiki/NetworkManager#Using_iwd_as_the_Wi-Fi_backend).
```

## 蓝牙

```sh
lsusb | grep Bluetooth              # 查看蓝牙型号
paru -S broadcom-bt-firmware        # install firmware `Broadcom Corp. BCM43142 Bluetooth`.
paru -S bluez bluez-utils blueman
lsmod | grep btusb
systemctl enable --now bluetooth.service

blueman-manager
blueman-applet
```

### 连接蓝牙

blueman:

```sh
blueman-manager
blueman-applet
```

bluetoothctl:

```sh
bluetoothctl
agent on
default-agent
power on        # 打开蓝牙

scan on         # 开始扫描
pair <mac>      # 配对
# [agent] Enter PIN code: 1234
trust <mac>
connect <mac>

# ## optional
pairable off
discoverable off
```

## Others

```sh
paru -S intel-ucode
# OR
paru -S amd-ucode
```
