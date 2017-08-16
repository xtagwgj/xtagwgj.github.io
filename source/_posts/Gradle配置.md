---
title: Gradle配置
tags:
  - Android Studio
  - Gradle
date: 2017-08-16 11:44:02
categories:
- 教程
---


### Gradle配置的完整文件
下面的就是Module级别下的配置文件，可以根据自己的需要自行修改删除

```groovy

//第一行代码应用了Android 程序的gradle插件，作为Android 的应用程序，这一步是必须的
apply plugin: 'com.android.application'
//如果我们要写一个library项目让其他的项目引用,就要改为library
//apply plugin: 'com.android.library'


android {
    compileSdkVersion 25
    buildToolsVersion "25.0.1"

    defaultConfig {
        applicationId "com.xxx.xx"
        minSdkVersion 19
        targetSdkVersion 22
        versionCode 11
        versionName "2.0.1"
        multiDexEnabled true

        //全局配置的key，在manifest中使用，如果buildTypes中未配置，则使用该处的值，否则替换
        manifestPlaceholders = [
                BAIDU_API_KEY: "key",
        ]

        //设置支持的SO库架构
        ndk {
            //显示指定支持的ABIs
            abiFilters 'armeabi-v7a', 'armeabi', 'x86', 'arm64-v8a'
        }

        //只保留xhdpi和xxhdpi文件夹厦门的资源文件
        resConfigs "xhdpi", "xxhdpi"
    }



    buildTypes {
        release {
            //动态修改app名称
            resValue "string", "app_name", "app_name"

            //正式服务器地址相关
            buildConfigField "String", "PROPERTY_URL", "\"https://xxx.com\""

            manifestPlaceholders = [
                    JPUSH_APPKEY: "key", //公网JPush上注册的包名对应的appkey.
            ]

            //是否进行混淆
            minifyEnabled true
            //是否支持zip
            zipAlignEnabled true
            //删除无用资源
            shrinkResources true
            //是否支持调试
            debuggable false
            //关闭jni调试
            jniDebuggable false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            //加载签名文件
            signingConfig signinfConfigs.release
        }

        debug {
            //动态修改app名称
            resValue "string", "app_name", "app_name"

            //测试服务器地址相关
            buildConfigField "String", "PROPERTY_URL", "\"http://xxxx:8080\""

            manifestPlaceholders = [
                    JPUSH_APPKEY: "key", //测试JPush的key
            ]

            //是否进行混淆
            minifyEnabled true
            //是否支持zip
            zipAlignEnabled true
            //删除无用资源
            shrinkResources false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }

    //Java 的版本配置
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_7
        targetCompatibility JavaVersion.VERSION_1_7
    }

    //默认的一些文件路径的配置
    sourceSets {
        main {
            assets.srcDirs = ['assets']    //资源文件
            jni.srcDirs 'src/main/jni'     //jni文件
            jniLibs.srcDir 'src/main/jniLibs' //jni库
        }
    }

    repositories {
        maven {
            url "http://repo.acmecorp.com/maven2"
            //如果仓库有密码，也可以同时传入用户名和密码
            credentials {
                username 'user'
                password 'secretPassword'

            }
        }

        flatDir {
            dirs 'aars' //就是你放aar的目录地址
        }
    }

    //签名文件配置
    signinfConfigs {

        release {
            storeFile file("release.keystore")
            storePassword "your keystore password"
            keyAlias "your keystore alias"
            keyPassword "your alias password"
        }

    }

    //multiDex的一些相关配置，这样配置可以让你的编译速度更快
    dexOptions {
        //让它不要对Lib做preDexing
        preDexLibraries = false
        //开启incremental dexing,优化编译效率，这个功能android studio默认是关闭的。
        incremental true
        //增加java堆内存大小
        javaMaxHeapSize "4g"
    }

    //程序在编译的时候会检查lint，有任何错误提示会停止build，我们可以关闭这个开关
    lintOptions {
        //即使报错也不会停止打包
        abortOnError false
    }

    packagingOptions {
        exclude 'META-INF/LICENSE'
        exclude 'META-INF/NOTICE'
        exclude 'META-INF/LICENSE.txt'
    }

    //修改输出apk包的文件名
    android.applicationVariants.all { variant ->
        variant.outputs.each { output ->
            output.outputFile = new File(
                    output.outputFile.parent,
                    "xxx_" + "_" + buildType.name + "_v" + defaultConfig.versionName + ".apk"
            );
        }
    }
}

dependencies {
    //引用所有的jar包
    compile fileTree(include: ['*.jar'], dir: 'libs')
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:25.0.0'
    //引用aar文件
    compile(name: 'openVideo-release', ext: 'aar')
}

//第三方依赖库的本地缓存路径
task showMeCache << {
    configurations.compile.each { println it }
}

```

在上面我们可以看到在`defaultConfig`和`dependencies`下面写明了具体的minSdkVersion或是第三方依赖等信息的版本号，在单个module中或许是比较方便，但是如果有多个module，此时要保证各个module依赖的统一性就比较麻烦。为了解决这个问题，我们可以在project的目录下改成统一的配置，让后面的module都使用该配置下的版本号。

```groovy
buildscript {
//第一种方式
    ext.kotlin_version = '1.1.3-2'
    ext.anko_version = '0.10.1'
    repositories {
        maven{ url 'http://maven.aliyun.com/nexus/content/groups/public'}
        jcenter()
    }

    dependencies {
        classpath 'com.android.tools.build:gradle:3.0.0-beta2'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"

    }
}

allprojects {

    repositories {
        maven{ url 'http://maven.aliyun.com/nexus/content/groups/public'}
        jcenter()
        maven { url "https://jitpack.io" }
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}

//第二种方式
ext {
    config = [
            buildToolsVersion: "25.0.2",
            compileVersion   : 25,
            minSdkVersion    : 21,
            targetVersion    : 25,
            versionCode      : 1,
            versionName      : "1.0",
            packageName      : "com.xx.xxx",
    ]

    lib = [
            support_version : "25.3.0",
            networking      : "1.0.0",
            flyco           : "2.1.2@aar",
    ]
}
```

- 使用第一种方式，我们可以直接在gradle文件中如下依赖
```groovy
  dependencies {
 	compile "org.jetbrains.kotlin:kotlin-stdlib-jre7:$kotlin_version"
 }
```

- 使用第二种方式
```groovy

def cfg = rootProject.ext.config
def libs = rootProject.ext.lib

android {
 	
 	compileSdkVersion cfg.compileVersion
    buildToolsVersion cfg.buildToolsVersion

    defaultConfig {
        applicationId cfg.packageName
        minSdkVersion cfg.minSdkVersion
        targetSdkVersion cfg.targetVersion
        versionCode cfg.versionCode
        versionName cfg.versionName

    }
}

dependencies {
 	compile "com.amitshekhar.android:android-networking:${libs.networking}"
 }
```