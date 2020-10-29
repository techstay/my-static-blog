---
title: 翻墙工具trojan-go篇
date: 2020-10-29 19:31:37
tags:
    - 翻墙
    - 代理工具
categories:
    - 翻墙
---

## trojan-go是什么

trojan是另外一个很好用的翻墙工具，名气虽然没有影梭大，但是功能也不错。而且trojan设计的时候就考虑到了防范检测的问题，通过HTTPS流量来达到非常安全的翻墙目的，抗干扰性也更好一点。trojan-go就是trojan的一个go语言实现，相较于原版trojan来说配置和使用更加简单，我试了一下效果也很不错，于是来给大家介绍一下。

## 准备条件：配置域名

### 购买和配置域名

trojan在使用的时候将自己伪装成一个普通的HTTPS网站，普通用户访问的时候，看到的就是一个普通的网页。一旦发现请求的是trojan协议，trojan就开始干正事了。正因为此，trojan的配置比影梭一些，一个必要的步骤就是购买域名，并在自己的服务器上将域名和证书配置好。所以首先就来讲讲如何配置域名。

第一步，去一个域名网站上买一个域名，最好选择国外的网站，如果嫌麻烦，国内的阿里云什么的也行，但是安全性可能要差一点。购买域名之后，在相应网站上配置好域名解析，将域名解析到你的服务器上面。这样第一步就做好了。

### 申请免费证书

为了使用HTTPS功能，我们还需要去相应的证书服务商那里申请一个免费证书。这样的网站有很多，不过大部分免费证书时间都只有几个月，到期以后还要自己重新申请一遍，操作比较麻烦。所以这里介绍另外一种方法，利用acme.sh这个脚本来自动化处理过程。该工具的详细用法可以参考[其wiki](https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E)。

首先安装acme.sh：

```sh
curl  https://get.acme.sh | sh
```

然后需要验证域名，如果你的服务器上现在运行着80端口服务，先把他关掉。然后运行下面的命令，acme.sh会临时启动一个服务器来完成验证过程。验证完毕后就会自动在其目录下生成证书。

```sh
acme.sh  --issue -d mydomain.com   --standalone
```

### 复制证书

虽然有了证书，但是证书放在acme.sh的内部，还需要将其复制出来。acme.sh也提供了相应的命令。这里假设我们的linux服务器用户名是hello，证书和trojan配置文件放在`/home/hello/config/`中。那么复制命令应该是这样的：

```sh
acme.sh --install-cert -d example.com \
--key-file       /home/hello/config/key.pem  \
--fullchain-file /home/hello/config/cert.pem \
--reloadcmd     "sudo service nginx force-reload"
```

这样在证书临近到期的时候，acme.sh就会自动开始验证并续期证书，然后将新的证书复制过来。如果指定了最后一行的`--reloadcmd`，那么还会自动用后面的命令来重新加载服务器，让新证书生效。命令成功运行之后，目录里应该会多出三个pem文件，这就是准备好的证书了。

## 配置trojan-go服务端

### 配置nginx服务器

好了，下面就是比较麻烦的一个部分了。trojan的配置需要我们服务器上运行两个HTTP服务，一个是80端口服务，用来当做假的HTTP服务器，另一个是1234端口（可以自定义其他端口号）的400状态码服务，用来在trojan协议遇到问题的时候来作为默认错误显示（这可以一定程度上防止防火墙的检测）。所以这里就来配置一下nginx这个东西。这里仍然假设我们linux用户名是hello，nginx的HTML文件是`/home/hello/html/index.html`。

然后编辑`/etc/nginx/sites-enabled/default`文件，将文件内容修改成类似下面这样的。

```sh
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /home/hello/html; # 这里修改为你的HTML文件存放目录

    index index.html index.htm index.nginx-debian.html;

    server_name _;

    location / {
            try_files $uri $uri/ =404;
    }
}

# 这里添加一个新的服务
server {
    listen 1234; # 这里可以自定义端口号
    return 400;
}
```

至于`index.html`文件的内容，这个就比较随意了。甚至你可以只写下面一行，不用惊讶，这也是合法的HTML文件，只不过省略了那些标签而已。

```html
<h1>主页</h1>
```

配置完毕之后，就可以启动nginx服务器看看效果了，如果配置正确，那么服务器就可以启动，我们就可以在浏览器中使用域名或者IP地址来访问服务了。查看一下80端口号的网页是否可以正常访问，再看看1234端口号是否可以显示400 Bad Request错误页面。验证正确之后，就可以将nginx服务器设为自启状态。

```cmd
# 验证服务器
sudo systemctl start nginx
# 验证无误后开机自启
sudo systemctl enable nginx
```

### 配置trojan-go服务

好了，下面终于可以开始配置trojan-go服务了。经过一番研究，我现在比较喜欢docker运行这类服务，一方面docker是一种非常方便的应用分发手段，另一方面，docker有个restart policy参数，允许docker服务在系统启动的同时也一起启动，这样就达到了类似开机自启动的目的，而且还不用配置supervisor那些东西，真的是非常方便。当然docker的问题在于容器内部和外部的配置文件不共通，所以命令看起来就比较冗长，而且如果不熟悉docker的话，可能会遇到一些小问题。所以这里还是建议大家事先学习一下docker的用法，真的非常好用。

前面其实我提到了将证书和trojan配置文件放到一个目录里，这是因为trojan-go的docker镜像会自动使用容器内部的配置文件，所以比较简便的做法就是把容器需要的配置文件放到一起，然后塞到容器里面。docker可以利用`-v`参数来支持这样的行为。这里的配置文件和证书假设放到`/home/hello/config/`中，配置文件名字务必是`config.json`，因为trojan-go默认会以这个名字来读取配置。如果想要了解更多trojan-go配置的话，可以直接参考[它的wiki](https://p4gefau1t.github.io/trojan-go/basic/config/)。

服务端配置文件大体上长这样，主要改一下密码就可以了，保存为`config.json`即可。

```json
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "localhost",
    "remote_port": 80,
    "password": [
        "longlonglongpassword"
    ],
    "ssl": {
        "cert": "/etc/trojan-go/cert.pem",
        "key": "/etc/trojan-go/key.pem",
        "fallback_port": 1234
    }
}
```

然后就可以使用docker来运行trojan-go服务了，虽然命令看起来挺长的，但是无需操心trojan-go的安装/更新/自启，全部由docker搞定，这也正是docker的魅力。第一次运行可能有错误，所以这里采用`--rm`参数，一旦停止服务就会删除容器，不留下垃圾，而且日志会在前台打印，方便定位和解决错误。一旦验证发现服务端没有问题，就可以将`--rm`参数改为`-d`参数，这样trojan端就可以在后台静静的运行，不用我们操心了。

```sh
docker run --rm --name trojan --restart always \
    -v /home/hello/config/:/etc/trojan-go/ \
    --network host \
    p4gefau1t/trojan-go
```

验证服务端是否正常运行的方法很简单，直接在浏览器中访问`https://你的域名`，因为nginx服务器上配置的只有80端口和1234端口，所以如果这里443端口可以正常访问，说明trojan-go服务端已经成功运行了。第一次访问`https`的时候会多等待几秒，因为trojan会故意停几秒，防止按时间探测的攻击。当然为了保险起见，最好再用客户端连接一下，如果可以正常翻墙，那么就是真的成功了。

## 配置trojan客户端

### 电脑客户端

从[这里](https://github.com/p4gefau1t/trojan-go/releases)下载trojan-go的客户端，解压。客户端配置文件和服务端类似。

```json
{
    "run_type": "client",
    "local_addr": "localhost",
    "local_port": 1080,
    "remote_addr": "你的域名",
    "remote_port": 443,
    "password": [
        "你的密码"
    ]
}
```

然后用下面的命令即可运行trojan客户端。

```sh
trojan-go.exe -config config.json
```

### 移动客户端

具体参考[这个页面](https://github.com/trojan-gfw/trojan/wiki/Mobile-Platforms)，安卓客户端推荐igniter，开源客户端，使用放心。

配置方法如图所示，SNI不用写，备注随意，域名和密码写服务端对应的即可，底下的三个选项全不选。

{% asset_img android.jpg 安卓客户端配置 %}

好吧，关于翻墙工具的介绍就到这里。其实每次写这类文章，我个人也感觉很麻烦，服务端的配置文件一大堆，而且客户端配置文件也一大堆。所以我准备做一个一站式的Web应用，将这些常用的翻墙工具囊括进去，做成可以在Web端轻松管理的工具，同时还可以直接导出客户端配置，到时候也不用在到处找一键配置脚本什么的了。不过这个计划可能要等些时间，到时候也请大家多多支持。
