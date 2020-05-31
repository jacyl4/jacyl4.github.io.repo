---
title: "给国外/国内虚拟机DD debian 系统"
tags: [ "debian", "DD" ]
categories: [ "linux" ]
date: 2020-05-27
timezone: UTC+8
draft: false
---

#### 查看虚拟机的ip跟其上级ip
`ip route show`    
把查到的ip填入最下面的dd命令里。

#### 安装DD运行所需的依赖

当前系统为centos时
`yum install -y xz openssl gawk file`

当前系统为debian时
`apt install -y xz-utils openssl gawk file`

#### DD命令

##### 国内
```
bash <(wget --no-check-certificate -qO- http://moeclub.org/attachment/LinuxShell/InstallNET.sh | sed 's/8.8.8.8/119.29.29.29/') \
-d 9 -v amd64 -a \
-p password \
--mirror "http://mirrors.aliyun.com/debian" \
--ip-addr 192.168.1.2 \
--ip-mask 255.255.255.0 \
--ip-gate 192.168.1.1
```

##### 国外
```
bash <(wget --no-check-certificate -qO- http://moeclub.org/attachment/LinuxShell/InstallNET.sh | sed 's/8.8.8.8/1.1.1.1/') \
-d 9 -v amd64 -a \
-p password \
-apt "http://deb.debian.org/debian/debian" \
--ip-addr 10.0.0.2 \
--ip-mask 255.255.255.0 \
--ip-gate 10.0.0.1
```

* 9 就是debian9
* password 就是密码
