---
title: dart基础知识
tags: [dart]
---
## 声明
1. var声明的时候，如果没有初始值，后面可以复制任意类型。如果有初始值，则类型被锁定
2. 使用强类型声明的时候，类型被锁定 
```dart
// 如
String a = 1;
a="aa";  //error
```
3.字符串模板
 ```dart
String str = "op";
String s1 = '$str 组合'；  // "op 组合"
String s2 = '一个大写${str.toUpperCase()}'; // "一个大写OP"
```
三个单引号可以断行
```dart
var s3 = '''
a 
sd
'''
```
4.数组list
```dart
var arr1 = ["t","c","d"];  // 字面量
var arr2 = List.of([1,2,3]);  // 构造函数
arr2.add(100);
arr2.forEach((v)=>{});
var map = {"a", "b","c"}; 
var map2 = new Map();
map2['name'] = 'jack';
map2['sex']='fame';
```
5. 私有变量 
var _a = "aa";
 一切变量初始值都是null， 包括bool
6. 时间
var now = DateTime.now(); // 2019-07-31 16:00:00
var d = DateTime(2019,7,31,21,55);
now,add(Duration(minutes: 5));  // 加5min
d.isAfter(now); // 比较时间先后
7. Object 和 dynamic
```dart
Object d2；
d2 = "ss";
d2.test(); // 执行不存在的方法时 编译的时候报错

dynamic d1;
d1="aa";
d1.test(); // 执行不存在的方法时，运行时报错
```
8. final  const
final,const生命时可以不指定类型，他会自动识别，之后值的类型不能更改
9.级联运算符
```dart
var lists = List<int>();
lists
..add(1)
..add(2)
..addAll([1,2,3]);
```
10.set
```dart
var set1 = {"li","zhang"};
 var set2={"li","wang","ye"};
  var dif1 = set1.difference(set2);
  print(dif1); // {zhang}
  var dif2 = set2.difference(set1);
  print(dif2); // {wang, ye}
// 交集
  print('${set1.intersection(set2)}');  // {li}
  // 并集
  print('${set1.union(set2)}');  //{li, zhang, wang, ye}
```
11. ??=
```dart
int add({int x, int y}){
   x ??=1;  // 如果x值空则赋值，
   y ??= 2;
   return x+y;
 }
print(add(x:2));  //4
```
12. 重定向构造函数
用于初始化的时候
```dart
class Point{
  num x,y;
  Point(this.x,this.y);
  Point.ar(num x):this(x,0);
  Point.ui(num y):this.ar(y);
}
main() {
  Point p = Point.ui(2);
  print(p.x); // 2
  print(p.y); // 0
}
```
13 dart的mixin
Mixin 是复用类代码的一种途径， 复用的类可以在不同层级，之间可以不存在继承关系。
通过 with 后面跟一个或多个混入的名称，来 使用 Mixin
```dart
class A{
  a(){
    print("a.a");
  }
}
class B{
  a(){
    print("b.a");
  }
  b(){
    print("b.b");
  }
}
class Point with B,A{
  
}
main() {
  Point p = Point();
  p.a();  // a.a
  p.b();   // b.b
}
// 这里混合的时候后面的相同名称的方法覆盖前面的，

//和extends的一起用的时候也是后面方法会覆盖前面的
class C {
  a(){
    print("c.a");
  }
  b(){
    print("c.b");
  }
  c(){
    print("c.c");
  }
}

class Point extends C with B,A{
}

main() {
  Point p = Point();
  p.a();   // a.a
  p.b();  //b.b
  p.c();      //c.c
}
// 但是当前方法里面的函数又会覆盖其他的
class Point extends C with B,A{
  @override
  c() {
    print("p.c");
  }
}

main() {
  Point p = Point();
  p.a();  // a.a
  p.b(); //b.b
  p.c();   //p.c    
}

```
14 异步
函数把自己放到队列中，返回一个Future对象，
获取Future中的值又两种方法
1.async和await 
2.使用Future接口
异步的函数使用的时候都要使用await，而await只能在带有async声明的函数中使用
```dart

Future<void> getName() async{
  await getStr();
  print("1");
}
getStr(){
  print("str1");
}
getStr2(){
  print("str2");
}
main() {
  getName();
  getStr2();
// str1 
// str2
// 1
}
// async里面的函数只会执行第一个await，后面会方式时间队列里面去，等到main函数里面的同步任务都执行完了，在去队列里面那到事件在去执行，所以这里为啥是看到的1在后面
```
```dart
getStr3(){
  print("str3");
}
getStr4(){
  print("str4");
}
getName() async{
  await getStr();
  print("1");
  await getStr3();
  await getStr4();
}
main() {
  getName();
  getStr2();
}![dartevent.png](https://upload-images.jianshu.io/upload_images/14227413-1a7c984d6a7ddc0a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

// str1 
// str2
//1
// str3
// str4
// Future只会执行第一个await，后面会放到事件队列里面去
```
```dart
Future(()=>2).then((v)=>v*3).then((v)=>print(v));  // 6
//Future执行完有个then可以回调

Future.delayed(const Duration(seconds: 5), () {
    print("delayed");
});  //五秒钟后执行
```

1.被添加到then里面的方法，会在future执行后立马执行（这个方法没有被加入到任何队列里面，只是被回调了）
2.如果在then调用之前，Future已经执行完毕，那么就有会有个任务 会被加入到microtask里面，这个任务就是被传入then里面的方法
```dart
//比如
var fn = Future(()=>2).then((v)=>v*2);
...
...
// 中间还有很多方法
fn.then((v)=>print(v));  //这里的then会被放到微任务里面，优先执行
```
3.Future和Future.delayed 构造方法并不会立刻完成
4.Future.value() 会在微任务中完成

Future捕获异常
```dart
Future(()=>1)
  .then(print)
  .then((_)=>Future.error("error"))
  .whenComplete(()=>print("finish"))
  .catchError(print);
// 处理错误
.catchError(print, test:(Object o){
  print(o);
return ture; // 这里返回true 表面错误被处理了，
});
```

一个dart又一个消息循环和两个消息队列
1.Event（事件）队列：i/o， 鼠标事件，键盘事件，绘制事件，定时器，isolate之间的message
2.microtask （微任务）队列：只包含当前的isolate隔离的代码
而microtask 优先级比event优先级高
执行顺序如下图

![dartevent.png](https://upload-images.jianshu.io/upload_images/14227413-507f8aa433b204db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

先从执行函数里面的所有同步方法，然后有微任务执行微任务，微任务里面有event事件的把event事件放到事件队列里面去，等待微任务执行完再去执行event事件，然后依次循环，直到所有事件全部执行完毕

scheduleMicrotask 添加一个微任务

练习
```dart
testFu(){
  Future f = new Future(()=>print("1"));
  Future f1 = new Future(()=>null);
  Future f2 = new Future(()=>null);
  Future f3 = new Future(()=>null);
  f3.then((_)=>print("2"));
  f2.then((_){
    print("3");
    new Future(()=>print("4"));
    f1.then((_){
      print("5");
    });
  });
  f1.then((m){
    print("6");
  });
  print("7");
}
main(List<String> args) {
  testFu();
}
```
分析：按照上表的，先执行main函数里面的方法，所以 7优先打印，new Future都放到了事件队列里面，然后按照从上到下的执行顺序， f1优先被执行，执行后直接直接then方法，不带间隔的，所以接下来是 打印6，然后是f2函数执行，打印了3.因为f1之前已经执行了，他的then被放到微任务里面了，所以这里3打印后会直接执行f1.then, 然后打印5，此时f2里面的新Future又被放到的时间队列的最后面，要等f3执行完成后才能轮到他执行，所以结果是 7163524

```dart
import "dart:async";
testFu(){
  scheduleMicrotask(()=>print("1"));
  var f1 = new Future.delayed(new Duration(seconds: 1), ()=>print("2"));
  var f2 = new Future(()=>print("3")).then((_){
    print("4");
    scheduleMicrotask(()=>print("5"));
  }).then((_)=>print("6"));
  var f3 = new Future(()=>print("7"));
  scheduleMicrotask(()=>print("8"));
  print("9");
}
main(List<String> args) {
  testFu();
}
```
分析：先执行函数里面的同步方法，所以先打印9，然后里面添加了两个微任务，主函数方法执行完后就去找微任务队列，一次打印1，8，然后执行事件队列里面的Future方法，f1延迟1秒，所以放在了最后，f2打印3,4，然后里面又创建了个微任务，把微任务扔进微任务队列，继续执行then，所以是6，这里的事件执行完后就把微任务拿出来执行完，打印5，然后这个事件函数里面的所有方法执行完了，然后在执行下一个事件队列里的函数


