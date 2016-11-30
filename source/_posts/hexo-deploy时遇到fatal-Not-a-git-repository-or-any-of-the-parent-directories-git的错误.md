---
title: >-
  使用hexo deploy时遇到fatal: Not a git repository (or any of the parent directories):
  .git的错误
date: 2016-11-26 22:33:27
tags:
---
<!-- more -->
## 描述信息
当使用hexo deploy命令将博客部署到github上是出现下面的错误信息：
```
Cosysn@Cosysn-PC MINGW64 /h/Dev/myBlog/cosysn.github.io
$ hexo deploy
INFO  Deploying: git
INFO  Clearing .deploy_git folder...
INFO  Copying files from public folder...
fatal: Not a git repository (or any of the parent directories): .git
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Error: fatal: Not a git repository (or any of the parent directories): .git

    at ChildProcess.<anonymous> (H:\Dev\myBlog\cosysn.github.io\node_modules\hexo-deployer-git\node_modules\hexo-util\lib\spawn.js:37:17)
    at emitTwo (events.js:87:13)
    at ChildProcess.emit (events.js:172:7)
    at ChildProcess.cp.emit (H:\Dev\myBlog\cosysn.github.io\node_modules\hexo-deployer-git\node_modules\hexo-util\node_modules\cross-spawn\lib\enoent.js:40:29)
    at maybeClose (internal/child_process.js:829:16)
    at Socket.<anonymous> (internal/child_process.js:319:11)
    at emitOne (events.js:77:13)
    at Socket.emit (events.js:169:7)
    at Pipe._onclose (net.js:486:12)
FATAL fatal: Not a git repository (or any of the parent directories): .git

Error: fatal: Not a git repository (or any of the parent directories): .git

    at ChildProcess.<anonymous> (H:\Dev\myBlog\cosysn.github.io\node_modules\hexo-deployer-git\node_modules\hexo-util\lib\spawn.js:37:17)
    at emitTwo (events.js:87:13)
    at ChildProcess.emit (events.js:172:7)
    at ChildProcess.cp.emit (H:\Dev\myBlog\cosysn.github.io\node_modules\hexo-deployer-git\node_modules\hexo-util\node_modules\cross-spawn\lib\enoent.js:40:29)
    at maybeClose (internal/child_process.js:829:16)
    at Socket.<anonymous> (internal/child_process.js:319:11)
    at emitOne (events.js:77:13)
    at Socket.emit (events.js:169:7)
    at Pipe._onclose (net.js:486:12)
```

检查git的配置没有问题

## 解决方案
将hexo根目录下的.deploy_git/删除掉，然后使用hexo deploy重新部署就可以了。
但是暂时不清楚是什么原因引起的错误。

## 参考文献
1. 搭建hexo博客并部署到github上(http://www.tuicool.com/articles/uE7FJba/)
