---
title: "RouterOS电信移动聚合实例"
tags: [ "ros", "分流" ]
categories: [ "路由" ]
date: 2020-04-22
timezone: UTC+8
draft: false
---


实例宽带接入：

电信（双拨）

移动

# 一，建立vrrp用于拨号

提前摸清ros里的网口顺序，实例中，电信插eth1口，移动插eth2口，桥接接口名为bridge1，给bridge1分配个ip，比如10.0.0.2，就是ros地址。
```
/interface vrrp add name=vrrp1 interface=eth1 vrid=1
/interface vrrp add name=vrrp2 interface=eth1 vrid=2
/interface vrrp add name=vrrp4 interface=eth2 vrid=3
```

给端口们预先分配好ip，随自己的习惯分配好了，因为vrrp需要预设好ip才会显示已连接
```
/ip address add address=192.168.10.1/24 interface=eth1
/ip address add address=192.168.20.1/24 interface=eth2
/ip address add address=192.168.10.11/24 interface=vrrp1
/ip address add address=192.168.10.12/24 interface=vrrp2
/ip address add address=192.168.20.11/24 interface=vrrp3
```

# 二，建立pppoe拨号

```
/interface pppoe-client add name=pppoe-CT1 max-mtu=1480 max-mru=1480 interface=vrrp1 user=宽带帐号 password=宽带密码 add-default-route=no disable=no
/interface pppoe-client add name=pppoe-CT2 max-mtu=1480 max-mru=1480 interface=vrrp2 user=宽带帐号 password=宽带密码 add-default-route=no disable=no
/interface pppoe-client add name=pppoe-CMCC1 max-mtu=1480 max-mru=1480 interface=vrrp3 user=宽带帐号 password=宽带密码 add-default-route=no disable=no
```

# 三，防火墙基础防护

下面第五行"src-address=10.0.0.0/24"，这个是我内网的网段，表示该ip段可以连入ros，进行设置。根据自己情况改。
```
/ip firewall filter
add chain=input connection-state=invalid action=drop comment="Drop Invalid connections"  
add chain=input connection-state=established action=accept comment="Allow Established connections"  
add chain=input protocol=icmp action=accept comment="Allow ICMP"  
add chain=input src-address=10.0.0.0/24 action=accept in-interface=bridge1
add chain=input action=drop comment="Drop everything else"

add chain=output action=accept comment="accept everything"

add chain=forward connection-state=invalid action=drop comment="Drop Invalid connections"
add chain=forward connection-state=established action=accept comment="Allow Established connections"
add chain=forward connection-state=related action=accept comment="allow related connections"

add chain=forward protocol=tcp action=jump jump-target=tcp
add chain=forward protocol=udp action=jump jump-target=udp
add chain=forward protocol=icmp action=jump jump-target=icmp

add chain=input protocol=tcp psd=21,3s,3,1 action=drop comment="Port scanners"
add chain=input protocol=tcp tcp-flags=fin,!syn,!rst,!psh,!ack,!urg action=drop comment="NMAP FIN Stealth scan"
add chain=input protocol=tcp tcp-flags=fin,syn action=drop comment="SYN/FIN scan"
add chain=input protocol=tcp tcp-flags=syn,rst action=drop comment="SYN/RST scan"
add chain=input protocol=tcp tcp-flags=fin,psh,urg,!syn,!rst,!ack action=drop comment="FIN/PSH/URG scan"
add chain=input protocol=tcp tcp-flags=fin,syn,rst,psh,ack,urg action=drop comment="ALL/ALL scan"
add chain=input protocol=tcp tcp-flags=!fin,!syn,!rst,!psh,!ack,!urg action=drop comment="NMAP NULL scan"
```

# 四，建立nat伪装与端口映射

```
/ip firewall nat
add chain=srcnat out-interface=pppoe-CT1 action=masquerade
add chain=srcnat out-interface=pppoe-CT2 action=masquerade
add chain=srcnat out-interface=pppoe-CMCC1 action=masquerade

add chain=dstnat protocol=tcp dst-port=1-65535 in-interface=pppoe-CT1 action=dst-nat to-addresses=10.0.0.5 to-ports=1-65535
```

# 五，PCC宽带聚合

## 0，导入国内运营商ip段

下载文件 [ros-dpbr-CT-CMCC.rsc](https://raw.githubusercontent.com/jacyl4/ros-pbr-CT-CMCC/master/ros-dpbr-CT-CMCC.rsc)

导入winbox的Files里

运行如下，把ip段导入ros firewall的address lists里，供下面标记时使用。（防止重复导入，前两行是删除现有的电信段与移动段）
```
/ip firewall address-list remove [find list="dpbr-CT"]
/ip firewall address-list remove [find list="dpbr-CMCC"]
/import ros-dpbr-CT-CMCC.rsc
```

## 1，排除内网通讯

```
/ip firewall address-list
add address=10.0.0.0/24 list=local comment=local

/ip firewall mangle
add chain=prerouting src-address-list=local dst-address-list=local action=accept comment="local"
```

## 2，源进标记

```
/ip firewall mangle
add chain=prerouting connection-mark=no-mark in-interface=pppoe-CT1 src-address-list=dpbr-CT action=mark-connection new-connection-mark=CT_conn1 passthrough=yes comment=in
add chain=prerouting connection-mark=no-mark in-interface=pppoe-CT2 src-address-list=dpbr-CT action=mark-connection new-connection-mark=CT_conn2 passthrough=yes
add chain=prerouting connection-mark=no-mark in-interface=pppoe-CMCC1 src-address-list=dpbr-CMCC action=mark-connection new-connection-mark=CMCC_conn1 passthrough=yes
add chain=prerouting connection-mark=no-mark in-interface=pppoe-CT1 action=mark-connection new-connection-mark=CT_conn1 passthrough=yes
add chain=prerouting connection-mark=no-mark in-interface=pppoe-CT2 action=mark-connection new-connection-mark=CT_conn2 passthrough=yes
add chain=prerouting connection-mark=no-mark in-interface=pppoe-CMCC1 action=mark-connection new-connection-mark=CMCC_conn1 passthrough=yes
```

## 3，v2线路标记（可选）

示例：

111.111.111.111是搬瓦工vps ip，注释名称bwg，走电信线路

222.222.222.222是CloudCone vps ip，注释名称cc，走移动线路
```
/ip firewall address-list
add address=111.111.111.111 list=CTv2 comment=bwg
add address=222.222.222.222 list=CMv2 comment=cc
```

```
/ip firewall mangle
add chain=prerouting connection-mark=no-mark in-interface=bridge1 dst-address-list=CTv2 action=mark-connection new-connection-mark=CT_conn1 passthrough=yes comment=v2
add chain=prerouting connection-mark=no-mark in-interface=bridge1 dst-address-list=CMv2 action=mark-connection new-connection-mark=CMCC_conn1 passthrough=yes
```

## 4，PCC标记

先做国内不同运营商指定出口，因为电信双拨，双拨的还得PCC聚合下，至于叠不叠带宽，各地随缘了。移动就单拨就直接标记一下就行了。
```
/ip firewall mangle
add chain=prerouting connection-mark=no-mark in-interface=bridge1 per-connection-classifier=both-addresses-and-ports:2/0 dst-address-list=dpbr-CT action=mark-connection new-connection-mark=CT_conn1 passthrough=yes comment="PCC spec"
add chain=prerouting connection-mark=no-mark in-interface=bridge1 per-connection-classifier=both-addresses-and-ports:2/1 dst-address-list=dpbr-CT action=mark-connection new-connection-mark=CT_conn1 passthrough=yes
add chain=prerouting connection-mark=no-mark in-interface=bridge1 dst-address-list=dpbr-CMCC action=mark-connection new-connection-mark=CMCC_conn1 passthrough=yes
```

ros防火墙规则自上而下顺序匹配，上面没匹配到的，就接下来整体聚合。
```
/ip firewall mangle
add chain=prerouting connection-mark=no-mark in-interface=bridge1 per-connection-classifier=both-addresses-and-ports:3/0 dst-address-type=!local action=mark-connection new-connection-mark=CT_conn1 passthrough=yes comment=PCC
add chain=prerouting connection-mark=no-mark in-interface=bridge1 per-connection-classifier=both-addresses-and-ports:3/1 dst-address-type=!local action=mark-connection new-connection-mark=CT_conn2 passthrough=yes
add chain=prerouting connection-mark=no-mark in-interface=bridge1 per-connection-classifier=both-addresses-and-ports:3/2 dst-address-type=!local action=mark-connection new-connection-mark=CMCC_conn1 passthrough=yes
```

## 5，让数据根据上面线路标记选择路由
```
/ip firewall mangle
add chain=prerouting connection-mark=CT_conn1 in-interface=bridge1 action=mark-routing new-routing-mark=CT1 passthrough=yes comment="dynamic pbr"
add chain=prerouting connection-mark=CT_conn2 in-interface=bridge1 action=mark-routing new-routing-mark=CT2 passthrough=yes
add chain=prerouting connection-mark=CMCC_conn1 in-interface=bridge1 action=mark-routing new-routing-mark=CMCC1 passthrough=yes

add chain=output connection-mark=CT_conn1 action=mark-routing new-routing-mark=CT1 passthrough=yes comment=out
add chain=output connection-mark=CT_conn2 action=mark-routing new-routing-mark=CT2 passthrough=yes
add chain=output connection-mark=CMCC_conn1 action=mark-routing new-routing-mark=CMCC1 passthrough=yes
```

# 六，设置路由
```
/ip route
add dst-address=0.0.0.0/0 gateway=pppoe-CT1 check-gateway=ping distance=1
add dst-address=0.0.0.0/0 gateway=pppoe-CT2 check-gateway=ping distance=2
add dst-address=0.0.0.0/0 gateway=pppoe-CMCC1 check-gateway=ping distance=2
add dst-address=0.0.0.0/0 gateway=pppoe-CT1 check-gateway=ping distance=1 routing-mark=CT1
add dst-address=0.0.0.0/0 gateway=pppoe-CT2 check-gateway=ping distance=1 routing-mark=CT2
add dst-address=0.0.0.0/0 gateway=pppoe-CMCC1 check-gateway=ping distance=1 routing-mark=CMCC1
```


# 乖巧 ☆⌒(*＾-゜)v THX!!

![给个赏吧](https://i.loli.net/2020/04/22/EaMjS1J8yfrVv4N.png)

