---
title: "Debian 9 升级 10"
tags: [ "debian", "DD", "buster", "stretch" ]
categories: [ "linux" ]
date: 2020-12-10T00:00:00+08:00
draft: false
---

#### 一，换源

##### 国内
```
cat << EOF >/etc/apt/sources.list
deb http://mirrors.aliyun.com/debian buster main contrib non-free
deb http://mirrors.aliyun.com/debian buster-updates main contrib non-free
deb http://mirrors.aliyun.com/debian-security buster/updates main contrib non-free
EOF
```

##### 国外
```
cat << EOF >/etc/apt/sources.list
deb http://deb.debian.org/debian buster main contrib non-free
deb http://deb.debian.org/debian buster-updates main contrib non-free
deb http://deb.debian.org/debian-security buster/updates main contrib non-free
EOF
```

#### 二，更新升级
```
apt update && apt upgrade -y
apt dist-upgrade
```

#### 三，重启查看当前系统版本
```
reboot
```

```
cat /etc/debian_version
```
