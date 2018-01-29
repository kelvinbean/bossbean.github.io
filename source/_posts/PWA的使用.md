---
title:  PWA的使用
---

去年就已经被很多前端工程师津津乐道的PWA，我感觉今年要大火。

原因是之前大家都在担心ios系统不支持service-worker，
但是最近苹果将在 iOS 11.3 和 macOS 10.13.4 版本上正式增加对 Service Worker 的支持，同时还支持了添加到桌面（Web App Manfiest）.
让我又重新把目光投放到这个PWA。这将会是又一次的前端大变革。

<!-- More -->

## 什么是PWA？

PWA全称是Progressive Web Apps,中文翻译过来的名称就是渐进增强的web应用。

这里的所谓渐进增强，就是说你如果在你现有的项目中使用PWA，在你的系统或者浏览器不支持PWA的时候， 对你的项目不会有任何影响。

但是如果支持PWA的话，可以把当前的页面加入到你的手机桌面上，就像一个APP一样。使用上会有APP的体验，同时也具有缓存的功能，

就算没有网络也可以访问你之前访问过的页面。

一个 PWA 应用首先是一个网页, 可以通过 Web 技术编写出一个网页应用. 随后添加上 App Manifest 和 Service Worker 来实现 PWA 的安装和离线等功能。


## PWA的优点

1.你只需要基于开放的 W3C 标准的 web 开发技术来开发一个app。不需要多客户端开发。

2.用户可以在安装前就体验你的 app。

3.不需要通过 AppStore 下载 app。app 会自动升级不需要用户升级。

4.用户会受到‘安装’的提示，点击安装会增加一个图标到用户首屏。

5.被打开时，PWA 会展示一个有吸引力的闪屏。

6.chrome 提供了可选选项，可以使 PWA 得到全屏体验。

7.必要的文件会被本地缓存，因此会比标准的web app 响应更快（也许也会比native app响应快）

8.安装及其轻量 -- 或许会有几百 kb 的缓存数据。

9.网站的数据传输必须是 https 连接。

10.PWAs 可以离线工作，并且在网络恢复时可以同步最新数据

等等。

## PWA的使用

说了那么多，那到底怎么使用呢。

把自己已有的项目加上PWA并不复杂。

### manifest.json的 编写

manifest.json这个文件的配置主要是用于添加到桌面的APP的功能，对添加到桌面的icon图片。打开这个PWA展示的背景色等等的设置。


manifest 中主要属性有：

name —— 网页显示给用户的完整名称

short_name —— 当空间不足以显示全名时的网站缩写名称

description —— 关于网站的详细描述

start_url —— 网页的初始 相对 URL（比如 /）

scope —— 导航范围。比如，/app/的scope就限制 app 在这个文件夹里。

background-color —— 启动屏和浏览器的背景颜色

theme_color —— 网站的主题颜色，一般都与背景颜色相同，它可以影响网站的显示

orientation —— 首选的显示方向：any, natural, landscape, landscape-primary, landscape-secondary, portrait, portrait-primary, 和 portrait-secondary。

display —— 首选的显示方式：fullscreen, standalone(看起来像是native app)，minimal-ui(有简化的浏览器控制选项) 和 browser(常规的浏览器 tab)

icons —— 定义了 src URL, sizes和type的图片对象数组。


例子：

```bash
    {
      "name": "Boss Bean",
      "short_name": "Bean",
      "display": "standalone",
      "start_url": "/",
      "theme_color": "#ccc",
      "background_color": "#29c360",
      "icons": [
        {
          "src": "bean.png",
          "sizes": "256x256",
          "type": "image/png"
        }
      ]
    }
```


在html页面文件引入

```bash
<link rel="manifest" href="manifest.json" />
```



### 添加service worker

Service Worker 在网页已经关闭的情况下还可以运行, 用来实现页面的缓存和离线, 后台通知等等功能。sw.js 文件需要在 HTML 当中引入:

```bash
    <script>
      if (navigator.serviceWorker != null) {
        navigator.serviceWorker.register('sw.js')
        .then(function(registration) {
          console.log('注册的事件的范围: ', registration.scope);
        });
      }
    </script>
```

在 Service Worker 当中会用到一些全局变量:

    self: 表示 Service Worker 作用域, 也是全局变量
    caches: 表示缓存
    skipWaiting: 表示强制当前处在 waiting 状态的脚本进入 activate 状态
    clients: 表示 Service Worker 接管的页面


#### 处理静态缓存

借助 Service Worker, 可以在注册完成安装 Service Worker 时, 抓取资源写入缓存:

在上文提到的sw.js文件内加入

```bash
    self.addEventListener('install', e => {
      e.waitUntil(
        caches.open(cacheStorageKey)
        .then(cache => cache.addAll(cacheList))
        .then(() => self.skipWaiting())
      )
    })
```

调用 self.skipWaiting() 方法是为了在页面更新的过程当中, 新的 Service Worker 脚本能立即激活和生效。


#### 处理动态缓存

网页抓取资源的过程中, 在 Service Worker 可以捕获到 fetch 事件, 可以编写代码决定如何响应资源的请求:

在sw.js文件内加入

```bash
    self.addEventListener('fetch', function(e) {
        e.respondWith(
            caches.match(e.request).then(function(response) {
              if (response != null) {
                return response
              }
              return fetch(e.request.url)
            })
        )
    })
```


## 总结

这里没有放出demo.但是可以手动测试一下。目前估计只能在一些高版本的系统和浏览器上看到添加到桌面的效果（暂时没测过）。而离线缓存这个，可以试试断开网络看看来测试一下。


## 参考文献

这两篇文章都写得很好。

[PWA 入门: 写个非常简单的 PWA 页面](https://zhuanlan.zhihu.com/p/25459319)

[改造你的网站，变身 PWA](https://segmentfault.com/a/1190000008880637)


## 更多

另外百度貌似也提供了个基于 Vue.js 的 PWA 解决方案，叫[LAVAS](https://lavas.baidu.com/)。感觉有兴趣也可以去研究一下。
