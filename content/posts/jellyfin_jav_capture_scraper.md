---
title: "Jellyfin 影库 JAV 刮削"
tags: [ "pve", "lxc", "jellyfin", "scraper" ]
categories: [ "linux" ]
date: 2020-06-28T00:00:00+08:00
draft: false
---


![gallery1.jpg](https://i.loli.net/2020/06/28/wbysakCxvQHLnIe.jpg)



#### 环境准备
预先规划好IP地址    
NAS IP: 10.0.0.4    
Jellyfin IP: 10.0.0.6

我JAV的文件在群晖的 `/volume1/Transmission/JAV` 文件夹下

在 PVE 上， 新建个 debian 的 lxc 容器    
新建lxc容器的时候，Unprivileged container: 不需要勾选

运行容器前，点击 即将要安装 Jellyfin 的容器， options， 编辑最后一个选项 Features ，勾上NFS。

#### 登录群晖，开启群晖的nfs共享。

控制面板--文件服务， 最底下， NFS

<img src="https://i.loli.net/2020/06/28/6ytReTVlPpmi5Jo.png" width="450">

控制面板--共享文件夹， 编辑共享文件夹， NFS权限 选项卡，

<img src="https://i.loli.net/2020/06/28/pAtg6UMTGFSfjXP.png" width="450">


## 搭建Jellyfin

#### 通过nfs连接nas存储

ssh登录jellyfin所在的 pve lxc debian 容器，把群晖里的Transmission文件夹挂载到 pve lxc debian容器的 `/anas`
```
mkdir -p /anas
cat << EOF >/etc/fstab
# UNCONFIGURED FSTAB FOR BASE SYSTEM
10.0.0.4:/volume1/Transmission /anas nfs nfsvers=3,hard,timeo=600,retrans=2,noatime,nodiratime,_netdev 0 0
EOF
systemctl daemon-reload
```

#### APT安装Jellyfin
```
apt install apt-transport-https gnupg2
wget -O - https://repo.jellyfin.org/debian/jellyfin_team.gpg.key | apt-key add -
echo "deb [arch=$(dpkg --print-architecture)] https://repo.jellyfin.org/debian $(cat /etc/os-release | grep VERSION= | cut -d'(' -f2 | cut -d')' -f1) main" > /etc/apt/sources.list.d/jellyfin.list

apt update
apt install jellyfin ffmpeg nginx

usermod -aG postfix jellyfin 
systemctl restart jellyfin

chmod -R 755 /anas
chown -R jellyfin:jellyfin /anas

reboot
```

至此，已经可以用 Jellyfin IP: `10.0.0.6:8096` 通过浏览器来访问jellyfin了。

#### 通过 AV_Data_Capture 分类整理 JAV 文件夹
https://github.com/yoshiko2/AV_Data_Capture/releases

就两个文件需要用的。AV_Data_Capture和config.ini    
登录pve lxc debian jellyfin, 进入挂载过的JAV文件夹，放入这两个文件

编辑config.ini
```
[common]
main_mode=2
failed_output_folder=failed
success_output_folder=JAV_output
soft_link=0

[proxy]
;proxytype: http or socks5
type=http
proxy=
timeout=10
retry=3

[Name_Rule]
location_rule=actor+'/'+'['+year+'] '+number
naming_rule=number+'-'+title

[update]
update_check=1

[priority]
website=javbus,javdb,fanza,xcity,mgstage,fc2,avsox,jav321,javlib

[escape]
literals=\()/
folders=failed,JAV_output

[debug_mode]
switch=0
```
在这个JAV文件夹路径下，`./AV_Data_Capture` 就开始移动整理视频文件了。最后整理好的文件会排放在JAV_output这个文件夹内，工具会生成这个文件夹的。

* （小技巧，一次整理的时候少放些，出错的时候好手动排查）

#### 手动添加 Jellyfin JavScraper 插件，来获取JAV封面之类的信息
https://github.com/JavScraper/Emby.Plugins.JavScraper/releases

下载，解压后得到 `JavScraper.dll` , 直接丢进 pve lxc debian jellyfin 的 `/var/lib/jellyfin/plugins`
```
chmod 644 /var/lib/jellyfin/plugins/JavScraper.dll
chown jellyfin:jellyfin /var/lib/jellyfin/plugins/JavScraper.dll
```

#### 为 Jellyfin 添加 JAV 媒体库

内容类型 为 电影

Movie 元数据下载器 只需要 勾选 JavScraper
Movie 图片获取程序 也只需要勾选 JavScraper

然后就开始自动扫描装载媒体库了。



## 配置Jellyfin https 访问 （可选）
由于上面已经通过 apt 安装过 nginx 了。下面直接给nginx写配置就行了
```
rm -rf /etc/nginx/nginx.conf
cat << EOF > /etc/nginx/nginx.conf
user  www-data www-data;
pid   /run/nginx.pid;

worker_processes auto;
worker_rlimit_nofile 100000;

events {
    worker_connections  100000;
    multi_accept on;
    use epoll;
}

http {
  include mime.types;
  default_type application/octet-stream;

  fastcgi_connect_timeout 300;
  fastcgi_send_timeout 300;
  fastcgi_read_timeout 300;
  fastcgi_buffer_size 64k;
  fastcgi_buffers 64 4k;
  fastcgi_busy_buffers_size 128k;
  fastcgi_temp_file_write_size 128k;
  fastcgi_intercept_errors on;

  server_tokens             off;
  sendfile                  on;
  tcp_nodelay               on;
  tcp_nopush                on;
  keepalive_timeout         60;
  client_header_timeout     60;
  client_body_timeout       60;
  reset_timedout_connection on;
  types_hash_max_size       2048;

  gzip                      on;
  gzip_disable              "MSIE [1-6]\.";
  gzip_vary                 on;
  gzip_proxied              any;
  gzip_comp_level           4;
  gzip_min_length           256;
  gzip_buffers              16 8k;
  gzip_http_version         1.0;
  gzip_types    text/plain
                text/javascript
                text/css
                text/js
                text/xml
                text/x-component
                text/x-json
                font/opentype
                application/x-font-ttf
                application/javascript
                application/x-javascript
                application/x-web-app-manifest+json
                application/json
                application/atom+xml
                application/xml
                application/xml+rss
                application/xhtml+xml
                application/vnd.ms-fontobject
                image/svg+xml
                image/x-icon;


  access_log off;
  error_log off;

  include /etc/nginx/conf.d/*.conf;
}
EOF
```

```
rm -rf /etc/nginx/conf.d/default.conf
cat << EOF > /etc/nginx/conf.d/default.conf
server {
  listen 8097 ssl http2;
  server_name 域名;

  ssl_certificate /var/www/ssl/域名.cer;
  ssl_certificate_key /var/www/ssl/域名.key;
  ssl_protocols TLSv1.2 TLSv1.3;
  ssl_prefer_server_ciphers on;
  ssl_ciphers TLS13+AESGCM+AES128:TLS13+AESGCM+AES256:TLS13+CHACHA20:EECDH+ECDSA+AESGCM+AES128:EECDH+ECDSA+CHACHA20:EECDH+ECDSA+AESGCM+AES256:EECDH+ECDSA+AES128+SHA:EECDH+ECDSA+AES256+SHA:EECDH+aRSA+AESGCM+AES128:EECDH+aRSA+CHACHA20:EECDH+aRSA+AESGCM+AES256:EECDH+aRSA+AES128+SHA:EECDH+aRSA+AES256+SHA:RSA+AES128+SHA:RSA+AES256+SHA:RSA+3DES;
  ssl_session_timeout 10m;
  ssl_session_cache builtin:1000 shared:SSL:10m;
  ssl_buffer_size 4k;

  add_header X-Frame-Options "SAMEORIGIN";
  add_header Referrer-Policy "no-referrer" always;
  add_header X-Content-Type-Options "nosniff" always;
  add_header X-Download-Options "noopen" always;
  add_header X-Frame-Options "SAMEORIGIN" always;
  add_header X-Permitted-Cross-Domain-Policies "none" always;
  add_header X-Robots-Tag "none" always;
  add_header X-XSS-Protection "1; mode=block" always;
  add_header Strict-Transport-Security "max-age=63072000" always;

location / {
  proxy_pass                http://127.0.0.1:8096;
  proxy_set_header Host \$host;
  proxy_set_header X-Real-IP \$remote_addr;
  proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto \$scheme;
  proxy_set_header X-Forwarded-Protocol \$scheme;
  proxy_set_header X-Forwarded-Host \$http_host;
  proxy_connect_timeout     432000;
  proxy_send_timeout        432000;
  proxy_read_timeout        432000;
  proxy_redirect            off;
  proxy_buffering           off;
  proxy_buffer_size         4k;
}
}
EOF
```
* 应用上面命令前，替换好域名字样为真正自己使用的域名。
* 提前放好ssl证书文件，路径我习惯 `/var/www/ssl` 域名.cer 和 域名.key 两个文件。（证书文件，一般情况下是 600 ）
* 8096是jellyfin默认的web访问的端口。
* 8097是设置给nginx的https端口，平时通过 https://域名:8097 来访问jellyfin。

