---
title: rn数据存储
tags: [react-native]
---

Android提供了多种选择来保存永久性的数据，根据不同的需求来使用不同的保存方式，一般情况下，保存数据的方式有下面几种
## SharedPreferences
（以下简称SP）以键值对形式进行存储，数据以xml形式存储在/data/data/项目包名/shared_prefs/xml.xml中。一般来说，SP只能存储基本类型的数据，如：布尔类型、浮点型、整形及字符串，默认情况下，SP保存的文件是应用的私有文件，其他应用（和用户）不能访问这些文件，
## 内部存储
一般是存在/data/data/项目包名/files/目录下的
File fileDire = getFilesDir(); 获取存储内部文件的文件系统目录的绝对路径/data/data/项目包名/files/
这层目录普通用户和程序一般无法直接访问，当前应用的文件也只能当前应用使用，其他应用无法访问。但是手机经过root之后，获取到了超级权限的话就可以看到这些数据了，获取到root权限的应用可以访问这些数据，并进行操作。但是其他未获得root权限的应用程序依旧无法访问，因此，在内部存储中的文件是最安全的。
卸载应用时，系统会自动清除掉这些文件。
## 外部存储
这部分文件是暴露在文件分区的，文件管理器能看到的文件，该存储可能是可移除的存储介质（例如 SD 卡）或内部（不可移除）存储。 保存到外部存储的文件是全局可读取文件。存储分为两种，一种是应用卸载后，存储数据也会被删除，一种永久存储，即使应用被卸载，存储的数据依然存在
第一种 是应用数据目录
context.getExternalFilesDir(null).getPath()可以获取到外部存储的路径
是/storage/emulated/0/Android/data/package_name/  这种是应用删除后数据就没了
低版本需要加上权限
```xml
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
                     android:maxSdkVersion="18">
```
第二种 非应用数据目录,这种是永久存储，即使应用被卸载，存储的数据依然存在
```java
($rootDir)
+- /data                -> Environment.getDataDirectory()
|   |
|   |   ($appDataDir)
|   +- data/com.srain.cube.sample
|       |
|       |   ($filesDir)
|       +- files            -> Context.getFilesDir() / Context.getFileStreamPath("")
|       |       |
|       |       +- file1    -> Context.getFileStreamPath("file1")
|       |   ($cacheDir)
|       +- cache            -> Context.getCacheDir()
|       |
|       +- app_$name        ->(Context.getDir(String name, int mode)
|
|   ($rootDir)
+- /storage/sdcard0     -> Environment.getExternalStorageDirectory()
    |                       / Environment.getExternalStoragePublicDirectory("")
    |
    +- dir1             -> Environment.getExternalStoragePublicDirectory("dir1")
    |
    |   ($appDataDir)
    +- Andorid/data/com.srain.cube.sample
        |
        |   ($filesDir)
        +- files        -> Context.getExternalFilesDir("")
        |   |
        |   +- file1    -> Context.getExternalFilesDir("file1")
        |   +- Music    -> Context.getExternalFilesDir(Environment.Music);
        |   +- Picture  -> ... Environment.Picture
        |   +- ...
        |
        |   ($cacheDir)
        +- cache        -> Context.getExternalCacheDir()
        |
        +- ???
```
这个结构清晰的描述了Android中存储的分类、路径以及访问方式。












