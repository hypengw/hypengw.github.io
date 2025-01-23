+++
title = 'HyperOS.eu KernelSU 卸载模块不工作'
date = 2025-01-18T12:55:59+08:00
draft = false
tags = ['科技','手机']

+++

给小米 13 解锁刷了 EU 版本并给了 KernelSU LKM，但是 momo 一直检测到 magisk 模块。

Debug 了一下下。

### KernelSU 卸载模块实现

kernel 内部使用 [path_umount](https://elixir.bootlin.com/linux/v5.15.31/C/ident/path_umount) 单独给某个 app 卸载模块 overlay 挂载点

```c
# https://github.com/tiann/KernelSU/blob/v1.0.2/kernel/core_hook.c#L451
static void ksu_umount_mnt(struct path *path, int flags)
{
	int err = path_umount(path, flags);
	if (err) {
		pr_info("umount %s failed: %d\n", path->dentry->d_iname, err);
	}
}
static void try_umount(const char *mnt, bool check_mnt, int flags)
{
	struct path path;
	int err = kern_path(mnt, 0, &path);
    ...
	ksu_umount_mnt(&path, flags);
}
int ksu_handle_setuid(struct cred *new, const struct cred *old)
{
	...
	try_umount("/system", true, 0);
	try_umount("/vendor", true, 0);
	try_umount("/product", true, 0);
	try_umount("/data/adb/modules", false, MNT_DETACH);
	...
}
```

### HyperOS 会使用 overlay 挂载 mi_ext

`mi_ext` 分区是小米用来添加一些额外 app 和 配置文件的  
`eu` 版本这个分区除了 `etc`，其他都是空文件夹

```bash
# cn 固件的 mi_ext 分区
$ tree                                                                  
.
├── etc
│   ├── build.prop
│   ├── init
│   │   └── init.miui.mi_ext.rc
│   └── NOTICE.xml.gz
├── product
│   ├── app
│   ├── bin
│   ├── data-app
│   ├── etc
│   │   ├── permissions
│   │   │   ├── hyperos.sdk2.xml
│   │   │   ├── privapp-permissions-product-miext.xml
│   │   │   └── system_launcher_private_permission.xml
│   │   ├── precust_theme
│   │   ├── preferred-apps
│   │   ├── security
│   │   └── sysconfig
│   ├── framework
│   │   └── hyperos.sdk2.jar
│   ├── lib
│   ├── lib64
│   ├── media
│   ├── opcust
│   ├── overlay
│   │   ├── GmsMiCSPTelecommOverlay.apk
│   │   ├── GmsMiCSPTelephonyOverlay.apk
│   │   └── MiuiStkResOverlay.apk
│   ├── priv-app
│   └── usr
├── system
│   ├── app
│   ├── etc
│   │   ├── permissions
│   │   │   └── privapp-permissions-system-miext.xml
│   │   └── sysconfig
│   ├── framework
│   └── priv-app
└── system_ext
    └── etc
        └── permissions
```

`fstab`:  

```bash
fuxi:/ $ cat /vendor/etc/fstab.qcom
mi_ext                                                  /mnt/vendor/mi_ext     erofs   ro                                                   wait,slotselect,logical,first_stage_mount,nofail
mi_ext                                                  /mnt/vendor/mi_ext     ext4    ro,barrier=1,discard                                 wait,slotselect,logical,first_stage_mount,nofail
/mnt/vendor/mi_ext                                      /mi_ext                erofs   ro,bind                                              wait,nofail
overlay                                                 /product/overlay          overlay ro,lowerdir=/mnt/vendor/mi_ext/product/overlay/:/product/overlay check,nofail
overlay                                                 /product/app              overlay ro,lowerdir=/mnt/vendor/mi_ext/product/app/:/product/app check,nofail
....
```

### 挂载混在一起

大致挂载顺序：

```text
fstab 中的 overlay
KSU 的 overlay
init 额外的 overlay
```

- `/mi_ext/etc/init/init.miui.mi_ext.rc`  
  启动时额外的 overlay 挂载

  ```text
  on boot
      mount overlay overlay product/usr lowerdir=/mnt/vendor/mi_ext/product/usr:/product/usr
      mount overlay overlay product/etc/precust_theme lowerdir=/mnt/vendor/mi_ext/product/etc/precust_theme:product/etc/precust_theme
  ```

  

- 某个 [commit](https://github.com/tiann/KernelSU/commit/b76d973f3af4b01a33c7f852599410fd530003a8) 会恢复一些原厂的挂载  
  不确定是不是有影响

### 解决

原本想的是直接修改 fstab 了事，但是发现 android 10 新加了 super 动态分区，把 `system/vender/...` 分段在一个分区里，类似套娃。  
而且这些分区基本都是 `erofs(readonly fs)`，想单独改某个分区还挺麻烦的。

看了下 [KernelSU Model guide](https://kernelsu.org/guide/module.html)，发现刚好有在 fs 完成后的脚本执行点（post-fs-data.sh）。  
那可以全局 umount，来回退 fstab 的挂载(`eu` 版本，这些挂载没有用处)  
以下是脚本：[模块下载](assets/xiaomi.eu.no.mi_ext-1.1.zip)

```bash
#!/system/bin/sh

MODDIR=${0%/*}
BINDDIR=/tmp/mi_ext

mkdir -p $BINDDIR

# log
exec 2>$BINDDIR/debug.log
set -x

PATHES="
/product/overlay
/product/app
/product/priv-app
/product/lib
/product/framework
/product/media
/product/opcust
/product/data-app
/product/etc/sysconfig
/product/etc/permissions
/system/app
/system/priv-app
/system/framework
/system/etc/sysconfig
/system/etc/permissions
/product/usr
/product/etc/precust_theme
/product/etc/preferred-apps
/product/etc/security
"

umount -lvf -t overlay /product/bin
umount -lvf -t overlay /product/lib64

for p in $PATHES; do
    umount -v -t overlay $p
done

mount --bind -o ro $BINDDIR /mnt/vendor/mi_ext/system
mount --bind -o ro $BINDDIR /mi_ext/system
mount --bind -o ro $BINDDIR /mnt/vendor/mi_ext/product
mount --bind -o ro $BINDDIR /mi_ext/product
mount --bind -o ro $BINDDIR /mnt/vendor/mi_ext/vendor
mount --bind -o ro $BINDDIR /mi_ext/vendor
```

### Changelog

- 1.1
  remove `/vendor/etc/camera` and`/vendor/lib/rfsa/adsp` which are used by camera and not part of mi_ext.   
  add `update.json`
