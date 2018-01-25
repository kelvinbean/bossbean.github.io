---
title: vue项目引入ueditor
---

最近很忙，所以都好久没更新了。好不容易松一口气来写写总结。

最近做的vue项目中，要是用到文本编辑器。其实现在有非常多的vue版本的轻量级文本编辑器。使用起来也都非常简单。

但是这些轻量级的编辑器都不支持本地上传视频，上传图片。

他们的上传图片的原理就是把图片转成base64，上传视频就只提供外链视频链接。但是无奈，项目需要本地上传视频功能。

所以我这里就选择了还是使用ueditor来做文本编辑器。

### 下载

首先先去官网下载包，[下载地址](http://ueditor.baidu.com/website/download.html)

然后把下载到的包解压放到static目录下。因为他不需要webpack打包，所以放到static的目录下。

<!-- More -->


### 编写组件

编写组件这一步就是简单封装一下编辑器。让以后需要引用编辑器的都可以简单调用就好。

下面是封装的代码

```bash
    <template>
        <div>
            <script id="editor" type="text/plain"></script>
        </div>
    </template>

    <script>
        // 百度编辑器(这是我的路径。你们就用你们的路径)
        import '../../../static/plugins/utf8-jsp/ueditor.config.js';
        import '../../../static/plugins/utf8-jsp/ueditor.all.min.js';
        import '../../../static/plugins/utf8-jsp/lang/zh-cn/zh-cn.js';
        import '../../../static/plugins/utf8-jsp/ueditor.parse.min.js';
        export default {
            name: 'editor',
            data () {
                return {
                    editor: null
                }
            },
            props: {
                config: {//编辑器配置（具体配置看官网介绍）
                    type: Object,
                    default:function(){
                        return {
                            initialFrameWidth: 800,//给编辑器定宽
                            initialFrameHeight: 350//给编辑器定高
                        };
                    }
                }
            },
            mounted() {
                this.editor = UE.getEditor('editor', this.config); // 初始化UE
                this.editor.addListener("ready",()=>{
                    this.editor.setContent(this.defaultMsg); // 确保UE加载完成后，放入内容。
                });
            },
            methods: {//下面声明的两个方法是用于父类的组件去使用的。（可以根据需要自己去扩展，实例化对象后的方法自行查看官方文档）
                getUEContent() { // 获取内容方法
                    return this.editor.getContent();
                },
                setUEContent(content) {//设置内容的方法
                    return this.editor.setContent(content);
                }
            },
            destroyed() {
                this.editor.destroy();
            }
        }
    </script>
```


### 调用

```bash
    <template>
        <div>
            <editor ref="bean"></editor>
            <button @click="submitForm">提交</button>
        </div>
    </template>
    <script>
        import editor from 'editor.vue';
        export default {
            mounted(){
                this.$refs['bean'].setUEContent('我要初始化的数据');
            },
            methods:{
                submitForm(){
                    var content = this.$refs['bean'].getUEContent();
                    //然后去下面去提交内容
                }
            }

        }
    </script>
```

这里可以巧用$refs拿到实例化后的editor组件对象。然后调用组件里面声明的方法。所以ref不是只有拿个dom对象的作用哦。

### 页面呈现

由于存的数据都是带标签的字符串。但是vue是有过滤html的功能的，所以你直接引入到页面中，拿到的就是把所有标签都带出来的文字。

所以这里要做的就是用v-html，即如下

```bash
    <div v-html>{{content}}</div>
```

这样页面的呈现就是你想要的效果啦。

### 上传文件和视频的配置 

对了，说了那么多。主要的上传文件和视频的配置是如何配的呢？

好了，其实我想说这部分的配置基本是后端的活。

前端其实只要修改一个地方。就是把一开始下载的包的。

找到ueditor.config.js文件。修改serverUrl的值，把它改成后端提供的接口。

那么后端方面的配置要如何配呢？

下面找了一篇比较详细的文章。可以供大家去参考。[参考地址](https://www.cnblogs.com/ocean-sky/p/7132319.html)。