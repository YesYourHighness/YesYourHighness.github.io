---
title: 02.Vue-从一个Todolist开始
date: 2019-07-19 17:44:36
tags: 
- Vue
categories: 
- 前端
- Vue
---

<center>
引言：

这一节用Vue做了一个Todolist的小实例，

并且给他拆成了组件

还有简单的组件间传值该怎么办呢？

加油加油！！
</center>
<!-- more -->

------------
##Vue做一个TodoList
做一个TodoList的完整功能，来复习的知识
```javascript
<body>
<div id="root">
    <div>
        <input 
        v-model="inputValue"/>
        <!--v-model代表双向绑定，
        当输入内容改变或者实例中的inputValue改变，
        另一个也会随之改变，
        是相互影响的！-->
        <button @click="handleSubmit">提交</button>
    </div>
    <ul>
        <li 
        v-for="(item,index) of list" 
        :key="index">
        {{item}}
        </li>
    </ul>
</div>
<script>
    new Vue({
        el: "#root",
        data: {
            inputValue: '',
            list: [],
        },
        methods: {
            handleSubmit: function () {
                this.list.push(this.inputValue);//记住要加this,添加到数组当中
                this.inputValue = '';//输入完成后清空输入框
            }
        }
    })
</script>
</body>
```
## 接下来我们对它进行组件拆分

### 定义局部的组件
```javascript
<div id="root">
    <div>
        <input v-model="inputValue"/>
        <button @click="handleSubmit">提交</button>
    </div>
    <ul>
        <todo-item
        v-for="(item,index) of list"
        :key="index"
        :content="item"
        ></todo-item>
        <!--这样就可以把参数传给组件-->
    </ul>

</div>
<script>
    let TodoItem = {
        props:['content'],
        template: '<li>{{content}}</li>'
    };
    new Vue({
        el: "#root",
        data: {
            inputValue: '',
            list: [],
        },
        methods: {
            handleSubmit: function () {
                this.list.push(this.inputValue);//添加到数组当中
                this.inputValue = '';//输入完成后清空输入框
            }
        },
        //局部变量要进行注册
        components: {
            'todo-item': TodoItem
        },
    })
</script>
</body>
</html>
```

### 定义全局的组件
```javascript
<body>
<div id="root">
    <div>
        <input v-model="inputValue"/>
        <button @click="handleSubmit">提交</button>
    </div>
    <ul>
        <li v-for="(item,index) of list" :key="index">{{item}}</li>
    </ul>
    <ul>
        <todo-item></todo-item>
        <!--我们在这里使用自定义的组件-->
    </ul>

</div>
<script>
    //Vue提供了我们自定义组件的功能
    Vue.component('todo-item',{
        //这里定义的名字可以是TodoItem,
        //但在使用的时候依然要用todo-Item
        template: '<li>全局组件</li>'
    });//这里是全局组件

    new Vue({
        el: "#root",
        methods: {
            handleSubmit: function () {
                this.list.push(this.inputValue);//添加到数组当中
                this.inputValue = '';//输入完成后清空输入框
            }
        }
    })
</script>
</body>
```

### 把li标签拆成了组件
```javascript
<body>
<div id="root">
    <div>
        <input v-model="inputValue"/>
        <button @click="handleSubmit">提交</button>
    </div>
    <ul>
        <todo-item
                v-for="(item,index) of list"
                :key="index"
                :content="item"
        ></todo-item>
        <!--
        ！！父组件给子组件传值：
        1. 
        对于外层的内容来说，
        我们自建的组件属于外部的一个子组件，
        我们可以使用v-bind命令来对子组件进行传值
        例如这里的 :content='item' 
        -->
        <!--这样就可以把参数传给组件-->
    </ul>

</div>
<script>
    //Vue提供了我们自定义组件的功能
    Vue.component('todo-item',{
        props:['content'],
        //2. 子组件光接收不行，还得接收!!
        template: '<li>{{content}}</li>'
    });//这里是全局组件

    new Vue({
        el: "#root",
        data: {
            inputValue: '',
            list: [],
        },
        methods: {
            handleSubmit: function () {
                this.list.push(this.inputValue);//添加到数组当中
                this.inputValue = '';//输入完成后清空输入框
            }
        }
    })
</script>
</body>
```
其实每一个Vue的实例都是一个组件，而当你不输入template的内容时，它默认会是挂载点下的所有内容作为其模板
```javascript
//Vue中的组件都是Vue的一个实例，可以给他们绑定一个事件
    Vue.component('todo-item', {
        props: ['content'],
        template: '<li @click="handleClick">{{content}}</li>',
        methods: {
            handleClick:function () {
                alert('clicked')
            }
        }
    });
```
我们再添加一个删除功能
```javascript
<body>
<div id="root">
    <div>
        <input v-model="inputValue"/>
        <button @click="handleSubmit">提交</button>
    </div>
    <ul>
        <todo-item
                v-for="(item,index) of list"
                :key="index"
                :content="item"
                :index="index"
                @delete="handleDelete"
        ></todo-item>
        <!--这样就可以把参数传给组件-->
    </ul>
</div>
<script>
    //Vue中的组件都是Vue的一个实例，可以给他们绑定一个事件
    Vue.component('todo-item', {
        props: ['content', 'index'],
        template: '<li @click="handleClick">{{content}}</li>',
        methods: {
            handleClick: function () {
                this.$emit('delete', this.index)
                //!!子组件向父组件传值
                //触发当前实例上的事件
            },
        }
    });
    new Vue({
        el: "#root",
        data: {
            inputValue: '',
            list: [],
        },
        methods: {
            handleSubmit: function () {
                this.list.push(this.inputValue);
                this.inputValue = '';
            },
            handleDelete: function (index) {
                this.list.splice(index, 1)//删除当前索引后的一个元素
            }
        }
    })
</script>
```

## 总结一下简单的组件间传值

- 父组件给子组件传值

1. 在子组件上用v-bind来对子组件传值
2. 子组件填入`props`的属性，接收传来的值，就可以使用了

- 子组件给父组件传值

1. 使用`this.$emit()`这个函数
2. 子组件要监听emit所发出的事件名并对其有相应的反应