## High-Perfomance-JavaScript

> 高性能JavaScript的简要笔记

- **第1章：加载和执行**
- **第2章：数据访问** 
- **第3章：DOM编程**
- **第4章：算法和流程控制**
- **第5章：字符串和正则表达式**
- **第6章：快速响应的用户界面**
- **第7章：Ajax**
- **第8章：编程实践**
- **第9章：构建并部署高性能JavaScript应用**
- **第10章：工具**

---

### 第1章：加载和执行

- 1.由于javascript的下载和执行，会阻塞页面的渲染，因此推荐将script放到</body>前面.
- 2.其中ie支持的`defer`可以不修改DOM，延迟执行脚本。`async`与`defer`相同点是并行下载，下载过程中都不会产生阻塞，但`async`加载完成后执行，`defer`页面加载完成后执行。

```java
<script type="text/javascript" src="file.js" defer></script>
```
- 3.加载一次script执行一次`HTTP`请求，所以尽量减少script文件数，减少HTTP请求次数。
- 4.动态脚本加载，放在`head`里，是通用的无阻赛加载解决方案。
```java
    var script=document.createElement("script");
    script.type="text/javascript";
    script.src="jquery.js";
    document.getElementsByTagName('head')[0].appendChild(script); 

    script.onload=function(){方法内容}//js加载完成执行方法
```
- 5.使用XHR对象下载Javascript代码并注入页面中。

### 第2章：数据存取

- 1.访问函数中的局部变量最快，全局变量最慢。由于全局变量在作用域链最后位置，访问最深，所以速度最慢。
- 2.访问全局变量将遍历整个作用域链，减少全局变量的使用，方法:使用局部变量存储全局变量。
- 3.`with`和`catch`会提升作用域链，可能将全局变量提升到局部变量之前，反而没得到提升还可能降低性能。少用with，适当catch，catch中可以用委托函数处理的方式来不改变性能。
- 4.闭包中包含了执行环境作用域链相同的应用，需要更多的内存开销，容易导致内存泄漏。将常用的跨作用域存储在局部变量中。
- 5.原型链的理解：
```java
function Person(){
    this.name = "Tom";
    this.sayHello = function(){
        return "Hello "+this.name;
    }
}
function Employee(email){
    this.email = email;
}
//Employee.prototype上定义属性和方法，可以被Emplyee实例使用
var person = new Person();
//实现了原形继承，Employee可以使用Person类的属性和方法
Employee.prototype = person;

//总结：原型继承本质是将构造函数的原型对象指向另外一个构造函数创建的实例
//Employee.prototype = new Person()
//Employee()构造函数继承了Person()构造函数
var emp = new Employee('keepfool@xxx.com');

```
> Employee()构造函数的原型引用了一个由Person()构造函数创建的实例，从而建立了Employee()和Person()的继承关系。

> 当设置了`Employee.prototype = new Person();`Employee的实例的`constructor`指向了Person()的构造函数。构造函数被改写了。

> 解决方法：`Employee.prototype.constructor = Employee`

**原型链**

| 编号 | 原型链                                      |	原型链指向的对象  |	描述  |
| :----: | :------------------------------------------ | :----------------- | :---- |
| 1	   | emp.__proto__	                            | Employee.prototype |	Employee()构造函数的原型对象 |
| 2    | emp.__proto__.__proto__	                | Person.prototype   |	Person()构造函数的原型对象 |
| 3	   | emp.__proto__.__proto__.__proto__	        | Object.prototype   |	Object()构造函数的原型对象 |
| 4	   | emp.__proto__.__proto__.__proto__.__proto__|	null            |	原型链的顶端 |

- 6.对象在原型链中存在的位置越深，访问速度就会越慢。其中点操作符同理，`location.href`比`window.location.href`要快。因此加快速度，少用或者存于局部变量之中。

**总结**

> 1.访问字面量和局部变量的速度最快，相反，访问数组元素和对象成员相对较慢。

> 2.访问局部变量比访问跨作用域更快，变量在作用域月深，访问时间越长，全局变量位于最末端，最慢。

> 3.避免使用`with`语句，会改变执行环境的作用域，`try-catch`语句中的catch也是如此，所以小心使用。

> 4.嵌套的对象成员会明显影响性能，少用。

> 5.属性或方法在原型链中的位置越深，访问的速度越慢。

> 6.为了提升速度，通常可以把对象成员、数组元素、跨域变量保存在局部变量中来改善性能。

### 第3章：DOM编程

- 1.Dom的访问与修改，用局部变量存储修改的内容，最后一次性写入，提供性能。
- 2.使用`innerHTML`比原生`DOM`方法更快。
- 3.读取集合的`length`比普通数组的`length`要慢很多。
```java
var coll = document.getElementByTagName('div');
var arr = toArray(coll);

//较慢
function loopCollection(){
    for(var count = 0; count < coll.length; count++){
        /*代码处理*/
    }
}
//较快
function looCopiedArray(){
    for(var count = 0; count < arr.length; count++){
        /*代码处理*/
    }
}
```

- 4.访问集合元素时使用局部变量。
- 5.重绘和重排代价昂贵，减少发生次数，合并多次对DOM和样式的修改，一次处理。防止重排:1.文档之外创建更新一个文档片段，然后附加到原始列表中。2.创建备份，对副本进行操作，然后新节点替代旧节点。
```java
//糟糕的写法，造成三次浏览器重排
var el = document.getElementById('mydiv');
el.style.borderLeft = '1px';
el.sytle.borderRight = '1px';
el.style.padding = '5px'

//较好的写法，一次重拍
var el = document.getElementById('mydiv');
el.style.cssText = 'border-left:1px; border-right:2px; padding:5px';
```
- 6.使用缓存信息

```java
//效率低，每次移动都要查询便宜量
myElement.style.left = 1 + myElement.offsetLeft + 'px';
myElement.style.top = 1 + myElement.offsetTop + 'px';
if(myElement.offsetLeft >= 500){
    stopAnimation()
}

//效率高，缓存偏移量
current++;
myElement.style.left = current + 'px';
myElement.style.left = current + 'px';
if(current >= 500){
    stopAnimation();
}
```

- 7.DOM标准，每个事件经历三个阶段：捕获、达到目标、冒泡。

**事件委托**

事件委托是利用事件冒泡原理(事件从最深的节点开始，向上传播。子节点事件委托父节点代为执行事件)实现。

```html

<ul id="ul1">
    <li>111</li>
    <li>222</li>
    <li>333</li>
    <li>444</li>
</ul>
//Dom操作，找ul->li才能执行操作，每次点击都要执行一次
window.onload = function(){
    var oUl = document.getElementById("ul1");
    var aLi = oUl.getElementsByTagName('li');
    for(var i=0;i<aLi.length;i++){
        aLi[i].onclick = function(){
            alert(123);
        }
    }
}
//采用事件委托的方式，有父节点ul代为执行子节点点击时间
window.onload = function(){
    var oUl = document.getElementById("ul1");
   oUl.onclick = function(){
        alert(123);
    }
}

```









