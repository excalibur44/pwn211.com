---
title: "在 vmware 配置 debian 8.6.0"
date: 2017-01-19T12:58:44+08:00
categories: ["server"]
tags: ["vmware", "debian"]
---

## 0x00 缘起
因为 debian 内存占用很少，所以选择它作为 VPS 的操作系统（曾经看到有人在内存为 32MB 的 VPS 上搭建 ss 和自己的[博客](https://32mb.space/)，膜拜大神！）。但是！新安装的 debian 非常干净，连 sudo 都没有（对，你没看错！），所以每次安装都要查各种博客来安装，很是麻烦，所以在一次安装中，把配置的过程写下来，以便以后翻阅。<!--more-->

## 0x01 环境
- VMware Workstation Pro 12
- Windows Home 10
- debian 8.6.0

## 0x02 安装 gcc
1. 打开终端，输入下面的命令
```bash
# 因为无法使用 sudo，所以要先 su 切换到 root 用户，下面同理
su
# 安装 gcc
apt-get install gcc
# 顺便更新升级下
agt-get update && apt-get upgrade
```
2. VMware Tools 安装需要依赖内核的头文件
```bash
su
# 查看系统linux-headers版本
uname -r
# sudo apt-get install linux-headers-你刚才查询的版本
apt-get install linux-headers-3.16.0-4-amd64
```

## 0x03 安装 VMware Tools
1. 将 vmtools 的安装光盘插进去：菜单栏 -> 虚拟机 -> 安装 VMware Tools
2. 进入光盘，把光盘中的压缩包复制到 Downloads 文件夹中
```bash
su
cp VMwareTools-10.0.6-3595377.tar.gz /home/pwn/Downloads/
```
3. 解压压缩文件 & 进入解压出来的目录 & 执行安装文件
```bash
# 解压 VMwareTools-10.0.6-3595377.tar.gz，zxvf 是解压的参数，暂时不用管它
tar zxvf VMwareTools-10.0.6-3595377.tar.gz
# 进入解压出来的目录 vmware-tools-distrib/
cd vmware-tools-distrib/
# 因为执行安装文件需要 root 权限，所以切换到 root 账户
su
# 执行安装文件
./vmware-install.pl
```
4. 系统它要求你确认，输入 y 然后回车，接着就一路回车就行了
5. 最后看到 Enjoy, -- the VMware team 说明安装成功了
6. 然后 log out 再登入，就可以看到 vmtools 奏效了

## 0x04 安装 sudo
1. 安装 sudo
```bash
apt-get install sudo
```
2. 将自己所在的用户加入 sudoer
```bash
vi /etc/sudoers (or visudo )
# 文件内容 begin
    ...
    # User privilege specification
    root       ALL=(ALL) ALL
    youruser   ALL=(ALL) ALL
    ...
# 文件内容 end
# 因为 /etc/sudoers 是只读文件，所以保存时要用 :wq!。
:wq! 
```

## 0x05 配置静态 IP
因为这个虚拟机是用来模拟服务器（VPS）的，所以经常要在本机通过 IP 访问，但是在虚拟机中， IP 是随机分配的，每次开机都会不同，导致每次开机都要用 ifconfig 查 IP，这就很不方便，所以要把 IP 固定下来，变成静态 IP。
```bash
# 首先切换到配置网络的文件夹 /etc/network
cd /etc/network/
# 然后修改 interfaces 文件
sudo vi interfaces
    # 把下面的复制到文件中 用 :wq! 保存
    auto eth0               # 使用的网卡
    iface eth0 inet static  # 静态分配
    address 192.168.17.126  # 你想要的静态 IP
    netmask 255.255.255.0   # 子网掩码
    gateway 192.168.17.2    # 网关（这个和子网掩码都可以在 vmware 的虚拟网络编辑器中查到）
    # 如果原来的文件中有下面那行，就在前面加一个#号注释掉
    #iface eth0 inet dhcp
# 接着在 /etc/resolv.conf 中设置 DNS 服务器，在这种情况下应该就是 gateway 的地址
vi /etc/resolv.conf
    # 修改 DNS 服务器，因为我这里网关就是 DNS 服务器所以是相同的
    nameserver 192.168.17.2
# 最后，重启网络服务。或者重启 reboot
/etc/init.d/networking restart
```

## 0x06 参考资料
- [Debian安装VMwareTools](http://wxmimperio.tk/2016/01/25/Install-VmwareTooles-In-Debian/)
- [安装debian小型映像，执行包含sudo的命令提示command not found](https://segmentfault.com/q/1010000000159631/a-1020000000159709)
- [Debian中配置静态IP](http://blog.csdn.net/shooter556/article/details/919776)