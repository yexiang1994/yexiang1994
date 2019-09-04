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
* 常用仓库
mavenLocal (本地仓库)/mavenCentral（远程仓库）/jcenter
依赖的传递性
b依赖a，如果c依赖b，那么c就依赖a

* 依赖阶段配置
compile编译阶段 runtime运行阶段
testCompile  testRuntime


* 解决冲突
    查看依赖报告
    排除传递性依赖
    强制一个版本
* 修改默认解决策略
```groovy
    //发现版本冲突就使他构建失败,可以清楚的看到版本冲突
    configurations.all{
        resolutionStrategy{
            failOnVersionConflict()
        }
    }
```
* 排除传递性依赖
```groovy
// 把hibernate里面依赖的slf4j的包给排除掉，引用本项目的
compile("org.hibernate:hibernate-core:3.6.3.Final"){
    exclude group: "org.slf4j", module:"slf4j-api"
}
```
* 强制指定一个版本
```groovy
// 强制指定一个最高版本的slf4j
configurations.all{
    resolutionStrategy{
        force 'org.slf4j:slf4j-api:1.7.24'
    }
}
```

* 本地组件互相依赖的配置
```groovy
dependencies {
    implementation project(":todoItem")  // todoItem是项目的最后一个名词如com.app.todoItem
}
```

在settings.gradle 里面引入所有的子项目
rootProject.name='todo'
include 'todoItem'
include 'domain'

* 统一配置java插件
```groovy
allprojects {
    // 所有项目都用到了java插件，和指定的java1.8版本，子项目里面的build.gradle里面就可以删除这两句话，而在主项目里面统一配置
    apply plugin: 'java' 
    sourceCompatibility = 1.8
}
```

* 统一添加日志组件
```groovy
subprojects {
    // 把repositories和dependencies加载的资源合并在subprojects方法里面，
    // 供子组件一起使用，子组件就不需要在配置这些东西了
    repositories {
        mavenCentral()
    }
    dependencies {
        implementation 'org.hibernate:hibernate-core:3.6.3.Final'
        implementation 'org.log4s:log4s-testing_sjs0.6_2.13.0-M5:1.7.0'
        testRuntime group: 'junit', name: 'junit', version: '4.12'
    }
}
```

* 统一配置group和version
再根目录里面创建gradle.properties文件，在文件里面提出
group 'com.todo'
version '1.0-SNAPSHOT'
全部组件就都是这个名称和版本了

* 发布
```groovy
apply plugin： 'maven-public' //发布插件
publishing{
    publications {
        // 定义函数名称，然后发布执行
        myPublish(MavenPublication){
            from components.java
        }
    }
    repositories{
        maven{
            // 发布的名称和地址
            name: "myrepo"
            url: "
        }
    }
}
```

<!-- 如果子集引用了私人的仓库，在最外层的项目中需要配置仓库地址，即使子集已经配置了。 -->