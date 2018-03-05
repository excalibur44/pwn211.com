---
title: "记：在 Debian8 安装 docker"
date: 2017-04-06T16:23:04+08:00
category: ["server"]
tags: ["server", "debian", "docker"]
---

在今年寒假的时候接触到了 docker，当时就觉得这是一个很棒的技术，想应用到我的 VPS 上去，但是不知道之前选择 VPS 系统时怎么想的，居然安装了一个32位的系统，但 docker 是不支持32位系统的，所以折腾了好久都没成功，就放弃了。。

现在想在服务器上搭一个 Jupyter Notebook，但是在本地的虚拟机尝试了官方说的两种安装方法，都没有很理想（不像老师提供的环境），于是就想要一个开箱即用的 Jupyter Notebook。开箱即用，这是 docker 最擅长的事了，所以又回去折腾 docker 了。。<!--more-->

----

## 0x01 安装 docker
第2次尝试安装 docker，虽然这段时间都没怎么搞过，但是其他方面的学习也会对这边有帮助的。

参考了[这篇文章](http://www.docker.org.cn/book/install/install-docker-on-debian-8.0-jessie-34.html)。根据这篇文章说的：

> Debian 8 自带了3.16的内核，已经满足了docker运行的要求。
但是因为安全方面的原因，docker.io包并没有放在debian的stable源里面，而是放在了backports源里面。

所以我们要添加源，文章提供的源为debian官方的源，但是在国内由于某些原因，访问国外的源会很慢，不过幸好我们有中科大，它免费提供各种源的镜像，但是不知道有没有这个backports源，所以我尝试的把官方的域名换成中科大的域名：

```bash
echo "deb http://mirrors.ustc.edu.cn/debian jessie-backports main" >> /etc/apt/sources.list
```

然后更新，结果成功了！感谢中科大！接下来就愉快的安装 docker 了，顺便测试一下。
```bash
sudo apt-get update
sudo apt-get install docker.io
sudo docker run --rm hello-world 
```
----

## 0x01 安装 docker(更新)
debian的仓库太久没更新了，安装的docker版本是1.6.2，一个极低的版本..因为不支持0x02的换源方法，所以用来[阿里云文档](https://help.aliyun.com/knowledge_detail/42851.html)中的安装脚本来安装docker：`curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -`，安装完成后，通过`docker version`查看版本：17.04.0。由此可见debian仓库的docker有多老了...

*PS. 其实版本没看起来那么夸张，docker飙了版本号，在 [github](https://github.com/docker/docker/releases) 上可以看到：v1.13.1 -> v17.03.0-ce-rc1*

## 0x02 为 docker 换源
没错，又要换源...
还是由于众所周知的原因，国内访问 docker 官方仓库很慢，所以要换成一个国内的源。
在网上找到了[这篇文章](http://www.datastart.cn/tech/2016/09/28/docker-mirror.html)，又见我心爱的中科大，果断用它！但是...在换了源之后，抓取镜像没有成功...不知道什么原因...留个坑以后再弄吧

OK，这个问题好像是解决了（ pull 的速度达到了正常网速），参考文章：[Docker 镜像使用帮助](https://lug.ustc.edu.cn/wiki/mirrors/help/docker)

> 新版的 Docker 使用 /etc/docker/daemon.json（Linux） 或者 %programdata%\docker\config\daemon.json（Windows） 来配置 Daemon。
> 请在该配置文件中加入（没有该文件的话，请先建一个）：

```conf
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
```

然后重启 docker：`service docker restart`

再次感谢中科大！
