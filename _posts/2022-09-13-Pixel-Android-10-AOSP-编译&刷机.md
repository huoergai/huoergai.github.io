---
title: Pixel Android 10 AOSP 编译&刷机
date: 2022-09-13 20:17
tags:
    - AOSP 

---

## 背景

自备的 Pixel 测试机的原系统带 Google 搜索，一直想去掉。同时新项目需要修改系统源码，顺便就将编译、烧录等过程做个完整记录。



## 目标

- 删除 Google 搜索
- 修改源码后，编译出镜像并烧录进 Pixel。



## 准备

**编译环境：**

- OS: Ubuntu 20.04
- AOSP: android-10.0.0_r17
- diver
  - Google Vendor image: https://dl.google.com/dl/android/aosp/google_devices-sailfish-qp1a.191005.007.a3-a1615a0f.tgz
  - Qualcomm: https://dl.google.com/dl/android/aosp/qcom-sailfish-qp1a.191005.007.a3-191228fe.tgz



## 下载

### AOSP 下载

```sh
# 下载、安装 repo
mkdir ~/bin
curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o ~/bin/repo
chmod +x ~/bin/repo
echo 'export PATH=$PATH:~/bin' >> ~/.bashrc 
echo 'export REPO_URL="https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/"' >> ~/.bashrc
source ~/.bashrc

# 新建目录
mkdir android_10
cd android_10

# 下载
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-10.0.0_r17

# 同步失败和不完整，可多次重复执行该命令
repo sync
```



### Driver 下载

将 Google 和 Qualcomm 的两个驱动链接下载到本地。

```sh
# 下载
wget https://dl.google.com/dl/android/aosp/google_devices-sailfish-qp1a.191005.007.a1-e42a3654.tgz
wget https://dl.google.com/dl/android/aosp/qcom-sailfish-qp1a.191005.007.a3-191228fe.tgz

# 解压，将其解压到 Android 源码根目录
tar -xzvf google_devices-sailfish-qp1a.191005.007.a1-e42a3654.tgz -C android_10/
tar -xzvf qcom-sailfish-qp1a.191005.007.a3-191228fe.tgz -C android_10/

# 执行解压出来的 shell 文件，提取、合并驱动。
cd android_10
./extract-google_devices-sailfish.sh
./extract-qcom-sailfish.sh
```



## 编译

```sh
source build/envsetup.sh
lunch aosp_sailfish-userdebug
# n 根据 CPU 核心数确定
make -jn
```

![lunch_result](/assets/img/sysdev/lunch.png)



![build_result](/assets/img/sysdev/build.png)



## 刷机

**如果手机是第一次刷机**，先不要刷入自己编译的镜像，否则会卡在开机动画界面，无法进入。先刷一次官方原厂镜像，成功后再刷自编的镜像。

### 烧录原厂镜像

- 从参考链接出下载对应版本的镜像，解压后得到如下目录。

  ![factory image](/assets/img/sysdev/factory_img.png)

- 参考连接中的”解锁和进入 fastboot 模式“。

- 进入 fastboot 模式后，运行解压镜像中的 flash-all 脚本即可自动刷机。



### 烧录自编镜像

- 将编译产物中的：boot.img ramdisk.img ramdisk-recovery.img system.img  vendor.img userdata.img 使用 fastboot 命令刷入后重启即可。

  ![flash](/assets/img/sysdev/flash.png)

  ```sh
  # flash.bat/flash.sh
  adb reboot bootloader
  fastboot flash boot .\boot.img
  fastboot flash ramdisk .\ramdisk.img
  fastboot flash ramdisk-recovery .\ramdisk-recovery.img
  fastboot flash system .\system.img
  fastboot flash vendor .\vendor.img
  fastboot flash userdata .\userdata.img
  fastboot reboot
  ```

  <center>
      <img src="/assets/img/sysdev/sailfish_home.png" alt="sailfish_home" style="zoom:50%;"/> 
      <img src="/assets/img/sysdev/sailfish_about.png" alt="sailfish_about" style="zoom:50%;"/>
  </center>





参考链接

[Nexus 和 Pixel 原厂镜像](https://developers.google.com/android/images)

[解锁和进入 fastboot 模式](https://source.android.com/source/running?hl=zh-CN)

