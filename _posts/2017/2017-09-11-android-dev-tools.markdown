---
layout: "post"
title: "android-dev-tools"
date: "2017-09-11 15:19"
---

## Android Debug Bridge(adb) 工具使用

```
# adb 位置
$ANDROID_SDK_ROOT/platform-tools/
├── adb
```

- 进入指定设备 `adb -s serialNumber shell`
- 查看版本 `adb version`
- 查看日志 `adb logcat`
- 查看设备 `adb devices`
- 连接状态 `adb get-state`
- 启动ADB服务 `adb start-server`
- 停止ADB服务 `adb kill-server`
- 电脑推送到手机 `adb push local remote`
- 手机拉取到电脑 `adb pull remote local`
- 通过 TCP 链接 Android

```
adb tcpip 9999
adb connect 192.168.8.103:9999
```

## am 、pm 工具

### am 全称activity manager

- 启动app `am start -n {packageName}/.{activityName}`
- 杀app的进程 `am kill <packageName>`
- 强制停止一切 `am force-stop <packageName>`
- 启动服务`am startservice`
- 停止服务 `am stopservice`
- 打开美拍 `am start -a android.intent.action.VIEW -d http://www.meipai.com`
- 拨打10086 `am start -a android.intent.action.CALL -d tel:10086`

### pm全称package manager

```
pm <command>
```

- 列出手机所有的包名 `pm list packages`
- 安装/卸载 `pm install/uninstall`



> 结合 adb shell 使用； 如 adb shell ifconfig

```
adb shell "cat /data/local/tmp/start_lldb_server.sh | run-as com.meitu.demo sh -c 'cat > /data/data/com.meitu.demo/lldb/bin/start_lldb_server.sh && chmod 700 /data/data/com.meitu.demo/lldb/bin/start_lldb_server.sh'"
```

## 常用的一些调试节点

> 查看节点值，例如：cat /sys/class/leds/lcd-backlight/brightness 修改节点值，例如：echo 128 > sys/class/leds/lcd-backlight/brightness

- LPM: `echo N > /sys/modue/lpm_levels/parameters/sleep_disabled`
- 亮度：`/sys/class/leds/lcd-backlight/brightness`
- CPU: `/sys/devices/system/cpu/cpu0/cpufreq`
- GPU: `/sys/class/ kgsl/kgsl-3d0/gpuclk`
- 限频：`cat /data/pmlist.config`
- 电流： `cat /sys/class/power_supply/battery/current_now`
- 查看Power： `dumpsys power`
- WIFI :`data/misc/wifi/wpa_supplicant.conf`
- 持有wake_lock: `echo a> sys/power/wake_lock`
- 释放wake_lock:`echo a> sys/power/wake_unlock`
- 查看Wakeup_source: `cat sys/kernel/debug/wakeup_sources`
- Display(关闭AD):`mv /data/misc/display/calib.cfg /data/misc/display/calib.cfg.bak 重启`
- 关闭cabc：`echo 0 > /sys/device/virtual/graphics/fb0/cabc_onoff`
- 打开cabc：`echo 3 > /sys/device/virtual/graphics/fb0/cabc_onoff`
- systrace：`sdk/tools/monitor`
- 限频：`echo /sys/devices/system/cpu/cpu0/cpufreq/scaling_max_freq 1497600`
- 当出现read-only 且 remount命令不管用时：`adb shell mount -o rw,remount /`
- 进入9008模式： `adb reboot edl`
- 查看高通gpio：`sys/class/private/tlmm 或者 sys/private/tlmm`
- 查看gpio占用情况：`sys/kernle/debug/gpio`

# Busybox【[下载](https://www.busybox.net/)】

```
$ busybox
BusyBox v1.21.1 (2013-07-08 10:26:30 CDT) multi-call binary.
BusyBox is copyrighted by many authors between 1998-2012.
Licensed under GPLv2. See source distribution for detailed
copyright notices.

Usage: busybox [function [arguments]...]
   or: busybox --list[-full]
   or: busybox --install [-s] [DIR]
   or: function [arguments]...

	BusyBox is a multi-call binary that combines many common Unix
	utilities into a single executable.  Most people will create a
	link to busybox for each function they wish to use and BusyBox
	will act like whatever it was invoked as.

Currently defined functions:
	[, [[, acpid, add-shell, addgroup, adduser, adjtimex, arp, arping, ash,
	awk, base64, basename, beep, blkid, blockdev, bootchartd, brctl,
	bunzip2, bzcat, bzip2, cal, cat, catv, chat, chattr, chgrp, chmod,
	chown, chpasswd, chpst, chroot, chrt, chvt, cksum, clear, cmp, comm,
	conspy, cp, cpio, crond, crontab, cryptpw, cttyhack, cut, date, dc, dd,
	deallocvt, delgroup, deluser, depmod, devmem, df, dhcprelay, diff,
	dirname, dmesg, dnsd, dnsdomainname, dos2unix, du, dumpkmap,
	dumpleases, echo, ed, egrep, eject, env, envdir, envuidgid, ether-wake,
	expand, expr, fakeidentd, false, fbset, fbsplash, fdflush, fdformat,
	fdisk, fgconsole, fgrep, find, findfs, flock, fold, free, freeramdisk,
	fsck, fsck.minix, fsync, ftpd, ftpget, ftpput, fuser, getopt, getty,
	grep, groups, gunzip, gzip, halt, hd, hdparm, head, hexdump, hostid,
	hostname, httpd, hush, hwclock, id, ifconfig, ifdown, ifenslave,
	ifplugd, ifup, inetd, init, insmod, install, ionice, iostat, ip,
	ipaddr, ipcalc, ipcrm, ipcs, iplink, iproute, iprule, iptunnel,
	kbd_mode, kill, killall, killall5, klogd, last, less, linux32, linux64,
	linuxrc, ln, loadfont, loadkmap, logger, login, logname, logread,
	losetup, lpd, lpq, lpr, ls, lsattr, lsmod, lsof, lspci, lsusb, lzcat,
	lzma, lzop, lzopcat, makedevs, makemime, man, md5sum, mdev, mesg,
	microcom, mkdir, mkdosfs, mke2fs, mkfifo, mkfs.ext2, mkfs.minix,
	mkfs.vfat, mknod, mkpasswd, mkswap, mktemp, modinfo, modprobe, more,
	mount, mountpoint, mpstat, mt, mv, nameif, nanddump, nandwrite,
	nbd-client, nc, netstat, nice, nmeter, nohup, nslookup, ntpd, od,
	openvt, passwd, patch, pgrep, pidof, ping, ping6, pipe_progress,
	pivot_root, pkill, pmap, popmaildir, poweroff, powertop, printenv,
	printf, ps, pscan, pstree, pwd, pwdx, raidautorun, rdate, rdev,
	readahead, readlink, readprofile, realpath, reboot, reformime,
	remove-shell, renice, reset, resize, rev, rm, rmdir, rmmod, route, rpm,
	rpm2cpio, rtcwake, run-parts, runlevel, runsv, runsvdir, rx, script,
	scriptreplay, sed, sendmail, seq, setarch, setconsole, setfont,
	setkeycodes, setlogcons, setserial, setsid, setuidgid, sh, sha1sum,
	sha256sum, sha3sum, sha512sum, showkey, slattach, sleep, smemcap,
	softlimit, sort, split, start-stop-daemon, stat, strings, stty, su,
	sulogin, sum, sv, svlogd, swapoff, swapon, switch_root, sync, sysctl,
	syslogd, tac, tail, tar, tcpsvd, tee, telnet, telnetd, test, tftp,
	tftpd, time, timeout, top, touch, tr, traceroute, traceroute6, true,
	tty, ttysize, tunctl, udhcpc, udhcpd, udpsvd, umount, uname, unexpand,
	uniq, unix2dos, unlzma, unlzop, unxz, unzip, uptime, users, usleep,
	uudecode, uuencode, vconfig, vi, vlock, volname, wall, watch, watchdog,
	wc, wget, which, who, whoami, whois, xargs, xz, xzcat, yes, zcat, zcip
```

## 安装

```
adb push busybox /data/local/tmp/busybox
adb shell su
mount -o remount,rw /system
cp /data/local/tmp/busybox /system/bin/
```

## Android NDK Tools

- cmake

  CMakeLists.txt

- nkd-build

  Android.mk Application.mk

[android-cmake-quick-build-script](https://gist.githubusercontent.com/jusonqiu/2cc7ac64ae3b87507a8ad18fae46159e/raw/fc239ba8f81f59787bbdb8e6713509ad754eee16/android-cmake-quick-build.py)

## JNI语法

### 基本数据类型

![DataType](/images/topic_jni_primitive_type.png)

![ObjectType](/images/topic-jobject.png)

### 常用函数

![methods](/images/topic-ndk_jmethods.png)

### 映射

#### 函数直接映射

HelloDemo.java

```java
package com.example.qiuzhaosheng.demo;
public class HelloDemo {
    private native void say(String who);
    private native String who();
}
```

使用 Javah 可以生产对应的 .h 头文件； 如

```
# com/example/qiuzhaosheng/demo/HelloDemo.java
$ pwd
/Users/qiuzhaosheng/AndroidStudioProjects/Demo/app/src/main/java

$ find $(pwd) -name HelloDemo.java
/Users/qiuzhaosheng/AndroidStudioProjects/Demo/app/src/main/java/com/example/qiuzhaosheng/demo/HelloDemo.java

$ javah javah -o ../cpp/HelloDemo.h -classpath . -jni com.example.qiuzhaosheng.demo.HelloDemo
```

HelloDemo.h

```c
/* DO NOT EDIT THIS FILE - it is machine generated */
#include <jni.h>
/* Header for class com_example_qiuzhaosheng_demo_HelloDemo */

#ifndef _Included_com_example_qiuzhaosheng_demo_HelloDemo
#define _Included_com_example_qiuzhaosheng_demo_HelloDemo
#ifdef __cplusplus
extern "C" {
#endif
/*
 * Class:     com_example_qiuzhaosheng_demo_HelloDemo
 * Method:    say
 * Signature: (Ljava/lang/String;)V
 */
JNIEXPORT void JNICALL Java_com_example_qiuzhaosheng_demo_HelloDemo_say
  (JNIEnv *, jobject, jstring);

/*
 * Class:     com_example_qiuzhaosheng_demo_HelloDemo
 * Method:    who
 * Signature: ()Ljava/lang/String;
 */
JNIEXPORT jstring JNICALL Java_com_example_qiuzhaosheng_demo_HelloDemo_who
  (JNIEnv *, jobject);

#ifdef __cplusplus
}
#endif
#endif

```

#### JNI_OnLoad 动态注册(推荐)

java 函数： ``public native int add(int a, int b);``

JNI_OnLoad 实现注册 :

```cpp
static const char *calss_name = "com/example/qiuzhaosheng/demo/MainActivity";
static JNINativeMethod methods[] = {
        {"add", "(II)I", (void *)add},
};

jint JNI_OnLoad(JavaVM *vm, void *reserved){
    JNIEnv *env;
    if (vm->GetEnv((void **)&env, JNI_VERSION_1_6) != JNI_OK){
        __android_log_print(ANDROID_LOG_ERROR, "DLL", "Get Jave ENV Failed");
        return JNI_ERR;
    }
    jclass clazz = env->FindClass(calss_name);
    if (JNI_OK == env->RegisterNatives(clazz, methods, 1)){
        return JNI_VERSION_1_6;
    }
    return JNI_ERR;
}
```



#### 使用 dlfcn.h 函数集操作

libdemo.so 符合 myAdd 函数实现

```c
extern "C" {
int myAdd(int a, int b);
}
int myAdd(int a, int b){
  return a+b;
}
```



使用 dl 函数加载 demo.so 后再调用 myAdd 函数

```c++
jint add(JNIEnv *env, jobject obj, int a, int b){
    const char *lib = "/data/data/com.example.qiuzhaosheng.demo/lib/libdemo.so";

    dll = dlopen(lib, RTLD_GLOBAL);
    if (dll == NULL){
        return -1;
    }
    demoAdd add = (demoAdd) dlsym(dll, "myAdd");
    if (add != NULL)
        return add(a, b);
    else
        return -2;
}
```



### C/C++ 下调用 Java Field，Java Method

```c
    jclass clazz = env->GetObjectClass(obj);
    jfieldID  version_code_id = env->GetFieldID(clazz, "version_code", "I");
    int version_code = env->GetIntField(obj, version_code_id);

    jmethodID versionName_id = env->GetMethodID(clazz, "versionName", "(I)Ljava/lang/String;");
    jstring versionName = (jstring)env->CallObjectMethod(clazz, versionName_id, version_code);
```


## 编译

- 编译脚本

  ```shell
  # Android.mk

  LOCAL_PATH := $(call my-dir)
  include $(CLEAR_VARS)

  LOCAL_MODULE := jni_utils
  LOCAL_SRC_FILES := jni_utils.cpp jni_init.cpp
  LOCAL_LDLIBS := -llog

  include $(BUILD_SHARED_LIBRARY)

  # Application.mk
  APP_ABI:=armeabi-v7a
  APP_PLATFORM := android-9
  NDK_TOOLCHAIN_VERSION := 4.9
  ```

  ```cmake
  # CMakeLists.txt
  cmake_minimum_required(VERSION 3.6)
  find_library( log)
  add_library( jni_utils SHARED jni_utils.cpp jni_init.cpp )
  ```

  ​

- 交叉编译工具

  ```shell
  $ ls $ANDROID_NDK_ROOT/toolchains/arm-linux-androideabi-4.9/prebuilt/darwin-x86_64/bin/
  arm-linux-androideabi-addr2line  arm-linux-androideabi-gcov
  arm-linux-androideabi-ar         arm-linux-androideabi-gcov-tool
  arm-linux-androideabi-as         arm-linux-androideabi-gprof
  arm-linux-androideabi-c++        arm-linux-androideabi-ld
  arm-linux-androideabi-c++filt    arm-linux-androideabi-ld.bfd
  arm-linux-androideabi-cpp        arm-linux-androideabi-ld.gold
  arm-linux-androideabi-dwp        arm-linux-androideabi-nm
  arm-linux-androideabi-elfedit    arm-linux-androideabi-objcopy
  arm-linux-androideabi-g++        arm-linux-androideabi-objdump
  arm-linux-androideabi-gcc        arm-linux-androideabi-ranlib
  arm-linux-androideabi-gcc-4.9    arm-linux-androideabi-readelf
  arm-linux-androideabi-gcc-4.9.x  arm-linux-androideabi-size
  arm-linux-androideabi-gcc-ar     arm-linux-androideabi-strings
  arm-linux-androideabi-gcc-nm     arm-linux-androideabi-strip
  arm-linux-androideabi-gcc-ranlib
  ```

  ​

- 例子

  ```shell
  cmake -DGTEST_ROOT=/Users/qiuzhaosheng/workspace/cplusplus_tests/google/out \
  -D__ANDROID__=ON \
  -DCMAKE_TOOLCHAIN_FILE=$ANDROID_NDK/android_toolchain.cmake ../output
  ```

  [小小轮子](https://gitlab.meitu.com/qzs/MyTools/blob/master/src/android_cmake.py)



# Android NDK 开发注意事项

- [STL 模版支持](https://developer.android.com/ndk/guides/cpp-support.html)

  ![STL](/images/topic-ndk-stl.png)

  ​

  由于 STL 需要加载多一个共享库，同时编译的时候也会增大原有的.so 库大小，会使 整体apk 变大。

- JNI 创建对象后要要记得释放引用
   使用 Jni 诸如 env->NewStringUTF() env->NewGlobalRef 等 env->New* 这类型的函数时，记得要使用对应的 env->Release* 或 env->Delete* 使用内存和引用。

- NDK 不一定比java运行快

- 编译命令行程序的时候记得加上 -PEI  -fPIE
