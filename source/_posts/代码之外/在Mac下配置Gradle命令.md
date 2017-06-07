---
layout: post
title: 在Mac下配置Gradle命令
date: 2016-03-12
category: 代码之外
tags: [Android,配置]
keywords: Android,配置
description: 在Mac下配置Gradle命令
---



## 推荐阅读
[Mac 下配置 Sdk,Gradle](http://blog.csdn.net/duqiuke/article/details/46057493)


## 相关下载
[下载gradle http://gradle.org/downloads/](http://gradle.org/downloads/)


## 配置环境变量
-  打开.bash-profile

```bash
touch .bash-profile
open -e .bash_profile
```

- 打开 bash_profile，配置环境变量。

```
//不使用ndk可以不配置
export NDK_PATH="/Users/Bob_ge/Documents/android_dev/ndk/android-ndk-r10d"
export SDK_PATH="/Users/Bob_ge/Library/Android/sdk/platform-tools"
export GRADLE_HOME="/Users/Bob_ge/Documents/android_dev/gradle/gradle-2.4/bin"
export PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/opt/X11/bin:/usr/local/ant/bin:/opt/reverse:${SDK_PATH}:${NDK_PATH}:${GRADLE_HOME}
```

- 更新 bash_profile

```bash
source .bash-profile
```

- 输入adb 检测是否配置正确

```bash
gradle -version
```
