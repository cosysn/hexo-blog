---
title: esxi_sense_decode - 解析VMware ESXi/ESX中SCSI感知代码的工具
date: 2016-11-26 21:05:44
tags:
---
注意：本文仅用于ESXi 5.0及以以后的版本，其他的版本暂不支持。
<!-- more -->
## 引言
我们在定位ESXi或ESX出现的问题的时候，经常会在ESXi/ESX主机上的vmkernel.log或messages.log的系统日志文件中，见到类似下面的日志信息：
```
2011-04-04T21:07:30.257Z cpu2:2050)ScsiDeviceIO: 2315: Cmd(0x4124003edb00) 0x12, CmdSN 0x51 to dev "naa.600508e000000000c9f6baa7c19f6900" failed H:0x0 D:0x2 P:0x0 Valid sense data: 0x5 0x24 0x0. 
Mar 9 23:53:24 esx405 vmkernel: 2:14:39:54.069 cpu3:4300)ScsiDeviceIO: 1688: Command 0x28 to device "naa.60000970000292600219533031453245" failed H:0x1 D:0x0 P:0x3 Possible sense data: 0x0 0x0 0x0.
```
这些日志中记录了SCSI状态代码表示I/O以某种给定的状态失败。此状况对任何给定的工作负载而言可能是临时的、暂时性的、良性的或严重的，具体取决于收到的状态。如果能够从日志中记录的状态码中获得错误的原因，那么对于定位问题有很好的指导作用。

找出 SCSI 事件，并关注下面十六进制值：
```
H:0x8 D:0x0 P:0x0 Possible sense data: 0x0 0x0 0x0
```
有关如何解析这些SCSI状态码，请参见[Interpreting SCSI sense codes in VMware ESXi and ESX (289902)](https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=289902) 

但是如果每次出现问题都要去查一下文档，不仅费时费力，效率还低。为了提高效率，我开发了esxi_sense_decode工具用来解析SCSI感知代码， 省去了查询文档的麻烦。


## esxi_sense_decode

### 示例
如果在日志文件中打印了下面的日志信息：
```
2011-04-04T21:07:30.257Z cpu2:2050)ScsiDeviceIO: 2315: Cmd(0x4124003edb00) 0x12, CmdSN 0x51 to dev "naa.600508e000000000c9f6baa7c19f6900" failed H:0x0 D:0x2 P:0x0 Valid sense data: 0x5 0x24 0x0. 
```
那么可以使用esxi_sense_decode解析日志中的状态码。
```
[root@cosysn src]# ./esxi_sense_decode -h 0x0 -d 0x2 -p 0x0 -s 0x5 -a 0x24/0x0
==================== Host Status ======================
Code: [0x0]
Name: OK
Description: This status is returned when there is no error on the host side. This is when you see if there is a status for a device or plugin. This status is also when you see valid sense data instead of possible sense data.

==================== Device Status ======================
Code: [0x2]
Name: CHECK_CONDITION
Description: This status is returned when a command fails for a specific reason. When a CHECK CONDITION is received, the ESX storage stack will send out a SCSI command 0x3 (REQUEST SENSE) in order to get the SCSI sense data (Sense Key, Additional Sense Code, ASC Qualifier, and other bits). The sense data is listed after Valid sense data in the order of Sense Key, Additional Sense Code, and ASC Qualifier.

========================== Plugin Status ==========================
Code: [0x0]
Name: GOOD
Description: No error.(ESXi 5.x / 6.x only)

==================== Sense Key ======================
Code: [0x5]
Name: ILLEGAL REQUEST

==================== Additional Sense Data ======================
Code: [0x24/0x0]
Description: INVALID FIELD IN CDB
```
可以看到上面解析了每个状态代码所代表的意思。

## 参考文献
1. [Interpreting SCSI sense codes in VMware ESXi and ESX (289902)](https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=289902)
2. [esxi_sense_decode](https://github.com/cosysn/esxi_sense_decode)
