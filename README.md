# 一份简单的CrossWalk使用指南

标签（空格分隔）： android

---

## 1 什么是CrossWalk？
Web技术的优势早已被广大应用开发者熟知，比如可与云服务轻松集成，基于响应式UI设计的精美布局，高度的开放性，跨平台能力, 高效的分发与部署等等。伴随着移动互联网的快速发展与HTML5技术的逐步成熟，Web应用已经成为移动端跨平台应用开发的热门解决方案。然而要在移动端充分利用Web技术的优势，仍然有许多障碍。

Crosswalk作为一款开源的web引擎，正是为了跨越这些障碍而生。目前Crosswalk正式支持的移动操作系统包括Android和Tizen，在Android 4.0及以上的系统中使用Crosswalk的Web应用程序在HTML5方面可以有一致的体验，同时和系统的整合交互方面(比如启动画面、权限管理、应用切换、社交分享等等)可以做到类似原生应用。

Crosswalk的核心是将Chrome的内核引入到应用当中，因此打包后的APK会增大，完整版每个平台增加20M，X86和arm两个平台就会增加40M。此外，低配置的手机在使用的时候可能会出现卡顿的情况。

## 2 CrossWalk实战
首先用Android Studio新建一个android应用。
### 2.1 在项目build.gradle中声明maven仓库
```
repositories {
        google()
        maven {
            url 'https://download.01.org/crosswalk/releases/crosswalk/android/maven2'
        }
        jcenter()
    }
```
### 2.2 在build.gradle中添加依赖
```
dependencies {
    implementation 'org.xwalk:xwalk_core_library:23.53.589.4'
}

```
添加完后dependencies里应该类似这样：
```
dependencies {

    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'com.android.support:appcompat-v7:26.1.0'
    implementation 'com.android.support.constraint:constraint-layout:1.1.0'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'com.android.support.test:runner:1.0.1'
    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.1'
    implementation 'org.xwalk:xwalk_core_library:23.53.589.4'
}

```
### 2.3 在build.greadle中添加productFlavors
productFlavors是在gradle中配置多渠道的打包的工具。我们利用productFlavors区分不同的产品（这里是arm和x86），定义不同的逻辑，使构建部分有差异的Android项目更加方便。

```
    productFlavors {
        armv7 {
            ndk {
                abiFilters "armeabi-v7a", ""
            }
            dimension "arm"
        }
        x86 {
            ndk {
                abiFilters "x86", ""
            }
            dimension "86"
        }
    }
```

### 2.4 在AndroidManifest.xml文件里添加应用权限
```
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
```

### 2.5 build一下，发现报错：
Manifest merger failed : uses-sdk:minSdkVersion 15 cannot be smaller than version 16 declared in library [org.xwalk:xwalk_core_library:23.53.589.4] /Users/sorahjy/.gradle/caches/transforms-1/files-1.1/xwalk_core_library-23.53.589.4.aar/4d63de3623aef3f8713541bd5435f90d/AndroidManifest.xml as the library might be using APIs not available in 15
	Suggestion: use a compatible library with a minSdk of at most 15,
		or increase this project's minSdk version to at least 16,
		or use tools:overrideLibrary="org.xwalk.core" to force usage (may lead to runtime failures)

### 2.6 解决兼容性问题
打开AndroidMainfest.xml，添加以下语句
```
xmlns:tools="http://schemas.android.com/tools"
```

```
<uses-sdk tools:overrideLibrary="org.xwalk.core" />
```
<uses-sdk> 用来描述该应用程序可以运行的最小和最大API级别，以及应用程序开发者设计期望运行的平台版本。通过在manifest清单文件中添加该属性，我们可以更好的控制应用在不同android系统版本上的安装和兼容性体验问题
。
最终我们的androidMainfest.xml文件应该像这样：

```xml
 <?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.sorahjy.crosswalk.ttttest">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>

```
### 2.7 修改布局文件
加入一个XWalkView
```
<org.xwalk.core.XWalkView
        android:id="@+id/xwalkWebView"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:orientation="vertical" />
```
### 2.8 最后敲一下MainActivity.java就可以啦
```
package com.sorahjy.crosswalk;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;

import org.xwalk.core.XWalkPreferences;
import org.xwalk.core.XWalkView;

public class MainActivity extends AppCompatActivity {
    private XWalkView xWalkWebView;
    //这个url就是你想要打开的页面。
    private String url="https://www.sorahjy.com";
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        xWalkWebView = findViewById(R.id.xwalkWebView);
        // turn on debugging
        XWalkPreferences.setValue(XWalkPreferences.REMOTE_DEBUGGING, true);
        xWalkWebView.load(url, null);
    }

    @Override
    protected void onPause() {
        super.onPause();
        if (xWalkWebView != null) {
            xWalkWebView.pauseTimers();
            xWalkWebView.onHide();
        }
    }

    @Override
    protected void onResume() {
        super.onResume();
        if (xWalkWebView != null) {
            xWalkWebView.resumeTimers();
            xWalkWebView.onShow();
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (xWalkWebView != null) {
            xWalkWebView.onDestroy();
        }
    }
}

```

## 3 本次的CrossWalk使用指南到底结束
感谢您的观看。
本次实例的源代码在我的github上，
网址为：https://github.com/sorahjy/crosswalk


作者 [@sorahjy][1]
我的github： https://github.com/sorahjy
2018 年 04月 22日

[1]: https://github.com/sorahjy







