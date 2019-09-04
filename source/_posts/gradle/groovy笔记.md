---
title: groovy学习
tags: [groovy]
---
groovy高效特性
1 可选的类型定义，变量类型可以不用定义，系统自动识别
def version = 1

2 assert 断言

3 括号是可选的
println(version)  等于 println version

4 字符串，有三种类型
```groovy
def s1='ab' // 仅仅是个字符串
def s2="bc ${version}" // 可以插入变量
def s3='''ml
sda'''  // 可以换行
```

5 集合api

```groovy
// list 类型
def tool=['ant','maven']
tool << 'gradle'  // 追加一个类型

// map
def bui =['vue': 100,'react': 35]
bui.jq=11
// 获取方法
hui.jq
hui['vue']
```
6 闭包
```groovy
def c1={
    v->
    print v
}
def c2={
    print 'htll'
}
def met1(Closure closure){
    cloure('param')
}
def met2(Closure closure){
    cloure()
}
met1(c1)
met2(c2)
```
