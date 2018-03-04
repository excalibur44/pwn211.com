---
title: "配置VPS"
date: 2017-01-19T13:02:20+08:00
category: ["server"]
draft: true
---

## 1. [修改登录端口](https://zhangge.net/4321.html)
## 2. [设置防火墙1](https://zhangge.net/4321.html)
## 3. [设置防火墙2](http://www.slyar.com/blog/vps-debian-iptables.html)
## 4. 配置ss<!--more-->

```
apt-get -y update && apt-get -y upgrade
apt-get -y install python-pip
pip install shadowsocks 
vi /etc/shadowsocks.json
ssserver -c /etc/shadowsocks.json -d start
```

```
#shadowsocks.json配置文件内容
{
    "server" : "you_server_ip",
    "port_password":{
        "user_1_name" : "user_1_password",
        "user_2_name" : "user_2_password",
        "user_2_name" : "user_3_password"
        },
    "timeout" : 600,
    "method" : "aes-256-cfb",
    "fast_open" : true,
    "workers" : 1
}
```

## 5. 安装 lnmp 全家桶

```
wget http://soft.vpser.net/lnmp/lnmp1.3.tar.gz
tar zxvf lnmp1.3.tar.gz
cd lnmp1.3/
./install.sh lnmp
#....
cd ..
rm lnmp1.3.tar.gz
rm -rf lnmp1.3/
```

## 6. 创建虚拟站点

```
lnmp vhost add
```

## 7. 下载配置 typecho

```
wget https://github.com/typecho/typecho/releases/download/v1.0-14.10.10-release/1.0.14.10.10.-release.tar.gz
tar zxvf 1.0.14.10.10.-release.tar.gz
mv build/* ./
rm 1.0.14.10.10.-release.tar.gz
rm -rf build/
```

## 8. 配置 https

```
wget https://dl.eff.org/certbot-auto
chmod a+x ./certbot-auto
./certbot-auto certonly --webroot -w /home/wwwroot/certbot -d  you.domain1 -d you.domain2
```

## 9. 配置反向代理
## 10. 为反向代理加上登录功能