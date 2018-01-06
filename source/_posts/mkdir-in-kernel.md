---
title: 内核中创建目录
date: 2018-01-06 23:46:34
tags:
---

在内核态中创建目录
vfs_mkdir创建目录
umode_t 访问权限
S_IRUGO
S_IWUSR

刚开始是参考mkdir系统调用的sys_mkdirat函数来实现的，其实现的代码如下：
```
static int my_mkdir(const char *name, umode_t mode)
{
    struct dentry *dentry;
    struct path path;
    int error;
    unsigned int lookup_flags = LOOKUP_DIRECTORY;

retry:
    dentry = user_path_create(AT_FDCWD, name, &path, lookup_flags);
    if (IS_ERR(dentry)) {
        return PTR_ERR(dentry);
    }

    if (!IS_POSIXACL(path.dentry->d_inode))
        mode &= ~current_umask();

    error = security_path_mkdir(&path, dentry, mode);
    if (!error) {
        error = vfs_mkdir(path.dentry->d_inode, dentry, mode);
    }

    done_path_create(&path, dentry);
    if (retry_estale(error, lookup_flags)) {
        lookup_flags |= LOOKUP_REVAL;
        goto retry;
    }

    return error;
}
```
但是在测试的时候会报Bad Address的错误
```
Mar 22 00:16:33 cosysn kernel: create a usr directory in kernel.
Mar 22 00:16:33 cosysn kernel: Failed to create directory, errno[-14]
```
定位后发现是user_path_create函数会调用strncpy_from_user从用户态拷贝目录名，但是我们
是在内核直接调用的，导致报错。后面找到了kern_path_create函数，能够实现和user_path_create
相同的功能，而且是在内核中使用的。

修改后测试通过，其中修改后的代码是：
```
static int my_mkdir(const char *name, umode_t mode)
{
    struct dentry *dentry;
    struct path path;
    int error;
    unsigned int lookup_flags = LOOKUP_DIRECTORY;

retry:
    dentry = kern_path_create(AT_FDCWD, name, &path, lookup_flags);
    if (IS_ERR(dentry)) {
        return PTR_ERR(dentry);
    }

    if (!IS_POSIXACL(path.dentry->d_inode))
        mode &= ~current_umask();

    error = security_path_mkdir(&path, dentry, mode);
    if (!error) {
        error = vfs_mkdir(path.dentry->d_inode, dentry, mode);
    }

    done_path_create(&path, dentry);
    if (retry_estale(error, lookup_flags)) {
        lookup_flags |= LOOKUP_REVAL;
        goto retry;
    }

    return error;
}
```
后面阅读内核的代码，发现内核之中也有类似的实现，如devtmpfs模块中的dev_mkdir：
```
static int dev_mkdir(const char *name, umode_t mode)
{
    struct dentry *dentry;
    struct path path;
    int err;

    dentry = kern_path_create(AT_FDCWD, name, &path, LOOKUP_DIRECTORY);
    if (IS_ERR(dentry))
        return PTR_ERR(dentry);

    err = vfs_mkdir(path.dentry->d_inode, dentry, mode);
    if (!err)
        /* mark as kernel-created inode */
        dentry->d_inode->i_private = &thread;
    done_path_create(&path, dentry);
    return err;
}
```
后面可以研究一下如何在内核中删除目录、创建文件、读文件、写文件、删除文件等操作。





后面附上完整的代码：
```
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/syscalls.h>
#include <linux/security.h>
#include <linux/namei.h>


static int my_mkdir(const char *name, umode_t mode)
{
    struct dentry *dentry;
    struct path path;
    int error;
    unsigned int lookup_flags = LOOKUP_DIRECTORY;

retry:
    dentry = kern_path_create(AT_FDCWD, name, &path, lookup_flags);
    if (IS_ERR(dentry)) {
        return PTR_ERR(dentry);
    }

    if (!IS_POSIXACL(path.dentry->d_inode))
        mode &= ~current_umask();

    error = security_path_mkdir(&path, dentry, mode);
    if (!error) {
        error = vfs_mkdir(path.dentry->d_inode, dentry, mode);
    }

    done_path_create(&path, dentry);
    if (retry_estale(error, lookup_flags)) {
        lookup_flags |= LOOKUP_REVAL;
        goto retry;
    }

    return error;
}

static int hello_init(void)
{
    int ret;
    printk(KERN_INFO "create a usr directory in kernel.\n");
    ret = my_mkdir("/root/abc", S_IRUGO | S_IWUSR);
    if (ret)
        printk(KERN_ERR "Failed to create directory, errno[%d]\n", ret);
    return ret;
}


static void hello_exit(void)
{
    printk(KERN_INFO "Byebye\n");
}


module_init(hello_init);
module_exit(hello_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Cosysn");
MODULE_DESCRIPTION("This is a demo!\n");

```