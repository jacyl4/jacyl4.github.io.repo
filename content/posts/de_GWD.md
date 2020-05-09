---
title: "de_GWD"
tags: [ "de_GWD", "旁路" ]
categories: [ "路由" ]
date: 2019-03-26
draft: false
---

[Github](https://github.com/jacyl4/de_GWD)

de_GWD(Debian Gateway&DNS)专注旁路，运行于debian的一个完善套件，带界面，纯粹是为了追求更高效更极速。

DNS去污染方式有别于以往各种，效率不是以往LEDE/梅林等兼容方案能比拟的。

需要64位，支持普通amd64平台 以及 树莓派，香橙派 等 arm64 平台。

![de_GWD](https://i.loli.net/2020/02/26/Sk7awvCJTLsUh8D.png)

# 部署

提前给自己的vps kvm 小机 准备好域名。哪怕是免费的只要能给cloudflare托管就行。二级域名不行。 脚本开始安装的时候，要制作证书的。

![dns record](https://i.loli.net/2019/04/04/5ca5beea00c91.png)

## 服务端

![Snipaste_2020-05-09_08-33-46.png](https://i.loli.net/2020/05/09/NbAYPetxiHWjKql.png)

ssh登录vps

```
apt install -y wget
bash <(wget --no-check-certificate -qO- https://raw.githubusercontent.com/jacyl4/de_GWD/master/server)
```
直接输出上面做好a记录的自己的域名。期间会自动生成uuid跟path。
脚本结束会打印出uuid跟path。

附：
[在线生成UUID](https://www.uuidgenerator.net/)

## 客户端

![Snipaste_2020-05-09_08-34-34.png](https://i.loli.net/2020/05/09/r1etxqvofSXlOGJ.png)

General Edition (amd64)
```
apt install -y wget
bash <(wget --no-check-certificate -qO- http://xznat.seso.icu:10290/client_do)
```

![Snipaste_2020-05-09_08-34-08.png](https://i.loli.net/2020/05/09/YKIRUT6JHbS71ak.png)

Compatible Edition (amd64&arm64)
```
apt install -y wget
bash <(wget --no-check-certificate -qO- http://xznat.seso.icu:10290/client)
```

通常用第一个脚本，如果是armbian平台的话，才选第二个脚本。

首次安装前，先维持上级路由的dhcp是普通状态，确保debian能正常获取ip联网。

直接联网安装，不需要挂代理。

选项2，可以用来强制重置pihole密码。

装完后，关闭上级路由的DHCP服务，在web UI上开启de_GWD的DHCP服务。

有公网ip的话，可以选项8安装wireguard组件。

- 自动每四小时校时

- 自动每天凌晨更新分流规则

更细致的疑问，[电报群](https://t.me/de_GWD)联系吧