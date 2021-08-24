---
title: "Debian 升级"
tags: [ "debian", "DD", "buster", "stretch" ]
categories: [ "linux" ]
date: 2020-12-10T00:00:00+08:00
draft: false
---

#### 一，Debian 9 升 10

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

```
apt update && apt -y full-upgrade
apt -y --purge autoremove && apt -y autoclean
```

#### 二，Debian 10 升 11

##### 国内
```
cat << EOF >/etc/apt/sources.list
deb http://mirrors.aliyun.com/debian bullseye main contrib non-free
deb http://mirrors.aliyun.com/debian bullseye-updates main contrib non-free
deb http://mirrors.aliyun.com/debian bullseye-backports main contrib non-free
deb http://mirrors.aliyun.com/debian-security bullseye-security main contrib non-free
EOF
```

##### 国外
```
cat << EOF >/etc/apt/sources.list
deb http://deb.debian.org/debian bullseye main contrib non-free
deb http://deb.debian.org/debian bullseye-updates main contrib non-free
deb http://deb.debian.org/debian bullseye-backports main contrib non-free
deb http://deb.debian.org/debian-security bullseye-security main contrib non-free
EOF
```

```
apt update && apt -y full-upgrade
apt -y --purge autoremove && apt -y autoclean
```

#### 三，重启查看当前系统版本
```
reboot
```

```
cat /etc/debian_version
```
