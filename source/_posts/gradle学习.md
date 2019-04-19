---
title: gradle学习
tags: [gradle]
---
开发rn的时候发现android一直用的是gradle构建的，不明白其中意思所有就学一下
gradle是用groovy语言编写的
所以先学下groovy吧

先来看下普通初始化构建的gradle项目里面的文件
gradle构建脚本默认都有个project实例
```groovy
group 'com.test.gradle'

version '1.0-SNAPSHOT' // 为project里面的属性

apply plugin: 'java'
// projcet里面的apply的方法可以 即调用 apply(plugin:'java') 方法
apply plugin: 'war' // 在ide会自动生成war包

sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    //下面声明的是gradle插件的版本
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```

属性 group，name，version唯一确定一个项目
方法 apply，dependencies，repositories（指定仓库），task（声明项目有那些任务）
属性的其他配置方式：ext,gradle.properties

任务（rask）
任务对应org.gradle.api.Task 主要包括任务动作和任务依赖，任务动作定义了一个最小的工作单元，可以定义依赖于其他任务，动作序列和执行条件。
里面的方法有：
dependsOn声明任务依赖
doFirst (动作任务列表前面添加任务),doLast<<（动作列表后面添加任务）  

实现创建目录的任务
```groovy
def createDir={
    path ->
        File dir=new File(path)
        if(!dir.exists()){
            dir.mkdir();
        }
}
// 任务一
task makeJavaDir(){
    def paths = ['sec/main/java','src/main/resources','src/test/java'];
    doFirst{
        paths.forEach(createDir);
    }
}
// 任务一
task makeWebDir(){
    dependsOn('makeJavaDir')  // 声明此任务依赖makejavadir任务
    def paths=['src/main/webapp','src/test/webapp'];
    doLast {
        paths.forEach(createDir)
    }
}
```





