---
title: FileReader的使用
---

![Image text](/images/fileReader/banner.jpg)

<!--  More -->

FileReader是HTML5就已经提出的一个用于处理上传文件的对象。

它使得我们在处理文件上变得更加简单了。


## 简单介绍

### 调用FileReader实例对象的方法

    方法名              参数                 描述
    abort               none                中断读取
    readAsBinaryString  file                将文件读取为二进制码
    readAsDataURL       file                将文件读取为 DataURL
    readAsText          file, [encoding]    将文件读取为文本


readAsText：该方法有两个参数，其中第二个参数是文本的编码方式，默认值为 UTF-8。
这个方法非常容易理解，将文件以文本方式读取，读取的结果即是这个文本文件中的内容。
readAsBinaryString：该方法将文件读取为二进制字符串，通常我们将它传送到后端，后端可以通过这段字符串存储文件。
readAsDataURL：这是例子程序中用到的方法，该方法将文件读取为一段以 data: 开头的字符串，这段字符串的实质就是 Data URL，
Data URL是一种将小文件直接嵌入文档的方案。这里的小文件通常是指图像与 html 等格式的文件。

### 处理事件

FileReader 包含了一套完整的事件模型，用于捕获读取文件时的状态，下面这个表格归纳了这些事件。

    事件         描述
    onabort      中断时触发
    onerror      出错时触发
    onload       文件读取成功完成时触发
    onloadend    读取完成触发，无论成功或失败
    onloadstart  读取开始时触发
    onprogress   读取中


## 实例运用

### 上传文件显示图片

一般的上传图片的处理都是把文件提交上去。之后让后端返回一个完整的url。然后再生成img标签，把url填进去。从而显示上传的图片。

那如果想要一上传，不需要等待服务器响应就显示出上传的图片这要这么做呢？

可以使用如下的方法:

``` bash
    html代码：
    <input type="file" id="file" onchange="fileChange(this)">
    <div id="result"></div>
    用onchange绑定一个叫fileChange的function,把本对象传入

    js代码：
    function fileChange(source){
        console.log(source.files)
        var file = source.files[0];
        var reader = new FileReader();
        reader.onloadend = function(){//reader读取完成后触发，无论成功或失败
            console.log(this.result);//读取完成后的结果在result里获取
            var img = new Image();
            img.src = this.result;
            document.getElementById('result').append(img);

        }
        reader.readAsDataURL(file);//读取文件As DataURL,触发onload和ondloadend，无论读取成功或失败，方法并不会返回读取结果，这一结果存储在 result属性中。
    }
```


如下是打印出来的source.files的信息

![Image text](/images/fileReader/FileReader_source.png)

其实就是个类似数组的对象，有最后的修改日期信息，文件名称，文件大小，文件的MIME类型等

上面的代码。只是简单举个例子，那文件数组的第一个对象，然后实例一个FileReader对象。
然后再给reader绑定一个加载后的时间。然后通过reader.readAsDataURL(file)把刚刚取到的第一个对象传入。readAsDataURL加载结束后就会触发刚刚上面绑定的onloadend方法。

onloadend可以通过this.result查看结果。
这里获取到的就是要给img需要的url。
其实readAsDataURLjiu就是把这个文件转出base64的格式然后创建img.把图片加载到div里任务就完成了。

