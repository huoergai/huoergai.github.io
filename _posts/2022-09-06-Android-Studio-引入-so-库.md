---
title: Android Studio 引入 so 库
date: 2022-09-06 13:54
tags:
    - Android Studio
---

之前在 Android Studio 中导入 so 库是将 so 库放置到项目的 jniLibs 目录下。或者放置在其它位置，并在 gradle 中指定，如下。

```groovy
sourceSets {
    main {
        jniLibs.srcDirs = ['libs']
    }
}
```

目前网上的“教程”不管哪个时间点的也都是这种写法，根本不适用。还是根据报错提示，查看官方文档才搞明白。

在 AGP 4.0.0 开始就不能这么写了，否则会报错。因为本地代码构建的时候会自动将这些打包，如果再在 gradle 中指定，或者将 so 放入默认会打包的 jniLibs 就会导致重复冲突。

解决办法就是，不在 gradle 中指定并放在 jinLibs 之外的地方，推荐 main/cpp/lib 目录。

![so 位置](load so.png)

在 CMakeLists.txt 中添加、链接 so 库。

```cmake
set(distribution_DIR ${CMAKE_SOURCE_DIR}/lib/${ANDROID_ABI})

add_library(avcodec SHARED IMPORTED)
set_target_properties(avcodec PROPERTIES IMPORTED_LOCATION ${distribution_DIR}/libavcodec.so)

target_link_libraries( # Specifies the target library.
        hello_native
        avcodec

        # Links the target library to the log library
        # included in the NDK.
        ${log-lib})
```



**参考链接：**

[自动打包 cmake 的预编译依赖]([Android Gradle plugin release notes  | Android Developers](https://developer.android.com/studio/releases/gradle-plugin#cmake-imported-targets))

