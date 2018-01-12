---
title: vue自定义组件递归
---

在用vue来做项目的时候，遇到了一个情况。就是当你不确定返回数据的层级是多少是，但是你又需要把底下的数据全都遍历出来。这时候直接用v-for来循环是无法循环出来的。那该怎么办呢？

### 举个栗子
比如有一个这样的数据结构,然后我们需要把这个每个对象的name都遍历出来而且呈现树形结构的形式。但是不确定到底有多少层。

```bash
[{
    name:'boss',
    child:[{
        name:'bean',
        child:[{
            ...
        }]
    }]
},{
    name:'boss2'
}]
```

<!-- More -->

### 解决办法

思路就是创建一个自定义的组件。然后让这个组件递归调用自己，从而的达到效果。

事不宜迟，就来试试吧。
```bash
    <template>
        <div v-for="item in list" >
            <div>{{item.name}}</div>
            <!-- 调用自身组件 -->
            <Bean :list="item.child"></Bean>
        </div>
    </template>
    <script>
    export default{
        name:'Bean',
        props:{
            list:{
                default:[]
            }         
        }
    }
    </script>
```

父级调用：

```bash
    <template>
        <Bean :list="formData"></Bean>
    </template>
    <script>
    import Bean from 'Bean.vue';
    export default{
        data(){
            return{
               formData:[] 
            };
        }
    }
    </script>    
```


就是这么简单。如果是要加上一些点击之后返回数据name给父级的话，要注意的问题是要把绑定的方法继续传递给下个组件。不然下个组件就获取不到父级定义的方法。

### 实例
下面是我编写的一个实例:

这个例子比较复杂。就是写了一个部门组件，同时用于新增员工的时候选择部门所用。也用于在系统中选择某个部门的员工的功能所用。

该例子的数据结构是，同样是不确定部门的层级，而且每个层级都可以添加员工。

```bash
    [{
        name:'财务部门',
        childDepts:[{
            name:'财务子部门1',
            employees:[{
                name:'员工1'
            }]
        },{
            name:'财务子部门2'
        }],
        employees:[{
            name:'财务主管'
        }]
    }]
```

下面是实现代码：

```bash
    <template>
        <div>
            <div class="tree_list" v-for="(item,key) in list" v-show="show">
                <div class="tree_item clearfix" @click="chooseItem(item)"  :title="item.description">
                    <i class="icon-plus-1" v-show="!item.active && (item.childDepts.length>0
                     || (item.employees && item.employees.length>0))" @click.stop="showChild(item,true)"></i>
                    <!-- 加号，点击后展开下一级的子元素 -->
                    <i class="icon-minus" v-show="item.active && (item.childDepts.length>0 
                     || (item.employees && item.employees.length>0))" @click.stop="showChild(item,false)"></i>
                    <!-- 减号，点击后展开收齐该层级下的所有子级 -->
                    <span>{{item.name}}</span>
                </div>
                <DepartTree :show="item.active" :list="item.childDepts" 
                :personlist="item.employees?item.employees:[]" @choose="chooseItem" :chooseId="chooseId"
                 @choosePerson="choosePerson"></DepartTree>
                <!-- 员工的数据，放在部门的后面 -->
                <div class="tree_item clearfix person noson" @click="choosePerson(personItem)" 
                    v-for="personItem in personlist" v-if="personlist && key+1 == list.length">
                    <span>{{personItem.name}}</span>
                </div>
            </div>
            <!-- 当没有下级部门时，有部门员工的话。则显示该部门的员工 -->
            <div class="tree_list" v-show="show" v-if="personlist && list.length <= 0">
                <div class="tree_item clearfix person noson" @click="choosePerson(personItem)" 
                v-for="personItem in personlist">
                    <span>{{personItem.name}}</span>
                </div>
            </div>
        </div>
    </template>
    <script>
        export default{
            name:'DepartTree',
            props:{
                show:{
                    default:false
                },
                list:{//部门列表
                    default:[]
                },
                personlist:{//员工列表
                    default:function(){
                        return [];
                    }
                },
                chooseId:{
                    default:''
                }
            },
            data(){
                return{
                    chooseTarget:{}
                };
            },
            methods:{
                chooseItem(item){//选择部门后触发的方法
                    this.$emit('choose',item);
                },
                choosePerson(item){//选择员工后触发的方法
                    this.$emit('choosePerson',item);
                },
                showChild(item,bool){//控制层级的显示与隐藏
                    this.$set(item,'active',bool);
                }
            }
        }
    </script>
```