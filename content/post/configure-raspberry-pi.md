---
title: "折腾树莓派"
date: 2017-04-15T20:45:57+08:00
category: ["others"]
draft: true
---

使用镜像：Raspberry + Debian = Raspbian

默认账号：pi，默认密码：raspberry 第一次登录马上修改密码（默认密码太难输了）

然后按惯例，拿到系统第一步先换源，再更新<!--more-->

```
# echo "deb http://mirrors.ustc.edu.cn/raspbian/raspbian/ jessie main non-free contrib
deb-src http://mirrors.ustc.edu.cn/raspbian/raspbian/ jessie main non-free contrib" > /etc/apt/sources.list
# apt-get update
```

但是不知道为什么，换了源之后还会从官方源更新...

因为只是想拿树莓派当服务器，所以没买显示器（主要是因为没钱），但是又想看看树莓派的图形界面，就安装了远程桌面，因为常用机是windows，所以直接使用RDP，树莓派那边有个软件叫xrdp，可以支持rdp协议远程。

初次安装有个坑，就是除了安装xrdp以外，还要安装vnc，因为xrdp是通过vnc连接桌面的。所以最终命令：
```
# apt-get install xrdp tightvncserver
```

从远程结果来看，树莓派可以支持2k分辨率，但我不想它那么大，所以在windows远程的软件对分辨率进行了限制。

联网：
配置静态ip的位置不是在`/etc/network/interfaces`了，而是在`/etc/dhcpcd.conf`中，以下代码copy自[这里](http://www.feifeiboke.com/pcjishu/3617.html)
```
vi /etc/dhcpcd.conf
# 使用 vi 编辑文件，增加下列配置项

# 指定接口 eth0
interface eth0
# 指定静态IP，/24表示子网掩码为 255.255.255.0
static ip_address=192.168.1.20/24
# 路由器/网关IP地址，这里要看路由器的设置
static routers=0.0.0.0
# 手动自定义DNS服务器，这里要看路由器的设置
static domain_name_servers=0.0.0.0

# 修改完成后，按esc键后输入 :wq 保存。重启树莓派就生效了
sudo reboot
```
注意：如果是配置wlan0，还要在远程树莓派进行wifi联网，输入密码，这样下次启动就会自动连上wifi，而且是静态ip。
