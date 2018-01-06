---
title: 折腾Hexo遇到的那些坑（长期更新中）
date: 2017-07-18 07:47:40
tags:
---

# 折腾Hexo遇到的那些坑

## 1. 标题中使用特殊字符如:[]等

### 1.1. 问题描述

在文章的标题中增加了:符号后，发现hexo生成html文档失败，并且会打印异常的堆栈：

```
15:49:15.773 ERROR Process failed: _posts/esxi-migrate-vm-failed-with-pbm-error-occurred.md
YAMLException: incomplete explicit mapping pair; a key node is missed at line 1, column 73:
     ... uring pre migrate check callback: connection refused
                                         ^
    at generateError (H:\Dev\myBlog\cosysn.github.io\node_modules\js-yaml\lib\js-yaml\loader.js:165:10)
    at throwError (H:\Dev\myBlog\cosysn.github.io\node_modules\js-yaml\lib\js-yaml\loader.js:171:9)
    at readBlockMapping (H:\Dev\myBlog\cosysn.github.io\node_modules\js-yaml\lib\js-yaml\loader.js:1000:9)
    at composeNode (H:\Dev\myBlog\cosysn.github.io\node_modules\js-yaml\lib\js-yaml\loader.js:1332:12)
    at readDocument (H:\Dev\myBlog\cosysn.github.io\node_modules\js-yaml\lib\js-yaml\loader.js:1492:3)
    at loadDocuments (H:\Dev\myBlog\cosysn.github.io\node_modules\js-yaml\lib\js-yaml\loader.js:1548:5)
    at Object.load (H:\Dev\myBlog\cosysn.github.io\node_modules\js-yaml\lib\js-yaml\loader.js:1569:19)
    at parseYAML (H:\Dev\myBlog\cosysn.github.io\node_modules\hexo-front-matter\lib\front_matter.js:80:21)
    at parse (H:\Dev\myBlog\cosysn.github.io\node_modules\hexo-front-matter\lib\front_matter.js:56:12)
    at H:\Dev\myBlog\cosysn.github.io\node_modules\hexo\lib\plugins\processor\post.js:52:18
    at tryCatcher (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\util.js:16:23)
    at Promise._settlePromiseFromHandler (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\promise.js:509:35)
    at Promise._settlePromise (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\promise.js:569:18)
    at Promise._settlePromise0 (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\promise.js:614:10)
    at Promise._settlePromises (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\promise.js:693:18)
    at Promise._fulfill (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\promise.js:638:18)
    at PromiseArray._resolve (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\promise_array.js:126:19)
    at PromiseArray._promiseFulfilled (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\promise_array.js:144:14)
    at PromiseArray._iterate (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\promise_array.js:114:31)
    at PromiseArray.init [as _init] (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\promise_array.js:78:10)
    at Promise._settlePromise (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\promise.js:566:21)
    at Promise._settlePromise0 (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\promise.js:614:10)
    at Promise._settlePromises (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\promise.js:693:18)
    at Promise._fulfill (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\promise.js:638:18)
    at PromiseArray._resolve (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\promise_array.js:126:19)
    at PromiseArray._promiseFulfilled (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\promise_array.js:144:14)
    at Promise._settlePromise (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\promise.js:574:26)
    at Promise._settlePromise0 (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\promise.js:614:10)
    at Promise._settlePromises (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\promise.js:693:18)
    at Promise._fulfill (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\promise.js:638:18)
    at Promise._resolveCallback (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\promise.js:432:57)
    at Promise._settlePromiseFromHandler (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\promise.js:524:17)
    at Promise._settlePromise (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\promise.js:569:18)
    at Promise._settlePromise0 (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\promise.js:614:10)
    at Promise._settlePromises (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\promise.js:693:18)
    at Promise._fulfill (H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\promise.js:638:18)
    at H:\Dev\myBlog\cosysn.github.io\node_modules\bluebird\js\release\nodeback.js:42:21
    at H:\Dev\myBlog\cosysn.github.io\node_modules\graceful-fs\graceful-fs.js:78:16
    at tryToString (fs.js:456:3)
    at FSReqWrap.readFileAfterClose [as oncomplete] (fs.js:443:12)
```

### 1.2. 解决方案

YAML的语法中，在冒号后不能跟随特殊符号。故此需要使用其他方式替代。

通常解决方案有两个：

- **使用短杠-替代**
- 将整个字符串用引号`''`/`""`进行界定

### 1.3. 参考文献

1. ["incomplete explicit mapping pair; a key node is missed"#7](https://github.com/kadamwhite/expresspress/issues/7)
2. [May I use `[]` square brackets and `:` colon in article title?](https://github.com/hexojs/hexo/issues/1113)
3. [[hexo]技巧 - 标题使用特殊符号如英文冒号和方括号的方式](https://github.com/xovel/xovel.github.io/issues/9)
4. [Hexo常见问题解决方案](http://xuanwo.org/2014/08/14/hexo-usual-problem/)
5. [更新至hexo 2.8.0之后报错 #722](https://github.com/hexojs/hexo/issues/722)



## 2. 更新日志

* 2017年07月18日 完成初稿