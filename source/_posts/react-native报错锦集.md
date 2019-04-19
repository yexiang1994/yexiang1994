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
    project(':react-native-baidu-map').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-baidu-map/android')
    include ':app'
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
**解决方法：**adb reverse tcp:8081 tcp:8081  关闭usb调试再重新打开

------

2. 初始化运行红屏错误 unable to load script from asset/index.android.bundle
assets中没有成功打包出index.android.bundle文件
**解决方法:**1.去（你的项目文件夹）\android\app\src\main目录下新建asset文件夹
　　　　　2.终端下运行
　　　　　react-native bundle --platform android --dev false --entry-file index.js --bundle-output android/app/src/main/assets/index.android.bundle --assets-dest android/app/src/main/res
此命令运行不成功把"babel-preset-react-native": "^2.1.0",
的配置改一下
再次执行 react-native run-android
　　注：由于0.49版本以后的react-native没有index.android.js和index.ios.js文件，而统一合并成了index.js，所以使用0.49及以后版本的同学请将第2步中的入口文件改为index.js

------

3. the development server trturned response error code:500
**解决方法：**删除node包重新install，要用npm不能用cnpm

------

4. bundlin failed: error:unable to resolve module  'AccessibilityInfo' form '...node_modules/...react-native-implementation.js':Module' AccessibilityInfo does not exist in the Hast..
**解决方法：**这个是0.56的版本问题，换成0.55.4版本就行了

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
下载后， 高级版本不支持从 react中直接引入proptype
需要改成 import PropTypes from  ’prop-types’ 把所有react-native-baidu-map里面的js文件都改掉，
同时其android/src/main/java/org/lovebing/reactnative/baidumap/BiduMapPackage.java里面
注释掉@Override 
如果还有问题，就找到BaiduMapViewManager.java 文件，注释掉里面含public boolean的@Override

------

11. 重要一点，开发时候需要用到安卓的debug.keystore的密钥库的SHA1密码来生成百度地图的key，不然测试的百度地图一直时网格状,

项目的包名必须与key申请的包名一致
注意，只有在 **.android**目录下生产的秘钥签名才是百度地图生产key所需要的秘钥
而不是打包的时候在java  jdk目录下生产的秘钥

react-native-camera  集成
按照网上配置好集成环境，然后再按照
https://github.com/react-native-community/react-native-camera/issues/1530
文件修改下配置，
最后只能用RNCamera, 不能用Camera组件

------

12. a problem occourred evaluating project: 'react-native-fast-image'
Could not find method implementation() for arguments[com.facebook.react:react-native:+]
**解决办法：**把node_module下的fast-image里面的compile修改成implementation，高版本的gradle语法变了，

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

