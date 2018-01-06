---
title: Linux内核入门（一） -- EXPORT_SYMBOL小结
date: 2016-11-12 02:30:35
tags:
---
<!-- more -->
EXPORT_SYMBOL能够将函数以符号的形式导出给其他模块使用。当使用EXPORT_SYMBOL导出函数之后，我们可以在/proc/kallsyms中查到导出的函数，其显示格式如下：
```
[root@cosysn ~]# tail /proc/kallsyms 
ffffffffa000a720 T dm_io        [dm_mod]
ffffffffa0005860 T dm_get_device        [dm_mod]
ffffffffa00006d0 t dm_set_mdptr [dm_mod]
ffffffffa0004360 t dm_table_any_busy_target     [dm_mod]
ffffffffa0003e20 t dm_table_get_immutable_target_type   [dm_mod]
ffffffffa00007a0 t dm_suspended_md      [dm_mod]
ffffffffa00042a0 t dm_table_postsuspend_targets [dm_mod]
ffffffffa0006570 t dm_target_iterate    [dm_mod]
ffffffffa00044b0 T dm_consume_args      [dm_mod]
ffffffffa0007630 t dm_copy_name_and_uuid        [dm_mod]
```

对于EXPORT_SYMBOL如何实现符号导出的，下面做下简要的分析。


EXPORT_SYMBOL宏是定义在include/linux/export.h的头文件中，其定义如下：
``` c
/* 
 * For every exported symbol, place a struct in the __ksymtab section 
 */
#define __EXPORT_SYMBOL(sym, sec) \                             
        extern typeof(sym) sym;   \
        __CRC_SYMBOL(sym, sec)    \
        static const char __kstrtab_##sym[]         \
        __attribute__((section("__ksymtab_strings"), aligned(1)))  \
        = VMLINUX_SYMBOL_STR(sym);                  \
        extern const struct kernel_symbol __ksymtab_##sym;\
        __visible const struct kernel_symbol __ksymtab_##sym        \
        __used                                                        \
        __attribute__((section("___ksymtab" sec "+" #sym), unused))        \
        = { (unsigned long)&sym, __kstrtab_##sym }

#define EXPORT_SYMBOL(sym)               \
        __EXPORT_SYMBOL(sym, "")
        
#define EXPORT_SYMBOL_GPL(sym)          \
        __EXPORT_SYMBOL(sym, "_gpl")

#define EXPORT_SYMBOL_GPL_FUTURE(sym)   \
        __EXPORT_SYMBOL(sym, "_gpl_future")

```

下面举例说明EXPORT_SYMBOL的作用：
如果需要将transport_lookup_cmd_lun导出为内核符号，可以调用EXPORT_SYMBOL(transport_lookup_cmd_lun);
1. 将其展开得到__EXPORT_SYMBOL(transport_lookup_cmd_lun, "")
2. 将__EXPORT_SYMBOL展开，得到
```
  extern typeof(transport_lookup_cmd_lun) transport_lookup_cmd_lun;                                        \
        __CRC_SYMBOL(transport_lookup_cmd_lun, sec)                                        \
        static const char __kstrtab_transport_lookup_cmd_lun[]                        \
        __attribute__((section("__ksymtab_strings"), aligned(1))) \
        = VMLINUX_SYMBOL_STR(transport_lookup_cmd_lun);                                \
        extern const struct kernel_symbol __ksymtab_transport_lookup_cmd_lun;        \
        __visible const struct kernel_symbol __ksymtab_transport_lookup_cmd_lun        \
        __used                                                        \
        __attribute__((section("___ksymtab" sec "+" transport_lookup_cmd_lun), unused))        \
        = { (unsigned long)&transport_lookup_cmd_lun, __kstrtab_transport_lookup_cmd_lun }
```

下面我们对展开的宏进行分析：
```
static const char __kstrtab_transport_lookup_cmd_lun[]                        \
        __attribute__((section("__ksymtab_strings"), aligned(1))) \
        = VMLINUX_SYMBOL_STR(transport_lookup_cmd_lun);
``` 
该段代码表示其定义了一个名为__kstrtab_transport_lookup_cmd_lun的字符数组，并将其放到__ksymtab_strings的section下；

```
extern const struct kernel_symbol __ksymtab_transport_lookup_cmd_lun;        \
        __visible const struct kernel_symbol __ksymtab_transport_lookup_cmd_lun        \
        __used                                                        \
        __attribute__((section("___ksymtab" sec "+" transport_lookup_cmd_lun), unused))        \
        = { (unsigned long)&transport_lookup_cmd_lun, __kstrtab_transport_lookup_cmd_lun }
```
上段代码中出现了一个新的数据结构kernel_symbol，其表示内核符号
```
struct kernel_symbol
{
        unsigned long value; // 该内核符号的地址
        const char *name; // 该内核符号的名字
};
```
那么上段代码就很好理解了，其定义了一个名字为__ksymtab_transport_lookup_cmd_lun的内核符号结构用来存放符号transport_lookup_cmd_lun的地址和名字，
并将该__ksymtab_transport_lookup_cmd_lun放到___ksymtab的section下。

总结：
在内核符号导出中，调用了EXPORT_SYMBOL(sym)，则会完成以下操作：
(1)  定义一个字符数组存放内核导出符号的名称，并放置到“__ksymtab_strings”的section中。
(2)  定义一个内核符号结构用于存放导出符号的内存地址和名称，并放置到”__ksymatab”中。

即通过EXPORT_SYMBOL(sym)告诉了内核以外的世界关于这个符号的两点信息：内核符号的名称和其内存地址。

---------

[1]:   谈EXPORT_SYMBOL使用  http://blog.csdn.net/macrossdzh/article/details/4601648
[2]:  关于EXPORT_SYMBOL http://blog.csdn.net/lisan04/article/details/4076013
