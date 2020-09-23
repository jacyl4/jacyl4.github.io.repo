---
title: "de_GWD"
tags: [ "de_GWD", "旁路" ]
categories: [ "路由" ]
date: 2019-03-26T00:00:00+08:00
draft: false
---

[Github](https://github.com/jacyl4/de_GWD)

de_GWD(Debian bypass Gateway & DNS)专注旁路，运行于debian的一个完善套件，带界面，纯粹是为了追求更高效更极速。    

DNS去污染方式有别于以往各种，效率不是以往LEDE/梅林等兼容方案能比拟的。    

双端配合，其于linux debian，特调tcp参数，可以从内核层面，略微提升连接性能。    

需要64位，支持普通amd64平台 以及 树莓派，香橙派 等 arm64 平台。    

![de_GWD](https://i.loli.net/2020/09/23/6J9PYjFbR2ocVq4.png)

- 只有一个doh地址的时候，doh1跟doh2填同一个地址。不要留空。    

# 部署

提前给自己的vps kvm 小机 准备好域名。哪怕是免费的只要能给cloudflare托管就行。二级域名不行。 脚本开始安装的时候，要制作证书的。

![dns record](https://i.loli.net/2019/04/04/5ca5beea00c91.png)    

## 服务端

![server](https://i.loli.net/2020/06/10/qWVbimIha9s1UeJ.png)


脚本结束会自动打印出doh，域名，uuid，path等信息。或者用 选项11 来复查    

服务端默认有自动更新功能，间歇查询github上的版本号，与本地比对，不一致后会自动生成计划任务，+8 凌晨4点30开始自动更新（pihole组件除外）

- nginx配置方面，如需要vps上同时运行wordpress nextcloud等程序。    

      /etc/nginx/conf.d    

      supp_head 用于存放nginx upstream模块的内容    

      supp_body 用于存放wordpress nextcloud等程序的伪静态内容    

      然后，平常更新时就可以共存了。    


## 客户端


General Edition (amd64)
![client_do](https://i.loli.net/2020/06/10/F1mkOphUEBTgxVr.png)    


Compatible Edition (amd64&arm64)
![client](https://i.loli.net/2020/06/10/iJ2vKInt8VNBWYz.png)


需要以root用户安装

通常用第一个脚本，如果是armbian平台的话，才选第二个脚本。如果x86 cpu比较落后，不能运行docker，那也只能选第二个版本。

首次安装时，确保debian能正常获取ip联网。

直接联网安装，不需要挂代理。

- 选项2，可以用来强制重置de_GWD与pihole密码。

- web ui 默认需要以https访问 （nat机上需要提前映射好443端口，以访问web ui）

- chrome内核浏览器打不开自签名证书网站的时候，在那个页面盲打 thisisunsafe 就能访问了
  
- 自动每二小时校时

- 自动每天凌晨 4:00 更新分流规则

- UDP代理开关的存在，是缘于v2本身的udp代理性能不佳。关闭udp代理后，可以让本地局域网中专门的游戏代理工具更好的工作。

- 组件通过install按钮安装后，刷新页面显示选项（页面能看到选项 即表示 组件已装上）
   - frp
      - 可以用于映射一个内网的服务端口，也可以用于内网穿透转接wireguard

      - 需要大陆服务器，或者nat机，提前确认好服务端端口

      - Bind-Port即de_GWD 到 大陆服务器 之前建立连接进行内网穿透的端口

      - Bind-Port，服务器端口，本地端口，可以为同一个，但是确认好tcp或udp

      - 生成的安装指令，可以在大陆服务器上直接快速安装与本地相匹配的frp服务端

      - 转接wireguard时，wireguard的endpoint域名与端口，直接填frp的服务器域名与服务器端口

      - kcp与wireguard皆为udp

- 更新出错时，可以点web ui上的救援按钮，再次进行更新。

***


更多疑问 

[![Telegram](https://cdn.rawgit.com/Patrolavia/telegram-badge/8fe3382b/chat.svg)](https://t.me/de_GWD)  
