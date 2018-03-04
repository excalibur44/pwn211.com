---
title: "docker 安装 jupyter 并用 nginx 进行反向代理"
date: 2017-04-08T11:30:00+08:00
category: ["server"]
---

这个过程的坑很多，所以记录下来帮助后人，也是帮助以后的自己。<!--more-->

## **0x01** 在 docker 运行 jupyter
如何安装 docker 和换 docker 的源在[这篇文章](https://pwn211.tk/index.php/archives/61/)有，就不多说了。

关于如何在 docker 运行 jupyter 的文章还是挺少的，不过幸好 jupyter 官方的 [github](https://github.com/jupyter/docker-stacks) 讲的挺详细，所以在安装时问题不大。

在这上个 jupyter docker 栈的图：
![inherit-diagram.png][1]

jupyter 的 docker 版本有这么多，其中的关系在图中也说清楚了，我选择的是 scipy-notebook，这样以后想要升级到 tensorflow-notebook 或 datascience-notebook 都比较方便。

按照（[官方说明](https://github.com/jupyter/docker-stacks/tree/master/scipy-notebook)）：运行`docker run -it --rm -p 8888:8888 jupyter/scipy-notebook`就可以运行 jupyter 了。

## **0x02** 配置 jupyter
因为 jupyter 是在 docker 的里面，所以比较难配置 jupyter，不过官方提供了脚本方便我们配置 jupyter。所以配置也很简单：

```bash
docker run -d --name jupyter \
-v /home/pwn/jupyter:/home/jovyan/work \
-p 8888:8888 \
jupyter/scipy-notebook start-notebook.sh \
--NotebookApp.password='sha1:74ba40f8a388:c913541b7ee99d15d5ed31d4226bf7838f83a50e'
```

解释一下这条很长的命令：

- \ 表示换行，把一条命令拆成几行以方便阅读
- -d 表示启动的容器进入到后台运行；
- --name 表示给启动的容器设定名字
- -v 表示把宿主机的目录挂载到容器中，直接查看配置文件可知 Jupyter 的活动目录是 /home/jovyan/work，并且可以挂载出来，所以就挂载出来方便以后的其他操作；
- -p 表示指定端口，这里把宿主机的8888端口映射到容器的8888端口（注：需要容器暴露端口才可以映射）；

然后后面的就是启动的镜像，最后面是 jupyter 给出的参数，用于把登录方式设置成密码，而不是通过 token 登录。

到这里 jupyter 已经配置完了，一条命令就搞定，是不是很快！

## **0x03** 用 nginx 做反向代理
因为 jupyter 访问是通过8888端口的，而输入域名之后再加个端口号很麻烦，而且多暴露一个端口服务器就越危险，所以我使用 nginx 做反向代理。配置文件是这样的：
```nginx.conf
server {
    listen 80;
    server_name your.domain.name;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name your.domain.name;
    index index.html index.php;
    root  /home/wwwroot/your.domain.name;

    ssl_certificate      /etc/letsencrypt/live/your.domain.name/fullchain.pem;
    ssl_certificate_key  /etc/letsencrypt/live/your.domain.name/privkey.pem;

    include enable-php.conf;

    location ^~ / {
        proxy_pass http://127.0.0.1:8888;
    }
}
```
用 nginx 把对这个域名的访问反向代理到本机的8888端口，顺手用 let's encrypt 搞了一个证书配置了 ssl，配置文件的最后一个 location 块是给 let's encrypt 验证用的。

这样看起来可以了，但是进入到 jupyter 发现创建不了文件，通过`docker logs jupyter`查看输出发现，jupyter 的安全机制把访问请求 block 了，原因是 http 请求的 origin 和 host 不同。这个简单，加几句配置命令的事，所以 / location块就变成这个样子：
```nginx.conf
    location ^~ / {
        proxy_pass http://127.0.0.1:8888;
        proxy_set_header host    "127.0.0.1:8888";
        proxy_set_header origin  "http://127.0.0.1:8888";
    }
```

这样就可以成功创建文件了，但是当创建了文件想运行时，却被提示连接不到 notebook 服务器...看一下 log，但这次 log 没有给出原因，只是说了 400 错误，这就棘手了...（漫长的 debug 过程：在 400 错误后面看到了 referer 的字样，那加个`proxy_set_header referer "http://127.0.0.1:8888";`行不行？然而不行；在 400 错误后面看到了 sessionID 的字样，虽然是通过 url 传的，但还是试着加上关于转发 cookies 的命令，但还是没用。）最后，在 chrome 的控制太看到 js 请求的错误信息，发现了一个没见过的协议：`wss://`，（后来才知道）问题就出在这！Google一下得知这是 Websocket 协议，nginx 没有对这个协议进行转发，所以才连接不上 notebook 服务器，查阅了 nginx 的[文档](http://nginx.org/en/docs/http/websocket.html)后，在 / location 块中加上了3条命令：
```nginx.conf
    location ^~ / {
        proxy_pass http://127.0.0.1:8888;
        proxy_set_header host    "127.0.0.1:8888";
        proxy_set_header origin  "http://127.0.0.1:8888";
        proxy_set_header X-Real-IP "127.0.0.1";
        
        # WebSocket proxying
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
```
重启 nginx `nginx -s reload` 再试试，就成功了哈哈！

[1]: /media/install-jupyter-with-docker-and-proxy-with-nginx/01.png
