---
title: "ProxmoxVE 安装及相关配置"
date: 2020-07-14T17:36:06+08:00
tags: [proxmox-ve]
categories: [proxmox-ve, linux]
---

<!--more-->

### 分区
```
# 安装过程就不详细叙述了，只记录下分区情况
# 分区大小
swapsize 4
maxroot 20
minfree 0
```

### 替换源
```bash
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

tee /etc/apt/sources.list.d/pve-enterprise.list <<-'EOF'
# deb https://enterprise.proxmox.com/debian/pve buster pve-no-subscription
deb https://mirrors.tuna.tsinghua.edu.cn/proxmox/debian buster pve-no-subscription
EOF

apt update && apt dist-upgrade -y
```

### 去除登录时订阅通知
```bash
sed -i.bak "s/data.status !== 'Active'/false/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js
```

### 开启pveproxy ipv6支持
```bash
sed -i.bak  -r "s/(.*pve)/::ffff:\1/" /etc/hosts
systemctl restart pveproxy
# 检测
# netstat -anlpt | grep 8006 # 需要安装net-tools
# curl -vk "https://127.0.0.1:8006" 2>&1 | grep "OK"
# curl -vk "https://[::1]:8006" 2>&1 | grep "OK"
```

### 卸载旧内核
```bash
# !!!谨慎操作!!!
# 先重启加载新内核再卸载

dpkg -l | \
grep ii | \
awk '{print $2}' | \
grep -E 'pve-kernel(.+)-pve' | \
grep -v $(uname -r) | \
xargs apt-get purge -y
```

### 硬盘无法休眠
由于pvestatd进程一直都再扫描硬盘所以导致硬盘无法正常休眠，可以修改如下配置让pvestatd只检测特定的硬盘
```bash
# /etc/lvm/lvm.conf
...
global_filter = [ "r|/dev/zd.*|", "r|/dev/mapper/pve-.*|" "r|/dev/mapper/.*-(vm|base)--[0-9]+--disk--[0-9]+|"]
...

# 添加以下内容到 global_filter 数组前面
# 将/dev/disk/by-id/YOUR_SYSTEM_DISK*替换为你的系统盘，当sdX不确定的情况下最好使用硬盘id或uuid等来确定硬盘
# 这样即可让pvestat监测系统盘，忽略所有的sd*和md*
"a|/dev/disk/by-id/YOUR_SYSTEM_DISK*|", "r|/dev/sd.*|", "r|/dev/md.*|",

# 最后重启pvestatd，现在就可以使用hdparm或者hd-idle配置自动休眠
pvestatd restart
```