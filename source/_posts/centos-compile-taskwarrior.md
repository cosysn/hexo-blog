---
title: CentOS编译Taskwarrior
date: 2018-01-06 23:17:33
tags:
---

编译task时出现问题，报下面的错误：
```
[root@localhost task-2.5.1]# cmake -DCMAKE_BUILD_TYPE=release .  -DENABLE_SYNC=OFF
CMake Warning at CMakeLists.txt:18 (message):
  ENABLE_SYNC=OFF.  Not building sync support.


CMAKE_SYSTEM_NAME Linux
-- Looking for SHA1 references
-- Looking for libuuid
CMake Error at CMakeLists.txt:152 (message):
  -- libuuid not found.


-- Configuring incomplete, errors occurred!
See also "/root/task-2.5.1/CMakeFiles/CMakeOutput.log".
See also "/root/task-2.5.1/CMakeFiles/CMakeError.log".

```
查看错误信息，发现是找不到libuuid，但是我已经安装了uuid，还是报这个错误。


通过查看cmake的脚本文件，发现是找不到"uuid/uuid.h"
``` c
message ("-- Looking for libuuid")
if (DARWIN OR FREEBSD OR OPENBSD)
  # Apple and FreeBSD include the uuid functions in their libc, rather than libuuid
  check_function_exists (uuid_unparse_lower HAVE_UUID_UNPARSE_LOWER)
else (DARWIN OR FREEBSD OR OPENBSD)
  find_path    (UUID_INCLUDE_DIR   uuid/uuid.h)
  find_library (UUID_LIBRARY NAMES uuid)
  if (UUID_INCLUDE_DIR AND UUID_LIBRARY)
    set (TASK_INCLUDE_DIRS ${TASK_INCLUDE_DIRS} ${UUID_INCLUDE_DIR})
    set (TASK_LIBRARIES    ${TASK_LIBRARIES}    ${UUID_LIBRARY})
    # Look for uuid_unparse_lower
    set (CMAKE_REQUIRED_INCLUDES  ${CMAKE_REQUIRED_INCLUDES}  ${UUID_INCLUDE_DIR})
    set (CMAKE_REQUIRED_LIBRARIES ${CMAKE_REQUIRED_LIBRARIES} ${UUID_LIBRARY})
    check_function_exists (uuid_unparse_lower HAVE_UUID_UNPARSE_LOWER)
  else (UUID_INCLUDE_DIR AND UUID_LIBRARY)
    message (FATAL_ERROR "-- libuuid not found.")
  endif (UUID_INCLUDE_DIR AND UUID_LIBRARY)
endif (DARWIN OR FREEBSD OR OPENBSD)
```

执行下面的命令，解决上面的问题。
```
sudo yum install e2fsprogs-devel
sudo yum install uuid-devel
sudo yum install libuuid-devel
```

按照下面的命令编译成功：
```
cmake -DCMAKE_BUILD_TYPE=release .  -DENABLE_SYNC=OFF
make
make install
```

编译并安装成功后，可以使用task命令
```
[root@localhost task-2.5.1]# task
[task next]
No matches.
```

## 参考文献
1. [Taskwarrior](https://taskwarrior.org/)
2. [CentOS #include <uuid/uuid.h> 找不到文件解决方法](http://blog.csdn.net/shell2008/article/details/40616911)
