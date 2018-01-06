---
title: 在VMware上安装Windows Server 2012失败-找不到Microsoft软件许可条款
date: 2016-12-02 00:28:59
tags:
---
## 描述信息
在VMware中安装Windows Server 2012时出现下面的错误：
```
Windows 找不到Microsoft软件许可证。请确保安装源有效，然后重新启动安装。
```

而且可以确认安装系统的ISO镜像是没有问题的，因为之前用过这个镜像安装过其他的系统。

## 解决方案
将虚拟机中的软盘删掉就可以正常安装了。
