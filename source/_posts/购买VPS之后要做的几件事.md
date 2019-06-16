---
title: 购买VPS之后要做的几件事
date: 2019-06-16 23:53:50
tags:
---

## 配置用户账户

刚买的 VPS，系统给你的肯定只有一个 root 账户，但是一般不要用 root 账户来登录，可能存在安全风险。所以我们这里就来创建一个新用户用来登录。由于这时候用的是 root 账户来操作，所以这里的命令不需要在前面加 sudo。有的系统管理员组叫做 wheel，有的叫做 sudo，大家根据自己的 VPS 系统来选择。

```
useradd yitian -m -G sudo,wheel
```

创建完用户之后，别忘了为用户添加密码，为了安全，密码应该设置的比较复杂，最好使用密码管理软件生成强密码，反正照着这里设置的话，可能以后再也用不到这个密码了。

```
passwd yitian
```

如果你怕麻烦，现在就可以设置免密码输入 sudo 命令。首先打开 sudoers 文件：

```
nano /etc/sudoers
```

然后找到类似下面的一行，然后在后面的`ALL`之前添加`NOPASSWD:`，改为下面这样的。这个操作一定要注意，整错了可能就没办法登录 VPS 了。设置成功的话，sudo 组里的用户就再也不需要输入密码确认了。

```
%sudo   ALL=(ALL:ALL) NOPASSWD: ALL
```

现在，使用刚刚创建的用户登录。可以顺便测试一下 sudo 命令，不需要输入密码就是爽！但是在使用 SSH 登录的时候还是需要密码来登录，所以现在就来设置免密码登录 SSH。

首先打开 SSH 的配置文件：

```
sudo nano /etc/ssh/sshd_config
```

找到下面一行，去掉前面的注释，开启公钥登录功能：

```
PubkeyAuthentication yes
```

然后打开本用户的用户目录下的 ssh 文件夹，没有则创建。在其中新建验证文件并输入你的 SSH 公钥。公钥文件就是你当前操作系统下`.ssh/id_rsa.pub`文件里的内容。

```
nano .ssh/authorized_keys
```

完成之后，重启 SSH 服务，然后退出用户并重新登录一下，应该可以看到不再需要密码就可以登录了。好了，现在你可以把刚刚的用户密码忘掉了，这样一来，几乎没人可以登录你的系统了，安全性妥妥的。甚至你可以做的绝一点，在 SSH 配置文件里直接禁用 ROOT 账户登录和密码登录，这样安全性几乎为 100%。但是这样做的问题就是一旦你的密钥对丢失（虽然这种情况几乎不可能发生），包括你自己都无法登录了。所以怎么做还是得看自己取舍。

## 升级系统

前面的做完了，下面就是系统维护工作了。首先是升级系统，刚安装的 VPS 系统一般都是比较老的，更新到最新可以体验系统的各项新功能。

如果是 Ubuntu：

```
sudo apt update
sudo apt upgrade
```

## 安装必要软件

如果是 Ubuntu：

```
sudo apt install zsh git python3-pip screenfetch
```

如果你想的话还可以安装 oh-my-zsh：

```
sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

## 安装并配置 shadowsocks

之前我用 Ubuntu 的时候，pip 里面的 shadowsocks 要比 Ubuntu 源里的更新。但是不知道为啥原因 pip 里面的影梭停更了，到了 Ubuntu 18.04，现在 Ubuntu 里的已经是更新的了。所以我们安装的时候，直接从 Ubuntu 源里安装就行了。

```
sudo apt install shadowsocks
```

然后随便建一个文件夹，准备存放翻墙软件的配置文件。

```
mkdir fq
cd fq
```

首先新建一个影梭的配置文件，比方说就叫`ss.json`。为了保证不被封，最好将端口号改一下，密码也改成一个比较复杂的。

```
{
    "server": "0.0.0.0",
    "server_port": 8388,
    "local_address": "127.0.0.1",
    "local_port": 1080,
    "password": "mypassword",
    "timeout": 300,
    "method": "aes-256-cfb",
    "fast_open": true
}
```

为了让影梭开机自动启动，我们可能还需要安装一些软件。如果是 Ubuntu 系统，可以在`/etc/local.rc`文件中添加开机脚本。或者向这里介绍的一样使用任务管理软件 supervisor。首先安装 supervisor：

```
sudo apt install supervisor
```

然后添加 ss 的配置文件：

```
sudo nano /etc/supervisor/conf.d/ss.conf
```

配置文件内容如下：

```
[program:ss]
command=ssserver -c /home/yitian/fq/ss.json
autostart=true
```

然后重启 supervisor 服务，就可以看到影梭成功启动了。

```
sudo systemctl restart supervisor
sudo supervisorctl restart ss

```

## 关于 kcptun

我现在用的服务器是 Hostwinds，速度还是挺快的，所以就不安装 kcptun 了。而且说实话 kcptun 安装起来太麻烦了，首先需要服务端下载、配置、设置自启，然后将配置文件传回电脑端，在电脑端上也下载客户端、配置、设置自启，这么一通下来也没啥用。所以我就先这么用了。等到将来速度不行的时候，我再配置一下 kcptun。不过我感觉过了 6 月搬瓦工应该就解禁了，没必要现在设置地这么麻烦。
