---
title: 用Hexo制作自己的静态博客
date: 2017-11-14 00:21:20
tags: 
 - 前端
 - Hexo
 - 静态博客
categories:
---
博客是一个老东西了，如果我们想写博客的话，有两种选择，第一种是在博客网站上，例如QQ空间、新浪博客、简书等网站上申请账号，然后编写博客；第二种就是自己找服务器搭一个博客。搭建博客也有两种选择，第一种是搭建动态博客，这方面最流行的就是WordPress，第二种就是搭建静态博客，这方面有很多选择。说到功能上，动态博客当然更胜一筹，但是所需的服务器资源比较大，如果想取得较好的效果，就必须花钱购买服务器资源。静态博客功能比较单一，但是由于省资源，所以可以找到很多免费部署的资源（例如Github Pages）。

搭建静态博客这方面有很多工具可供选择，我看了看[Hexo](https://hexo.io/)是一个很不错的选择，使用人数比较多，功能也挺丰富，所以这里我就选择Hexo来搭建静态博客。这篇文章在很多地方也参考了[Hexo 官方文档](https://hexo.io/docs/)。

## 安装Hexo
在安装Hexo之前，首先需要安装[Node.js](http://nodejs.org/)和[Git](http://git-scm.com/)两个工具。

然后输入下面命令来安装Hexo。
```
$ npm install -g hexo-cli
```

## 建立博客
安装好Hexo之后，就可以建立博客了。建立博客需要输入以下命令。
```
$ hexo init <folder>
$ cd <folder>
$ npm install
```

等待npm安装好所有依赖包之后，博客就算建立好了。博客的文件结构如下面所示。
```
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```

## 配置博客
在项目外层文件夹中有一个`_config.yml`，这是博客项目的全局配置文件，在这里有很多选项需要我们修改。例如博客主标题、子标题、描述、作者、语言、时区、博客地址和根地址等等。这里列举的这些地址都需要我们根据自己需求进行修改。
```
title: 易天的静态个人博客
subtitle:
description:
author: 易天
language: zh-CN
timezone: Asia/Shanghai

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://yoursite.com
root: /
```

## Hexo命令
下面介绍一下Hexo的常用命令，方括号`[]`中包括的是可选项，尖括号`<>`中包括的是必选项。

### 创建新博客项目
如果未指定文件夹，hexo会在当前文件夹创建项目文件。
```
$ hexo init [folder]
```

### 创建新文章
如果未指定布局，会使用配置文件中的默认布局选项。如果文章标题中含有空格等字符，需要使用双引号包括标题。
```
$ hexo new [layout] <title>
```

### 生成静态博客
该命令会生成博客的静态文件。
```
$ hexo generate
```

### 启动本地服务器
启动本地服务器来开发博客，默认地址为 [http://localhost:4000/](http://localhost:4000/) 。
```
$ hexo server
```

还有一些命令这里就不介绍了，需要详细了解的话可以直接查看官方文档。

## 编写新文章
编写新文章使用下面的命令。
```
$ hexo new [layout] <title>
```

默认情况下布局有`post`、`page`、`draft`三个，它们所在的文件夹位置也不同。默认使用`post`布局，生成的文章会放在`source/_posts`下。执行完这个命令之后，在该文件夹会出现名为`<title>.md`的markdown格式文件，这就是我们的博客文件，我们可以按照markdown语法来编辑它。

项目中还有一个名为`scaffolds`的文件夹，里面存放的不同布局的模板。我希望所有文章都有分类属性，所以需要在`post.md`中添加`categories:`一行。这里在`---`之间包括的代码是文章的属性，将会由Hexo渲染为实际的样式。我们的博客文章需要写在这一部分的后面。
```
---
title: {{ title }}
date: {{ date }}
tags:
categories:
---
```

这里我写了一篇小文章，介绍了一点经验。
```
---
title: 在客户端上登录微软邮箱时提示您输出的用户名或密码不起作用的解决办法
date: 2017-11-13 18:42:56
tags:
 - 疑难杂症
 - 电子邮箱
categories:
 - 疑难杂症
---

有些同学可能会在用微软邮箱登录outlook或者其他邮箱客户端的时候，明明输入的是正确的用户名和密码，但是却提示“您输入的用户名或密码不起作用”。其实原因很简单，这是因为你的微软账号开启了二次验证，而邮箱客户端并不支持这个功能。

当然解决办法也很简单，登录[微软账户更多安全选项](https://account.live.com/proofs/Manage/additional)中，然后找到应用密码这个，将应用密码作为邮箱密码输入到邮箱客户端中即可解决问题。当然对于Xbox等无法登陆的问题，也可以使用这个方法来解决。
```

写完之后，使用下面的命令启动本地服务器，然后访问[http://localhost:4000/](http://localhost:4000/)查看一下博客效果。
```
$ hexo server
```

![博客效果](http://upload-images.jianshu.io/upload_images/832668-63febdf496729638.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 添加Disqus评论支持
静态博客因为是静态的，所以没有办法支持评论等功能。不过很多第三方评论服务都可以通过添加JS代码的方式让博客可以支持评论功能。原来国内比较著名的第三方评论系统有多说，可惜因为无法盈利已经关闭了。Hexo官方支持[Disqus](https://disqus.com/)，一个国外的第三方评论平台。当然，为了使用Disqus，首先你需要去注册一个账号，并添加一个网站。

默认情况下，Hexo的默认主题landscape是支持Disqus的，相应的源代码可以查看`themes\landscape\layout\_partial/after-footer.ejs`文件。当主配置文件中存在`disqus_shortname`选项，而且相应URL正确配置的话，Hexo就会自动显示Disqus评论。这是我的配置，这里的名称是我的网站的名称。
```
disqus_shortname: yitian-static-blog
```

成功配置之后，在每篇文章下面，应该就会看到一个Disqus评论框了。

## 静态资源处理
假如整个博客只有少量图片等静态资源，我们可以在`source`文件夹中新建一个`image`文件夹，然后将图片放置进去，在文章中使用MarkDown标准格式`[图片上传失败...(image-180063-1510595721877)]`即可访问。但是假如大部分文章都需要图片，那么这种方式就不太适用了。

这时候，我们可以在配置文件中设置`post_asset_folder`选项为`true`。这样在创建文章时，Hexo会同时创建一个和文章同名的文件夹，我们可以将每个文章单独的资源放入该文件夹，然后以相对路径的方使引用。
```
post_asset_folder: true
```

举个例子，假如图片名为`hello.jpg`，已经放置到文章同名的文件夹中，那么在文章中引用图片，可以使用标准Markdown形式`[图片上传失败...(image-22e4b0-1510595721878)]`。不过这种方式仅适用于在文章页面下，假如在主页或者归档页面查看文章，由于相对路径不同，图片是无法正常显示的。

对于这个问题，我们需要使用Hexo的标签插件来解决。这个插件在Hexo 3中已经包括到核心包中，所以我们可以直接使用，使用语法如下。如果图片名或标题有空格，需要使用双引号包括。
```
{% asset_link 图片名 图片标题 %}
```

这样一来，不管在哪个页面，图片都可以正常显示了。

## 发布博客
发布博客有很多种方式，如果你有一个自己的服务器，可以选择FTP、RSync、Git等多种方式发布到服务器。当然这里为了省事就直接发布到Github Pages上。由于Github Pages要求静态网站在项目的根目录或者`docs`目录下，所以这里还要对项目进行一下小修改，在配置文件中将发布路径改为`docs`。
```
public_dir: docs
```

然后生成静态页面。
```
$ hexo generate
```

然后将所有代码一起提交到Github上，并在设置中选择`docs`选项，然后保存。

![Github Pages设置](http://upload-images.jianshu.io/upload_images/832668-3f6285f159931e02.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后访问Github Pages的路径，就会发现项目已经出现了，当然样式都是乱的。因为我们还没有设置合适的URL。本地开发的话，路径直接就是域名。但是Github Pages的路径一般都不是以域名开头的，所以需要我们按照自己的项目路径进行修改，下面是我的项目配置。
```
url: https://techstay.github.io
root: /my-static-blog/
```

修改完毕之后别忘了执行`hexo generate`重新生成静态文件，然后再次提交，就可以发现这次项目完美的出现了。

当然这篇文章仅仅介绍了Hexo的一点点知识，Hexo还有丰富的主题、插件可供探索。这里就仅仅抛砖引玉，各位如果有兴趣的话请自行探索吧！最后附上我的静态博客地址[https://techstay.github.io/my-static-blog/](https://techstay.github.io/my-static-blog/)，欢迎大家查看！