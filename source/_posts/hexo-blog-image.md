---
title: Hexo博客支持图片
date: 2017-01-14 13:45:03
tags:
---

之前写博客都是文字描述，今天在写文章需要引用图片时才发现我的博客不支持图片，无法将图片加载到博客当中去。 查询之后发现[CodeFalling/hexo-asset-image](https://github.com/CodeFalling/hexo-asset-image)插件可以非常方便加载本地的图片。

## 安装

1. 首先确认`_config.yml`中有`post_asset_folder:true`。
Assets指的是那些不在source目录下的资源，比如图片、CSS文件或者Javascript文件。Hexo提供一种更方便的方法来管理这些资源（Assets）。想使其生效，首先修改 post_asset_folder 字段的设置，将其值改为 true 。
当生效后，在你创建文章的时候，Hexo会创建一个同名目录，你可以将该文章关联的资源全部放到该目录下。这样就可以更加方便的使用它们了。

2. 在Hexo目录下，使用npm安装hexo-asset-image
```
npm install hexo-asset-image --save
```

3. 安装完成后，用Hexo新建文章的时候就会在_post目录下创建一个和文章名字相同的目录，该文章相关联的图片可以放在这个目录下面。

## 用法
安装之后，就以本文为例介绍如何使用该插件在文章中插入图片。
1. 创建名字为hexo-blog-image的文章，其命令如下：

```
hexo new "hexo-blog-image"
```

文章创建好了之后，在_post目录下可以看到hexo-blog-image/目录，将准备好的图片放到该目录下面，如下的结构:
```shell
hexo-blog-image
├── logo.png
└── rules.jpg
hexo-blog-image.md
```

这样的目录结构（目录名和文章名一致），只要使用 `![logo](hexo-blog-image/logo.png)` 就可以插入图片。

如下图所示：
![logo](hexo-blog-image/logo.png)

生成的结构为
```
public/2017/01/14/hexo-blog-image
├── index.html
├── logo.png
└── rules.jpg
```
同时，生成的 html 是
`<img src="/2017/01/14/hexo-blog-image/logo.png" alt="logo">`
而不是愚蠢的
`<img src="hexo-blog-image/logo.png" alt="logo">`

## 缺陷
因为我的博客现在是托管在`Github`上面的，现在是有容量限制的，使用本地的图片可能会导致空间很快地消耗。

## 参考文献
[1] [hexo博客图片问题](http://www.jianshu.com/p/c2ba9533088a)
[2] [在 hexo 中无痛使用本地图片](http://www.tuicool.com/articles/umEBVfI)
[3] [CodeFalling/hexo-asset-image](https://github.com/CodeFalling/hexo-asset-image)