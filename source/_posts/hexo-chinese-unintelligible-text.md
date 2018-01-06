---
title: Hexo自动生成的RSS中出现中文乱码
date: 2017-07-17 08:47:41
tags:
---

#  Hexo自动生成的RSS中出现中文乱码

## 0. 问题描述

添加了hexo-generator-feed插件后自动生成RSS，但是在生成的atom.xml文件中如果存在中文的话，就会出现乱码，导致订阅RSS功能会出现异常。下面截取部分出现问题的atom.xml：

![](atom_chinese_unicode_num.png)

这个问题比较严重，需要解决。

## 1. 问题分析

1. 首先在hexo-generator-feed中添加调试信息，然后复现该问题，确认是否是hexo-genrator-feed引起的异常。

   ![](atom_debug_code.png)

2. 复现后，查看打印的调试信息

   ![](atom_xxx.png)

   可以看到很多乱码，而且发现是在hexo-generator-feed这个插件在处理之前就是乱码了，因此这些乱码不是这个插件处理有问题导致的。

   [Local Variables](https://hexo.io/api/locals.html)

3. 那么排除了这个插件的嫌疑点之后，那谁才是导致出现这个问题的罪魁祸首呢。

   我开始怀疑，是不是hexo在处理markdown生成相应的html文档时存在问题，导致生成的html文档就存在乱码。

   我找到了生成的html文档，发现生成的html文档中确实存在乱码，下面截取一部分片段：

   ![](generator_html.png)

   那么在这里就可以确定是hexo生成html文档时出现问题，导致生成的html文档中存在乱码。

4. 但是hexo生成html时，调用了很多插件，很难确定导致问题的原因。那么首先到网上去查询一下是否存在类似的问题，查询后发现有很多类似的问题，大多都跟文档的编码设置不正确有关。

   [解决hexo博客的乱码问题](http://www.cognize.me/2014/10/31/hexoluanma/)

   但是我将所有的博客的编码都设置为utf-8，尝试多次之后都无法解决这个问题，我开始怀疑可能跟文件的编码没有关系。

5. 后面的hexo的问题库中找到了问题的原因，是因为[cheerio](https://github.com/cheeriojs/cheerio)库存在编码处理的问题，而使用了hexo-asset-image这个插件，插件中使用了cheerio导致的。

   - [網頁能正常顯示中文，但查看HTML源代碼卻出現亂碼……](https://github.com/hexojs/hexo/issues/657)
   - [中文转码问题 #13](https://github.com/PaicHyperionDev/hexo-generator-search/issues/13)
   - [let cheerio do not decode entities #15](https://github.com/bubkoo/hexo-toc/pull/15)
   - [.html() method returns html with UNICODE when Chinese words are passed ](https://github.com/cheeriojs/cheerio/issues/565)
   - [中文编码问题 #12](https://github.com/CodeFalling/hexo-asset-image/issues/12)

   上面这些链接都是有这个原因导致的问题。

6. 分析到这里，问题的原因已经清楚了，下面将讲述如何解决这个问题。



## 2. 问题解决

查询hexo-asset-image的问题单库，发现该问题已经在 [中文编码问题 #12](https://github.com/CodeFalling/hexo-asset-image/issues/12) 中解决了，其解决的代码如下所示：

```javascript
 var $ = cheerio.load(data[key], {
        ignoreWhitespace: false,
        xmlMode: false,
        lowerCaseTags: false,
        decodeEntities: false // 解决问题的关键
      });
```

既然hexo-assert-image最新的版本已经解决了这个问题，我就想直接升级hexo-assert-image来解决这个问题。

```
npm update hexo-assert-image
```

使用上述命令升级hexo-assert-image后发现问题没有解决，应该是作者在修改了这个问题后，没有及时将修改发布到npm当中，导致使用npm无法更新到最新版本。



使用npm无法更新hexo-assert-image库，后面找到npm可以直接使用github安装库。其升级步骤如下：

1. 卸载老版本的hexo-assert-image函数库

   ```
   npm uninstall hexo-assert-image
   ```

2. npm直接安装github上的hexo-assert-image函数库

   ```
   npm install git+ssh://git@github.com:CodeFalling/hexo-asset-image.git
   ```

3. hexo重新生成静态博客

   ```
    hexo clean
    hexo g
   ```

4. 检查重新生成后的网页，发现问题已经解决，没有发现乱码

   ![](generator_cor_html.png)



## 3. 经验总结

1. 合理利用hexo的问题库，如果能够找到别人已经解决的问题，这样自己就可以少走一些弯路。
2. 大胆假设，小心求证。

## 4. 参考文献

1. [解决hexo博客的乱码问题](http://www.cognize.me/2014/10/31/hexoluanma/)
2. [網頁能正常顯示中文，但查看HTML源代碼卻出現亂碼……](https://github.com/hexojs/hexo/issues/657)
3. [中文转码问题 #13](https://github.com/PaicHyperionDev/hexo-generator-search/issues/13)
4. [let cheerio do not decode entities #15](https://github.com/bubkoo/hexo-toc/pull/15)
5. [.html() method returns html with UNICODE when Chinese words are passed ](https://github.com/cheeriojs/cheerio/issues/565)
6. [中文编码问题 #12](https://github.com/CodeFalling/hexo-asset-image/issues/12)
7. [npm 安装远程包（github的）](http://www.cnblogs.com/so-letitgo/p/5833595.html)
8. [npm如何安装github里的指定分支](https://segmentfault.com/q/1010000006244639)
9. [CodeFalling/hexo-asset-image](https://github.com/CodeFalling/hexo-asset-image)
10. [Hexo高级教程之插件开发](http://blog.csdn.net/melordljm/article/details/51985157)