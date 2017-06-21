---
layout: post
title: 将自己写的库发布到Jcenter
date: 2016-03-12
category: 代码之外
tags: [Android,开源库]
keywords:
---

本文主要介绍如何最简单的将自己的 `Library` 发布到 `jcenter`。

2017.6.16 更新为使用远程上传脚本打包上传，简化操作流程

使用 `gradle` 开发过程中，我们可以使用 `compile` 命令，依赖 `Library`，这种做法的好处时，我们不需要再去关注一堆 `jar` 文件，当我们需要升级 `Library` 时，只需要改变依赖的版本号，就可以完成升级。

我们之所以可以使用一句 `gradle` 脚本完成依赖的下载，升级，是因为这些 `Library` 都被存储在了一个公共的服务器上，通过 `compile` 命令可以唯一的映射到服务器存储的 `Library`。 `jcenter` 就是一个这样的服务，`AndroidStudio` 现在新建项目都会默认依赖 `repositories {jcenter()}`，当执行 `compile` 脚本时，就会去 `jcenter` 检查对应的 `Library` 进行下载和依赖。
 
<!--more-->
 
## 踩过的坑
1. 删除 `gradle.properties` 里面会有 `systemProp.http.proxyHost=127.0.0.1 systemProp.http.proxyPort=1080 `字段，使用代理可能会导致无法上传发布。

2. 打包 `javadoc` 时会出问题，因为他会将你所有的注释打包进去，如果注释写的不规范就会报错，不过这个是有提示的，按照提示改掉就好了，注释要符合要求。

3. 如果出现了 `gradle commond not found` ,说明你的 `gradle` 路径没有配置，[mac可以参考这篇文章进行配置](http://blog.csdn.net/chendong_/article/details/50865767)。

4. 你的 `library` 中的 `manifest` 文件有 `allowback` 属性最好删掉， 当别人或者你的别的工程引用这个库时，需要进行 `manifest` 的 `merge` 操作，如果某些属性冲突了会导致合并失败，因此库文件的 `manifest` 文件只保留必要的属性，即可。
 

## 注册账号
我们要发布自己的库到 `JCenter`,首先要将代码发布到 `maven` 库，[官方网站www.bintray.com](https://bintray.com/) ，进去网站注册登录就好了，注册之后可以拿到你的用户名和 `apikey`。

`apiKey` 的获取方法 －> 右上角点击头像 －> `your profile `-> 头像下面edit -> 最下面的 `APIkey` 就可以获取了，后面有用。


## 构建项目
使用 `AS` 开发你的项目应该是有多个 `module` 的，通常你的项目至少应该有 `app` 这个 `module` ，创建一个 `library` 类型的 `module`（点击工具栏 File - > new moudle -> library,就可以建立一个library moudle)，因为我们要发布到 `maven` 的是你的 `library`，所以这个 `moudle` 需要有。然后你应该把这个项目发布到 `github`上去，后续需要填写项目的地址，当然这不是必选的。

## 配置 project/build.gradle

配置这个文件主要是依赖一些必要的插件来完成后续的打包上传操作，可能当你看到时下面的插件版本已经不是最新的了，你可以自己依赖最新的插件。

```java
//最后写完应该是这样的
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.3'
        //主要是下面的两行
        classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.0'
        classpath 'com.github.dcendents:android-maven-gradle-plugin:1.4.1'
    }
}

allprojects {
    repositories {
        jcenter()
    }
}
```

## 配置 module/build.gradle

需要在 `ext{}` 内配置项目的相关信息，在每个信息上我都加了比较详细的注释。

完整的配置文件如下，引用了我 `git` 上面远程的一个上传脚本，当然你也将全部内容复制到本地进行修改，使用远程脚本是为了简化书写脚本的流程。

```gradle
apply plugin: 'com.android.library'

android {
    ...
}

dependencies {
    ...
}

ext {
    // 项目的主页
    yourSiteUrl = 'https://github.com/chendongMarch/LightAdapterExample'
    // Git仓库的url
    yourGitUrl = 'https://github.com/chendongMarch/LightAdapterExample'
    // issue 地址
    YourIssueTrackerUrl = 'https://github.com/chendongMarch/LightAdapterExample/issues'

    // 项目的名称，将会显示在bintray
    yourProName = 'LightAdapter'
    // 项目的描述
    yourProDesc = 'Java Recyclerview Library'

    // bintrayID
    yourId = 'chendongmarch'
    // nickName
    yourName = 'chendong'
    // email
    yourEmail = 'helloworld4x@gmial.com'

    // 最后生成依赖 compile 'yourGroup:yourArtifactId:yourVersion'
    // 最后生成依赖 compile 'com.march.lightadapter:lightadapter:1.0.1-beta1'
    yourGroup = 'com.march.lightadapter'
    yourArtifactId = 'lightadapter'
    yourVersion = '1.0.1-beta4'

    
    // 此文件中存储的是 bintray 的敏感信息，用户名和key
    // 你需要在该文件中如下声明
    // bintray.user=xxxx
    // bintray.apikey=xxxx
    // 你可以使用项目中的 local.properties
    yourBintrayUserPath = '/Users/march/AndroidRes/file/common_local.properties'
}

apply from: 'https://raw.githubusercontent.com/chendongMarch/Resource/master/jcenter/bintrayUpload.gradle'
```

### 需要注意的点

下面配置的三个值，决定了你最后生成的依赖是什么样的，我用我现在的库举了个例子，这个是可以后期修改的。

```gradle
// 最后生成依赖 compile 'yourGroup:yourArtifactId:yourVersion'
// 最后生成依赖 compile 'com.march.lightadapter:lightadapter:1.0.1-beta1'
yourGroup = 'com.march.lightadapter'
yourArtifactId = 'lightadapter'
yourVersion = '1.0.1-beta4'
```

用户名和 `key`，这些敏感信息是不应该被传到 `git` 上面的， 你应该在 `.gitignore` 中避免上传这个文件，习惯的做法你可以将这些信息写在 `local.properties` 中，路径配置为 `yourBintrayUserPath = 'local.properties'`，我的做法是写在本地文件中然后使用绝对路径引用。

然后你在文件中需要按照如下格式声明 `bintray.user` 和 `bintray.apikey`；

```gradle
// 此文件中存储的是 bintray 的敏感信息，用户名和key
// 你需要在该文件中如下声明
// bintray.user=xxxx
// bintray.apikey=xxxx
// 你可以使用项目中的 local.properties
yourBintrayUserPath = '/Users/march/AndroidRes/file/common_local.properties'
```

## 发布类库
我在 `mac` 下开发，相关命令可能有所差异。

打开 `terminal`，运行 `install` 命令

```bash
./gradlew install
```

成功后运行上传命令

```bash
./gradlew bintrayUpload
```

当提示 `Build Success` 后，去[网站](www.bintray.com)打开所在的项目，注意右上角有个链接，类似 `https://dl.bintray.com/chendongmarch/maven`，这是你 `maven`库的链接，也就是你私有的空间。注意左下角有三个很小的选项 `maven`、`gradle`、`lvy`，点击 `gradle` 可以看到一个依赖地址，这是 `compile` 的地址，就算大功告成啦。

### 在私有空间使用类库

此时你已经可以使用这个类库了，但是因为你的类库没有发布到共有空间，因此使用时需要在 `project/build.gradle` 声明你的私有空间。

```
//在 project/build.gradle 文件中添加你私有 maven 空间
allprojects {
    repositories {
        jcenter()
        // 现在之所以添加这一句是因为你的类库只是发布到了maven
        // 但是没有发布到JCenter
        // 因此需要单独依赖你的私有空间的地址。
        maven {url 'https://dl.bintray.com/chendongmarch/maven'}
    }
}

//在app/build.gradle文件中依赖
compile 'com.march.lightadapter:lightadapter:1.0.1-beta1'
```

### 发布到 jcenter

然后我们将类库发布到公共空间让别人可以快速使用，还是在网站查看项目右下角可以看到 `Add to jcenter` 点击之后可以很简单的添加你的库到 `JCenter`,但是审核需要时间，第一次会慢一点，以后会很快。等到审核通过，就可以将 `maven{url=""}` 这一块去掉，直接 `compile`，因为已经可以在 `jcenter` 找到你的类库了， 不需要使用私有空间了。

### 在本地空间使用类库

单单运行 `./gradlew  install` 任务， `gradle` 会在 `maven` 的本地仓库中生成工件，只需将 `mavenLocal` 添加到 `repositories`，我们可以像发布到 `JCenter` 一样引用自己的库，方便打包那些多个项目共享又不想发布的私有库, 在`project/build.gradle` 文件添加下面的依赖，可以使用本地库，类比上面的私有空间，这个应该算本地空间。

```gradle
allprojects {
    repositories {
        mavenLocal()
        jcenter()
    }
}
```


## 一些理解
总体来说，最最重要的是打包上传的脚本，他负责打包源文件，生成 `aar`，生成 `doc` 然后上传到 `bintray`，我之前采用过别的方式打包类库，本文介绍的方法有个很大的优点就是把这个任务独立出来，写在 `bintrayUpload.gradle` 文件中，使用远程脚本，需要打包上传的 `library` 只需要配置一下类库相关信息，引用这个脚本就可以上传，不需要进行复杂的配置。
