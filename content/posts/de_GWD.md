---
title: "de_GWD"
tags: [ "de_GWD", "旁路" ]
categories: [ "路由" ]
date: 2019-03-26
draft: false
---

[Github](https://github.com/jacyl4/de_GWD)

de_GWD(Debian bypass Gateway & DNS)专注旁路，运行于debian的一个完善套件，带界面，纯粹是为了追求更高效更极速。    

DNS去污染方式有别于以往各种，效率不是以往LEDE/梅林等兼容方案能比拟的。    

需要64位，支持普通amd64平台 以及 树莓派，香橙派 等 arm64 平台。    

![de_GWD](https://i.loli.net/2020/02/26/Sk7awvCJTLsUh8D.png)    
内网设备分流ip，用空格分隔。    
只有一个doh地址的时候，doh1跟doh2填同一个地址。    

# 部署

提前给自己的vps kvm 小机 准备好域名。哪怕是免费的只要能给cloudflare托管就行。二级域名不行。 脚本开始安装的时候，要制作证书的。

![dns record](https://i.loli.net/2019/04/04/5ca5beea00c91.png)    

## 服务端

![Snipaste_2020-05-12_01-01-12.png](https://i.loli.net/2020/05/12/bKfZG3BXWJ9wjD1.png)


提前准备好域名，做好a记录。    

脚本结束会自动打印出doh，域名，uuid，path等信息。/ 或者用选项11来复查    

服务端默认有自动更新功能，间歇查询github上的版本号，与本地比对，不一致后会自动生成计划任务，+8 凌晨4点30开始自动更新（pihole组件除外）

nginx配置方面，如需要vps上同时运行wordpress nextcloud等程序。    

/etc/nginx/conf.d    

supp_head 用于存放nginx upstream模块的内容    

supp_body 用于存放wordpress nextcloud等程序的伪静态内容    

然后，平常更新时就可以共存了。    

附：
[在线生成UUID](https://www.uuidgenerator.net/)

## 客户端


General Edition (amd64)
![Snipaste_2020-05-12_00-58-45.png](https://i.loli.net/2020/05/12/EdTgwehSUJGpLaB.png)    


Compatible Edition (amd64&arm64)
![Snipaste_2020-05-12_00-59-03.png](https://i.loli.net/2020/05/12/SYc8MIi3pzTWDla.png)



通常用第一个脚本，如果是armbian平台的话，才选第二个脚本。如果x86 cpu比较落后，不能运行docker，那也只能选第二个版本。

首次安装前，先维持上级路由的dhcp是普通状态，确保debian能正常获取ip联网。

直接联网安装，不需要挂代理。

选项2，可以用来强制重置pihole密码。

装完后，关闭上级路由的DHCP服务，在web UI上开启de_GWD的DHCP服务。


有公网ip的话，可以选项8安装wireguard组件。

wireguard组件，在每次debian内核更新后，需要重新编译安装。

- 自动每二小时校时

- 自动每天凌晨更新分流规则    

    
    



更多疑问 

[![Telegram](https://cdn.rawgit.com/Patrolavia/telegram-badge/8fe3382b/chat.svg)](https://t.me/de_GWD)  
