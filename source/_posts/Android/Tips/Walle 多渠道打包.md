---
layout: post
title: Walle 多渠道打包
categories:
  - Android
tags:
  - Android
  - AndroidTips
keywords:
  - Android
  - Walle
  - 多渠道打包
  - 加固
abbrlink: 317ee1ad
date: 2017-8-16 00:00:00
---

`Walle` 是 `APK Signature Scheme v2` 下新的多渠道打包方案，速度快，也支持 360 加固，下面是本文相关链接。

[Android Signature V2 Scheme 介绍](http://blog.bihe0832.com/android-v2-signature.html)

[美团 Walle 多渠道打包 原理介绍](https://tech.meituan.com/android-apk-v2-signature-scheme.html)

[项目地址 Meituan-Dianping/Walle](https://github.com/Meituan-Dianping/walle)

<!--more-->

[与 360 加固配合使用](http://18e0c209.wiz01.com/share/s/0oUc890scQDx2tkMAj02NI0c3Ubmms31ckdr2UwE0E2X-bzY)

[GitHub Python脚本 多渠道打包+360加固](https://github.com/Jay-Goo/ProtectedApkResignerForWalle)

[Walle 命令行使用](https://github.com/Meituan-Dianping/walle/blob/master/walle-cli/README.md)



## 使用 Walle


参考 `GitHub-Walle` 进行配置，下面是项目最基本的配置，更多的使用方法参照 [Meituan-Dianping/Walle README](https://github.com/Meituan-Dianping/walle)

```gradle
buildscript { 
    repositories {
        jcenter()
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.3.2'
        // walle 多渠道打包
        classpath 'com.meituan.android.walle:plugin:1.1.5'
    }
}

// walle 多渠道打包
apply plugin: 'walle'

android{
...
}

dependencies {
    //
    compile 'com.meituan.android.walle:library:1.1.5'
}

walle {
    // 指定渠道包的输出路径
    apkOutputFolder = new File("${project.buildDir}/outputs/channels");
    // 定制渠道包的APK的文件名称，支持如下属性
    // projectName - 项目名字
    // appName - App模块名字
    // packageName - applicationId (App包名packageName)
    // buildType - buildType (release/debug等)
    // channel - channel名称 (对应渠道打包中的渠道名字)
    // versionName - versionName (显示用的版本号)
    // versionCode - versionCode (内部版本号)
    // buildTime - buildTime (编译构建日期时间)
    // fileSHA1 - fileSHA1 (最终APK文件的SHA1哈希值)
    // flavorName - 编译构建 productFlavors 名
    apkFileNameFormat = '${appName}-${packageName}-${channel}-${buildType}-v${versionName}-${versionCode}-${buildTime}.apk';
    // 渠道配置文件
    channelFile = new File("${project.getProjectDir()}/channel")
}
```

生成渠道包

```bash
./gradlew clean assembleReleaseChannels
// 支持 productFlavors 
./gradlew clean assembleMeituanReleaseChannels
```


## 360 加固 + 多渠道打包


使用 [360 加固网页版](http://jiagu.360.cn/qcms/manager.html#record) 上传 apk，加固完成后下载没有签名的 apk。

### 对齐优化

对下载的 apk 使用 `apkalign` 工具进行对齐优化，`zipalign` 工具位于 `Android SDK/build-tools/25.x.x` 下。

```
zipalign -v 4 [源apk地址] [目标apk地址]

eg:
zipalign -v 4 /Users/march/Downloads/walle/app-release.encrypted.align.apk
```

对齐完成后，对 apk 进行检测是否对齐成功。

```
zipalign -c -v 4 [目标apk地址]

eg:
zipalign -c -v 4 /Users/march/Downloads/walle/app-release.encrypted.align.apk
```

### 二次签名

对齐成功后，使用 `apksigner` 工具对 apk 进行签名，`apksigner` 也位于 `Android SDK/build-tools/25.x.x` 下，但是版本需要在 `25.x.x` 以上，才能支持 `v2` 签名。

```
apksigner sign --ks [密钥地址] [apk 地址]

eg:
apksigner sign --ks /Users/march/AndroidPro/DevKitSample/app/march.keystore /Users/march/Downloads/walle/app-release.encrypted.align.apk
```

### 渠道写入

签名完成后，检测是否签名成功和是否是 `v2` 签名，检查需要使用 `CheckAndroidV2Signature.jar` 文件，[下载地址](https://github.com/bihe0832/AndroidGetAPKInfo/blob/master/CheckAndroidV2Signature.jar)

```
java -jar CheckAndroidV2Signature.jar [apk 地址]

eg:
java -jar /Users/march/Downloads/CheckAndroidV2Signature.jar  /Users/march/Downloads/walle/app-release.encrypted.align.apk
```

将会返回类似下面的结果

```
{"ret":0,"msg":"ok","isV2":true,"isV2OK":true}
```

使用 `Walle` 写入渠道信息，需要借助 `walle-cli-all.jar` 来执行，[下载地址](https://github.com/Meituan-Dianping/walle/blob/master/walle-cli/walle-cli-all.jar)

这一块更多的可以参照 [Walle 命令行使用](https://github.com/Meituan-Dianping/walle/blob/master/walle-cli/README.md)

```
java -jar walle-cli-all.jar put -c [渠道名称] [源 apk 地址] [目标 apk 地址]

eg:
java -jar /Users/march/Downloads/walle-cli-all.jar put -c meituan /Users/march/Downloads/walle/app-release.encrypted.align.apk /Users/march/Downloads/walle/app-release.encrypted.align.meituan.apk
```

同样也支持多渠道同时打包

```
java -jar walle-cli-all.jar batch -f [channel文件] [签名后的apk路径] [生成渠道app的文件夹路径]

eg:
java -jar /Users/march/Downloads/walle-cli-all.jar batch -f /Users/march/AndroidPro/DevKitSample/app/channel /Users/march/Downloads/walle/app-release.encrypted.align.apk /Users/march/Downloads/walle/channel
```
最后检查渠道是否写入成功

```
java -jar walle-cli-all.jar show [apk路径]

eg:
java -jar /Users/march/Downloads/walle-cli-all.jar show /Users/march/Downloads/walle/app-release.encrypted.align.meituan.apk
```