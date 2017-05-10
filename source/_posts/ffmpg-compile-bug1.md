---
title: ffmpg cheating native code不能断点调试
date: 2017-05-10 20:43:24
categories:
 - work
tags:
 - android
 - ndk
 - ffmpeg
---

为什么明明上午还能断点的，下午就不能断点了？
<!-- more -->

折腾了整整一天，一点点代码回溯，终于搞清楚了。

先抛问题：在Android studio 2.3上用cmake直接编译调试c++ native代码时，不能断点调试。

起因是，在c++代码引用到了ffmpeg里的头文件`libavutil/timestamp.h`时，编译出错，提示`__STDC_FORMAT_MACROS`这个宏没有定义，导致`PRId64`这个牛逼闪闪的跨平台变量未定义。
怎么解决这个问题呢？首先想到的是cmake过程中用到的，那么缺少这个宏，就要在cmake的编译选项里加上呗！于是，就在工程的app目录下`CmakeList.txt`文件里加了一条语句
```bash
add_library( # Sets the name of the library.
             native-lib

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             src/main/cpp/native-lib.cpp )

set(CMAKE_CXX_FLAGS "-D__STDC_FORMAT_MACROS")

add_library( ffmpeg
             SHARED
             IMPORTED )
set_target_properties( ffmpeg
                       PROPERTIES IMPORTED_LOCATION
                       ${CMAKE_SOURCE_DIR}/src/main/jniLibs/${ANDROID_ABI}/libffmpeg.so )

```

加上后，编译是过了。但是引出来这个文章的主要问题了！
原来能在native代码里step调试的，现在完全跟不进native代码！

一整天的找问题，才找这个原因，网络上的资源几乎没有谈到这个的。
解决方案其实很简单，把`CmakeList.txt`里的这句`set(CMAKE_CXX_FLAGS "-D__STDC_FORMAT_MACROS")`放到app的`build.gradle`文件里的`cppFlags`标签就行了。

`build.gradle`文件内容如下所示：
```bash
apply plugin: 'com.android.application'

android {
    compileSdkVersion 25
    buildToolsVersion "25.0.3"
    defaultConfig {
        applicationId "com.suning.sports.videocutter"
        minSdkVersion 14
        targetSdkVersion 25
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
        externalNativeBuild {
            cmake {
                cppFlags "-D__STDC_FORMAT_MACROS"
            }
        }
        ndk {
            abiFilters "armeabi-v7a"
        }
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
    externalNativeBuild {
        cmake {
            path "CMakeLists.txt"
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:25.3.1'
    compile 'com.android.support.constraint:constraint-layout:1.0.2'
    compile 'com.android.support:design:25.3.1'
    testCompile 'junit:junit:4.12'
}
```

这个坑踩的不划算啊！
