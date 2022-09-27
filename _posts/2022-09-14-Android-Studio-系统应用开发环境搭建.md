---
title: Android Studio 系统应用开发环境搭建
date: 2022-09-14 20:35
tags:
    - Android Studio 
---



## 背景

新项目需要开发一款集成到 Android 10 系统的应用，准备开发环境时遇到系统权限、隐藏 API、编译版本、插件和依赖库版本导致的一系列问题，遂在此记录以备查询和参考。



## 目标

- 在 Android Studio 上为开发系统应用，实现近似于开发普通应用的体验。



## 步骤

- 系统应用是有权限使用系统隐藏 API 的，但在 Android Studio 中要引用隐藏 API，需要替换编译版本的 android.jar。到该地址 https://github.com/Reginer/aosp-android-jar  下载对应版本的 android jar 包并替换 sdk platform 目录下的即可。后期亦可使用该地址 https://github.com/JetpackDuba/android-jar-with-hidden-api 下的脚本手动生成。替换后将 comileSdk 和 targetSdk 设置为相应版本。

- 因为设置了 compileSdk 和 targetSdk 版本为 29，目前最新为 33，导致编译不过，与各个依赖库冲突。主要是以下库：

  ```groovy
  androidx.core:core-ktx:1.6.0
  androidx.appcompat:appcompat:1.3.0
  com.google.android.material:material:1.4.0
  androidx.lifecycle:lifecycle-xxx:2.3.1
  androidx.fragment:fragment-ktx:1.3.6
  androidx.navigation:navigation-ui-ktx:2.3.5
  androidx.navigation:navigation-fragment-ktx:2.3.5
  
  com.google.dagger:hilt-android:2.38.1
  ```

  目前在 AGP 7.2.2 下 Kotlin 1.5.21 与上面的依赖库版本可以通过编译。

