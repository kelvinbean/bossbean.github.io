---
title: 如何使用axios来处理地狱回调
---

axios是一个类似于ajax的工具。可以实现与后端数据交互。而且现在在vue官方也推荐使用axios来进行与后端的数据传递与交互。

Axios 是一个基于 promise 的 HTTP 库，可以用在浏览器和 node.js中。

其实axios的使用非常简单。很多的文档，以及一些官方文档已经有很多详细的讲解。[文档地址](https://www.npmjs.com/package/axios])。我这里就不赘述了。这里我主要讲axios的小小的使用技巧。

<!-- More -->

### axios的好处

我个人认为axios的好处，第一个可以处理ajax的地狱回调问题。

第二个就是可以多个请求同时发起，之后同时处理数据。如使用axios.all。

我主要讲下axios是怎么处理地狱回调的。

### 地狱回调的处理

所谓地狱回调，就用下面的代码让你自行体会。

下面用jquery的ajax来举个例子

```bash
    $.get('/boss',function(result){
        $.get('/bean',{bossName:result.name},function(result2){
            ...
        })
    });

```

这种代码一直写下去，就形成了我所形容的地狱回调。首先地狱回调会影响代码的阅读，而且容易出错，代码显得也不那么整洁。

那如何处理axios按顺序等待上个请求完成以后再去执行对应的下个请求呢？

其实很简单。

一般的axios是这样的

```bash
    axios.get('/boss')
    .then(res => {
        //处理数据
    })
```

但是要继续传递下去，那就返回一个pomise对象。因为本身axios处理请求返回的就是promise对象。
然后通过promise对象的then方法去处理回调方法。只是如果你需要一个axios的请求，那么就能有继续传递下去。
就像jquery的连缀语法一样。所以就是下面那么写。

then的方法里最好也使用es6的写法来写function,这样的话this的指向就不用再去声明。（就不用在外部写var that = this;）


那么一开始那个地狱回调就可以这么写

```bash
    axios.get('/boss')
    .then(res => {
        return axios.get('/bean',{
             {bossName:result.name}
        });
    }).then(res => {
        ...
    });
```

这样的话，就方法看似都在同一层级上，从上往下执行。非常清晰。

还有另外一种方法。

### async和await

```bash
    async function test(){
        var response = await axios.get('/boss');
        //处理方法
        if(response.code == 'success'){

        }
    }     

```

这种方法其实，使用的es6的语法。不需要再把方法放到回调函数里去等待请求完成。

可以直接写在外部。而且会等待await执行结束，才会执行接下去的代码。这种写法也非常直观。但是要求在async函数下。

基本规则是：

    async 表示这是一个async函数，await只能用在这个函数里面。
    await 表示在这里等待promise返回结果了，再继续执行。
    await 后面跟着的应该是一个promise对象（当然，其他返回值也没关系，只是会立即执行，不过那样就没有意义了…）

详细具体的说明可以自行搜索哦。