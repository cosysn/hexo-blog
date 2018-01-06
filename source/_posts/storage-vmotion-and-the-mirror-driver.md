---
title: Storage vMotion and the Mirror Driver
date: 2017-01-14 19:09:01
tags:
---

在使用ESXi迁移虚拟机的时候，我在vmkernel.log中发现下面一段日志：
```
	Line 2964: 2016-05-19T12:59:29.720Z cpu17:808643)SVM: 5139: SVM_MakeDev.5139: Creating device 10a80a6-819390-svmmirror: Success
	Line 2965: 2016-05-19T12:59:29.720Z cpu17:808643)SVM: 5188: Created device 10a80a6-819390-svmmirror, primary 10a80a6, secondary 819390
	Line 3094: 2016-05-19T13:00:25.159Z cpu21:34821)Vol3: 2675: Could not open device '10a80a6-819390-svmmirror' for probing: Busy
	Line 3095: 2016-05-19T13:00:25.159Z cpu21:34821)Vol3: 1072: Could not open device '10a80a6-819390-svmmirror' for volume open: Busy
	Line 3096: 2016-05-19T13:00:25.159Z cpu21:34821)Vol3: 1072: Could not open device '10a80a6-819390-svmmirror' for volume open: Busy
	Line 3097: 2016-05-19T13:00:25.159Z cpu21:34821)FSS: 5327: No FS driver claimed device '10a80a6-819390-svmmirror': No filesystem on the device
	Line 3643: 2016-05-19T13:07:32.032Z cpu22:34821)Vol3: 2675: Could not open device '10a80a6-819390-svmmirror' for probing: Busy
	Line 3644: 2016-05-19T13:07:32.032Z cpu22:34821)Vol3: 1072: Could not open device '10a80a6-819390-svmmirror' for volume open: Busy
	Line 3645: 2016-05-19T13:07:32.032Z cpu22:34821)Vol3: 1072: Could not open device '10a80a6-819390-svmmirror' for volume open: Busy
	Line 3646: 2016-05-19T13:07:32.032Z cpu22:34821)FSS: 5327: No FS driver claimed device '10a80a6-819390-svmmirror': No filesystem on the device
	Line 4145: 2016-05-19T13:16:10.123Z cpu15:34821)Vol3: 2675: Could not open device '10a80a6-819390-svmmirror' for probing: Busy
	Line 4146: 2016-05-19T13:16:10.123Z cpu15:34821)Vol3: 1072: Could not open device '10a80a6-819390-svmmirror' for volume open: Busy
	Line 4147: 2016-05-19T13:16:10.123Z cpu15:34821)Vol3: 1072: Could not open device '10a80a6-819390-svmmirror' for volume open: Busy
	Line 4148: 2016-05-19T13:16:10.123Z cpu15:34821)FSS: 5327: No FS driver claimed device '10a80a6-819390-svmmirror': No filesystem on the device
	Line 4849: 2016-05-19T13:25:12.841Z cpu9:34803)Vol3: 2675: Could not open device '10a80a6-819390-svmmirror' for probing: Busy
	Line 4850: 2016-05-19T13:25:12.842Z cpu9:34803)Vol3: 1072: Could not open device '10a80a6-819390-svmmirror' for volume open: Busy
	Line 4851: 2016-05-19T13:25:12.842Z cpu9:34803)Vol3: 1072: Could not open device '10a80a6-819390-svmmirror' for volume open: Busy
	Line 4852: 2016-05-19T13:25:12.842Z cpu9:34803)FSS: 5327: No FS driver claimed device '10a80a6-819390-svmmirror': No filesystem on the device
	Line 5021: 2016-05-19T13:27:20.668Z cpu16:808643)SVM: 2428: SVM Mirrored mode IO stats for device: 10a80a6-819390-svmmirror
	Line 5024: 2016-05-19T13:27:20.668Z cpu16:808643)SVM: 2450: Destroyed device 10a80a6-819390-svmmirror
```

有很多`10a80a6-819390-svmmirror busy`的打印，当时从这里能够找到一些线索，于是到查找`svmmirror`相关的资料，发现找到一些有用的资料，虽然没有从`svmmirror`中得到更多的线索，但是对于`ESXi`的`Storage vMotion`有了更深的理解。

`Svmmirror`是由`mirror driver`实现的，下面我就简单介绍一下`mirror driver`：

`Mirror Mode`是在`vSphere 5.0`合入到`Storage vMotion`的一个新特性，它能够让`Storage vMotion`更快、更高效。

在`Storage vMotion`迁移虚拟机过程中会遇到一个的问题：拷贝虚拟机的虚拟磁盘时，部分数据块已经拷贝到目的磁盘，当正在运行过程中的虚拟机向已经被拷贝到目的磁盘所在的数据块中写数据，那么如何将修改后的数据块同步到目的磁盘保证数据的一致性？

`Storage vMotion`提出了很多办法来解决这个问题：
1.	快照技术
2.	`Change Block Tracking(CBT)`
3.	`Mirror Mode`
其中`Mirror Mode`的性能最好。其工作原理很简单，就是将I/O做下镜像。换句话说，当一个正在进行`Storage vMotio`n的虚拟机要写数据到虚拟机磁盘当中，写数据的I/O会同时向源磁盘和目的磁盘下发，只有当源磁盘和目的磁盘都写成功之后才会通知虚拟机I/O已完成。 

`Mirror Mode`对于迁移虚拟机的性能提升是非常巨大的，详情请参考这个[论文](https://www.usenix.org/legacy/events/atc11/tech/final_files/Mashtizadeh.pdf)。

了解完了`Mirror Mode`，那么我们看看对`Storage vMotion`迁移流程的影响：
1.	由`VPXA`将虚拟机的工作目录拷贝到目的磁盘上；
2.	使用拷贝过去的文件在目标磁盘上创建并启动“影子”虚拟机，这个虚拟机会处于空闲状态并等待拷贝数据完成。为防止虚拟机的工作目录被转移，虚拟机被创建为失效保护模式；
3.	`Storage vMotion`启动`Mirror Driver`将已拷贝到目的磁盘的数据镜像写入源和目标磁盘；
4.	经过一轮拷贝，使用镜像`I/O`就可以将虚拟机的磁盘文档完整的拷贝到目的磁盘上。
5.	`Storage vMotion`调用虚拟机的快速挂起和恢复（类似于`vMotion`），将正在运行的虚拟机转移到空闲影子虚拟机。
6.	在快速挂起和恢复完成后，旧的工作目录和虚拟磁盘文件将从源磁盘中删除。


然后我通过测试`Storage vMotion`过程中实际使用了`Mirror Mode`。打开虚拟机的日志文件，找打下面的打印：
```
2016-05-19T12:59:29.721Z| vcpu-0| I120: DISKLIB-LIB_MISC   : Opening mirror node /vmfs/devices/svm/10a80a6-819390-svmmirror
2016-05-19T12:59:29.721Z| vcpu-0| I120: Creating virtual dev for scsi0:0
2016-05-19T12:59:29.721Z| vcpu-0| I120: DumpDiskInfo: scsi0:0 createType=11, capacity = 33554432, numLinks = 1, allocationType = 1
2016-05-19T12:59:29.721Z| vcpu-0| I120: SCSIDiskESXPopulateVDevDesc: Using FS backend
2016-05-19T12:59:29.721Z| vcpu-0| I120: DISKUTIL: scsi0:0 : geometry=2088/255/63
2016-05-19T12:59:29.723Z| vcpu-0| I120: VMXNET3 user: Ethernet1 Driver Info: version = 16778496 gosBits = 2 gosType = 1, gosVer = 0, gosMisc = 0
2016-05-19T12:59:29.723Z| Worker#0| I120: SVMotion: Enter Phase 8
2016-05-19T12:59:30.012Z| Worker#0| I120: Disk/File copy started for /vmfs/volumes/5739734e-f309ac87-4491-f8bc123449c2/W1_2/W1_2.vmdk.
2016-05-19T12:59:35.681Z| vcpu-0| I120: HBACommon: First write on scsi0:0.fileName='/vmfs/volumes/5739734e-f309ac87-4491-f8bc123449c2/W1_2/W1_2.vmdk'
2016-05-19T12:59:35.681Z| vcpu-0| I120: DDB: "longContentID" = "40d56f2945fea90820e1995ff9f13b36" (was "31f679521f5e9755ae9f840e8c55ccfc")
```
