---
title: js笔记
tags: [js]
---
## 数组的reduce
reduce()方法接收一个函数callback作为累加器，数组中的每个值（从左到右）开始合并，最终为一个值。
array.reduce(callback,[initialValue])
function callback(preValue,curValue,index,array){}
preValue: 上一次调用回调返回的值，或者是提供的初始值（initialValue）
curValue: 数组中当前被处理的数组项
index: 当前数组项在数组中的索引值
array: 调用 reduce()方法的数组
```js
var ar=[-1,2,5,-2]
let obj = ar.reduce(function(o,b,c,d){
    console.log(o, b, c, d)
    return b+o
},10)
console.log(obj)
```
![](http://thdqn.cn:8081/imagePath/1557043634134.jpg)

```js
var ar=[-1,2,5,-2]
let obj = ar.reduce(function(o,b,c,d){
    console.log(o, b, c, d)
    return b+o
})
console.log(obj)
```
![](http://thdqn.cn:8081/imagePath/1557043782103.jpg)

## reduceRight
reduceRight和reduce功能一样但是他是从右到左累加
```js
var ar=[-1,2,5,-2]
let obj = ar.reduceRight(function(o,b,c,d){
    console.log(o, b, c, d)
    return b+o
})
console.log(obj)
```
![](http://thdqn.cn:8081/imagePath/1557043903212.jpg)

## filter
filter() 方法创建一个新的数组，新数组中的元素是通过检查指定数组中符合条件的所有元素。
注意： filter() 不会对空数组进行检测。
注意： filter() 不会改变原始数组。
array.filter(function(currentValue,index,arr), thisValue)
currentValue	必须。当前元素的值
index	可选。当前元素的索引值
arr	可选。当前元素属于的数组对象
thisValue  可选。对象作为该执行回调时使用，传递给函数，用作 "this" 的值。如果省略了 thisValue ，"this" 的值为 "undefined"
```js
const students = [{
    name: '张三',
    score: 73,
    sex: 'male',
}, {
    name: '刘丽',
    score: 62,
    sex: 'female'
}, {
    name: '李四',
    score: 93,
    sex: 'male'
}, {
    name: '王五',
    score: 100,
    sex: 'male'
}];
const isExcellent = student => student.score > 80;
const excellentStudentNames = students => students.filter(isExcellent)
```

![](http://thdqn.cn:8081/imagePath/1557048938071.jpg)

## splice
删除数组里面的元素
let a = [1,2,3];
let b = a.splice(2,1)
删除数组里面的第2个的第一位起的元素;  结果a为[1,2]， b为[3]
