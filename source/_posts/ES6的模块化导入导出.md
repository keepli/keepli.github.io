---
title: ES6的模块化导入导出
date: 2020-10-30
updated: 2020-10-30
tags: es6
categories: 前端
---

### ES6的模块化导入导出

- export导出的变量/常量，在import导入的时候<font color=red>需要</font>知道导出时候的名称（必须指定）
- export default导出的变量/常量，在import导入的时候<font color=red>不需要</font>指定导出时候的名称（可以自定义），<span style="background-color:yellow">但是一个js文件中只能存在一个default</span>

**aaa.js文件：**

- 导出数据

```js
var flag = true

function sum(num1,num2){
    return num1 + num2;
}

// 1.导出方式一（一次性导出多个）
export {
    flag, sum
}

// 2.导出方式二（一次导出单个）
// 2.1导出变量
export var num = 1000

// 2.2导出函数
export function mul(num1,num2){
    return num1 + num2;
}

// 2.3导出类
export class Person {
    run() {
        console.log("person")
    }
}

// 3.导出方式三（导出default，注意：一个js文件中只有一个default）
//export default var test = "default测试" （这样写是错误的，去掉default是可以的）

var test = "default测试"
export default test
```

**bbb.js文件：**

- 导入数据

```js
// 1.导入的{}中定义的变量
import {flag,sum} from './aaa.js';
if (flag) {
    console.log('输出')
    console.log(sum(20,30))
}

// 2.直接导入export定义的变量
import {num} from './aaa.js';
console.log(num)

// 3.导入export的function
import {mul} from './aaa.js';
console.log(mul(10,10))

// 4.导入export的class
import {Person} from './aaa.js';
new Person().run()

// 5.导入export的default（可以自己指定命名）
import c from './aaa.js';
console.log(c)

// 6.统一全部导入（将所有的导入数据存储到aaa对象中）
import * as aaa from './aaa.js';
console.log(aaa.flag)
```
**ccc.html文件：**

- 使用js文件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <script src="./bbb.js" type="module"></script>  
</body>
</html>
```

**可以看到ccc.html文件中正常输出：**

<img src="https://img-blog.csdnimg.cn/img_convert/734f5ccd6cdbefc85db7775ccabe24a9.png" style="zoom: 80%;" />