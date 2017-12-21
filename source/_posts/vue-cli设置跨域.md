---
title: vue-cli设置跨域
---

在开发vue项目的时候，在开发项目的时候，启动的本地服务器要与后端人员进行联调的时候就会出现跨域问题。
但是实际正式项目中并不用跨域，所以这里要进行开发环境的代理设置。

<!-- More -->
### 方法1

vue-cli项目下，

修改 config/index.js

```
    module.exports = {
        ...
        dev:{

           ...

            proxyTable: {//加上这项来设置代理
                '/bean':{
                    target: 'http://www.bossbean.com/', //目标地址
                    changeOrigin: true, //必须设置为true
                    pathRewrite: { //一些需要替换的接口，没有需要替换的就不用写
                      '^/list': '/bean_list'
                    }
                }
            }
        }
    }
```

加上之后再重新启动服务，这样就完美解决跨域的问题。

其实原理是利用了[http-proxy-middleware](https://github.com/chimurai/http-proxy-middleware)来做了一个路由的映射，这个在vue-cli脚手架里已经封装进去了，所以只需要在config/index.js上修改即可达到效果。

优点：各个浏览器都可以适用，无需设置浏览器跨域

缺点：每次切换环境时，都需要重新启动服务。


### 方法2

第二种方法就是设置浏览器，让它支持跨域的请求。
其实这个浏览器的快捷方式，用这个快捷方式打开的，支持跨域请求。

下面是设置谷歌浏览器的跨域方法：

    1.49版本之前的跨域。 --disable-web-security 和exe要有一个空格。
    2.49版本之后的跨域。
          1.在电脑上新建一个目录，例如：C:\MyChromeDevUserData
          2.在属性页面中的目标输入框里加上   --disable-web-security --user-data-dir=C:\MyChromeDevUserData，--user-data-dir的值就是刚才新建的目录。
          3.点击应用和确定后关闭属性页面，并打开chrome浏览器。
    再次打开chrome，发现有“--disable-web-security”相关的提示，说明chrome又能正常跨域工作了。
    注： 跨域成功后，首页换成了google的welcome页面，同时原来收藏的链接和历史记录都不见了，而C:\MyChromeDevUserData目录下则生成了新的个人信息相关的文件

而ie浏览器的跨域方法上面可以去搜下，至于火狐浏览器的跨域，在现在较新的版本已经无法再设置了，以前旧版本还可以设置。

优点：每次切换环境，无需重启服务。

缺点：每个浏览器的跨域方法都不一样，而且要把设置了跨域的快捷方式与没设置的区别开。