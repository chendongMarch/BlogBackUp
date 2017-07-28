---
layout: post
title: Android使用CMAKE编译libjpeg-turbo
categories: 
	- Android
	- Ndk
tags:
  - Android
  - Ndk
keywords:
	- Android
	- Ndk
	- libjpeg-turbo
	- cmake
	- 图片压缩
comments: true
abbrlink: fc67caea
date: 2017-07-19 19:56:37
password:
---

本文主要介绍使用  `CMAKE` 编译 `libjpeg-turbo` 类库，本文相关代码请在[GitHub-TurboJpegSample](https://github.com/chendongMarch/TurboJpegSample) 查看。

`libjpeg-turbo` 附[ GitHub 地址](https://github.com/libjpeg-turbo/libjpeg-turbo)，`libjpeg-turbo` 是个运用极其广泛的库，可以说，基本上电脑上手机上能见到的 `JPEG` 压缩的地方用的一般都是 `libjpeg-turbo`，本文只介绍使用了图片压缩的功能。

<!--more-->

使用 `Android` 保存图片时，我们通常使用的是 `Bitmap.compress()` 方法，但是使用该方法时，就算 `quality` 设置为 100，图片质量还是会越来越模糊，颜色也会越来越绿～，至于为什么会这样，请看 [知乎回答](https://www.zhihu.com/question/29355920)。 这个问题在贴吧上体现尤为明显，贴吧里面经常很多绿绿的图片就是因为大家保存下来上传上去，保存下来上传上去 ... 导致质量越拉越低，我解压了 美图秀秀 和 in 的 apk 发现里面都引用了 `libjpeg.so`，所以这个应该是一个比较通用的解决方案。

我们对图片使用质量压缩时它的底层就是用 `skia` 引擎进行处理，如我们调用`bitmap.compress(Bitmap.CompressFormat.JPEG...)` 他实际会 使用一个`libjpeg.so` 的动态库进行编码压缩。`android` 在进行 `jpeg` 压缩编码的时候，考虑到了效率问题使用了定长编码方式进行编码（因为当时的手机性能都比较低），而 `ios` 使用了变长编码的算法——哈夫曼算法。而且 `IOS` 对 `skia` 引擎也做了优化，所以我们看到同样的图片在 `ios` 上压缩会好一点。

文档就整理到这里吧，其实上面说的都是看了一些博客的介绍然后进行了整理和记录，来清楚为什么我们需要自己来编译 so 实现图片压缩。

## 资源准备

我们需要下载源码编译出 `libjpeg.so` 来使用，clone 源代码。

```bash
git clone git://git.linaro.org/people/tomgall/libjpeg-turbo/libjpeg-turbo.git -b linaro-android
```

重命名根目录 `libjpeg-turbo` 为 `jni`，进入 `jni` 目录，首先你应该配置好 `ndk`， 使用 `ndk-build`  命令进行编译。

```bash

mv libjpeg-turbo jni

ndk-build APP_ABI=armeabi-v7a,armeabi
```
编译完成之后我们就可以获取到 `libjpeg.so`，目录如下：

```bash
jni
libs
	- armeabi
		- libjpeg.so
	- armeabi-v7a
		- libjpeg.so
obj
	- local
		- armeabi
		- armeabi-v7a
```

此时如果出现了以下异常：

```bash
Users/march/AndroidRes/sdk/ndk-bundle/build/core/build-binary.mk:702: 
*** Android NDK: Aborting (set APP_ALLOW_MISSING_DEPS=true to allow missing dependencies)    .  Stop.
```
打开 `build-binary.mk` 添加 `APP_ALLOW_MISSING_DEPS=true`

```bash
ifdef undefined_libs
    $(call __ndk_warning,Module $(LOCAL_MODULE) depends on undefined modules: $(undefined_libs))

    # https://github.com/android-ndk/ndk/issues/208
    # ndk-build didn't used to fail the build for a missing dependency. This
    # seems to have always been the behavior, so there's a good chance that
    # there are builds out there that depend on this behavior (as of right now,
    # anything using libc++ on ARM has this problem because of libunwind).
    #
    # By default we will abort in this situation because this is so completely
    # broken. A user may define APP_ALLOW_MISSING_DEPS to "true" in their
    # Application.mk or on the command line to revert to the old, broken
    # behavior.
    APP_ALLOW_MISSING_DEPS=true # here
    ifneq ($(APP_ALLOW_MISSING_DEPS),true)
        $(call __ndk_error,Aborting (set APP_ALLOW_MISSING_DEPS=true to allow missing dependencies))
    endif
endif
```


## 创建工程

使用 `AndroidStudio` 新建工程，勾选 `support c++`，生成的工程中会包含 `CMakeLists.txt` 文件，在下面的介绍中没有截图，对目录的说明如果不够清晰，请至 [GitHub-TurboJpegSample](https://github.com/chendongMarch/TurboJpegSample) 查看工程代码。

在 `app` 目录下新建文件夹 `libjpeg`，用来存放 `so` 文件和头文件，目录如下：

```bash
app
	- libjpeg
		- prebuilt
			- armeabi
				- libjpeg.so
		- include
			- 头文件
	- src 
		- main
			- cpp
			- java
			- res
```

编写 `compress.h` 和 `compress.c`，这个就是压缩的核心代码了，篇幅较长，但是这里就不贴了，详情请查看 [项目的 cpp 目录](https://github.com/chendongMarch/TurboJpegSample/tree/master/app/src/main/cpp)，使用的算法是网上查找的，看起来资料介绍的都是这一种。


## CMakeLists.txt

```bash

# 最低版本
cmake_minimum_required(VERSION 3.4.1)

#设置生成的 so 动态库最后输出的路径
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/../jniLibs/${ANDROID_ABI})

# 初始化目录变量
set(libjpeg_dir ${CMAKE_SOURCE_DIR}/libjpeg)
set(INC_DIR ${libjpeg_dir}/include)
set(libjpeg_lib_dir ${libjpeg_dir}/prebuilt)

# 添加头文件目录
include_directories(${INC_DIR})

# 设置资源路径
set(SOURCE_FILES src/main/cpp/compress.c)

# 
add_library(compress SHARED
            ${SOURCE_FILES})


find_library(log-lib log)
find_library(graphics jnigraphics)

add_library(libjpeg SHARED IMPORTED)
set_target_properties(libjpeg PROPERTIES IMPORTED_LOCATION ${libjpeg_lib_dir}/${ANDROID_ABI}/libjpeg.so)


target_link_libraries(compress libjpeg ${log-lib} ${graphics})
```

## build.gradle

相比普通的 `Android` 工程，`app/build.gradle` 文件也有些许不同，我只编译了 `armeabi` 。

```gradle
android {
    compileSdkVersion 25
    buildToolsVersion "25.0.3"
    defaultConfig {
	     ...
        externalNativeBuild {
            cmake {
                abiFilters 'armeabi'
            }
        }

        ndk {
            //打包进APK的ABI类型
            abiFilters 'armeabi'
        }
    }
    
    externalNativeBuild {
        cmake {
            path "CMakeLists.txt"
        } 
    }
    sourceSets {
        main {
            java.srcDirs 'src/java'
            jniLibs.srcDirs  'libs'
//          jniLibs.srcDirs '../libjpeg/prebuilt', 'libs'

            // 这里没有添加libjpeg.so这个动态库，也是可以执行的。
            // 原因在于android本身使用了 libjpeg.so这个动态库，
            // 这个库存放在/system/lib下，如果我们没有加入
            // libjpeg.so的话，他会去/system/lib下加载这个动态库

            // 如果android手机上没有 libjpeg.so这个动态库的话，
            // 也可以使用: jniLibs.srcDirs '../libjpeg/prebuilt' 'libcd
            // 将libjpeg.so加入到apk中
        }
    }
}

```

## 编写 java 调用

```java
public class TurboJpegUtils {

    static {
        System.loadLibrary("compress");
    }

    /**
     * 使用native方法进行图片压缩。
     * Bitmap的格式必须是ARGB_8888 {@link android.graphics.Bitmap.Config}。
     *
     * @param bitmap   图片数据
     * @param quality  压缩质量
     * @param dstFile  压缩后存放的路径
     * @param optimize 是否使用哈夫曼算法
     * @return 结果
     */
    public static native int compress(Bitmap bitmap, int quality, String dstFile, boolean optimize);
}
```

运行之后就可以测试结果，同时在 配置的 `jniLibs` 目录中也会生成 so 文件，在别的地方使用同包名下的 `TurboJpegUtils` 类也可以调用了。


## 总结

测试对比结果，确实比 `Andorid` 原生的算法压缩图片的效果好很多，但是压缩次数多了以后也会有明显的模糊现象。