---
title: "迁移虚拟机失败并显示错误：pbm error occurred during pre migrate check callback: connection refused"
date: 2017-07-17 23:20:57
tags:
---

# 0. 问题描述

冷迁移虚拟机时出现下面的错误：

![](migrate_fail.png)

需要解决这个迁移虚拟机失败的问题。



# 1. 问题分析

查询资料发现pbm error occurred during pre migrate check callback: connection refused这个错误是因为vmware-sps服务没有开启导致的。

[A general system occured: PBM error occurred during PreMigrateCheckCallback: Connection refused](http://blog.jamesblakeley.ca/?p=75)

[克隆虚拟机失败并显示错误：A general system error occurred:在 PreCloneCheckCallback 过程中出现 PBM 错误 (2124969)](https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2124969)[Issue with vMotion: PBM error during pre-check](http://vinfrastructure.it/2016/01/issue-with-vmotion-pbm-error/)

那么登陆到vcenter上查询vmware-sps服务有没有开启

![](esxi_vmware_sps.png)

查询后发现vmware-sps服务确实没有开启

# 2. 问题解决

那么该问题的解决办法：

![](esxi_slove.jpg)

直接重启vmware-sps服务即可。

![](esxi_vmware_sps_restart.png)

vmeare-sps服务重启之后，再重新迁移虚拟机，发现迁移虚拟机正常，不会再报错

开始迁移虚拟机：

![](migrate_start.png)

迁移虚拟机完成：

![](migrate_finish.png)

# 3. 经验总结

1. 善用搜索引擎，不要重复解决别人已经解决的问题；

# 4. 参考文献

1. [A general system occured: PBM error occurred during PreMigrateCheckCallback: Connection refused](http://blog.jamesblakeley.ca/?p=75)
2. [克隆虚拟机失败并显示错误：A general system error occurred:在 PreCloneCheckCallback 过程中出现 PBM 错误 (2124969)](https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2124969)[Issue with vMotion: PBM error during pre-check](http://vinfrastructure.it/2016/01/issue-with-vmotion-pbm-error/)



