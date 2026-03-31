---
title: AOSP repo 转大 Git 仓库实践
date: 2022-09-07 20:14
tags:
    - repo 转 Git
---



## 背景

Google 为方便管理和提高效率，将 AOSP 切分为许多个单独的 Git 仓库，再由最外层的 repo 统一管理。因为代码仓库过于庞大，这种化整为零、分而治之的办法确实是对的。但对于一般公司开发，存在两个问题。一是对于 repo 的思想和技术没有 Git 那么熟悉、熟练，存在学习成本。二是对于 AOSP 系统的开发、修改往往是跨 Git 仓库的，导致完成一个任务需要不同目录多次提交，容易遗漏、出错。考虑到一般对 AOSP 修改频次和数量不会太大，统一为一个 Git 仓库可能会慢很多，但应该是在可以接受的范围，反而会提高管理和开发的效率。 



## 目标

- 将 AOSP 由 repo 转为一个 Git 仓库。
- 或者将内核与 Android 代码分为两个单独 Git 仓库。



## 步骤

1. 备份

   在操作之前先备份源码，以防万一，同时方便比对查找差异文件，以用于添加忽略规则。

2. 清理代码仓库，得到干净、没有编译产物的代码。

   如果是新下载的 AOSP 则是纯净的，可直接下一步。现实一般是方案商提供的或者历史项目都是编译过的，需要手动清理。借助以下命令和其它经常观察到的常见编译产物，手动删除。

   ```sh
   make installclean
   make dataclean
   make clean 
   rm -rf out/
   ```

3. 清理 repo、git 和 gitignore 文件。

   ```sh
   # 删除 repo 目录
   find . -type f -name ".repo"|xargs rm -rf
   
   # 删除所有 .git 目录
   find . -type d -name ".git"|xargs rm -rf
   
   # 删除所有 .gitignore 文件
   find .-type f -name ".gitignore"|xargs rm -rf
   ```

   

4. 新建仓库

   ```sh
   git init
   
   # 这一步有点慢
   git add .
   git commit -m "init project"
   ```

   

5. 编译

6. 将差异文件添加 gitignore

   ```sh
   git status > status_0.txt
   ```

   根据 status_x.txt 文件 将忽略规则添加到 .gitignore 文件。以下为参考。

   ```
   # Android 目录 .gitignore
   
   /out/
   
   # 忽略编译期生成文件
   /device/softwinner/peony-perf1/sunxi.dtb
   /device/softwinner/peony-perf1/kernel
   /device/softwinner/peony-perf1/modules/modules/*.ko
   /device/softwinner/peony-perf1/modules/modules/Module.symvers
   
   # 内核 目录 .gitignore
   /.vscode/
   /out/
   *.su
   *.o.cmd
   *.o.d
   *.order
   *.tmp
   *.builtin
   *.mod.c
   *.ko
   
   # 编译后自动变动的文件
   device/config/chips/r818/bin/u-boot-sun50iw10p1.bin
   kernel/linux-4.9/.version
   kernel/linux-4.9/include/generated/compile.h
   
   # 忽略 .o 文件但保留以下文件
   *.o
   !./brandy/brandy-2.0/tools/toolchain/gcc-linaro-7.2.1-2017.11-x86_64_arm-linux-gnueabi/lib/gcc/arm-linux-gnueabi/7.2.1/*.o
   !./brandy/brandy-2.0/tools/toolchain/gcc-linaro-7.2.1-2017.11-x86_64_arm-linux-gnueabi/arm-linux-gnueabi/lib/libasan_preinit.o
   !./brandy/brandy-2.0/tools/toolchain/gcc-linaro-7.2.1-2017.11-x86_64_arm-linux-gnueabi/arm-linux-gnueabi/libc/lib/libasan_preinit.o
   !./brandy/brandy-2.0/tools/toolchain/gcc-linaro-7.2.1-2017.11-x86_64_arm-linux-gnueabi/arm-linux-gnueabi/libc/usr/lib/*.o
   
   # 处理 cmd 结尾的文件
   *.cmd
   !./kernel/linux-4.9/include/config/auto.conf.cmd
   !./brandy/brandy-2.0/u-boot-2018/board/k+p/bootscripts/tpcboot.cmd
   !./brandy/brandy-2.0/u-boot-2018/board/samsung/common/bootscripts/bootzimg.cmd
   !./brandy/brandy-2.0/u-boot-2018/board/samsung/common/bootscripts/autoboot.cmd
   
   # 保留 *.install.cmd 文件
   !./brandy/brandy-2.0/tools/toolchain/gcc-linaro-7.2.1-2017.11-x86_64_arm-linux-gnueabi/arm-linux-gnueabi/libc/usr/include/*.install.cmd
   
   brandy/brandy-2.0/u-boot-2018/.config
   brandy/brandy-2.0/u-boot-2018/System.map
   brandy/brandy-2.0/u-boot-2018/arch/arm/include/asm/arch
   brandy/brandy-2.0/u-boot-2018/arch/arm/lib/asm-offsets.s
   brandy/brandy-2.0/u-boot-2018/arch/arm/lib/lib.a
   brandy/brandy-2.0/u-boot-2018/examples/standalone/hello_world
   brandy/brandy-2.0/u-boot-2018/examples/standalone/hello_world.bin
   brandy/brandy-2.0/u-boot-2018/examples/standalone/hello_world.srec
   brandy/brandy-2.0/u-boot-2018/include/autoconf.mk
   brandy/brandy-2.0/u-boot-2018/include/autoconf.mk.dep
   brandy/brandy-2.0/u-boot-2018/include/config.h
   brandy/brandy-2.0/u-boot-2018/include/config/
   brandy/brandy-2.0/u-boot-2018/include/generated/
   brandy/brandy-2.0/u-boot-2018/lib/asm-offsets.s
   brandy/brandy-2.0/u-boot-2018/scripts/basic/fixdep
   brandy/brandy-2.0/u-boot-2018/scripts/kconfig/conf
   brandy/brandy-2.0/u-boot-2018/scripts/kconfig/zconf.hash.c
   brandy/brandy-2.0/u-boot-2018/scripts/kconfig/zconf.lex.c
   brandy/brandy-2.0/u-boot-2018/scripts/kconfig/zconf.tab.c
   brandy/brandy-2.0/u-boot-2018/tools/common/
   brandy/brandy-2.0/u-boot-2018/tools/dumpimage
   brandy/brandy-2.0/u-boot-2018/tools/fdtgrep
   brandy/brandy-2.0/u-boot-2018/tools/img2srec
   brandy/brandy-2.0/u-boot-2018/tools/lib/
   brandy/brandy-2.0/u-boot-2018/tools/mkenvimage
   brandy/brandy-2.0/u-boot-2018/tools/mkimage
   brandy/brandy-2.0/u-boot-2018/tools/mksunxiboot
   brandy/brandy-2.0/u-boot-2018/tools/proftool
   brandy/brandy-2.0/u-boot-2018/tools/sunxi-spl-image-builder
   brandy/brandy-2.0/u-boot-2018/u-boot
   brandy/brandy-2.0/u-boot-2018/u-boot-nodtb.bin
   brandy/brandy-2.0/u-boot-2018/u-boot-sun50iw10p1.bin
   brandy/brandy-2.0/u-boot-2018/u-boot.bin
   brandy/brandy-2.0/u-boot-2018/u-boot.cfg
   brandy/brandy-2.0/u-boot-2018/u-boot.cfg.configs
   brandy/brandy-2.0/u-boot-2018/u-boot.lds
   brandy/brandy-2.0/u-boot-2018/u-boot.map
   brandy/brandy-2.0/u-boot-2018/u-boot.srec
   brandy/brandy-2.0/u-boot-2018/u-boot.sym
   device/config/chips/r818/dtbo/sunxi_board1.dtbo
   device/config/chips/r818/dtbo/sunxi_board2.dtbo
   device/config/chips/r818/dtbo/sunxi_board3.dtbo
   kernel/linux-4.9/.missing-syscalls.d
   kernel/linux-4.9/.tmp_System.map
   kernel/linux-4.9/.tmp_kallsyms1.S
   kernel/linux-4.9/.tmp_kallsyms2.S
   kernel/linux-4.9/.tmp_versions/
   kernel/linux-4.9/.tmp_vmlinux1
   kernel/linux-4.9/.tmp_vmlinux2
   kernel/linux-4.9/System.map
   kernel/linux-4.9/arch/arm64/boot/Image
   kernel/linux-4.9/arch/arm64/boot/Image.gz
   kernel/linux-4.9/arch/arm64/boot/dts/sunxi/board.dtb
   kernel/linux-4.9/arch/arm64/kernel/asm-offsets.s
   kernel/linux-4.9/arch/arm64/kernel/vdso/vdso.lds
   kernel/linux-4.9/arch/arm64/kernel/vdso/vdso.so
   kernel/linux-4.9/arch/arm64/kernel/vdso/vdso.so.dbg
   kernel/linux-4.9/arch/arm64/kernel/vmlinux.lds
   kernel/linux-4.9/arch/arm64/lib/lib.a
   kernel/linux-4.9/kernel/bounds.s
   kernel/linux-4.9/kernel/config_data.gz
   kernel/linux-4.9/kernel/config_data.h
   kernel/linux-4.9/lib/crc32table.h
   kernel/linux-4.9/lib/gen_crc32table
   kernel/linux-4.9/lib/lib.a
   kernel/linux-4.9/modules/gpu/img-rgx/android/rogue_km/binary_sunxi_android_release/target_aarch64/
   kernel/linux-4.9/output/Image.gz
   kernel/linux-4.9/output/arisc
   kernel/linux-4.9/output/bImage
   kernel/linux-4.9/output/boot.img
   kernel/linux-4.9/output/lib/
   kernel/linux-4.9/output/rootfs.cpio.gz
   kernel/linux-4.9/output/sunxi.dtb
   kernel/linux-4.9/output/sys_config_fix.fex
   kernel/linux-4.9/output/vmlinux.tar.bz2
   kernel/linux-4.9/scripts/mod/devicetable-offsets.s
   kernel/linux-4.9/security/selinux/av_permissions.h
   kernel/linux-4.9/security/selinux/flask.h
   kernel/linux-4.9/usr/.initramfs_data.cpio.d
   kernel/linux-4.9/usr/gen_init_cpio
   kernel/linux-4.9/usr/initramfs_data.cpio.gz
   kernel/linux-4.9/vmlinux
   ```

   指导思想是根据 status_x.txt 文件中的文件后缀，猜测是否为中间产物，到备份目录中用查找命令确认看是否有这种文件，没有就可以统一用 “*.xxx”忽略。添加后在使用 “git status” 导出对比，再重复上一步骤。这是我的方法，不是很厉害但简单实用，第一个 status 文件有近两万行，最后的 .gitignore 文件只有一百来行。

7. 提交

   添加完忽略规则后直至 "git status" 干净后，就可以提交了。