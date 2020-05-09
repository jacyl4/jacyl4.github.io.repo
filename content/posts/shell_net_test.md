---
title: "终端内用于查看网络情况的指令"
tags: [ "debian", "shell" ]
categories: [ "linux" ]
date: 2020-05-09
draft: false
---

#### 列出适配器名称

`ifconfig | grep -E -o "^[a-z0-9]+" | grep -v "lo" | uniq`

<br>

####查看端口占用

`netstat -anp|grep 5390`

或

`lsof -i:5390`

<br>

#### 查看进程pid

`ps -ef | grep nginx`

<br>

#### 根据pid查看端口占用

`netstat -nap | grep 10191`

<br>

#### 查看连接数

`netstat -n | awk '/^tcp/ {++state[$NF]} END {for(key in state) print key,"t",state[key]}'`

<br>

#### 查看tcpfastopen

`grep '^TcpExt:' /proc/net/netstat | cut -d ' ' -f 87-92 | column -t`

<br>

#### 入口流量信息统计GB (ens18)

`echo "$(awk -v eth=ens18 -F'[: ]+' '{if ($0 ~eth){print $3,$11}}' /proc/net/dev | awk '{print $1}') /1024 /1024 /1024" | bc`

<br>

#### 出口流量信息统计GB (ens18)

`echo "$(awk -v eth=ens18 -F'[: ]+' '{if ($0 ~eth){print $3,$11}}' /proc/net/dev | awk '{print $2}') /1024 /1024 /1024" | bc`

<br>

