---
title: Armbian的常用配置
date: 2019-09-04 11:29:27
tags:
  - armbian
  - n1
categories:
  - linux
  - armbian
---

最近入了个斐讯n1，安装了Armbian，在家里做个微服务器。记录下安装完成后常用的配置修改：
<!--more-->
### 配置国内镜像
```
tee /etc/apt/sources.list <<-'EOF'
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free
EOF

tee /etc/apt/sources.list.d/armbian.list <<-'EOF'
# deb http://apt.armbian.com buster main buster-utils buster-desktop
deb https://mirrors.tuna.tsinghua.edu.cn/armbian/ buster main buster-utils buster-desktop
EOF
```
### 常规配置
```
# 修改时区
timedatectl set-timezone Asia/Shanghai
# 删除 LC_ALL默认配置
sed -i '/LC_ALL/d' /etc/environment
# swappiness参数设置与内存交换 
sed -i 's/vm.swappiness=100/vm.swappiness=10/' /etc/sysctl.conf
sysctl -p
```
### 安装oh-my-zsh
```
apt-get install -y zsh
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
# 替换主题
sed -i 's/ZSH_THEME="robbyrussell"/ZSH_THEME="ys"/' ~/.zshrc
```
### usb自动挂载
```
# 创建udev规则
tee /etc/udev/rules.d/10-usb-mount.rules <<-'EOF'
KERNEL!="sd*", GOTO="exit"
SUBSYSTEM!="block", GOTO="exit"
IMPORT{builtin}="blkid"
ENV{ID_FS_LABEL}=="EFI|BOOT|Recovery|RECOVERY|SETTINGS|boot|root0|share0", GOTO="exit"
ENV{DEVTYPE}!="partition", GOTO="exit"
ENV{ID_FS_USAGE}!="filesystem", GOTO="exit"
ENV{ID_FS_LABEL}!="", ENV{dir_name}="%E{ID_FS_LABEL}"
ENV{ID_FS_LABEL}=="", ENV{dir_name}="Untitled-%k"
ACTION=="add", PROGRAM="/bin/sh -c '/bin/grep -E ^/dev/%k\  /proc/mounts || true'", RESULT=="", RUN+="/usr/bin/systemd-mount --no-block --automount=yes --collect /dev/%k /media/%E{dir_name}"
ACTION=="remove", ENV{dir_name}!="", RUN+="/usr/bin/systemd-mount --umount /media/%E{dir_name}", RUN+="/bin/rmdir /media/%E{dir_name}"
LABEL="exit"
EOF

#执行命令重新加载配置
udevadm control --reload
```

### 安装docker
```
#安装
export DOWNLOAD_URL=https://mirrors.tuna.tsinghua.edu.cn/docker-ce
sh -c "$(curl -fsSL https://get.docker.com)"

#配置加速镜像
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn"
  ]
}
EOF

systemctl daemon-reload
systemctl restart docker

#安装GUI工具
docker volume create portainer_data
docker run -d -p 9000:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
```