---
title: react-native报错锦集
tags: [react-native]
---

下面是一些rn问题的分类：
> 1.rn的版本问题，如0.55.6版本就是这样的，安装的时候就会报错，
> 2.link组件的问题，有些时候会发生自动link不全或者路径错误的情况，
> 3.安卓版本问题，组件的版本和rn项目版本统一
> 4.其他问题，

其实我建议还是学一下android原生，包括gradle，不然后面涉及太多原生问题很难解决。

## 1.rn版本问题 ##

    rn版本更新的很快，有些版本说实话根本用不了，比如0.55.6，都启动不了，
    我推荐几个我用过的版本，** 0.55.4 ， 0.57.0**。 这两个相对来说还算稳定。

## 2.react-native link  xxx   问题 ##
link的时候很多时候会出现不全的问题，这也是很多人害怕的问题，尤其一开始做这个项目的时候，项目写的好好的一link就出错，网上半天找不到解决方案。其实主要把link这三个地方注意些全就基本没问题了。以react-native-baidu-map组件为例

1. android/settings.gradle里面

```java
    rootProject.name = 'apps'
    include ':react-native-baidu-map'
    project(':react-native-baidu-map').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-baidu-map/android');
    include ':app';
```

2. android/app/build.gradle里面

```java
dependencies {
    implementation project(':react-native-baidu-map')
}
```

3. android/app/src/main/java/../MainApplication.java

```java
@Override
protected List<ReactPackage> getPackages() {
    return Arrays.<ReactPackage>asList(
        new MainReactPackage(),
        new BaiduMapPackage(getApplicationContext()),
    );
}
```

## 3.android版本问题 ##
这是个大问题，我现在用的androidsdk28的，rn一开始建议的是23，其实这个已经是很老的版本的，很多组件都是在新版本上构建的
如果组件里面的版本大多不统一，那么最好就全部改成最高的版本，android向下兼容一般改成高版本没什么大问题，
有一个经常遇到的一个语法问题，在link的时候低版本  implementation要改成compile，

```java
dependencies {
    implementation project(':react-native-baidu-map')
}
```

## 4.其他问题 ##
其他问题么就不多说了，各种各样的都有，

接下来就说下其他的报错案例

1. Execution failed for task ':app:installDebug'.com.android.builder.testing.api.DeviceException: com.android.ddmlib.InstallException: device '860BDMP22B63' not found
**解决方法：** adb reverse tcp:8081 tcp:8081  关闭usb调试再重新打开

------

2. 初始化运行红屏错误 unable to load script from asset/index.android.bundle
assets中没有成功打包出index.android.bundle文件
**解决方法:** 1.去（你的项目文件夹）\android\app\src\main目录下新建asset文件夹
　　　　　2.终端下运行
　　　　　react-native bundle --platform android --dev false --entry-file index.js --bundle-output android/app/src/main/assets/index.android.bundle --assets-dest android/app/src/main/res
此命令运行不成功把"babel-preset-react-native": "^2.1.0",
的配置改一下
再次执行 react-native run-android
　　注：由于0.49版本以后的react-native没有index.android.js和index.ios.js文件，而统一合并成了index.js，所以使用0.49及以后版本的同学请将第2步中的入口文件改为index.js

------

3. the development server trturned response error code:500
**解决方法：** 删除node包重新install，要用npm不能用cnpm

------

4. bundlin failed: error:unable to resolve module  'AccessibilityInfo' form '...node_modules/...react-native-implementation.js':Module' AccessibilityInfo does not exist in the Hast..
**解决方法：** 这个是0.56的版本问题，换成0.55.4版本就行了

------

5. 运行断开，有时候会出现运行成功了但是又断开了，直接新运行就好了

------

6. 使用AsyncStorage  debug不出来数据，也就是本地存储失效了，需要取消debug模式，这和react-native运行模式有关

------

7. Error(1,0) Your project path contains non-ASCII characters,This will most likely cause the
    build to fail on Windows,
**解决方法：** 项目不能有中文路径

------

8. 引用react-native-camera保存问题，rn0.55.4版本
**解决问题的地址：** [git连接](https://github.com/react-native-community/react-native-camera/issues/1530)

------

9. com.android.builder.testing.api.DeviceException:com.android.ddmlib.InstallException:Failed to finalize pession:INSTALL_FAILED_UPDATE_INCOMPATIBLE: Package com.app signatures do not match the prevously installed version:ignoring!
**解决方法：** 创建项目名的时候不能重名，就算之前删除了，除非彻底关闭所有的相关运行程序，还有一中情况是安装了打包后的app没有卸载，在运行项目就不行

------

10. react-native-baidu-map插件用法
下载后， 高级版本不支持从 react中直接引入proptype需要改成
```js
import PropTypes from 'prop-types';
```
把所有**react-native-baidu-map**里面的js文件都改掉，同时其android/src/main/java/org/lovebing/reactnative/baidumap/BiduMapPackage.java里面注释掉@Override如果还有问题，就找到BaiduMapViewManager.java 文件，注释掉里面含public boolean的@Override

------

11. 重要一点，开发时候需要用到安卓的debug.keystore的密钥库的SHA1密码来生成百度地图的key，不然测试的百度地图一直时网格状,

项目的包名必须与key申请的包名一致
注意，只有在 **.android** 目录下生产的秘钥签名才是百度地图生产key所需要的秘钥
而不是打包的时候在java  jdk目录下生产的秘钥

react-native-camera  集成
按照网上配置好集成环境，然后再按照
https://github.com/react-native-community/react-native-camera/issues/1530
文件修改下配置，
最后只能用RNCamera, 不能用Camera组件

------

12. a problem occourred evaluating project: 'react-native-fast-image'
Could not find method implementation() for arguments[com.facebook.react:react-native:+]
**解决办法：** 把node_module下的fast-image里面的compile修改成implementation，高版本的gradle语法变了，

------

13. 低版本rn有长屏幕手机撑不开的问题，需要加
```html
<meta-data android:name="android.max_aspect" android:value="2.2"/>
```

------

14. 高版本手机请求不到数据，因为高版本8.0以上手机默认支持https请求，
**解决办法：** 在AndroidManifest.xml文件里面添加android:usesCleartextTraffic="true"

------

15. ./gradlew assembleRelease的时候的错误
* What went wrong:
Execution failed for task ':app:generateReleaseBuildConfig'.
> java.nio.file.DirectoryNotEmptyException: D:\work\2018\08-03\water\App\apps\android\app\build\generated\source\buildConfig\release\com

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output. Run with --scan to get full insights.

BUILD FAILED in 2s
11 actionable tasks: 1 executed, 10 up-to-date

要重新清理下文件， ./gradlew clean

------

16. 调用的百度地图一直是网格状，显示不出来
这里需要注意这几个点
* 1：调试的时候只能用 debug.keystore,
* 2：正式打包需要用 在./android 目录下的（一般是在c盘的appdada里面）生成的keystore才行，而不是在java sdk目录下生成的keystore，这两个有区别。

------

17. A problem occurred evaluating project ':react-native-file-selector'.
> Could not find method google() for arguments [] on repository container.

解决办法： gradle-wrapper.properties  里面的distributionUrl = https\://services.gradle.org/distributions/gradle-4.1-all.zip

------

18. Execution failed for task ':app:preDebugBuild'.
> Android dependency 'com.android.support:support-fragment' has different version for the compile (26.1.0) and runtime (27.1.1) classpath. You should manually set the same version via DependencyResolution

解决办法： 我所用的没有support-fragment这个依赖，所以应该是build冲突，清除build文件，./gradlew clean后就好了

------

19. > Task :react-native-fast-image:compileDebugJavaWithJavac
注: [2] Wrote GeneratedAppGlideModule with: [com.bumptech.glide.integration.okhttp3.OkHttpLibraryGlideModule, com.dylanvann.fastimage.FastImageOkHttpProgressGlideModule]

解决办法删了他吧react-native-fast-image ，换个版本吧，最新版本有问题啊，我之前用的5.1.0

------

20. 打包的时候可能出现的问题 node_modules_reactnavigation_src_views_assets_backicon.png, but the error uncompiled PNG file passed as argument. Must be compiled first into .flat file.. error: failed parsing overlays.
在gradle.properties文件中添加以下行：android.enableAapt2 = false，让他使用旧的aapt方式来获取资源打包，虽然不推荐，我用的是0.57.0的版本，一开始打包是没有这个问题的，不知道后面改了android哪里的资源导致的，因为改的东西太多了，没办法。我建议还是升级下app版本，把项目移植过去，以免后面继续出现其他问题。

------

21. Exception in thread "main" javax.net.ssl.SSLException: Connection has been shutdown: javax.net.ssl.SSLException: SSL peer shut down incorrectly

网的问题，下载不到distributionUrl=https\://services.gradle.org/distributions/gradle-4.10.2-all.zip文件，可以陪下镜像阿里云
```groovy
buildscript {
    repositories {
        google()
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
        jcenter()
    }
}

allprojects {
    repositories {
        mavenLocal()
        google()
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
        jcenter()
        maven {
            url "$rootDir/../node_modules/react-native/android"
        }
    }
}
```
配置了这个也没有，翻墙也下载不下来的话去官网下载吧，
[gradle地址](https://services.gradle.org/distributions/)

------

22. Error: Unable to resolve module `./index` from `\node_modules\react-native\scripts/.`: The module `./index` could not be found from `\node_modules\react-native\scripts/.`. Indeed, none of these files exist:
解决方法： 这个是用了0.59.0-0.59.2版本的问题，换成0.59.4就可以了

------

23. Error: null, Cannot fit requested classes in a single dex file (# methods: 71853 > 65536)
这个是由于android 的64K 引用限制，具体参照[原文](https://developer.android.com/studio/build/multidex)
在gradle里面添加multiDexEnabled true 就行了
```java
android {
    defaultConfig {
        ...
        minSdkVersion 21
        targetSdkVersion 28
        multiDexEnabled true
    }
    ...
}

dependencies {
  implementation 'com.android.support:multidex:1.0.3'
}
```

添加这个后项目高版本手机可以运行了，但是低版本的android就会crash，起都起不来，会报错主类加载不到。我用到的是**4.4.4**版本的手机，文档写的是**android5.0之前**的手机不行。然后解决办法是， 主类继承**MultiDexApplication** ，而不是继承Application类，文档里面还写了引用的时候换成android:name="android.support.multidex.MultiDexApplication" ，但是换了后却是无法使用，所有我这里没换，是可以使用的；还有问题可以参照[连接](https://developer.android.com/studio/build/multidex)，仔细看下。

------

24. android 底版本沉浸栏不起效，
解决办法：  正常安装rn官网的沉浸栏设置，高版本是可以实现沉浸栏的，但是我的测试手机4.4版本不行，沉浸栏还是原来的，需要在 android/app/src/main/res/styles.xml 里面的
```xml
<style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
```
里面添加
```xml
<item name="android:windowTranslucentStatus">true</item>
```
------

25. Execution failed for task ':app:transformDexArchiveWithExternalLibsDexMergerForDebug'.
> com.android.builder.dexing.DexArchiveMergerException: Error while merging dex archives:
The number of method references in a .dex file cannot exceed 64K.
Learn how to resolve this issue at https://developer.android.com/tools/building/multidex.html

这个问题是android引用第三方库方法超过了64k大小的限制，
解决办法： 在build.gradle的 defaultConfig里面添加 **multiDexEnabled true** 设置下就行了，
但是设置后又会出现低版本不兼容的情况，我的是4.4.4版本的android，问题如下

------

26. Process: com.socket0594, PID: 27759
java.lang.RuntimeException: Unable to get provider android.support.v4.content.FileProvider: java.lang.ClassNotFoundException: Didn't find class "android.support.v4.content.FileProvider" on path: DexPathList[[zip file "/mnt/asec/com.socket0594-2/pkg.apk"],nativeLibraryDirectories=[/mnt/asec/com.socket0594-2/lib, /vendor/lib, /system/lib, /data/datalib]]
这里需要android的support.v4 库来做兼容，support.v4库一般rn都引入了没有引入的这样引入下
```groovy
// \android\app\build.gradle
implementation 'com.android.support:support-v4:28.0.0'
```
然后在 main/java/xxx  目录下的 MainApplication.java 里面修改MainApplication的继承类
```java
import android.support.multidex.MultiDexApplication;

public class MainApplication extends MultiDexApplication implements ReactApplication {
    ...
}
// 之前是这样的
//public class MainApplication extends Application implements ReactApplication {
    //...
// }
```
------

27. rn使用StatusBar无效，正常来说不会无效的，我按照其他的的方法设置了
```xml
//在 android\app\src\main\res\values\styles.xml 里面
<style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
       <!-- <item name="android:windowTranslucentStatus">true</item> -->
  </style>  
```
这个的意思其实是设置沉浸栏的状态，是否是半透明的，设置了这个windowTranslucentStatus就会导致StatusBar无效，网上的其他人的方法谨慎参考啊

28. 安装react-native-image-crop-picker 插件的时候Could not find com.github.yalantis:ucrop:2.2.1-native，因为缺少maven源， 
解决方法：/android/build.gradle 下
```groovy
allprojects {
    repositories {
        mavenLocal()
        google()
        jcenter()
        maven {
            // All of React Native (JS, Obj-C sources, Android binaries) is installed from npm
            url "$rootDir/../node_modules/react-native/android"
        }
        maven{
            url "https://jitpack.io"
        }
        maven{
            url 'https://maven.google.com/'
            name 'Google'
        }
    }
}
```
------

29. 安装最新版的react-native-baidu-map时候出现的问题
InnerClass annotations are missing corresponding EnclosingMember annotations. Such InnerClass annotations are ignored.
Message{kind=WARNING, text=InnerClass annotations are missing corresponding EnclosingMember annotations. Such InnerClass annotations are ignored., sources=[Unknown source file], tool name=Optional.of(D8)}

在app.gradle里面添加
```groovy
android{
    ...
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}
```
