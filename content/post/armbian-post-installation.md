---
title: Armbian的常用配置
date: 2019-09-04T11:29:27+08:00
lastmod: 2019-09-06T10:39:10+08:00
tags: [armbian, n1]
categories: [linux, armbian]
---

最近入了个斐讯n1，安装了Armbian，在家里做个微服务器。记录下安装完成后常用的配置修改：
<!--more-->
### 修改dtb
```
cd /boot`sed -n "s#dtb_name=\(.*\)\/.*dtb#\1#p" /boot/uEnv.ini`
dtc -I dtb -O dts -o meson-gxl-s905d-phicomm-n1.dts meson-gxl-s905d-phicomm-n1.dtb > /dev/null 2>&1
cp meson-gxl-s905d-phicomm-n1.dts meson-gxl-s905d-phicomm-n1-mod.dts
sed -i "$(expr `cat -n meson-gxl-s905d-phicomm-n1-mod.dts | grep "linux,cma" | head -n 1 |awk '{print $1}'` + 3)s#38000000#8000000#" meson-gxl-s905d-phicomm-n1-mod.dts
sed -i "$(expr `cat -n meson-gxl-s905d-phicomm-n1-mod.dts | grep "interrupt-controller@9880" | head -n 1 |awk '{print $1}'` + 7)s#phandle.#\#\0#" meson-gxl-s905d-phicomm-n1-mod.dts
diff meson-gxl-s905d-phicomm-n1.dts meson-gxl-s905d-phicomm-n1-mod.dts
dtc -I dts -O dtb -o meson-gxl-s905d-phicomm-n1-mod.dtb meson-gxl-s905d-phicomm-n1-mod.dts > /dev/null 2>&1
rm -rf meson-gxl-s905d-phicomm-n1.dts meson-gxl-s905d-phicomm-n1-mod.dts
sed -i "s#\(dtb_name.*/\).*dtb#\1meson-gxl-s905d-phicomm-n1-mod.dtb#" /boot/uEnv.ini
grep dtb_name /boot/uEnv.ini
```

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

apt-get update && apt-get dist-upgrade -y
```

### 常规配置
```
# 安装常用工具
apt-get install -y tmux
# 修改时区
timedatectl set-timezone Asia/Shanghai
# 删除 LC_ALL默认配置
sed -i '/LC_ALL/d' /etc/environment
# swappiness参数设置与内存交换 
sed -i 's#\(^vm.swappiness=\).*#\110#' /etc/sysctl.conf
sysctl -p
```

### usb自动挂载
```
apt-get install -y udevil
# 修改默认挂载路径到/media
sed -i 's#\(^allowed_media_dirs = \)\(.*\)#\1\/media, \2#' /etc/udevil/udevil.conf
systemctl enable devmon@your-username
systemctl start devmon@your-username
```

### 安装oh-my-zsh
```
apt-get install -y zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)" "" --unattended
# 替换主题
sed -i 's#\(^ZSH_THEME="\).*\("\)#\1ys\2#' ~/.zshrc
# 修改默认终端（需要验证密码）
chsh -s $(which zsh) && zsh
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
    "https://dockerhub.azk8s.cn",
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