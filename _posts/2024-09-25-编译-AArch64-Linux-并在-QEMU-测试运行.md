---
title: 编译 AArch64 Linux 并在 QEMU 测试运行
date: 2024-09-25 09:34
tags: 
    - Linux 内核
    - BusyBox
    - QEMU
    - AArch64
---



## 目的

编译一个 64 位的 arm Linux，在命令行通过 QEMU 运行。



## 环境

- 主机
  - Ubuntu 24.04.1 LTS  x86_64 6.8.0-45
- 工具
  - 交叉编译器：arm-gnu-toolchain-13.3.rel1-x86_64-aarch64-none-linux-gnu
  - BusyBox 1.36.1
  - QEMU 9.1.0
  - Linux 5.15.167



## 编译 BusyBox 制作根文件系统



### 配置

```shell
cd busybox-1.36.1
mkdir build

make O=build ARCH=arm64 defconfig
make O=build ARCH=arm64 menuconfig
```

在 menuconfig 菜单中修改以下配置

- 选中 Settings --> Don't use /usr
- 选中 Settings --> Build static binary (no shared libs)
- 设置 Settings --> Cross compiler prefix 为 aarch64-none-linux-gnu-

设置完成保存退出。



### 编译 BusyBox

```bash
make O=build -j6
make O=build install
cd build/_install
```

以上命令分别以之前的配置进行编译，再安装到 build/_install 目录，执行成功该目录如下：

![busybox_install](/assets/img/linux/busybox_install.png)



### 制作根文件

创建目录

```sh
mkdir -pv {etc,proc,sys,usr/{bin,sbin}}
```

创建 init 文件，并添加如下内容。

```sh
#!/bin/sh

mount -t proc none /proc
mount -t sysfs none /sys

echo -e "\nboot took $(cut -d' ' -f1 /proc/uptime) seconds\n"

exec /bin/sh
```

为 init 文件添加可执行权限

```sh
chmod +x init
```

此时 build/_install 目录内容如下

![build/_install content](/assets/img/linux/rootfs.png)

打包目录和文件

```sh
find . -print0|cpio --null -ov --format=newc|gzip > ../initramfs.cpio.gz
```

此时生成的 BusyBox ramdisk 放在 build/initramfs.cpio.gz。



## 编译 Linux 内核

首先到[Linux Kernel 官网](https://www.kernel.org/)下载源码，再解压。

### 配置

```sh
cd linux-5.15.167
mkdir build

make O=build ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu- allnoconfig
make O=build ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu- menuconfig
```

初始化最小配置 allnoconfig 后，在 menuconfig 中修改以下配置：

- General setup
  - Initial RAM filesystem and RAM disk (initramfs/initrd) support  选中
- Exectuable file formats
  - Kernel support for ELF binaries  选中
  - Kernel support for scripts starting with #!  选中
- Device Drivers
  - Generic Driver Options
    - Maintain a devtmpfs filesystem to mount at /dev  选中
    - ​       Automount devtmpfs at /dev, after the kernel mounted the rootfs  选中
  - Character devices
    - Enable TTY 选中
    - Serial drivers
      - ARM AMBA PL010 serial port support 选中
      - ​      Support for console on AMBA serial port 选中
      - ARM AMBA PL011 serial port support 选中
      - ​     Support for console on AMBA serial port  选中
- File Systems
  - Pseudo filesystems
    - /proc file system support  选中
    - sysfs file system support 选中，如果没有此选项，参见[这里](https://www.kernel.org/doc/Documentation/kdump/kdump.txt)，并搜索“sysfs file system support” 关键字。



### 编译内核

```sh
make O=build ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu- -j6
```

编译完成后，得到 build/vmlinux 可用于在 GDB 中加载调试信息的 ELF 格式内核，以及可启动的内核镜像文件 build/arch/arm64/boot/Image。



## 编译 QEMU

```sh
cd qemu-9.1.0
mkdir build
cd build

../configure --tartget-list=aarch64-softmmu
make -j6
```

编译生成 build/qemu-system-aarch64



## 运行测试

```sh
./qemu-system-aarch64 \
-machine virt -cpu cortex-a53 -smp 1 -m 2G \
-kernel ../../../ws/linux/linux-5.15.167/build/arch/arm64/boot/Image \
-append "console=ttyAMA0" \
-initrd ../../busybox-1.36.1/build/initramfs.cpio.gz \
-nographic
```

![linux_run](/assets/img/linux/linux_run.png)

