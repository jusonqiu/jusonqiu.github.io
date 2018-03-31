---
title: Android-NDK 设置NDK编译选项
date: 2015-03-06
---

在Android NDK 开发中,编译选项配置能够帮助我们很好的使用NDK/C/C++的特性,这些编译选项中那个包括:APP_ABI,LOCAL_LDLIBS, LOCAL_CFLAGS, APP_STL ...等,下面是这些选项中的一些配置.

# 选项概述

(一)  Android.mk

| 选项 | 解释 |
|:----|:-----|
|LOCAL_MODULE| 模块名|
|LOCAL_SRC_FILES| 模块依赖的编译源文件|
|LOCAL_STATIC_LIBRARIES| 第三方依赖库(静态.a)|
|LOCAL_SHARED_LIBRARIES| 第三方依赖库(动态.so)|
|LOCAL_LDLIBS,LOCAL_CFLAGS |  编译/链接选项 |

更多参考[官方文档 Android.mk](https://developer.android.com/ndk/guides/android_mk.html)

(二)  Application.mk

| 选项 | 解释 |
|:------|:------|
|APP_ABI | 目标平台的ABI类型|
|APP_STL |  C++标准库类型, 默认 system|
|APP_OPTIM| release/debug|

更多参考[官方文档 application.mk](https://developer.android.com/ndk/guides/application_mk.html)

ABI 全称是: Application binary interface, 应用程序二进制接口,不同的CPU芯片（如：ARM、Intel x86、MIPS）支持不同的ABI架构，常见的ABI类型包括：armeabi，armeabi-v7a，x86，x86_64，mips，mips64，arm64-v8a等。

最新 Android NDK 所支持的ABI类型及区别:
![android-abi.png](/images/android-abi.png)


Android NDK 支持的LOCAL_LDLIBS :
(1) Bionic libc 库
(2) Pthread 库(-lpthread)
(3) math 数学库(-lmath)
(4) C++ support library (lstdc++)

添加到“LOCAL_LDLIBS”中的不同版本的Android NDK所支持的库：
![android-libs.png](/images/android-libs.png)

Android NDK C++模板支持:
如在Application.mk下
```makefile
APP_STL := gnustl_static
```
NDK C++运行苦中支持情况:

| C++ Exceptions | C++ RTTI | Standard Library |
|:----------:|:-----------:|:------------:|
|system | no | no | no|
|gabi++ | no | yes| no|
|stlport| no| yes| yes|
|gnustl | yes| yes| yes|

对应的Applicaion.mk APP_STL 配置:
```
system          -> Use the default minimal system C++ runtime library.
gabi++_static   -> Use the GAbi++ runtime as a static library.
gabi++_shared   -> Use the GAbi++ runtime as a shared library.
stlport_static  -> Use the STLport runtime as a static library.
stlport_shared  -> Use the STLport runtime as a shared library.
gnustl_static   -> Use the GNU STL as a static library.
gnustl_shared   -> Use the GNU STL as a shared library.
```

# 编译选项
## 通用的编译
- ``-O2 或者 -O3 或者 -Os``
编译优化选项，一般选择O2，兼顾了优化程度与目标大小;

- `` -Wall ``
打开所有编译过程中的Warning, ``-Werror`` 打开所有错误编译输出, ``-Wno-int-to-pointer-cast`` 关闭int强制转换;

- ``-fPIC``
编译位置无关的代码，一般用于编译动态库;

- ``shared``
编译动态库, 如:
```shell
gcc -c -Wall -Werror -fpic test.c
gcc -shared -fPIC -o libtest.so test.o
```

- ``-fopenmp``
打开多核并行计算;

- ``-I<dir>``
配置头文件搜索路径，如果有多个-I选项，则路径的搜索先后顺序是从左到右的，即在前面的路径会被选搜索;

- ``-nostdinc``
该选项指示不要标准路径下的搜索头文件，而只搜索-I选项指定的路径和当前路径;

- ``–sysroot=<dir>``
用dir作为头文件和库文件的逻辑根目录，例如，正常情况下，如果编译器在/usr/include搜索头文件，在/usr/lib下搜索库文件，它将用dir/usr/include和dir/usr/lib替代原来的相应路径;

- ``-nostdlib``
该选项指示链接的时候不要使用标准路径下的库文件;

- ``-L<dir>, -l<name>`` 在路径 dir 下查找名为 libname.so 的库进行链接.

## ARM 平台相关的编译选项
- ``-marm -mthumb``
二选一，指定编译thumb指令集还是arm指令集;

- ``-march=name ``
指定特定的ARM架构，常用的包括：-march=armv6， -march=armv7-a;

- ``-mfpu=name``
给出目标平台的浮点运算处理器类型，常用的包括：-mfpu=neon，-mfpu=vfpv3-d16;

- ``-mfloat-abi=name``
给出目标平台的浮点预算ABI，支持的参数包括：“soft”, “softfp” and “hard”;

# 一些快速模板
- 编译静态库
```bash
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE = libhellos
LOCAL_CFLAGS = $(L_CFLAGS)
LOCAL_SRC_FILES = hellos.c
LOCAL_C_INCLUDES = $(INCLUDES)
LOCAL_SHARED_LIBRARIES := libcutils
LOCAL_COPY_HEADERS_TO := libhellos
LOCAL_COPY_HEADERS := hellos.h
include $(BUILD_STATIC_LIBRARY)
```

- 编译动态库
```bash
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE = libhellod
LOCAL_CFLAGS = $(L_CFLAGS)
LOCAL_SRC_FILES = hellod.c
LOCAL_C_INCLUDES = $(INCLUDES)
LOCAL_SHARED_LIBRARIES := libcutils
LOCAL_COPY_HEADERS_TO := libhellod
LOCAL_COPY_HEADERS := hellod.h
include $(BUILD_SHARED_LIBRARY)
```

- 使用静态库
```bash
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := hellos
LOCAL_STATIC_LIBRARIES := libhellos
LOCAL_SHARED_LIBRARIES :=
LOCAL_LDLIBS += -ldl
LOCAL_CFLAGS := $(L_CFLAGS)
LOCAL_SRC_FILES := mains.c
LOCAL_C_INCLUDES := $(INCLUDES)
include $(BUILD_EXECUTABLE)
```

- 使用动态库
```bash
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := hellod
LOCAL_MODULE_TAGS := debug
LOCAL_SHARED_LIBRARIES := libc libcutils libhellod
LOCAL_LDLIBS += -ldl
LOCAL_CFLAGS := $(L_CFLAGS)
LOCAL_SRC_FILES := maind.c
LOCAL_C_INCLUDES := $(INCLUDES)
include $(BUILD_EXECUTABLE)
```

-  父子模块

```bash
# 路径结构
jni_root
├── Android.mk --> 父目录mk
└── test
    └── Android.mk --> 子目录mk


# 父目录 Android.mk
LOCAL_PATH := $(call my-dir)
include $(call all-subdir-makefiles) 	# 打开编译子目录
include $(CLEAR_VARS)
LOCAL_MODULE := main
LOCAL_LDLIBS := -llog -landroid
LOCAL_STATIC_LIBRARIES := test			# 子模块依赖
LOCAL_SRC_FILES := main.c
include $(BUILD_SHARED_LIBRARY)

#子目录 Android.mk
INNER_SAVED_LOCAL_PATH := $(LOCAL_PATH) # 保存父目录路径
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := test 					# 子模块
LOCAL_SRC_FILES :=  test.c
include $(BUILD_STATIC_LIBRARY)
LOCAL_PATH := $(INNER_SAVED_LOCAL_PATH) # 父目录路径恢复
```
- Android NDK CLI 命令行可执行程序
```
 include $(CLEAR_VARS)
 LOCAL_CFLAGS := -fPIE # 用于生成位置无关的代码段, 否则无法正常执行
 LOCAL_LDFLAGS := -llog -rdynamic -fPIE -pie
 LOCAL_MODULE := test
 LOCAL_SRC_FILES := test.c
 include $(BUILD_EXECUTABLE)
```

# 参考
- [Android NDK](https://developer.android.com/ndk/guides/index.html)
- [GNU GCC Manual](https://gcc.gnu.org/onlinedocs/4.9.3/)
- [clang](http://clang.llvm.org/get_started.html)
- [libstdc++](https://gcc.gnu.org/onlinedocs/libstdc++/)
- [android dev NDK](https://developer.android.com/ndk/guides/cpp-support.html)
