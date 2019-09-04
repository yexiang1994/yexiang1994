---
title: hook学习
tags: [react]
---

## 一些初始化
使用react class方法初始化时，比如请求数据之间在componentDidMount里面操作，这里只执行一次，但是在hooks里面useEffect是在每次渲染的时候都会执行， 因为重新渲染，都会生成新的 effect，我们在useEffect里面请求数据，并复制给useState就会造成无限循环，这时给useEffect的第二个参数 数组里面添加一个false，
```js
useEffect(()=>{
  query()
}, [false])
```
如果useEffect需要响应多次执行，则可以设置第二个参数，这样参数改变后useEffect也会重新执行
```js
useEffect(()=>{
  query()
}, [params])
```
