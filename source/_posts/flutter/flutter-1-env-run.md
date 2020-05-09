---
layout: post
title: Flutter开发-1-入门篇
category: Android
tags:
  - Android
  - Flutter
  - Dart
abbrlink: '1e731081'
keywords:
  - Flutter
  - Dart
location: 青岛
date: 2019-01-17 09:18:00
photos: https://images.pexels.com/photos/1760962/pexels-photo-1760962.jpeg?auto=compress&cs=tinysrgb&dpr=2&w=500
---


> 推荐阅读
 
[在macOS上搭建Flutter开发环境](https://flutterchina.club/setup-macos/)

<!--more-->

## Q&A

- 出现问题，`iOS` 模拟器和 `Android` 设备可以运行，但是 `iOS` 真机没有办法运行?

```bash
brew update
brew uninstall --ignore-dependencies libimobiledevice
brew uninstall --ignore-dependencies usbmuxd
brew install --HEAD usbmuxd
brew unlink usbmuxd
brew link usbmuxd
brew install --HEAD libimobiledevice
```