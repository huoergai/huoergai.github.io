---
title: FFmpeg 编译并集成到 Android
date: 2022-09-06 14:02
tags:
    - FFmpeg
---

## 目标

- 在 Ubuntu 上编译出可供 Android 使用的 FFmpeg 动态链接库(so)。

## 编译

**准备**

- NDK 

  选择 ndk-r24，支持 API 19+，将其下载到本地并解压。 

- FFmpeg 源码

  当前最新 release 版本是5.1，clone 的 master 分支。将 ndk 与 FFmpeg 放在同根目录下将。

  在 FFmpeg 目录下新建 build_android.sh 文件，并填入以下内容。主要注意其中 tag1、tag2 修改点。

  ```sh
  #!/bin/bash
  # 用于编译android平台的脚本
  
  # NDK所在目录
  NDK_PATH=../ndk_r24/ # tag1
  # 编译平台，具体查看 $NDK_PATH/toolchains/llvm/prebuilt/ 下的文件夹名称
  HOST_PLATFORM=linux-x86_64  #tag2
  # minSdkVersion
  API=23
  
  TOOLCHAINS="$NDK_PATH/toolchains/llvm/prebuilt/$HOST_PLATFORM"
  SYSROOT="$NDK_PATH/toolchains/llvm/prebuilt/$HOST_PLATFORM/sysroot"
  # 生成 -fpic 与位置无关的代码
  CFLAG="-D__ANDROID_API__=$API -Os -fPIC -DANDROID "
  LDFLAG="-lc -lm -ldl -llog "
  
  # 输出目录
  PREFIX=`pwd`/android-build
  # 日志输出目录
  CONFIG_LOG_PATH=${PREFIX}/log
  # 公共配置
  COMMON_OPTIONS=
  # 交叉配置
  CONFIGURATION=
  
  build() {
    APP_ABI=$1
    echo "======== > Start build $APP_ABI"
    case ${APP_ABI} in
    armeabi-v7a)
      ARCH="arm"
      CPU="armv7-a"
      MARCH="armv7-a"
      TARGET=armv7a-linux-androideabi
      CC="$TOOLCHAINS/bin/$TARGET$API-clang"
      CXX="$TOOLCHAINS/bin/$TARGET$API-clang++"
      LD="$TOOLCHAINS/bin/$TARGET$API-clang"
      # 交叉编译工具前缀
      CROSS_PREFIX="$TOOLCHAINS/bin/arm-linux-androideabi-"
      EXTRA_CFLAGS="$CFLAG -mfloat-abi=softfp -mfpu=vfp -marm -march=$MARCH "
      EXTRA_LDFLAGS="$LDFLAG"
      EXTRA_OPTIONS="--enable-neon --cpu=$CPU "
      ;;
    arm64-v8a)
      ARCH="aarch64"
      TARGET=$ARCH-linux-android
      CC="$TOOLCHAINS/bin/$TARGET$API-clang"
      CXX="$TOOLCHAINS/bin/$TARGET$API-clang++"
      LD="$TOOLCHAINS/bin/$TARGET$API-clang"
      CROSS_PREFIX="$TOOLCHAINS/bin/$TARGET-"
      EXTRA_CFLAGS="$CFLAG"
      EXTRA_LDFLAGS="$LDFLAG"
      EXTRA_OPTIONS=""
      ;;
    x86)
      ARCH="x86"
      CPU="i686"
      MARCH="i686"
      TARGET=i686-linux-android
      CC="$TOOLCHAINS/bin/$TARGET$API-clang"
      CXX="$TOOLCHAINS/bin/$TARGET$API-clang++"
      LD="$TOOLCHAINS/bin/$TARGET$API-clang"
      CROSS_PREFIX="$TOOLCHAINS/bin/$TARGET-"
      #EXTRA_CFLAGS="$CFLAG -march=$MARCH -mtune=intel -mssse3 -mfpmath=sse -m32"
      EXTRA_CFLAGS="$CFLAG -march=$MARCH  -mssse3 -mfpmath=sse -m32"
      EXTRA_LDFLAGS="$LDFLAG"
      EXTRA_OPTIONS="--cpu=$CPU "
      ;;
    x86_64)
      ARCH="x86_64"
      CPU="x86-64"
      MARCH="x86_64"
      TARGET=$ARCH-linux-android
      CC="$TOOLCHAINS/bin/$TARGET$API-clang"
      CXX="$TOOLCHAINS/bin/$TARGET$API-clang++"
      LD="$TOOLCHAINS/bin/$TARGET$API-clang"
      CROSS_PREFIX="$TOOLCHAINS/bin/$TARGET-"
      #EXTRA_CFLAGS="$CFLAG -march=$CPU -mtune=intel -msse4.2 -mpopcnt -m64"
      EXTRA_CFLAGS="$CFLAG -march=$CPU -msse4.2 -mpopcnt -m64"
      EXTRA_LDFLAGS="$LDFLAG"
      EXTRA_OPTIONS="--cpu=$CPU "
      ;;
    esac
  
    echo "-------- > Start clean workspace"
    make clean
  
    echo "-------- > Start build configuration"
    CONFIGURATION="$COMMON_OPTIONS"
    CONFIGURATION="$CONFIGURATION --logfile=$CONFIG_LOG_PATH/config_$APP_ABI.log"
    CONFIGURATION="$CONFIGURATION --prefix=$PREFIX"
    CONFIGURATION="$CONFIGURATION --libdir=$PREFIX/libs/$APP_ABI"
    CONFIGURATION="$CONFIGURATION --incdir=$PREFIX/includes/$APP_ABI"
    CONFIGURATION="$CONFIGURATION --pkgconfigdir=$PREFIX/pkgconfig/$APP_ABI"
    CONFIGURATION="$CONFIGURATION --cross-prefix=$CROSS_PREFIX"
    CONFIGURATION="$CONFIGURATION --arch=$ARCH"
    CONFIGURATION="$CONFIGURATION --sysroot=$SYSROOT"
    CONFIGURATION="$CONFIGURATION --cc=$CC"
    CONFIGURATION="$CONFIGURATION --cxx=$CXX"
    CONFIGURATION="$CONFIGURATION --ld=$LD"
    # nm 和 strip
    CONFIGURATION="$CONFIGURATION --nm=$TOOLCHAINS/bin/llvm-nm"
    CONFIGURATION="$CONFIGURATION --strip=$TOOLCHAINS/bin/llvm-strip"
    CONFIGURATION="$CONFIGURATION $EXTRA_OPTIONS"
  
    echo "-------- > Start config makefile with $CONFIGURATION --extra-cflags=${EXTRA_CFLAGS} --extra-ldflags=${EXTRA_LDFLAGS}"
    ./configure ${CONFIGURATION} \
    --extra-cflags="$EXTRA_CFLAGS" \
    --extra-ldflags="$EXTRA_LDFLAGS"
  
    echo "-------- > Start make $APP_ABI with -j1"
    make -j6
  
    echo "-------- > Start install $APP_ABI"
    make install
    echo "++++++++ > make and install $APP_ABI complete."
  
  }
  
  build_all() {
    #配置开源协议声明
    COMMON_OPTIONS="$COMMON_OPTIONS --enable-gpl"
    #目标android平台
    COMMON_OPTIONS="$COMMON_OPTIONS --target-os=android"
    #取消默认的静态库
    COMMON_OPTIONS="$COMMON_OPTIONS --disable-static"
    COMMON_OPTIONS="$COMMON_OPTIONS --enable-shared"
    COMMON_OPTIONS="$COMMON_OPTIONS --enable-protocols"
    #开启交叉编译
    COMMON_OPTIONS="$COMMON_OPTIONS --enable-cross-compile"
    COMMON_OPTIONS="$COMMON_OPTIONS --enable-optimizations"
    COMMON_OPTIONS="$COMMON_OPTIONS --disable-debug"
    #尽可能小
    COMMON_OPTIONS="$COMMON_OPTIONS --enable-small"
    COMMON_OPTIONS="$COMMON_OPTIONS --disable-doc"
    #不要命令（执行文件）
    COMMON_OPTIONS="$COMMON_OPTIONS --disable-programs"  # do not build command line programs
    COMMON_OPTIONS="$COMMON_OPTIONS --disable-ffmpeg"    # disable ffmpeg build
    COMMON_OPTIONS="$COMMON_OPTIONS --disable-ffplay"    # disable ffplay build
    COMMON_OPTIONS="$COMMON_OPTIONS --disable-ffprobe"   # disable ffprobe build
    COMMON_OPTIONS="$COMMON_OPTIONS --disable-symver"
    COMMON_OPTIONS="$COMMON_OPTIONS --disable-network"
    COMMON_OPTIONS="$COMMON_OPTIONS --disable-x86asm"
    COMMON_OPTIONS="$COMMON_OPTIONS --disable-asm"
    #启用
    COMMON_OPTIONS="$COMMON_OPTIONS --enable-pthreads"
    COMMON_OPTIONS="$COMMON_OPTIONS --enable-mediacodec"
    COMMON_OPTIONS="$COMMON_OPTIONS --enable-jni"
    COMMON_OPTIONS="$COMMON_OPTIONS --enable-zlib"
    COMMON_OPTIONS="$COMMON_OPTIONS --enable-pic"
    COMMON_OPTIONS="$COMMON_OPTIONS --enable-muxer=flv"
    #COMMON_OPTIONS="$COMMON_OPTIONS --enable-avresample"
    COMMON_OPTIONS="$COMMON_OPTIONS --enable-decoder=h264"
    COMMON_OPTIONS="$COMMON_OPTIONS --enable-decoder=mpeg4"
    COMMON_OPTIONS="$COMMON_OPTIONS --enable-decoder=mjpeg"
    COMMON_OPTIONS="$COMMON_OPTIONS --enable-decoder=png"
    COMMON_OPTIONS="$COMMON_OPTIONS --enable-decoder=vorbis"
    COMMON_OPTIONS="$COMMON_OPTIONS --enable-decoder=opus"
    COMMON_OPTIONS="$COMMON_OPTIONS --enable-decoder=flac"
    echo "COMMON_OPTIONS=$COMMON_OPTIONS"
    echo "PREFIX=$PREFIX"
    echo "CONFIG_LOG_PATH=$CONFIG_LOG_PATH"
    mkdir -p ${CONFIG_LOG_PATH}
    build "armeabi-v7a"
    build "arm64-v8a"
    build "x86"
    build "x86_64"
  }
  
  echo "-------- Start --------"
  build_all
  echo "-------- End --------"
  ```

  编译成功后会在 FFmpeg 目录下生成 android-build 目录，其中的 libs 目录为 so 库输出，includes 为头文件输出。导入项目时，将这两个目录引入即可。

  

  ## 集成

  

  ![ffpeg](/assets/img/sysdev/ffmpeg_0.png)

​		CMakeLists.txt 文件内容

​		 

```cmake
# For more information about using CMake with Android Studio, read the
# documentation: https://d.android.com/studio/projects/add-native-code.html

# Sets the minimum version of CMake required to build the native library.

cmake_minimum_required(VERSION 3.18.1)

# Declares and names the project.

project("hello_ffmpeg")

# 引入头文件
include_directories(include)

# 添加 ffmpeg 的 avcodec、swresample、avutil 模块 start======
set(distribution_DIR ${CMAKE_SOURCE_DIR}/lib/${ANDROID_ABI})

add_library(avcodec SHARED IMPORTED)
set_target_properties(avcodec PROPERTIES IMPORTED_LOCATION ${distribution_DIR}/libavcodec.so)

add_library(swresample SHARED IMPORTED)
set_target_properties(swresample PROPERTIES IMPORTED_LOCATION ${distribution_DIR}/libswresample.so)

add_library(avutil SHARED IMPORTED)
set_target_properties(avutil PROPERTIES IMPORTED_LOCATION ${distribution_DIR}/libavutil.so)
# 添加 ffmpeg 的 avcodec、swresample、avutil 模块 end======

# Creates and names a library, sets it as either STATIC
# or SHARED, and provides the relative paths to its source code.
# You can define multiple libraries, and CMake builds them for you.
# Gradle automatically packages shared libraries with your APK.

add_library( # Sets the name of the library.
        hello_ffmpeg

        # Sets the library as a shared library.
        SHARED

        # Provides a relative path to your source file(s).
        hello_ffmpeg.cpp)

# Searches for a specified prebuilt library and stores the path as a
# variable. Because CMake includes system libraries in the search path by
# default, you only need to specify the name of the public NDK library
# you want to add. CMake verifies that the library exists before
# completing its build.

find_library( # Sets the name of the path variable.
        log-lib

        # Specifies the name of the NDK library that
        # you want CMake to locate.
        log)

# Specifies libraries CMake should link to your target library. You
# can link multiple libraries, such as libraries you define in this
# build script, prebuilt third-party libraries, or system libraries.

target_link_libraries( # Specifies the target library.
        hello_ffmpeg
        avcodec
        swresample
        avutil

        # Links the target library to the log library
        # included in the NDK.
        ${log-lib})
```



hello_ffmpeg.cpp

```cpp
// D&T: 2022-09-06 09:34
// DES: 
//
#include <jni.h>
#include <string>

extern "C" {
#include "libavcodec/version.h"
#include "libavformat/version.h"
#include "libavutil/version.h"
#include "libavfilter/version.h"
#include "libswresample/version.h"
#include "libswscale/version.h"
#include "libavcodec/avcodec.h"
}

extern "C" JNIEXPORT jstring JNICALL
Java_com_huoergai_demo_FfmpegUtil_getVersion(JNIEnv *env, jobject thiz) {
    std::string version = "libavcode: ";
    version.append(AV_STRINGIFY(LIBAVCODEC_VERSION));

    version.append("\navformat: ");
    version.append(AV_STRINGIFY(LIBAVFORMAT_VERSION));

    version.append("\navutil: ");
    version.append(AV_STRINGIFY(LIBAVUTIL_VERSION));

    version.append("\navfilter: ");
    version.append(AV_STRINGIFY(LIBAVFILTER_VERSION));

    version.append("\nswresample: ");
    version.append(AV_STRINGIFY(LIBSWRESAMPLE_VERSION));

    version.append("\nswscale: ");
    version.append(AV_STRINGIFY(LIBSWSCALE_VERSION));

    version.append("\navcodec_license: ");
    version.append(avcodec_license());

    version.append("\navcodec_config: ");
    version.append(avcodec_configuration());

    return env->NewStringUTF(version.c_str());
}
```

FfmpegUtil.kt

```kotlin
package com.huoergai.demo

/**
 * D&T: 2022-09-06 09:36
 * DES:
 */
object FfmpegUtil {

    init {
        System.loadLibrary("hello_ffmpeg")
    }

    external fun getVersion(): String

}
```

