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

### 第二章：数据存取

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
| ---- |:-------------------------------------------:| -----------------:|
| 1	   | emp.__proto__	                            | Employee.prototype |	Employee()构造函数的原型对象 |
| 2    |	emp.__proto__.__proto__	                | Person.prototype   |	Person()构造函数的原型对象 |
| 3	   | emp.__proto__.__proto__.__proto__	        | Object.prototype   |	Object()构造函数的原型对象 |
| 4	   | emp.__proto__.__proto__.__proto__.__proto__|	null            |	原型链的顶端 |


| Tables        | Are           | Cool  |

| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |







