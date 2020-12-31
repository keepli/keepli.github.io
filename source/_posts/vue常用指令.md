---
title: vue常用指令
date: 2020-09-15
updated: 2020-11-01
tags: vue
categories: 前端
---

#### v-bind（有简写方式）

> 该指令用来动态修改标签中的属性信息

动态绑定class有两种语法实现

- 对象语法
- 数组语法


```HTML
<div id="app">
    <a v-bind:href="url"></a>
    <!--以下是简写方式-->
    <a :href="url"></a>
    <!--可以动态切换class来改变样式（以下使用的是对象语法）-->
    <!--方法一-->
    <div :class="{a:is-a,b:is-b}">动态切换class</a>
    <!--方法二-->
    <div :class="getClasses()">动态切换class</a>
</div>
```

```JavaScript
var app = new Vue({
  el: '#app',
  data: {
    url: "https://www.baidu.com",
    is-a: true,
    is-b: false  
  },
  methods: {
      getClasses: function(){
          return {a: this.is-a, b: this.is-b}
      }
  }  
})
```

<!-- more -->

#### v-on（有简写方式）

> 该指令用来绑定事件

- 在调用方法时省略了小括号，但是方法本身是需要一个参数的，这个时候vue会默认将浏览器生成的event事件对象作为参数传入到方法中

```html
<div id="app">
    <a v-on:click="doSomething">{{message}}</a>
    <!--以下是简写方式-->
    <a @click="doSomething">{{message}}</a>
</div>
```

```javascript
var app = new Vue({
  el: '#app',
  data: {
    message: '点击'
  },
  methods: {
      doSomething: function(event){
          alert('我被点击了');
          console.log(event);//接收来自浏览器生成的event
      }
  }  
})
```

- 如果往单击事件的方法中传入多个参数，要想在接收到浏览器生成的event事件，需要$event

```html
<div id="app">
    <a v-on:click="doSomething">{{message}}</a>
    <!--以下是简写方式-->
    <a @click="doSomething('你好',$event)">{{message}}</a>
</div>
```

```javascript
var app = new Vue({
  el: '#app',
  data: {
    message: '点击'
  },
  methods: {
      doSomething: function(msg,event){
          alert(msg);
          console.log(event);//接收来自浏览器生成的event
      }
  }  
})
```



#### v-if、v-else、v-else-if

>该指令用来做判断，如果条件满足就渲染

v-else-if用的少，因为判断条件比较复杂的话可以使用计算属性来判断，注意的是v-else-if不能单独使用

```html
<div id="app">
    <div v-if="isShow">true显示</div>
    <div v-else="isShow">false显示</div>
</div>
```

```javascript
var app = new Vue({
    el: "#app",
    data: {
        isShow: true
    }
});
```



#### v-show

> 该指令用来隐藏标签

它和v-if指令的区别是v-if条件不满足不会进行渲染，而v-show一定会渲染，只是设置了隐藏属性（display）

- 当显示隐藏切换频率高用v-show
- 当只有一次切换使用v-if

```html
<div id="app">
    <div v-show="isShow">显示</div>
    <button @click="isShow=!isShow">切换</button>
</div>
```

```javascript
var app = new Vue({
    el: "#app",
    data: {
        isShow: true
    }
});
```



#### v-for

> 该指令用来遍历数组和对象

- 遍历数组

```html
<div id="app">
    <ul>
        <!--itm是元素，index是索引，items是数组-->
        <li v-for="itm,index in items">{{index}}--{{itm}}</li>
    </ul>
</div>
```

```javascript
var app = new Vue({
    el: "#app",
    data: {
        items: ["海王","狮王","虎王"]
    }
});
```

- 遍历对象

```html
<div id="app">
    <!-- 写法一（只获取value） -->
    <ul>
        <li v-for="value in info">{{value}}</li> <!--单个只获取value -->
    </ul>

    <!-- 写法二（获取key和value） -->
    <ul>
        <li v-for="value,key in info">{{key}}--{{value}}</li> 
    </ul>

    <!-- 写法三（获取key、value和索引） -->
    <ul>
        <li v-for="value,key,index in info">{{index}}--{{key}}--{{value}}</li> 
    </ul>
</div>
```

```javascript
var app = new Vue({
    el: "#app",
    data: {
        info: {
            name: "李狗蛋",
            age: 20,
            sex: "男"
        }
    }
});
```



#### v-model

>该指令用来对表单元素数据进行双向绑定

- 文本输入框text

```html
<div id="app">
    <!--v-model跟value属性绑定的-->
    <input type="text" v-model="message">
    {{message}}
</div>
```

```javascript
var app = new Vue({
    el: "#app",
    data: {
        message: "hello"
    }
});
```

- 单选框radio

```html
<div id="app">
    <!-- label标签可以实现点击文字就可触发相应绑定的标签 -->
    <label for="male">男</label>
    <input type="radio" name="sex" id="male" value="男" v-model="message"/>
    <br />
    <label for="female">女</label>
    <input type="radio" name="sex" id="female" value="女" v-model="message"/>
    你选择的性别是：{{message}}
</div>
```

```javascript
var app = new Vue({
    el: "#app",
    data: {
        message: ""
    }
});
```

- 复选框checkbox（有单选和多选）

```html
<div id="app">
    <!-- 单选框 -->
    <label for="agree">同意协议</label>
    <input type="checkbox" id="agree" name="agree" v-model="isAgree"/>
    <p>你选择的是：{{isAgree}}</p>
    <button :disabled="!isAgree">下一步</button>
    </br>

    <!-- 多选框 -->
    篮球：<input type="checkbox" value="篮球" v-model="hobbys">
    足球：<input type="checkbox" value="足球" v-model="hobbys">
    台球：<input type="checkbox" value="台球" v-model="hobbys">
    <p>爱好有：{{hobbys}}</p>
</div>
```

```javascript
var app = new Vue({
    el: "#app",
    data: {
        isAgree: false,
        hobbys: []
    }
});
```

- 下选框select（单选和多选）

```html
<div id="app">
    <!-- 单选 -->
    <select name="select" v-model="fruit">
        <option value="苹果">苹果</option>
        <option value="香蕉">香蕉</option>
        <option value="西瓜">西瓜</option>
        <option value="葡萄">葡萄</option>
    </select>
    <p>喜欢的水果是：{{fruit}}</p>

    <!-- 多选 -->
    <select name="select" v-model="fruits" multiple>
        <option value="苹果">苹果</option>
        <option value="香蕉">香蕉</option>
        <option value="西瓜">西瓜</option>
        <option value="葡萄">葡萄</option>
    </select>
    <p>喜欢哪些水果：{{fruits}}</p>
</div>
```

```javascript
var app = new Vue({
    el: "#app",
    data: {
        fruit: "葡萄",
        fruits: []
    }
});
```



#### v-once

> 使用该指令表示元素和组件只渲染一次，不会随着数据的变化而变化


```HTML
<div id="app" v-once>{{ message }}</div>
```

```JavaScript
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
```



#### v-html

> 使用该指令可以把字符串的数据在插值中显示时自动转换为HTML格式


```HTML
<div id="app" v-html="url">{{ url }}</div>
```

```JavaScript
var app = new Vue({
  el: '#app',
  data: {
    url: "<a href='https://www.baidu.com'>百度一下</a>"
  }
})
```



#### v-pre

> 该指令可以让插值语法不做任何解析，将原本的数据进行原封不动的展示


```HTML
<div id="app" v-pre></div>
```

```JavaScript
var app = new Vue({
  el: '#app',
  data: {
    message: "不生效"
  }
})
```

