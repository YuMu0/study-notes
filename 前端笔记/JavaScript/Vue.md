# Vue基础

### el挂载点

* 父元素不会被渲染，内部子元素会继承渲染
* CSS选择器都可以使用，推荐使用`id选择器`，推荐使用`<div>`，因为其没有内部自带样式

### data数据对象

* Vue中用到的数据定义在`data`中，`data`中可以写复杂类型的数据
* 渲染复杂类型数据时，遵守JS的语法即可

# 本地应用

### v-text指令

设置标签的文本值(text content)，使用**插值表达式**可以保证只渲染部分内容，而使用`v-text`会使得标签内容全部被渲染

```html
 <div id = "app">
 	<h2 v-text = "message"></h2>
    <h2>只渲染内部文本{{message}}</h2>
 </div>
```

### v-html指令

* 设置元素的`innerHTML`

* 对于普通文本，和`v-text`没有区别；但对于**html语法**，会被解析出来

### v-on指令

为元素绑定事件

完整写法和简写写法：

```html
<div id = "app">
    <input type = "button" value = "v-on指令" v-on:click = "doIt">
    <input type = "button" value = "v-on简写" @click = "doIt">
</div>

<script>
    var app = new Vue
    ({
        el: "#app",
        methods:
        {
            doIt: function(){}
        }
    });
</script>
```

函数中也可以传递参数

```html
<input type = "button" value = "v-on简写" @click = "doIt(p1, p2)">
```

事件修饰符 [官网详细内容及API](https://cn.vuejs.org/v2/api/#v-on)

```html
<input type = "button" value = "v-on简写" @keyup.enter = "doIt">
```

### v-show指令

* 根据表达式的真假，切换元素的显示和隐藏
* 原理是修改元素的`display`属性，实现显示或隐藏
* 指令后面的内容，最终都会被解析为**布尔值**

### v-if指令

* 根据表达式的真假，切换元素的显示和隐藏——和`v-show`一样
* `v-show`是改变元素的`display`属性，而`v-if`是**操纵DOM树**直接增加或去除元素

### v-band指令

设置元素的属性

```html
<div>
    <img v-bind: src = "imgSrc">
    <img v-bind: title = "imgtitle">
    <img v-bind: class = "isActive ? 'active' : ''">
    <img v-bind: class = {active: isActive}>
</div>
```

其中` {active: isActive}`是对象的写法，`active`这个类名是否生效，取决于`isActive`的布尔值

`v-bind`可以省略，只留下冒号

### v-for指令

根据数据生成列表结构，常与数组结合使用

```html
<div>
    <ul>
        <li v-for = "item in arr">
            数组每一项：{{item}}
        </li>
    </ul>
     <ul>
        <li v-for = "(item,index) in arr">
            {{index+1}}数组每一项：{{item}}
        </li>
    </ul>
</div>
```

### v-model指令

获取和设置表单元素的值(**双向数据绑定**：更改任意一方的值，另一方都会同步更新)

```html
<input type = "text" v-model = "message">
<h2>{{message}}</h2>
...
<script>
	var app = new Vue
    ({
        el: "#app",
        data:
        {
            message: "new message"
        }
    });
</script>
```

