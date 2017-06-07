---
layout: post
title: Python学习之安装WingIDE并破解
date: 2015-05-05
category: 代码之外
tags: [Python]
keywords: Python
description: Python学习之安装WingIDE并破解
---


buildscript {
    repositories {
        jcenter()
        maven { url "https://jitpack.io" }
        maven { url "https://dl.bintray.com/chendongmarch/maven" }
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.2'
        classpath 'com.antfortune.freeline:gradle:0.8.1'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}


apply plugin: 'com.antfortune.freeline'//阿里增量编译



freeline {
        hack true
        productFlavor 'babyphoto'
        applicationProxy false

    }