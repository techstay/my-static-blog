---
title: shadowsocks安装和配置
date: 2020-01-21 20:33:41
tags: 
    - shadowsocks
    - 影梭
    - gost
categories:
    - linux
---


## shadowsocks-libev配置

### 服务端配置

以下配置全部以Ubuntu系统为例。首先安装shadowsocks：

```sh
sudo apt install shadowsocks-libev
```

安装完成之后编辑配置文件`/etc/shadowsocks-libev/config.json`，改成类似下面的内容。其中端口号port以及密码password需要修改，其他配置无需变动。特别提一点，原来默认的aes-256-cfb有安全漏洞，所以推荐使用下面的全新协议chacha20-ietf-poly1305.

```json
{
    "server":["::0", "0.0.0.0"],
    "server_port":12345,
    "local_port":1080,
    "password":"123456",
    "timeout":60,
    "method":"chacha20-ietf-poly1305",
    "fast_open":true,
    "prefer_ipv6":true,
}
```

配置完成之后启动服务端。

```sh
sudo systemctl start shadowsocks-libev-server@config
```

然后查看一下服务状态，如果成功运行的话，使用客户端连接，看看能否科学上网。

```sh
sudo systemctl status shadowsocks-libev-server@config
```

成功连接上的话，就可以让服务开机自启动了，这样以后我们就不用管它了。

```sh
sudo systemctl enable shadowsocks-libev-server@config
```

### 客户端配置

虽然一般我们都是用windows客户端或者安卓客户端之类的来连接，但是既然提到了，就顺便说一下shadowsocks-libev客户端的配置方法吧。首先还是需要一个配置文件`/etc/shadowsocks-libev/config.json`。

```json
{
    "server":["这里改成你的服务器的IP地址"],
    "server_port":12345,
    "local_port":1080,
    "password":"123456",
    "timeout":60,
    "method":"chacha20-ietf-poly1305",
    "fast_open":true,
    "prefer_ipv6":true,
}
```

然后启动客户端服务。

```sh
sudo systemctl start shadowsocks-libev-local@config
# 成功之后让客户端服务开机自启
sudo systemctl enable shadowsocks-libev-local@config
```

### Windows客户端配置

这里推荐的是影梭Windows版，一个利用.NET实现的小软件，直接就可以在[Github](https://github.com/shadowsocks/shadowsocks-windows/releases)下载到。

然后打开软件，由于是图形界面配置，这里我就不介绍了，是个人就会用。

{% asset_img 影梭客户端配置.png 影梭客户端配置 %}

## gost配置

[gost](https://github.com/ginuerzh/gost)是go语言实现的一个安全隧道，可以用来做很多事情。而且它还非常贴心的内置了影梭协议，用起来比shadowsocks-libev更加方便，而且支持kcp等其他一些协议。

### 服务端配置

gost有中文官方，介绍的非常详细，这里就不多做介绍了。gost是一个单纯的命令行工具，而且并不是每个发行版的软件仓库中都有gost。所以最简单的办法就是利用docker来运行gost，恰好docker可以通过配置重启策略的方式来达到类似于自启服务的效果。

先来安装docker。

```bash
# 安装docker
sudo apt install docker.io
# 将用户添加到docker用户组中，方便直接执行命令无需管理员权限
sudo gpasswd docker -a $(whoami)
# 启动docker
sudo systemctl start docker
# 设置docker开机自启
sudo systemctl enable docker
```

以上命令成功执行以后，测试一下docker是否可以正常启动。

```sh
docker run --rm hello-world
```

然后安装gost镜像。

```bash
docker pull ginuerzh/gost
```

然后就可以启动gost服务了。如果你不熟悉docker的话，可以先用下面的命令试试，它结束之后会自动删除docker容器，不用担心有残留文件。命令在前台运行成功之后，在客户端中测试一下，看看能否成功上网。

```bash
docker run --rm \
    --net=host \
    ginuerzh/gost -L=ss://AEAD_CHACHA20_POLY1305:password@:8338
```

等到测试成功之后，用`Ctrl-C`停止容器，因为它有`--rm`参数，所以停止后会自动销毁。然后改用这个命令让容器在后台持续运行，并在docker服务重启或者系统重启之后自动跟着重启。

```bash
docker run -d \
    --net=host \
    --restart=always \
    ginuerzh/gost -L=ss://AEAD_CHACHA20_POLY1305:password@:8338
```

这里建议大家学习一下docker的用法，一来他也算个重要的软件，二来以后应该会经常和docker容器打交道，如果不熟悉的话，遇到问题还是不太好解决的。

### 客户端配置

服务端运行成功之后，就可以使用客户端来连接了。gost的影梭协议完全兼容shadowsocks-libev，所以可以直接用它来连接，配置方法如上。

如果你用linux服务端的话，也可以像配置服务端一样来配置客户端。安装方法和上面一样，用docker方式安装即可。直接来看看如何用docker来运行客户端连接吧。

```bash
docker run -d \
    --net=host \
    --restart=always \
    ginuerzh/gost -L=:10800 -F=ss://AEAD_CHACHA20_POLY1305:password@server_ip:8338
```

## 启用kcp协议加速

### kcp默认配置

gost内置了kcp协议的支持，使用kcp非常简单，只需要在协议那里添加`+kcp`即可。

```bash
docker run -d \
    --net=host --restart=always \
        ginuerzh/gost -L=ss+kcp://AEAD_CHACHA20_POLY1305:123456@:12222
```

gost客户端也需要做相应的修改。

```bash
docker run -d \
    --net=host --restart=always \
        ginuerzh/gost -L :1080 \
         -F=ss+kcp://AEAD_CHACHA20_POLY1305:123456@1.2.3.4:12222
```

这种方式启动的kcp使用的是默认配置，对应的配置文件如下。

```json
{
    "key": "it's a secrect",
    "crypt": "aes",
    "mode": "fast",
    "mtu" : 1350,
    "sndwnd": 1024,
    "rcvwnd": 1024,
    "datashard": 10,
    "parityshard": 3,
    "dscp": 0,
    "nocomp": false,
    "acknodelay": false,
    "nodelay": 0,
    "interval": 40,
    "resend": 0,
    "nc": 0,
    "sockbuf": 4194304,
    "keepalive": 10,
    "snmplog": "",
    "snmpperiod": 60,
    "tcp": false
}
```

### 自定义kcp配置

如果你的VPS只有自己用的话，可以考虑将加速模式改成fast2或者fast3，来增强加速效果，相对应地，流量消耗也会增大。不过根据我几年的VPS使用经验来看，一般一个月1T的流量，我平时最多用个50G，照这个用法，就算有个fast10模式，也不用担心流量。例如我有下面的配置文件。

```json
{
    "key": "it's a secrect",
    "crypt": "aes",
    "mode": "fast3",
    "mtu" : 1350,
    "sndwnd": 1024,
    "rcvwnd": 1024,
    "datashard": 10,
    "parityshard": 3,
    "dscp": 0,
    "nocomp": false,
    "acknodelay": false,
    "nodelay": 0,
    "interval": 40,
    "resend": 0,
    "nc": 0,
    "sockbuf": 4194304,
    "keepalive": 10,
    "snmplog": "",
    "snmpperiod": 60,
    "tcp": false
}
```

把上面的配置文件保存为`~/.kcp/kcp.json`文件。然后运行服务端的时候手动指定配置文件。注意shell会转义`?`字符，所以整个参数需要用单引号括起来。后面使用`?c=xxx.json`的时候同样需要注意这个问题。因为容器的文件系统和本地的文件系统并不一致，所以需要`-v`参数将文件传递到容器内部，因此命令看起来稍微复杂一点。

```bash
docker run -d \
    --net=host --restart=always \
    -v ~/.kcp/kcp.json:/kcp.json \
        ginuerzh/gost '-L=ss+kcp://AEAD_CHACHA20_POLY1305:123456@:12222?c=/kcp.json'
```

同样地，客户端命令也需要变成这种形式。当然，客户端需要同样的配置文件。

```bash
docker run -d \
    --net=host --restart=always \
    -v ~/.kcp/kcp.json:/kcp.json \
        ginuerzh/gost \
        -L=:10800 \
        '-F=ss+kcp://AEAD_CHACHA20_POLY1305:123456@1.2.3.4:12222?c=/kcp.json'
```

### Windows客户端配置

默认情况下影梭的Windows客户端不支持kcptun，所以需要进行额外配置。

首先下载[kcptun](https://github.com/xtaci/kcptun/releases)，然后将`client_windows_amd64.exe`重命名为`kcptun.exe`，然后把它放到影梭程序的同级目录下。

{% asset_img 影梭文件夹.png 影梭文件夹 %}

然后在服务器编辑对话框中额外添加几个参数。

- 插件程序，就是刚刚改完名字的kcptun。如果不和影梭在同一目录的话，需要改成绝对路径。
- 插件选项，你改了多少项默认配置，这里就要添加多少项。所以为了省事，我建议只改一下mode。
- 需要命令行参数的对勾必须打上，然后在对话框中添加`-l %SS_LOCAL_HOST%:%SS_LOCAL_PORT% -r %SS_REMOTE_HOST%:%SS_REMOTE_PORT%`。这样IP、端口号等参数就会传递给kcptun。

{% asset_img 插件参数配置.png 插件参数配置 %}

配置完毕之后，应该就可以连接了。测试一下，速度应该比裸的影梭更快。

## 脚本快捷配置

我有个计划，准备编写一个脚本来帮助快速配置程序并生成相应的客户端配置信息。不过脚本编写过程中，因为想得太多，最后脚本难产了。后来我又重新缕了一下需求，把实现比较困难的去掉了，最后终于把脚本写出来了。大家可以试用一下。

脚本使用Python写成，使用的时候要保证无需管理员权限即可运行docker命令。脚本运行成功之后，会显示gost以及影梭客户端的配置文件，方便用户在客户端上配置。为了方便起见，在脚本中我没有提供设置其他kcp参数的选项，可以设置的只有mode参数，这也是比较常用的一个参数。

```sh
# 下载脚本
wget https://raw.githubusercontent.com/techstay/climbthewall/master/gost_ss.sh

# 用python运行脚本
python gost_ss.py -h
usage: gost_ss.py [-h] [-password PASSWORD] [-port PORT] [-k] [-mode {fast,fast2,fast3}]

optional arguments:
  -h, --help            show this help message and exit
  -password PASSWORD    密码，未指定则使用随机密码
  -port PORT            端口号，未指定则使用随机端口号

kcp:
  -k                    是否使用kcp协议加速
  -mode {fast,fast2,fast3}
                        kcp协议的加速模式，流量充足可使用fast3
```
