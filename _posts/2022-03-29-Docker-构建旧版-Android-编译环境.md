---
title: Docker 构建旧版 Android 编译环境
date: 2022-03-22 08:08
tags: 
    - Docker
    - Android
---

## 系统配置

- OS：Ubuntu 20.04
- CPU：Intel i5 8500
- RAM：16G
- SSD：600G



为了编译 Android 4.4 到 Android 7.1 的旧版 AOSP，最近折腾了很久。因为一直使用的 Ubuntu 20.04，又不想装虚拟机或者换系统，开始直接在上面手动安装旧版需要的环境。最后编译始终通不过，看来各个系统版本中的工具链版本还是有差异的。

其实在 Android 8.0 以后的 AOSP 应该就比较好配置编译环境了，因为在这之后，AOSP 自带有很多预构建的工具版本，包括 JDK、Make等。相反，在这之前可能需要特定的系统版本 Ubuntu 12.04、14.04、16.04，特定的 JDK 版本 OpenJDK 6、7、8等。

最后还是按照官方建议使用 Docker，不过[官方 Docker](https://android.googlesource.com/platform/build/+/master/tools/docker) 兼容 Android 5.0 到 Android 8.0，看 docker 文件里面没有 OpenJDK 6，说明确实是不支持 Android 4.4的。所以我在其它地方找了一个包含 JDK 6 的版本。



## 安装 Docker

第一次接触 Docker，先在网上找了各种安装教程，五花八门的各不相同，最后还是还是官网的简单明了。具体参见链接，[在 Ubuntu 上安装 Docker](https://docs.docker.com/engine/install/ubuntu/)。注意按照官网一步步安装，并且测试没问题才算成功。



## 下载、构建 Docker 镜像

```shell
git clone https://github.com/qiushao/aosp_builder.git
cd aosp_builder
docker build -t aosp_builder:V1.0
```

构建出来的镜像的详细信息请查看 Dockerfile 和 user.txt 两个文件。

![构建成功的镜像](/assets/img/docker_aosp/docker_images.png)





## 创建容器

```shell
docker run -it --name aosp_builder -v ~/workspace/source/docker:/home/builder/source -u builder aosp_builder:V1.0 /bin/bash
```

说明：

- -V ~/workspace/source/docker:/home/builder/source：是将宿主机目录 ~/workspace/source/docker 映射到 Docker 容器的 /home/builder/source 目录。
- -u builder：使用用户 builder 登录，参见下载的 user.txt 文件。指定用户，避免默认 root 用户登录，便于在宿主机上编辑和删除编译生成的文件。



## 启动容器

```shell
docker start aosp_builder
docker exec -it aosp_builder /bin/bash
```

这样就可以按找 AOSP  标准的编译流程编译旧版 Android 了，对于不同的旧版 Android 依赖的 JDK 版本不同，默认是 JDK 6 的环境，使用时按需修改 JAVA_HOME 环境变量即可。

```shell
# 查看当前使用的 jdk 路径
which javac
# 查看所有已安装 jdk 路径
ls /usr/lib/jvm/
# 修改 JAVA_HOME 环境变量
vim /home/builder/.bashrc
# 使生效
source /home/builder/.bashrc
```

在 Android 5.1 编译刷机成功。

![build_succ](/assets/img/docker_aosp/build_succ.png)



![flash_succ](/assets/img/docker_aosp/flash_succ.png)



参考链接：

- [搭建构建环境](https://source.android.google.cn/setup/build/initializing?hl=zh-cn)
- [要求](https://source.android.google.cn/setup/build/requirements?hl=zh-cn#os)

- [支持旧版本](https://source.android.google.cn/setup/build/older-versions?hl=zh-cn#jdk)
- [ubuntu 18.04 docker构建Android编译环境](http://qiushao.net/2019/11/14/Linux/docker-aosp-build-env/)