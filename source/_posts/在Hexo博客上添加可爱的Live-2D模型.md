---
title: 在Hexo博客上添加可爱的Live 2D模型
date: 2018-09-15 22:56:47
tags: 
 - 前端
 - Hexo
 - 静态博客
categories:
 - 前端
---

在查找资料的偶然间，我发现一个博客上有非常可爱的Live 2D模型，当时我就被打动了，马上开启审查元素，试图找出这个Live 2D模型的信息，可是找了半天没找到。最后通过截图->谷歌图片的方式，终于一层一层的找到了相关资料，我正好有一个Hexo博客，所以今天就来在博客上添加一波Live 2D模型！

首先，安装npm包：
```
npm install --save hexo-helper-live2d
```
然后在hexo的配置文件`_config.yml`中添加如下配置，详细配置可以参考[文档](https://github.com/EYHN/hexo-helper-live2d/blob/master/README.zh-CN.md)：
```
live2d:
  enable: true
  scriptFrom: local
  pluginRootPath: live2dw/
  pluginJsPath: lib/
  pluginModelPath: assets/
  tagMode: false
  debug: false
  model:
    use: live2d-widget-model-shizuku
  display:
    position: right
    width: 150
    height: 300
  mobile:
    show: true
```
然后下载模型，模型名称可以到[这里](https://github.com/xiazeyu/live2d-widget-models)参考，一些模型的预览可以在[这里](https://huaji8.top/post/live2d-plugin-2.0/)。
```
npm install live2d-widget-model-shizuku
```


所有模型列表如下：

- `live2d-widget-model-chitose`
- `live2d-widget-model-epsilon2_1`
- `live2d-widget-model-gf`
- `live2d-widget-model-haru/01` (use `npm install --save live2d-widget-model-haru`)
- `live2d-widget-model-haru/02` (use `npm install --save live2d-widget-model-haru`)
- `live2d-widget-model-haruto`
- `live2d-widget-model-hibiki`
- `live2d-widget-model-hijiki`
- `live2d-widget-model-izumi`
- `live2d-widget-model-koharu`
- `live2d-widget-model-miku`
- `live2d-widget-model-ni-j`
- `live2d-widget-model-nico`
- `live2d-widget-model-nietzsche`
- `live2d-widget-model-nipsilon`
- `live2d-widget-model-nito`
- `live2d-widget-model-shizuku`
- `live2d-widget-model-tororo`
- `live2d-widget-model-tsumiki`
- `live2d-widget-model-unitychan`
- `live2d-widget-model-wanko`
- `live2d-widget-model-z16`

下载完之后，在Hexo根目录中新建文件夹live2d_models，然后在node_modules文件夹中找到刚刚下载的live2d模型，将其复制到live2d_models中，然后编辑配置文件中的`model.use`项，将其修改为live2d_models文件夹中的模型文件夹名称。

一切就绪之后，用`hexo server`命令启动服务器，稍等一下就可以看到右下角出现了一个可爱的萌萌哒的妹纸！


