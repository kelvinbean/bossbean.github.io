---
title: Array数组简单写法汇总
---

其实数组真的有很多妙用的方法来写很多，而且如果巧妙的使用起来，可以用简单的一句代码来实现一些比较复杂的功能。

就像一个简单的功能，不同的人写的代码，有的只要一行，但是有的人可能需要十行。

因此我们都尽量做到用最少的代码实现同样的功能。

这篇文章用来汇总，列举的都是我遇到觉得比较好的数组写法以及数组的一些相关知识。

以后只要有发现新的写法，会不断更新这篇文章。


### Array.apply(null, {length: 10})和Array(10)有什么区别?


```bash
    var arr1 = Array.apply(null, {length: 10});
    var arr2 = Array(10);
    0 in arr1  //true
    0 in arr2  //false
    arr1.map(function(item, index){console.log(index)}) //0, 1, 2, 3...
    arr2.map(function(item, index){console.log(index)}) //直接不打印，因为只是一个空数组。根本不循环
```

<!-- More -->

解释
首先Array(1,2,3,4)，你知道的吧,生成一个数组[1,2,3,4]
然后是apply的问题,要求第二个参数是一个数组
那么Array.apply(null,[1,2,3,4])生成的和上述的一样的[1,2,3,4]数组
但apply有个奇怪的地方,当第二个参数是一个带有length属性的对象时,会当成一个数组使用
所以Array.apply(null,{length:4})生成[undefined,undefined,undefined,undefined]
相当于Array.apply(null,[undefined,undefined,undefined,undefined])


所以如果要快速生成一个1-100的递增数组。以下是方法

```bash
var result = Array.apply(null,{length:100}).map(function(item,index){
    return index+1;
})
```
或者
```bash
var result = Array.apply(null,Array(100)).map(function(item,index){
    return index+1;
})
```


### 随机生成10个1-100的不重复随机数

```bash
    var arr = new Array(100)
    .fill(0)
    .map((v,i) => i+1)
    .sort(()=>0.5-Math.random())
    .filter((v,i) => i<10)
```
