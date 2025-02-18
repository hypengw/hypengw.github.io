+++
title = 'Kindle 也有自己的 Magisk 模块'
date = 2025-02-17T21:25:18+08:00
draft = false
tags = ['硬件','Linux']

+++

## KUAL
TODO

## Kindle(kpw5) 系统环境
![Kindle Neofetch](https://pan.bluempty.com/d/Public/Blog/kindle-module/neofetch.webp)

```txt
OS: Linux 4.9.77-lab126 armv7l 
CPU: MT8110 Bellatrix device 
GLIBC: 2.20
LD: /lib/ld-linux-armhf.so.3
```

```text
[root@kindle root]# cat /etc/ld.so.conf 
/usr/java/lib
/app/lib
/usr/lib/lua
```

```text
[root@kindle root]# mount | grep tmpfs
tmpfs on /dev type tmpfs (rw,relatime,mode=755)
tmpfs on /dev/shm type tmpfs (rw,relatime)
tmpfs on /var type tmpfs (rw,relatime,size=65536k)
tmpfs on /mnt/kfxcache type tmpfs (rw,relatime,size=256000k)
cgroup_root on /sys/fs/cgroup type tmpfs (rw,relatime)
tmpfs on /chroot/dev type tmpfs (rw,relatime,mode=755)
tmpfs on /chroot/var/cache type tmpfs (rw,relatime,size=65536k)
tmpfs on /chroot/var/lock type tmpfs (rw,relatime,size=65536k)
tmpfs on /chroot/var/run type tmpfs (rw,relatime,size=65536k)
tmpfs on /chroot/tmp/session_token type tmpfs (rw,relatime,size=65536k)
tmpfs on /etc/shadow type tmpfs (ro,relatime,size=65536k)
```
```text
[root@kindle 4.9.77-lab126]# lsmod
Module                  Size  Used by
g_mass_storage          2340  0 
wmt_cdev_bt            17908  0 
wlan_drv_gen4m       1792220  0 
wmt_chrdev_wifi        11440  1 wlan_drv_gen4m
wmt_drv               965568  4 wmt_cdev_bt,wlan_drv_gen4m,wmt_chrdev_wifi
usb_f_mass_storage     32216  2 g_mass_storage
libcomposite           32776  2 g_mass_storage,usb_f_mass_storage
configfs               20880  3 usb_f_mass_storage,libcomposite
pt_i2c                  4448  0 
pt                    146584  2 pt_i2c
opt3001                11908  1 
falcon                 31568  0 [permanent]
hwtcon_v2             142808  3
```

```text
[root@kindle root]# xwininfo -tree -root
xwininfo: Window id: 0x51 (the root window) (has no name)
...
     children:
     "Webreader"
     "Pillowd"
     "KPPMainApp"
     "Mesquite"
     "Kfxreader"
     "Kb"
     "JunoStatusBarDriver"
     "L:A_N:application_ID:com.lab126.booklet.home_M:false_PC:TSB_RC:true_WT:true_ASR:true_O:U"
     "L:D_N:overlay_AKB:true_HIDET1:200_ID:system_A:QuickSettingsWindow_LB:ON_M:dismissible_CD:true_S:-1_KIWI:com.lab126.kppQuickSettings_SHOWT1:250": ("KPPMainApp" "KPPMainApp")
     "L:A_N:application_AKB:true_ASR:true_ID:com.lab126.krpp_A:mainWindow_WS:true_WT:true_PC:N_ALS:com.lab126.booklet.reader_O:UDLR_SHOWT1:70_S:-2": ("KPPMainApp" "KPPMainApp")
     "L:D_N:overlay_AKB:true_HIDET1:150_RKB:default_ID:system_A:kppFullScreenSearch_M:dismissible_PAIRID:JunoStatusBarWindow_S:-1_KIWI:com.lab126.KPPMainApp_SHOWT1:50": ("KPPMainApp" "KPPMainApp")
     "L:D_N:overlay_AKB:true_CD:true_FH:S_ID:com.lab126.kppContextMenu_A:ChromeContextMenu_LB:OFF_owner:com.lab126.KPPMainApp_M:dismissible_S:-1_SHOWT1:0_KIWI:com.lab126.KPPMainApp_HIDET1:80": ("KPPMainApp" "KPPMainApp")
     "L:SS_N:screenSaver_FH:F_module:screensaver_ID:blanket-screensaver_FS:F_O:U"
     "L:C_N:titleBar_PAIRID:JunoStatusBarWindow_KIWI:com.lab126.gtkstatusbar_HIDET1:50_ID:system_A:titleBar": ("JunoStatusBarDriver" "JunoStatusBarDriver")
     "L:A_N:application_AKB:true_ASR:true_ID:com.lab126.KPPMainApp_A:KPPMainApp_WS:true_WT:true_PC:T_O:UD_SHOWT1:200_S:-1": ("KPPMainApp" "KPPMainApp")
     "L:A_N:application_ID:com.lab126.booklet.reader_M:false_PC:N_RC:true_WT:true_ASR:true_O:URL_WTNB:true_WTPB:true_DM:N_S:-7"
     "L:A_N:application_module:blankwindow_ID:blankBackground_WS:true"
     "L:C_N:searchBar_A:kppTopChrome_KIWI:com.lab126.KPPMainApp_AKB:true_S:-1_SHOWT1:30_ID:system_HIDET1:50": ("KPPMainApp" "KPPMainApp")
     "L:KB_N:keyboard_DMINSETRIGHT:5_DM:KB_DMINSETLEFT:5_KBS:H_DMINSETTOP:84_LanH:567_PorH:567_DMINSETBOTTOM:5": ("kb" "Kb")
     "L:A_N:application_HIDE:background_S:1500_PC:TS_ID:com.lab126.store_O:U": ("mesquite" "Mesquite")
     "L:C_N:bottomBar_KIWI:com.lab126.KPPMainApp_AKB:true_S:-1_SHOWT1:50_ID:system_A:kppBottomChrome": ("KPPMainApp" "KPPMainApp")
```


### 越狱
[2025 Kindle 越狱教程：不限 Kindle 型号，不限固件版本](https://bookfere.com/post/1145.html)

### armv5 armv6 armv7 armhf 的区别
Kindle 在 5.16.3 固件版本后，切换到了 armhf(hard float)，这直接影响到了 c abi，包括 ld 也变成了 `/lib/ld-linux-armhf.so.3`，无法和原来的 armel 程序兼容。  

- 架构版本维度
  - armv5：经典 ARM9 处理器架构
  - armv6：ARM11 系列
  - armv7：Cortex-A 系列
  - armv8：64 位 ARM 架构
- ABI 应用二进制接口维度
  - armel (ARM EABI)：使用软浮点（软件模拟浮点运算）
  - armhf (ARM Hard Float)：使用硬浮点（直接调用 FPU 单元）

| 架构  | 主要 CPU | 指令集 | SIMD/NEON | 浮点运算 | 典型设备 |
| --- | --- | --- | --- | --- | --- |
| **ARMv5** | ARM9, ARM10 | ARM, Thumb | ❌   | ❌（仅软件模拟） | 旧款路由器、嵌入式系统 |
| **ARMv6** | ARM11 | ARM, Thumb-2 | ❌   | VFP（可选） | 树莓派 1、早期智能手机 |
| **ARMv7** | Cortex-A8/A9/A15 | ARM, Thumb-2 | ✅（NEON） | ✅（VFPv3） | 树莓派 2/3、智能手机 |


## 开发环境
> 最终的 [Dockerfile 仓库](https://github.com/hypengw/dockerfiles/tree/main/kindle)

珍爱生命，远离交叉编译。  
我选择用 `qemu` 来跑 `arm` 的容器，具体来说，是通过 `binfmt` 实现的。  
接下来就是选能用的发行版，对于这种非主流架构的 cpu，还得是 debian。  
然后就是能对上 `glibc` 版本的分支，要退到 Debian 8(jessie), released on 2015。`jessie` 的 `glibc` 版本是 `2.19`，满足 `<=2.20` 的要求。

这里给一个 Dockerfile 的开头，需要安装 `qemu-user-static-arm` 和 `qemu-user-static-binfmt`, 并让 `binfmt` 在后台运行。
```Dockerfile
FROM --platform=linux/arm/v7 debian:jessie

RUN echo "\n\
deb [check-valid-until=no] http://archive.debian.org/debian jessie main\n\
deb [check-valid-until=no] http://archive.debian.org/debian-security jessie/updates main\n\
deb [check-valid-until=no] http://archive.debian.org/debian jessie-backports main\n\
deb [check-valid-until=no] http://archive.debian.org/debian jessie-backports-sloppy main\n\
" > /etc/apt/sources.list
...
```

可以用 `cat /proc/sys/fs/binfmt_misc/qemu-arm` 来看 `binfmt` 是否正常工作，会有类似输出:
```text
enabled
interpreter /usr/bin/qemu-arm-static
flags: F
offset 0
magic 7f454c4601010100000000000000000002002800
mask ffffffffffffff00fffffffffffffffffeffffff
```

### gcc
`jessie` 仓库里最新的 `gcc` 版本是 `4.9`, 相当老。  
对于 c 项目，可能会在一些编译选项上报错。  
对于 c++ 的项目，`4.x` 勉强支持到 `c++ 14` 标准，要编译新点的项目，建议还是自行编译新版的 `gcc`。考虑到 `libstdc++` 的兼容问题，这里不建议用太高的版本，比如 `12/13..`，可能会有坑。 

### go
go 提供了 `arm` 通用的 [binary 包](https://go.dev/dl/go1.24.0.linux-armv6l.tar.gz)，而且不用考虑 `glibc` 的兼容。  
参考官方的[文档](https://go.dev/wiki/GoArm), 如果要指定 `arm7l` 目标，需要设置 `GOARCH=arm` 和 `GOARM=7` 两个环境变量。 

### rust
TODO

## 我维护的插件

- openssh server
- syncthing

[下载](https://pan.bluempty.com/Kindle/extensions)