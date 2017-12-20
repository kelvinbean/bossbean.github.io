---
title: vue 2.x的组件之间双向绑定
---

vue在2.x版本就取消了组件之间数据的双向影响。因此如果想要实现这个功能。使用以下方法来解决。

## 解决方法

以下我使用一个自己写的组件其中一部分代码来展示一下解决办法

<!-- More -->

### 父组件
假设子组件的名称为popup
``` bash
    <template>
        <popup v-model="showModel"></popup>
    </template>
    <script>
        import popup from 'popup'
        export default{
            data(){
                return {
                    showModel:false
                }
            },
            components:{
                popup
            }
        }
    </script>
```

### 子组件
```bash
    <template>
        <div v-if="visible">
        </div>
    </template>
    <script>
        export default{
            porps:{
                value:{
                    type:Boolean,
                    default:true
                }
            },
            data(){
                return {
                    visible:false
                }
            },
            watch:{
                value(val) {
                    this.visible = val;
                },
                visible(val) {
                    this.$emit('input', val);
                }
            }
        }
    </script>    
```

讲解一下，父级的调用子组件是，加入的v-model相当于给子组件传入一个value属性。然后监听父级的value值改变时，修改子组件的visible属性。

监听当子组件的visible属性修改时，通过触发input事件，修改父级对应的showModel的值。

这里其实也是利用vue的数据双向绑定的原理。因为像v-model绑定一个输入框，当输入框触发input事件的时候，对应的js内存的值就会改变。只是这里我们是通过$emit来给父组件传递数据，来手动触发input事件。

### 其他

如果说没有使用v-model。而是一个别的属性。或者自定义的属性。那么就需要代码可能会多写一点。

父级的代码可能需要改成这样：
``` bash
    <template>
        <popup :test="showModel" @changeTest="changeTest"></popup>
    </template>
    <script>
        import popup from 'popup'
        export default{
            data(){
                return {
                    showModel:false
                }
            },
            method:{
                changeTest(val){
                    this.showModel = val;
                }
            },
            components:{
                popup
            }
        }
    </script>
```

子级的代码就会是这样
```bash
    <template>
        <div v-if="visible">
        </div>
    </template>
    <script>
        export default{
            porps:{
                test:{
                    type:Boolean,
                    default:true
                }
            },
            data(){
                return {
                    visible:false
                }
            },
            watch:{
                test(val) {
                    this.visible = val;
                },
                visible(val) {
                    this.$emit('changeTest', val);
                }
            }
        }
    </script>    
```

这样的话，麻烦的地方就是每个调用这个子组件的方法都需要加一个changeTest的方法。这样使用起来就很麻烦。所以能用value可以解决的就不要使用自定义的属性去处理这个双向绑定。
自定义的属性更多的用在初始化组件的数据，然后不会再变。或者是子级单向监听父级数据数据变化的时候来使用。