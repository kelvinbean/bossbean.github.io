---
title: 浏览器tab切换事件处理
---

![Image text](/images/banner/banner3.jpg)

有这么一种情况。就是一个网站，你先登录了A用户，之后你再在同一浏览器打开一个tab页，登录了B用户。
这时候再切换原先的tab页。这时候该页面显示的还是A用户的信息。这时候除非你刷新，否则信息就一直不会变，不会更新成B用户的信息。

<!-- More -->
其实很多的网站是没有处理这个问题，其实也可以理解。因为只要在原先的页面点击一些操作，或许就会自动变成B用户的信息。
但是如果是单页面的网站，这时候你每次的点击其实都不会去刷新页面。这时候我们就可以通过一个监听事件来监听页面时候切换到当前页。

登登登登，这个方法就是visibilitychange。


### visibilityState

在讲visibilitychange前，先介绍一下visibilityState属性

document对象上有一个叫visibilityState的属性。代表这该页面是处于当前选中状态还是被隐藏起来。

这个属性有两个值。
当前tab页被选中时，document.visibilityState 为 visible，否则为hidden

要注意的一点是，各个浏览器可能会有不同，可能需要添加浏览器前缀。
visibilitychange，visibilityState以及visibilityState的值都可能会有前缀的兼容问题。这里就不扩展了

### 使用方法

visibilitychange这个方法就是在tab切换的时候会触发的方法。

所以要解决一开始的问题，使用如下：

```bash
    document.addEventListener('visibilitychange',function(){
        if(document.visibilityState == 'visible'){//当切到当前tab页时，
            if(判断登录的信息是否一致){
               location.reload(); //当如果不一致就刷新页面。拿最新数据。
            }
        }
    });
```

大致的思路就是这样。信息判断可以通过各自tab页面的内存变量与浏览器的本地存储的数据进行比较。
