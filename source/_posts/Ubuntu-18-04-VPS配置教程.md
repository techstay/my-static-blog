---
title: Ubuntu 18.04 VPS配置教程
date: 2020-03-19 19:26:23
tags:
    - linux
categories:
    - linux
---

最近我hostwind服务器又不能用了，正好干脆把它重装一下算了。这里简单记录一下重装之后简单的配置过程，以后也好作为参考。

## 配置系统

VPS启动之后，只会给你一个root账号，使用root账号工作起来不太安全也不方便，所以肯定需要创建一个自己的账号。选择你喜欢的SSH软件，登录到VPS上。这里我推荐Git for Windows，自带的Git Bash就可以充当SSH的作用。配置好以后简单小巧，使用也方便。关于如何配置Git Bash，稍后就会看到。

```sh
ssh root@1.2.3.4 -p 22
```

新的VPS刚刚安装完，还没有更新过，因此第一件事情就是更新系统。以下命令都以root账户执行，所以无需sudo，不过有些命令是我从别处贴过来的，带了sudo，复制粘贴的时候带上也无妨。安装过程中可能会弹出一些提示，简单意思就是新版本的配置文件和当前版本的不一样，询问我们使用哪个。一般情况下选择本地版本（这也是默认选择）即可。

```sh
apt update
apt upgrade
```

接下来就是区域和时间配置，配置完毕之后，一些软件的提示就会变成中文。我个人比较喜欢这样。

```sh
# 区域和时间配置
sed -i 's/^# zh_CN.UTF-8 UTF-8/zh_CN.UTF-8 UTF-8/g' /etc/locale.gen
locale-gen
# 这一步可能会出现错误，没关系，等会配置完重启系统在运行一遍即可
localectl set-locale zh_CN.UTF-8
echo 'LANG=zh_CN.UTF-8' | sudo tee /etc/locale.conf
sudo timedatectl set-timezone Asia/Shanghai
sudo timedatectl set-ntp 1
```

## 配置用户

首先需要安装以下软件，下面会用到。

```sh
apt install git zsh
```

新建账户，ubuntu用的是sudo用户组，而arch用的是wheel组，所以还要注意一下。

```sh
useradd yitian -m -g sudo -s /bin/zsh
# 别忘了为用户添加密码
passwd yitian
```

如果你喜欢的话，还可以这里干脆把root账号的默认shell改成zsh。

```sh
chsh -s /bin/zsh
```

默认的zsh其实并不好用，所以需要配置一下。这里简单用antigen包管理器并复制我的配置。

```sh
curl -L git.io/antigen >.antigen.zsh
wget https://raw.githubusercontent.com/techstay/dotfiles/master/zsh/.zshrc
wget https://raw.githubusercontent.com/techstay/dotfiles/master/zsh/.p10k.zsh
```

对于我们的账户，同样要进行一遍。首先先切换到我们的账户，然后继续。

```sh
su - yitian
# 登录以后应该会出现zsh的选择，选择q，然后继续
curl -L git.io/antigen >.antigen.zsh
wget https://raw.githubusercontent.com/techstay/dotfiles/master/zsh/.zshrc
wget https://raw.githubusercontent.com/techstay/dotfiles/master/zsh/.p10k.zsh
```

前面将用户添加到sudo（或者wheel）组以后，用户就可以使用sudo命令临时获得管理员权限执行一些命令。但是有时候特别是我们个人服务器这样做不太方便，如果你的服务器很安全，而且只有自己用的话，可以将用户设置为无需命令直接sudo执行。当然这样做略有风险，在安全和方便之间请自行权衡。

```sh
sudo mkdir -p /etc/sudoers.d/
echo "$(whoami) ALL=(ALL) NOPASSWD: ALL" | sudo tee "/etc/sudoers.d/$(whoami)"
```

这样一来VPS就算基本配置好了。

## 配置密钥登录

虽然VPS配置好了，但是每次登录还是有点麻烦。所以为了方便，这里推荐启用SSH的密钥登录，无需密码，配合上面的sudo无需密码，你可能很快就会把自己的用户密码给彻底忘掉。

安装好Git之后打开Git Bash。如果你之前没有创建过密钥，推荐先创建一个。

```sh
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

创建完毕之后，在`~/.ssh/`下应该会出现`id_rsa`和`id_rsa.pub`两个文件，分别是我们创建的私钥和公钥。有了密钥，就可以开始配置了。

### 手动配置

这里先介绍一下完全化的手动配置，然后在介绍稍微简单一点的配置。嫌麻烦可以直接看下面。

首先还是登录VPS。

```sh
ssh yitian@1.2.4.5 -p 22
```

登录之后，找到当前用户的SSH信任文件，路径为`~/.ssh/authorized_keys`，如果没有则创建。

```sh
mkdir -p ~/.ssh
```

接下来另开一个本地的Git Bash窗口，然后查看一下我们的公钥，选中点击右键将其复制到剪贴板中。

```sh
less ~/.ssh/id_rsa.pub
```

接着上面的linux SSH终端，这次将公钥粘贴到`authorized_keys`文件中，然后按`Ctrl+X`并选择Y保存。

```sh
nano ~/.ssh.authorized_keys
```

这样以后我们用ssh命令登录VPS的时候，就不需要密码了。

### 稍微简单一点的手动配置

上面的配置过程还是有点麻烦，而且配置完以后，每次登录还是得输入又臭又长的ssh命令。有没有更简单的办法呢？当然有，而且用了这种方法以后，你登录VPS仅需敲几个字母，简单到你以后可能没事就想登录VPS看看。

这需要利用SSH提供的配置文件功能。首先在本地打开或创建`~/.ssh/config`文件。

```sh
nano ~/.ssh/config
```

然后添加以下内容，第一段是让所有主机都开启自动保活功能，防止时间长不用终端导致连接断开。第二段就是具体的服务器配置，这里只需要输入用户名和端口号即可。为了安全起见，这里不允许指定密码，而且接下来开启密钥登录之后也不需要密码了。`Host`右面的名字不一定得是`vps`，你可以改成自己任意喜欢的名字，当然为了方便最好两三个字母就够了。

```txt
Host *
    ServerAliveInterval 10
    ServerAliveCountMax 20
Host vps
    Hostname 1.2.3.4
    User yitian
    Port 22
```

配置完毕之后，你就可以用`ssh vps`来取代又臭又长的命令`ssh yitian@1.2.3.4 -p 22`了。

这里我们仅仅让登录时候能少输入几个字母，密钥登录还是没配置完。接下来就来配置SSH登录。其实这个配置也很简单，SSH自带了一个简化命令，自动读取公钥并复制到服务器中，仅仅需要下面的命令即可。

```sh
ssh-copy-id yitian@1.2.3.4 -p 22
```

ssh-copy-id和ssh是一家，所以其实不用这么麻烦。配置了服务器信息之后，可以直接用`ssh-copy-id vps`来复制公钥。

完成之后，我们以后登录VPS的时候，仅需要输入`ssh vps`即可，是不是非常方便呢？

## 安装Meslo NF字体

如果你直接用了我前面的zsh主题，使用终端的时候会发现一大堆乱码。我配置的主题使用了增强的字体，所以如果你没有安装对应字体，显示就会乱码。解决办法很简单就，安装Meslo NF字体即可。

复制下面的链接下载字体。

```txt
https://github.com/ryanoasis/nerd-fonts/releases/download/v2.1.0/Meslo.zip
```

解压之后，全选，点击右键安装字体。然后在Git Bash左上角右键点击，然后选择Option，然后找到Fonts，将字体改成MesloLGS NF字体。

{% asset_img GitBash字体修改.png GitBash修改字体 %}

这时候再看看，是不是发现终端主题非常酷呢。

{% asset_img 主题效果.png 主题效果 %}

## 配置影梭

众所周知，VPS就是用来翻墙的，这里在介绍一下如何配置影梭。传统配置方法需要安装`shadowsocks-libev`，然后编辑配置文件并启动，这样稍微有点麻烦，而且不能方便的支持kcp等加速协议。所以这里我为大家介绍一个更简单的配置办法，利用docker和gost，使用我编写的脚本，自动配置并启动服务端。

### 安装docker

首先需要安装docker。

```sh
sudo apt install docker.io
```

然后将用户添加到docker组中，这样以后使用docker的时候无需sudo权限即可执行命令。

```sh
sudo gpasswd docker -a $(whoami)
```

然后启动docker服务，并令其开机自启。

```sh
sudo systemctl enable docker
sudo systemctl start docker
```

配置完毕之后，退出终端重新登录一下。然后运行下面的命令，看看能否正常运行，成功以后，就可以进行下一步了。

```sh
docker run --rm hello-world
```

### 使用gost_ss脚本启动服务

gost_ss是我写的一个利用docker和gost便捷启动影梭服务的脚本，支持影梭和影梭+kcp两种模式。使用方法很简单，首先下载脚本。

```sh
wget https://raw.githubusercontent.com/techstay/climbthewall/master/gost_ss.py
```

然后看看脚本的命令行参数。

```sh
python3 gost_ss.py -h
```

`-password`和`-port`参数是可选的，如果不指定会自动随机生成。如果指定了`-k`参数则使用kcp协议，如果使用kcp协议的话，还可以使用`-mode`参数指定kcp协议的模式。一般情况下我们的VPS每个月流量可能有500G-1T之间，而一个人用的话，也就用50G左右，根本用不完。所以可以将模式指定为`fast3`来获取最好的加速效果，相对应的，流量消耗最多。

```sh
python3 gost_ss.py -k -mode fast3
```

运行成功之后，脚本会显示出对应的客户端配置。linux用户的话，保存linux配置命令行。安卓等用户的话，复制配置字符串即可。当然如果你用kcp的话，需要客户端支持才行（例如安卓端安装kcp插件）。如果不懂的话，可以先配置影梭，以后再慢慢研究kcp。也可以看我博客那篇配置影梭的文章，里面介绍了客户端如何配置kcp。

好了，下面你应该可以享受自己的VPS了。如果有兴趣的话，还可以买个域名，然后自己搭个网站什么的。
