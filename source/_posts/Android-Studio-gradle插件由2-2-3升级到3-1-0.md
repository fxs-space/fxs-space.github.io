---
title: Android Studio gradle插件由2.2.3升级到3.1.0
date: 2018-07-07 19:14:01
categories: React Native
tags: 爬坑
---
## Android Studio gradle插件由2.2.3升级到3.1.0

升级gradle插件的原因是因为项目里集成第三方依赖都已经需要gradle3.x的版本去编译，所以我也需要将项目的gradle插件进行升级；插件升级的过程中也遇到了一些乱七八糟编译失败的报错，网上搜了一大堆解决办法，结果没有一个方案让升级过程变得畅通无阻，没办法只好网上找了gradle3.x版本的Demo进行比对尝试，后来结果还算令人欣喜；现在开始奉上我升级gradle插件的方案，希望可以帮助有需要的人少走点弯路。

<!--more-->

##### 1. 更新 Android Gradle Plugin 版本
打开 `android/build.gradle` 文件

将 `classpath 'com.android.tools.build:gradle:2.2.3'` 修改为 `com.android.tools.build:gradle:3.1.0`

##### 2. 更新 Gradle 版本
打开 `android/gradle/wrapper/gradle-wrapper.properties` 文件

将 `distributionUrl=https\://services.gradle.org/distributions/gradle-2.14.1-all.zip` 修改为 `distributionUrl=https\://services.gradle.org/distributions/gradle-4.4-all.zip`

##### 3. 更新 Android SDK Build Tools 版本
打开 `android/app/build.gradle`文件

先将 `compileSdkVersion 23 buildToolsVersion "23.0.1"` 修改为 `compileSdkVersion 26 buildToolsVersion "27.0.3"`；

再将 `targetSdkVersion 22` 修改为 `targetSdkVersion 26`。

##### 4. 更新 uses-sdk 中的 targetSdkVersion 版本
打开 `android/app/src/main/AndroidManifest.xml` 文件

将 `<uses-sdk android:minSdkVersion="16" android:targetSdkVersion="22" />` 修改为 `<uses-sdk android:minSdkVersion="16" android:targetSdkVersion="26" />` 。

##### 5. 升级 appcompat-v7 的版本
打开 `android/app/build.gradle` 文件

将`compile "com.android.support:appcompat-v7:23.0.1"` 修改为 `compile "com.android.support:appcompat-v7:27.1.1"`;

同时将 `dependencies` 中所有的 `compile` 修改为 `implementation`，

如：`compile "com.android.support:appcompat-v7:27.1.1"` 修改为 `implementation "com.android.support:appcompat-v7:27.1.1"`

具体原因看[这里](https://developer.android.google.cn/studio/build/gradle-plugin-3-0-0-migration)


##### 6. 设置 maven 仓库
在 `android/build.gradle` 文件中进行maven仓库的设置，请参考下面的配置：

```
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
  repositories {
    jcenter()
    google()
    maven {
      url 'https://maven.google.com'
      name 'Google'
    }
    maven{
      url 'http://maven.aliyun.com/nexus/content/groups/public/'
      name 'aliyun'
    }
    maven {
      url "https://jitpack.io"
      name 'jitpack'
    }
  }
  dependencies {
    classpath 'com.android.tools.build:gradle:3.1.0'

    // NOTE: Do not place your application dependencies here; they belong
    // in the individual module build.gradle files
  }
}

allprojects {
  repositories {
    mavenLocal()
    jcenter()
    google()
    maven {
      // All of React Native (JS, Obj-C sources, Android binaries) is installed from npm
      url "$rootDir/../node_modules/react-native/android"
    }
    maven {
      url 'https://maven.google.com' // Google's Maven repository
      name 'Google'
    }
    maven{
      url 'http://maven.aliyun.com/nexus/content/groups/public/'
      name 'aliyun'
    }
    maven {
      url "https://jitpack.io"
      name 'jitpack'
    }
  }
}
```

现在你可以看到提供了好几个数据源，你可以选择一个，也可以都配置上去。


##### 7. error: uncompiled PNG file passed as argument报错处理
如遇到 `error: uncompiled PNG file passed as argument. Must be compiled first into .flat file.` 的报错，解决方法如下：

打开 `android/gradle.properties`文件

添加 `android.enableAapt2=false`