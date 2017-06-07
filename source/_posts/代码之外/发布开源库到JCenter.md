---
layout: post
title: 将自己写的库发布到Jcenter
date: 2016-03-12
category: 代码之外
tags: [Android,开源库]
keywords:
---


## 1. 前言
> 1. 我们使用gradle开发，可以通过compile命令将别人的library下载到本地使用，使用这种方式简单快速，而且当需要版本升级时改一下版本号就可以了，简直比使用jar包好了几百倍。  
2. 我们之所以可以使用别人的library，是因为别人将自己的library发布到了JCenter,我们可以通过compile命令将aar文件文档等内容下载下来使用，也就是说只要我们将自己的library发布到JCenter,那么自己和别人就都可以使用了。  
3. 网上有很多资料，介绍了各家各户的服务器，但是JCenter目前是library最多的服务器，AndroidStudio现在新建项目都是默认依赖 `repositories {jcenter()}`但是看着容易做着难，走了很多弯路，现在把我的经验分享出来，希望可以帮到别人。  
 
<!--more-->

## 2. 推荐阅读
- [推荐这篇文章](http://www.jianshu.com/p/c721f9297b2f?utm_campaign=hugo&utm_medium=reader_share&utm_content=note)


## 3. 踩过的坑
1. 删除gradle.properties里面会有 systemProp.http.proxyHost=127.0.0.1 systemProp.http.proxyPort=1080 字段，使用代理可能会导致无法上传发布。

2. 打包javadoc时会出问题，因为他会将你所有的注释打包进去，如果注释写的不规范就会报错，不过这个是有提示的，按照提示改掉就好了，注释要符合要求。

3. 如果出现了gradle commond not found,说明你的gradle路径没有配置，[mac可以参考这篇文章，windows的话自己查一下如何配置](http://blog.csdn.net/chendong_/article/details/50865767)

4. 你的library中的manifest文件有allowback属性最好删掉， 当别人或者你的别的工程引用这个库时，需要进行manifest的merge操作，如果某些属性冲突了会导致合并失败，因此库文件的manifest文件只保留必要的属性，即可。
 

## 4. 注册账号
我们要发布自己的库到JCenter,首先要将代码发布到maven库，[官方网站www.bintray.com](https://bintray.com/) ，进去网站注册登录就好了，注册之后可以拿到你的用户名和apikey。

apiKey的获取方法 －> 右上角点击头像 －> your profile -> 头像下面edit -> 最下面的APIkey 就可以获取了，后面有用.


## 5. 构建项目
使用As开发你的项目应该是有多个module的，通常你的项目至少应该有app这个module，创建一个library类型的module（点击file - > new moudle -> library,就可以建立一个library moudle)，因为我们要发布到maven 的是你的library，所以这个moudle需要有。然后你应该把这个项目发布到github上去，后续需要填写项目的地址，当然这不是必选的。


## 6. 配置(重要)

### 6.1 首先是 projectName.gradle
配置这个文件主要是依赖一些必要的插件来完成后续的打包上传操作
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

### 6.2 配置local.properties
`local.properties`不会被上传到git上面，因此一些个人用户的信息需要配置在这里面，避免泄漏

```java
// 这里是你在bintray的id同下面bintray.user
developer.id=chendongmarch
// 开发者的名字，随意
developer.name=chendong
// 开发者邮箱
developer.email=helloworld4x@gmail.com

// bintrayId 
bintray.user=chendongmarch
// bintrayApiKey
bintray.apikey=f452fcdf8...xxxx...4b1a925331
```
### 6.3 配置Library
在你新创建的lib module下面的project.propertie文件，如果没有就新建一个  


```java
groupId artifactId version决定了最后你的类库的依赖的地址
例如我下面的配置最后的地址就是
compile 'groupId:artifactId:version'
compile 'com.march.lib-dev:lib-dev:0.0.1-beta6'

// 项目的名称
project.name=lib-dev
// groupId
project.groupId=com.march.lib-dev
// artifactId
project.artifactId=lib-dev
// version
project.versionName=0.0.1-beta6
// 打包类型
project.packaging=aar
// 类库的主页
project.siteUrl=git@github.com:chendongMarch/CommonLib
// 类库的git地址
project.gitUrl=git@github.com:chendongMarch/CommonLib.git

javadoc.name=lib-dev
```


### 6.4 打包上传的脚本
上面配置那么多，其实都在这个脚本中将这些数据读出来，作为打包上传时候使用，将下面的脚本放到project根目录下面即可，命名为`bintrayUpload.gradle`，拷贝下面的脚本到项目根目录就可以使用了。

```java
apply plugin: 'com.github.dcendents.android-maven'
apply plugin: 'com.jfrog.bintray'

// load properties
Properties properties = new Properties()

File projectPropertiesFile = project.file("project.properties");
if (projectPropertiesFile.exists()) {
    properties.load(projectPropertiesFile.newDataInputStream())
}
// read properties
def projectName = properties.getProperty("project.name")
def projectGroupId = properties.getProperty("project.groupId")
def projectArtifactId = properties.getProperty("project.artifactId")
def projectVersionName =  properties.getProperty("project.versionName")
def projectPackaging = properties.getProperty("project.packaging")
def projectSiteUrl = properties.getProperty("project.siteUrl")
def projectGitUrl = properties.getProperty("project.gitUrl")

File localPropertiesFile = project.file("../local.properties");
if (localPropertiesFile.exists()) {
    properties.load(localPropertiesFile.newDataInputStream())
}


def developerId = properties.getProperty("developer.id")
def developerName = properties.getProperty("developer.name")
def developerEmail = properties.getProperty("developer.email")

def bintrayUser = properties.getProperty("bintray.user")
def bintrayApikey = properties.getProperty("bintray.apikey")

def javadocName = properties.getProperty("javadoc.name")

group = projectGroupId

// This generates POM.xml with proper parameters
install {
    repositories.mavenInstaller {
        pom {
            project {
                name projectName
                groupId projectGroupId
                artifactId projectArtifactId
                version projectVersionName
                packaging projectPackaging
                url projectSiteUrl
                licenses {
                    license {
                        name 'The Apache Software License, Version 2.0'
                        url 'http://www.apache.org/licenses/LICENSE-2.0.txt'
                    }
                }
                developers {
                    developer {
                        id developerId
                        name developerName
                        email developerEmail
                    }
                }
                scm {
                    connection projectGitUrl
                    developerConnection projectGitUrl
                    url projectSiteUrl
                }
            }
        }
    }
}

// This generates sources.jar
task sourcesJar(type: Jar) {
    from android.sourceSets.main.java.srcDirs
    classifier = 'sources'
}

task javadoc(type: Javadoc) {
    source = android.sourceSets.main.java.srcDirs
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
}

// This generates javadoc.jar
task javadocJar(type: Jar, dependsOn: javadoc) {
    classifier = 'javadoc'
    from javadoc.destinationDir
}

artifacts {
//    archives javadocJar
    archives sourcesJar
}


//javadoc configuration
javadoc {
    options {
        encoding "UTF-8"
        charSet 'UTF-8'
        author true
        version projectVersionName
        links "http://docs.oracle.com/javase/7/docs/api"
        title javadocName
    }
}

////打包doc,避免注解报错
android.libraryVariants.all { variant ->
    println variant.javaCompile.classpath.files
    if (variant.name == 'release') { //我们只需 release 的 javadoc
        task("generate${variant.name.capitalize()}Javadoc", type: Javadoc) {
            // title = ''
            // description = ''
            source = variant.javaCompile.source
            classpath = files(variant.javaCompile.classpath.files, project.android.getBootClasspath())
            options {
                encoding "utf-8"
                links "http://docs.oracle.com/javase/7/docs/api/"
                linksOffline "http://d.android.com/reference", "${android.sdkDirectory}/docs/reference"
            }
            exclude '**/BuildConfig.java'
            exclude '**/R.java'
        }
        task("javadoc${variant.name.capitalize()}Jar", type: Jar, dependsOn: "generate${variant.name.capitalize()}Javadoc") {
            classifier = 'javadoc'
            from tasks.getByName("generate${variant.name.capitalize()}Javadoc").destinationDir
        }
        artifacts {
            archives tasks.getByName("javadoc${variant.name.capitalize()}Jar")
        }
    }
}

// bintray configuration
bintray {
    user = bintrayUser
    key = bintrayApikey
    configurations = ['archives']
    pkg {
        repo = "maven"
        name = projectName
        websiteUrl = projectSiteUrl
        vcsUrl = projectGitUrl
        licenses = ["Apache-2.0"]
        publish = true
    }
}
```


## 7. 打包发布
### 7.1 执行脚本
在需要打包上传的lib module的`build.gradle`文件中如下代码就可以执行上面的脚本啦

```gradle
apply from: '../bintrayUpload.gradle'
```
打开terminal，进入lib module目录下面执行如下命令开始打包上传，第一次可能需要下载一些以来文件会慢一些。

```bash
../gradlew bintrayUpload
```





### 7.2 去网站查看
打开[bintray.com](https://bintray.com)打开所在的项目，注意右上角有个链接，类似`https://dl.bintray.com/chendongmarch/maven`，这是你maven库的链接。注意左下角有三个很小的选项maven、gradle、lvy,点击gradle可以看到类似这样的`compile 'com.march.lib-dev:lib-dev:0.0.1-beta6'`一个依赖地址，这是compile的地址，就算大功告成啦。


### 7.3 使用库
```java
//在项目名称.gradle文件中添加你bintray的链接
allprojects {
    repositories {
        jcenter()
        // 现在之所以添加这一句是因为你的类库只是发布到了maven
        // 但是没有发布到JCenter
        // 因此需要单独依赖你的bintray的地址。
        maven {url 'https://dl.bintray.com/chendongmarch/maven'}
    }
}
//在app.gradle文件中添加
compile 'com.march.lib-dev:lib-dev:0.0.1-beta6'
```


### 7.4 发布到JCenter
在网站打开你的项目右下角可以看到Add to jcenter点击之后可以很简单的添加你的库到JCenter,但是审核需要时间，第一次会慢一点。等到审核通过，就可以将maven{url=""}这一块去掉，直接compile


### 7.5 发布本地库
不用上传到 jcenter 单单运行 `./gradlew  install` 任务， gradle 会在 maven 的本地仓库中生成工件（artifact），只需将 mavenLocal 添加到 repositories，我们可以像发布到 JCenter 一样引用自己的库，方便打包那些多个项目共享又不想发布的私有库, 在project/build.gradle文件添加下面的依赖，可以使用本地库
```java
allprojects {
    repositories {
        mavenLocal()
        jcenter()
    }
}
```


## 8. 一些理解
总体来说，最最重要的是打包上传的脚本，他负责打包源文件，生成aar，生成doc然后上传到bintray，我之前采用过别的方式打包类库，本文介绍的方法有个很大的优点就是把这个任务独立出来，写在`bintrayUpload.gradle`文件中，需要打包上传的library只需要配置一下类库相关信息，引用这个脚本就可以上传，不需要进行复杂的配置，而且可以将一个项目中的多个类库同时上传，每个类库只要在它的build.gradle文件中引用一下该脚本就可以啦。
