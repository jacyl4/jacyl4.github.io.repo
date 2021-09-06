---
title: "Debian 升级"
tags: [ "debian", "stretch", "buster", "bullseye" ]
categories: [ "linux" ]
date: 2020-12-10T00:00:00+08:00
draft: false
---

#### 一，Debian 9 升 10

```
cat << EOF >/etc/apt/sources.list
deb http://cloudfront.debian.net/debian buster main contrib non-free
deb http://cloudfront.debian.net/debian buster-updates main contrib non-free
deb http://cloudfront.debian.net/debian-security buster/updates main contrib non-free
EOF

sed -i "s/ stretch / buster /g" /etc/apt/sources.list.d/*
apt update && apt -y full-upgrade
apt -y --purge autoremove && apt -y autoclean
```

#### 二，Debian 10 升 11

```
cat << EOF >/etc/apt/sources.list
deb http://cloudfront.debian.net/debian bullseye main contrib non-free
deb http://cloudfront.debian.net/debian bullseye-updates main contrib non-free
deb http://cloudfront.debian.net/debian bullseye-backports main contrib non-free
deb http://cloudfront.debian.net/debian-security bullseye-security main contrib non-free
EOF

sed -i "s/ buster / bullseye /g" /etc/apt/sources.list.d/*
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
