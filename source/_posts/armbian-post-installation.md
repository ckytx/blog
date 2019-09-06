---
title: Armbian的常用配置
date: 2019-09-04 11:29:27
updated: 2019-09-06 10:39:10
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
apt-get install -y udevil
# 修改默认挂载路径到/media
sed -i 's/allowed_media_dirs\ =\ \/media\/$USER,\ \/run\/media\/$USER/allowed_media_dirs\ =\ \/media,\ \/media\/$USER,\ \/run\/media\/$USER/' /etc/udevil/udevil.conf
systemctl start devmon@your-username.service
systemctl enable devmon@your-username.service
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