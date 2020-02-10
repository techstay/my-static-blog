---
title: 翻墙工具wireguard篇
date: 2020-02-10 21:01:48
tags:
    - 翻墙
    - 代理工具
categories:
    - 翻墙
---

## wireguard是什么

wireguard是一个全新的高性能的VPN，使用最新的技术实现的强大VPN。它设计精巧，核心代码仅4000多行，被Linux之父林纳斯亲切的称为“艺术品”，可见wireguard是多么的优秀。所以如果有使用VPN需求的话，可以考虑使用wireguard。

## 安装wireguard

wireguard安装起来稍微有些麻烦，因为它需要内核的支持，所以如果你是OpenVZ架构的VPS的话，基本不用想了，万年2.7的内核，什么也干不了。

Ubuntu 18.04目前可以通过手动安装的方式来使用wireguard。等到今年4月份，Ubuntu 20.04长期支持版出来以后，我们就有原生的wireguard可以使用了。

```sh
# Ubuntu 19.10及更新的系统
$ sudo apt install wireguard

# Ubuntu 19.04及更旧的系统
$ sudo add-apt-repository ppa:wireguard/wireguard
$ sudo apt-get update
$ sudo apt-get install wireguard
```

这里注意一下，如果你用的是搬瓦工的的VPS，可能会提示`add-apt-repository`命令未找到，这需要安装另外一个包才可以正常使用。

```sh
sudo apt-get install software-properties-common
```

Arch系发行版因为一直都是滚动更新，所以安装wireguard十分简单，随系统更新新版本内核，然后安装一下wireguard的命令行工具即可。

## 配置wireguard

说实话这里耗费了我不少时间，因为和一般程序服务端、客户端配置文件不同，wireguard是点对点的，而且涉及到了一些网络和防火墙的配置，所以导致我很不理解配置文件的各项含义。

不过熟悉了之后，我完全可以表示，wireguard配置起来一点也不难，相对于OpenVpn之类的甚至还简单多了。为了省事我这里还写了一个配置脚本，自动生成wireguard配置文件和客户端配置文件，方便大家使用。

这里简单复制一下脚本，大家看看内容就明白了，配置起来很简单，顶多修改一下端口号就行了。密钥对都是随机生成的，绝对安全可靠。当然脚本后续可能更新，如果想要最新版的话，请大家关注我的[Gitub项目](https://github.com/techstay/climbthewall)。

使用脚本文件也很简单，直接下载我项目中的最新版脚本，用bash运行即可，注意事项写在项目的README中了，大家注意一下即可。

```sh
wget https://raw.githubusercontent.com/techstay/climbthewall/master/wg.sh
bash wg.sh
```

以下是脚本文件内容。

```sh
#! /bin/bash

config_dir="$HOME/.wireguard/"

mkdir -p "$config_dir"
cd "$config_dir" || {
    echo 切换目录失败，程序退出
    exit
}
# 生成两对密钥，分别用作服务器和客户端使用
wg genkey | tee pri1 | wg pubkey >pub1
wg genkey | tee pri2 | wg pubkey >pub2

# 设置密钥访问权限
chmod 600 pri1
chmod 600 pri2

interface=$(ip -o -4 route show to default | awk '{print $5}')
ip=$(ip -4 addr show "$interface" | grep -oP '(?<=inet\s)\d+(\.\d+){3}')

# 生成服务端配置文件
cat >wg0.conf <<EOL
[Interface] 
PrivateKey = $(cat pri1)
Address = 10.10.10.1
ListenPort = 54321
PostUp   = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o $interface -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o $interface -j MASQUERADE
[Peer]
PublicKey = $(cat pub2)
AllowedIPs = 10.10.10.2/32
EOL

# 生成客户端配置文件
cat >client.conf <<EOL
[Interface]
PrivateKey = $(cat pri2)
Address = 10.10.10.2
DNS = 8.8.8.8

[Peer]
PublicKey = $(cat pub1)
Endpoint = $ip:54321
AllowedIPs = 0.0.0.0/0
EOL

# 复制配置文件并启动
sudo cp wg0.conf /etc/wireguard/ || {
    echo 复制失败,请检查/etc/wireguard目录或wg0.conf是否已经存在
    exit
}
sudo wg-quick up wg0 || {
    echo 启动wireguard失败，请检查/etc/wireguard/wg0.conf是否存在错误
    exit
}

# 显示客户端配置文件
echo "----------以下是客户端配置文件，请保存并在客户端中使用----------"
cat client.conf
```





