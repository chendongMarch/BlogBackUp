---
layout: post
title: 搭建 Weex 开发环境和集成到 Android 平台 [Weex]
categories:
  - weex
tags: weex
keywords:
  - weex
abbrlink: 39fe7888
date: 2017-10-25 10:46:00
---

`Weex` 是阿里开源的一款跨平台移动开发工具，`Weex` 能够完美兼顾性能与动态性，让移动开发者通过简捷的前端语法写出 `Native` 级别的性能体验，并支持 `IOS`、`android`、 `Web`等多端部署。

对于移动开发者来说，`Weex` 主要解决了频繁发版和多端研发两大痛点，同时解决了前端语言性能差和显示效果受限的问题。

不足：
布局上 `UI` 嵌套层级太深。

<!--more-->

[Weex GitHub](https://github.com/apache/incubator-weex)

[Weex 官方教程](https://weex.apache.org/cn/guide/)

[Mac 重装 Node](http://linyehui.me/2016/03/03/reinstall-nodejs-on-osx/)

[Weex Android Dev Tools](https://github.com/weexteam/weex-devtools-android/blob/master/README-zh.md)


## 环境安装

解决 安装  `weex-toolkit` 出现 `npm ERR! code ELIFECYCLE` 错误。

```bash
chmod -R 777 ~/.xtoolkit
sudo chmod -R 777 ~/.xtoolkit
```
使用命令开启 `watch` 模式和静态服务器

```bash
npm run dev & npm run serve
```

## weex-toolkit

[WeexToolKit 教程](https://weex.apache.org/cn/guide/tools/toolkit.html)

`weex-toolkit` 是官方提供的一个脚手架命令行工具，你可以使用它进行 Weex 项目的创建，调试以及打包等功能。

具体的安装和初始化流程可以参照官网文档，这里记录几个常用命令。

调试方面，更多特性参照文档。

```bash
# 初始化项目
weex init {project}
weex init my_weex_project

# 实时预览
weex {file.vue}
weex src/page.vue

# 预览整个目录
weex src --entry {entry_file.vue}
weex src --entry src/entry_page.vue

# 打包 weex 项目
weex compile {src_file/src_dir} {dst_dir}
weex compile src/foo.vue dist
```

调试

```bash
weex debug [options] [file.vue/dir]

# 开启调试服务器
weex debug 

# 调试某个文件
weex debug {vue 文件}
weex debug test.vue

# 调试文件夹
weex debug {文件夹} -e {入口文件}
weex debug test_dir -e src/index.vue
```
## Android 端简单集成

配置 `app/build.gralde`

```gradle
android {
    compileSdkVersion 25 // 23 以上
    buildToolsVersion "26.0.2" // 23.0.1 以上
    defaultConfig {
        // ...
        ndk {
            abiFilters "x86"
            abiFilters "armeabi"
        }
    }
     
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_7
        targetCompatibility JavaVersion.VERSION_1_7
    }
}
dependencies {
	// ...
	compile 'com.android.support:appcompat-v7:25.3.1'
	compile 'com.android.support:recyclerview-v7:25.3.1'
	compile 'com.alibaba:fastjson:1.1.46.android'
	compile 'com.taobao.android:weex_sdk:0.10.0'
}
```
在 `Application` 中进行初始化，同时必须在 `AndroidManifest.xml` 文件中进行声明。

```java
WXSDKEngine.initialize(this, new InitConfig.Builder().setImgAdapter(new IWXImgLoaderAdapter() {
    @Override
    public void setImage(String url, ImageView view, WXImageQuality quality, WXImageStrategy strategy) 

    }
}).build());
```

将编译完成的 `js` 文件拷贝到 `assets` 目录，加载本地 `js`

```java
public class MainActivity extends AppCompatActivity implements IWXRenderListener {

    WXSDKInstance mWXSDKInstance;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mWXSDKInstance = new WXSDKInstance(this);
        mWXSDKInstance.registerRenderListener(this);
        /**
         * WXSample 可以替换成自定义的字符串，针对埋点有效。
         * template 是.we transform 后的 js文件。
         * option 可以为空，或者通过option传入 js需要的参数。例如bundle js的地址等。
         * jsonInitData 可以为空。
         * width 为-1 默认全屏，可以自己定制。
         * height =-1 默认全屏，可以自己定制。
         */
        mWXSDKInstance.render("WXSample", WXFileUtils.loadAsset("index.js", this), null, null,  WXRenderStrategy.APPEND_ASYNC);
    }


    @Override
    public void onViewCreated(WXSDKInstance instance, View view) {
        Log.e("chendong","onViewCreated");
        setContentView(view);
    }

    @Override
    public void onRenderSuccess(WXSDKInstance instance, int width, int height) {
        Log.e("chendong","onRenderSuccess");
    }

    @Override
    public void onRefreshSuccess(WXSDKInstance instance, int width, int height) {
        Log.e("chendong","onRefreshSuccess");
    }

    @Override
    public void onException(WXSDKInstance instance, String errCode, String msg) {
        Log.e("chendong",msg + "  "  +errCode);
    }


    @Override
    protected void onResume() {
        super.onResume();
        if(mWXSDKInstance!=null){
            mWXSDKInstance.onActivityResume();
        }
    }

    @Override
    protected void onPause() {
        super.onPause();
        if(mWXSDKInstance!=null){
            mWXSDKInstance.onActivityPause();
        }
    }

    @Override
    protected void onStop() {
        super.onStop();
        if(mWXSDKInstance!=null){
            mWXSDKInstance.onActivityStop();
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if(mWXSDKInstance!=null){
            mWXSDKInstance.onActivityDestroy();
        }
    }
}
```

运行发现如下问题 

```bash
initWXBridge HackAssertionException 
com.taobao.weex.utils.WXHack$HackDeclaration$HackAssertionException: 
java.lang.ClassNotFoundException: com.taobao.weex.devtools.debug.DebugServerProxy
```

解决上述问题，在 `app/build.gradle` 中加入 `debug` 依赖

```gradle
compile 'com.taobao.android:weex_inspector:0.0.8.5'
```
