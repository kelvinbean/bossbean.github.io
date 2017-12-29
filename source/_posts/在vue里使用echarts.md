---
title: 在vue中使用echarts
---

最近项目需要做一些图表，如柱状图，圆饼图等等。

一说到做图表，我们就能想到一个非常强大的框架Echarts。它可以实现各种需求的图，基本做这种展示数据的图，都用它。

那今天我就来介绍一下怎么在vue项目中使用echarts。


### vue-echarts-v3

其实非常简单，也没什么好说的。主要就是使用vue版本的echarts。

首先安装vue-echarts-v3，这是echarts 3的版本。之前的我没去细究，反正就使用最新的。

[项目仓库地址](https://github.com/xlsdg/vue-echarts-v3)

```bash
    npm install --save vue-echarts-v3
```


### 修改webpack配置

因为在webpack打包的时候需要做resolve，不然打包出来的代码就无法正常显示。

如果是使用脚手架vue-cli的话，就在build/webpack.base.conf.js里对js代码处理的那里加上对echarts的resolve.

```bash
    module: {
        rules: [
          {
            test: /\.vue$/,
            loader: 'vue-loader',
            options: vueLoaderConfig
          },
          {
            test: /\.js$/,
            loader: 'babel-loader',
            include: [resolve('src'), resolve('test'), resolve('node_modules/vue-echarts-v3/src')]//加上这个resolve('node_modules/vue-echarts-v3/src')
          },
          {
            test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
            loader: 'url-loader',
            options: {
              limit: 10000,
              name: utils.assetsPath('img/[name].[hash:7].[ext]')
            }
          },
          {
            test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
            loader: 'url-loader',
            options: {
              limit: 10000,
              name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
            }
          }
        ]
    }
```

### 使用

使用上非常简单。

```bash
    <template>
        <div>
            <div class="echarts">
                <IEcharts :option="option"></IEcharts>
            </div>
        </div>
    </template>
    <script>
        import IEcharts from 'vue-echarts-v3';
        export default{
            data(){
                return{
                    option:{//这里就是传echarts的配置。跟官网的配置完全一致，下面只是一个简单例子
                        title:{
                            text:'简单示例',
                            x:'center'
                        },
                        legend: {
                            orient: 'vertical',
                            x: 'right',
                            data: ['视频广告','联盟广告','邮件营销','直接访问','搜索引擎']
                        },
                        series : {
                            type: 'pie',
                            radius: '50%',
                            data:[
                                {value:100, name:'视频广告'},
                                {value:274, name:'联盟广告'},
                                {value:310, name:'邮件营销'},
                                {value:335, name:'直接访问'},
                                {value:400, name:'搜索引擎'}
                            ],
                            label:{
                                normal:{
                                    formatter: '{d}%',
                                    position:'inner'
                                }
                            }
                        }
                    }
                }
            },
            components:{
                IEcharts
            }
        }
    </script>
    <style>
        .echarts{//设置外部宽高
            width:400px;
            height:400px;
        }
    </style>
```


一些用的比较多的option配置

    title   设置图表标题
    legend  设置图表项的说明
    series  设置图表的类型等多个属性。
    color   设置颜色
    xAxis   X轴设置
    yAxis   Y轴设置


这里就不具体讲配置项的内容。官网上有非常详细的介绍。[Echarts配置](http://echarts.baidu.com/option.html#xAxis)

主要就是根据设置外部的宽高。来控制echarts生成的图片大小。然后配置想要的视图。

另外这个组件还提供多个属性。如loading属性，通过控制它的值是true还是false来显示和隐藏加载图层。

还提供了click事件，用来监听点击操作。

```bash
    onClick(event, instance, echarts){
        if(event.dataIndex==0){//点击了第一个柱体
            //逻辑判断  
        }else{////点击了第二个柱体
            //逻辑判断
        }
    }
```

上面就是很笼统的介绍了一下。其实github和echarts官网都很有详细的介绍，如果想详细了解还是要去查看官方的文档。