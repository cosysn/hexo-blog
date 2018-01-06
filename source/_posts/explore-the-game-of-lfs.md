---
title: 摸索LFS的玩法
date: 2017-12-04 08:30:36
tags:
---

# 0. 简述
Beyond Linux From Scratch
BLFS
在LFS的基础上，增加了网络、X Windows、声音、打印机等，进一步完善了Linux系统。
从5.0版本开始，BLFS和LFS的版本号保持同步

Cross Linux from Scratch
CLFS
交叉编译系统

Hardened Linux From Scratch
HLFS
专注于安全

# 1. 
# 2. 准备宿主机环境
## 2.1. 软件要求
LFS对于宿主机的软件版本是有要求的，可以使用下列的脚本去查询所需软件的版本。
依赖软件的版本要求详情见[2.2. Host System Requirements](http://www.linuxfromscratch.org/lfs/view/stable/chapter02/hostreqs.html)
```
cat > version-check.sh << "EOF"
#!/bin/bash
# Simple script to list version numbers of critical development tools
export LC_ALL=C
bash --version | head -n1 | cut -d" " -f2-4
MYSH=$(readlink -f /bin/sh)
echo "/bin/sh -> $MYSH"
echo $MYSH | grep -q bash || echo "ERROR: /bin/sh does not point to bash"
unset MYSH

echo -n "Binutils: "; ld --version | head -n1 | cut -d" " -f3-
bison --version | head -n1

if [ -h /usr/bin/yacc ]; then
  echo "/usr/bin/yacc -> `readlink -f /usr/bin/yacc`";
elif [ -x /usr/bin/yacc ]; then
  echo yacc is `/usr/bin/yacc --version | head -n1`
else
  echo "yacc not found" 
fi

bzip2 --version 2>&1 < /dev/null | head -n1 | cut -d" " -f1,6-
echo -n "Coreutils: "; chown --version | head -n1 | cut -d")" -f2
diff --version | head -n1
find --version | head -n1
gawk --version | head -n1

if [ -h /usr/bin/awk ]; then
  echo "/usr/bin/awk -> `readlink -f /usr/bin/awk`";
elif [ -x /usr/bin/awk ]; then
  echo awk is `/usr/bin/awk --version | head -n1`
else 
  echo "awk not found" 
fi

gcc --version | head -n1
g++ --version | head -n1
ldd --version | head -n1 | cut -d" " -f2-  # glibc version
grep --version | head -n1
gzip --version | head -n1
cat /proc/version
m4 --version | head -n1
make --version | head -n1
patch --version | head -n1
echo Perl `perl -V:version`
sed --version | head -n1
tar --version | head -n1
makeinfo --version | head -n1
xz --version | head -n1

echo 'int main(){}' > dummy.c && g++ -o dummy dummy.c
if [ -x dummy ]
  then echo "g++ compilation OK";
  else echo "g++ compilation failed"; fi
rm -f dummy.c dummy
EOF

bash version-check.sh
```
执行上述脚本，查询所有依赖软件的版本如下图所示：
![](version_check.png)
检查后发现依赖的软件版本都满足条件。

## 2.2. 构建LFS的阶段
## 2.3. 创建新分区
LFS

## 2.5. 创建文件系统
```
mkfs -v -t ext4 /dev/<xxx>
```

如果存在交换分区，是不需要格式化该分区的，但是需要执行下面的命令初始化该分区
```
mkswap /dev/<yyy>
```

## 2.6. 设置$LFS变量
```
export LFS=/mnt/lfs
```
由于在构建LFS过程中会多次使用$LFS变量。需要确保在编译LFS的过程中$LFS的变量一直是被定义过的。

>注意：
>  当你离开并重新进入到当前环境都不要忘记检查$LFS的值有没有被设置。可以使用echo $LFS检查LFS的值是否设置正确。

## 2.7. mount新的分区
需要将格式化后的分区挂载到指定的挂载点上。
```
mkdir -pv $LFS
mount -v -t ext4 /dev/<xxx> $LFS
```
将其中的<xxx>替换成LFS分区

如果你的LFS使用了多个分区，可以使用下面的命令挂载
```
mkdir -pv $LFS
mount -v -t ext4 /dev/<xxx> $LFS
mkdir -v $LFS/usr
mount -v -t ext4 /dev/<yyy> $LFS/usr
```
需要将<xxx>和<yyy>替换成指定的分区

如果还使用了swap分区，可以使用swapon命令启用该分区
```
/sbin/swapon -v /dev/<zzz>
```
将<zzz>替换成swap分区的名字


# 3. 源码包和补丁
创建源代码编译用目录，修改权限并下载文件
需要将下载的源码包和补丁下载到$LFS/sources目录下
```
mkdir -v $LFS/sources
chmod -v a+wt $LFS/sources
wget http://mirror.jaleco.com/lfs/pub/lfs/lfs-packages/lfs-packages-8.0.tar
tar -xf lfs-packages-8.0.tar
mv 8.0/* sources/;rm -rf 8.0
```

校验下载的文件的md5值
```
pushd $LFS/sources
md5sum -c md5sums
popd
```

LFS依赖的所有的包和补丁的路径：
[All Packages](http://www.linuxfromscratch.org/lfs/view/stable/chapter03/packages.html)
[Needed Patches](http://www.linuxfromscratch.org/lfs/view/stable/chapter03/patches.html)

# 4. 最终的准备

## 4.1. 创建tools目录
使用root用户创建tools目录：
```
mkdir -v $LFS/tools
ln -sv $LFS/tools /
```

## 4.2. 创建LFS用户
如果我们使用root用户进行操作，当执行一个错误的操作可能会损坏系统。随意我们推荐使用非特权用户编译包。
```
groupadd lfs
useradd -s /bin/bash -g lfs -m -k /dev/null lfs
passwd lfs
chown -v lfs $LFS/tools
chown -v lfs $LFS/sources
su - lfs
```

## 4.3. 设置环境变量
```
cat > ~/.bash_profile << "EOF"
exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
EOF
```

```
cat > ~/.bashrc << "EOF"
set +h
umask 022
LFS=/mnt/lfs
LC_ALL=POSIX
LFS_TGT=$(uname -m)-lfs-linux-gnu
PATH=/tools/bin:/bin:/usr/bin
export LFS LC_ALL LFS_TGT PATH
EOF
```

```
source ~/.bash_profile
```

## 4.4. SBU

### 4.4.1. 定义
> The SBU measure works as follows. The first package to be compiled from this book is Binutils in Chapter 5. The time it takes to compile this package is what will be referred to as the Standard Build Unit or SBU. All other compile times will be expressed relative to this time.

### 4.4.2. 加速编译
```
export MAKEFLAGS='-j 2'
```


# 5. 构建临时系统
## 5.1. Binutils-2.29 - Pass 1


# m. 配置系统参数
## m.1. 设置环境变量
```
cat > ~/.bash_profile << "EOF"
exec env -i HOME=$HOME TERM=$TERM PS1='[\u:\H \w]\$ ' /bin/bash
EOF
```

# n. 参考文献
1. [Linux from scratch](http://www.linuxfromscratch.org/lfs/)
