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

> <script type="text/javascript" src="file.js" defer></script>

- 3.加载一次script执行一次`HTTP`请求，所以尽量减少script文件数，减少HTTP请求次数。
- 4.动态脚本加载，放在`head`里，是通用的无阻赛加载解决方案。

```javascript

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

```javascript
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

```javascript
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

```javascript
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

```javascript
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
//采用事件委托的方式，有父节点ul代为执行子节点点击事件

window.onload = function(){
    var oUl = document.getElementById("ul1");
   oUl.onclick = function(){
        alert(123);
    }
}

//采用如上事件委托问题：当点击ul的时候也会触发事件，就用到了Event的target
window.onload = function(){
　　var oUl = document.getElementById("ul1");
　　oUl.onclick = function(ev){
　　　　var ev = ev || window.event;
　　　　var target = ev.target || ev.srcElement;
　　　　if(target.nodeName.toLowerCase() == 'li'){
　 　　　　　　	alert(123);
　　　　　　　  alert(target.innerHTML);
　　　　}
　　}
}

```
**如果遇到每个li的点击事件不同，则采用如下代码：**

```html
//采用四次Dom操作方式
<div id="box">
        <input type="button" id="add" value="添加" />
        <input type="button" id="remove" value="删除" />
        <input type="button" id="move" value="移动" />
        <input type="button" id="select" value="选择" />
    </div>
window.onload = function(){
    var Add = document.getElementById("add");
    var Remove = document.getElementById("remove");
    var Move = document.getElementById("move");
    var Select = document.getElementById("select");
    
    Add.onclick = function(){
        alert('添加');
    };
    Remove.onclick = function(){
        alert('删除');
    };
    Move.onclick = function(){
        alert('移动');
    };
    Select.onclick = function(){
        alert('选择');
    }
    
}

//事件委托优化方式，采用一次Dom操作，效果更好
window.onload = function(){
    var oBox = document.getElementById("box");
    oBox.onclick = function (ev) {
        var ev = ev || window.event;
        var target = ev.target || ev.srcElement;
        if(target.nodeName.toLocaleLowerCase() == 'input'){
            switch(target.id){
                case 'add' :
                    alert('添加');
                    break;
                case 'remove' :
                    alert('删除');
                    break;
                case 'move' :
                    alert('移动');
                    break;
                case 'select' :
                    alert('选择');
                    break;
            }
        }
    }
}
```
### 第3章：算法和流程控制

- 1.JS循环体有`for`,`while`,`do while`,`for in`四种，其中`for in`循环体速度比其他慢，其他循环体之间的速度差异不大。
```java
for(var i = 0; i < item.length; i++){
    process(item[i]);
}

var j = 0;
while(j < items.length){
    process(item[j++]);
}

var k = 0;
do{
    process(items[k++]);
}while(k < items.length);
```
- 2.优化循环减少对象成员及数组项的查找次数，保存在局部变量中。
```
for(var i = 0;len = items.length; i < len; i++){
    process(items[i]);
}
```

- 3.循环中使用倒叙的方式可以提神性能，同时，减少属性查找（赋值到局部变量）。
- 4.达夫设备，每次循环中最多调用八次循环，同时存放余数。

```javascript
var i = items.length % 8;
while(i) {
  process(items[i--]);
}
 
i = Math.floor(items.length / 8);
 
while(i) {
  process(items[i--]);
  process(items[i--]);
  process(items[i--]);
  process(items[i--]);
  process(items[i--]);
  process(items[i--]);
  process(items[i--]);
  process(items[i--]);
}
```

- 5.基于函数的迭代`forEach`,接受三个参数：当前数组值value、索引index、数组本身array。循环迭代比函数迭代快。
```java
items.forEach(function(value, index, array){
    process(value);
})
```
- 6.`switch`比`if-else`运行的快，优化if-else，最有可能出现的情况放在前面。

### 第5章：字符串和正则表达式

- 1.字符串拼接，避免产生临时字符串来提升性能。

```javascript
//第一行会产生临时字符串存储“onetwo”；
//用两行的方式避免了产生临时字符串，从而提升性能；
str += "one" + "two";

str += "one";
str += "two";

```
- 2.数量巨大和尺寸巨大的字符串连接时，数组项合并是推荐的方法。
- 3.数组项合并是最慢的字符串连接方法，推荐使用`+`和`+=`。

- 4.正则表达式的分支和回溯（是低效率之源）

> /h(ello|appy) hippo/.test("hello there, happy hippo");

过程：匹配开始，首先查找`h`，接下来，子表达式`(ello|appy)`提供了两个处理选项，检查`hello`匹配成功，然后匹配空格，由于`hippo`中的`h`无法匹配`t`，匹配无法继续，回溯到最近的点，匹配首字母`h`后面的第二个分支，失败。知道匹配到`happy hippo`。

- 5.正则表达式不是最好的工具，例如在搜索字面字符串的时候。
- 6.用正则表达式来去掉字符串首尾空白，是最好的方式。

### 第6章：快速响应的用户界面

- 1.Js任务执行超过`100ms`，用户会感觉到明显的延迟，体验感不好。
- 2.Js定时器让出UI线程的控制权(UI)可以更新。函数`greeting`将在250ms后执行，在此期间UI更新和其他Js可以执行。

```javascript
function greeting(){
    alert("Hello world");
}
setTimeOut(greeting, 250);
```
- 3.Javascript定时器不太精确，通常与几毫秒，推荐延时25ms。
- 4.如果函数运行时间太长，用定时器进行分割任务。

```javascript
//文件操作
function saveDocument(id){
    openDocument(id);
    writeText(id);
    closeDocument(id);
    //更新界面
    updateUI(id);
}

//定时器分割任务
function saveDocument(id){
    var tasks = [openDocument, writeText, closeDocument, updateUI];
    setTimeout(function(){
        //执行下个任务
        var task = tasks.shift();
        task(id);

        //检查是否还有其他任务
        if(tasks.length > 0){
            //松耦合递归调用
            setTimeout(arguments.callee, 25);
        }
    },25)
}

//将以上方法重写，以便复用
function multistep(steps, args, callback){
    //克隆数组
    var tasks = steps.concat();
    setTimeout(function{
        //执行下一个任务
        var task = tasks,shift();
        task.apply(null, args || []);
        //检查是否还有其他任务
        if(tasks.length > 0){
            setTimeout(arguments.callee, 25);
        }else{
            callback();
        }
    })
}

//调用
function saveDocument(id){
    var tasks = [openDocument, writeText, closeDocument, updateUI];
    multistep(tasks,[id],function{
        alert("Save completed!")
    })
}
```
- 5.`Web works`新版浏览器的新特性，允许在UI线程外执行js代码。


### 第7章：Ajax

- 1.Ajax特性：1.延迟下载体积大的资源文件，使页面加载速度快。2.异步方式在客户端和服务端传输数据。3.只有一个HTTP请求获取整个页面的资源。
- 2.高性能的向服务器请求数据的三种技术：XHR、动态脚本注入、multipart XHR.
- 3.XHR(XHLHttpRequest)异步发送和接收数据。`readState == 4`:整个响应接受完毕，`==3`正在和服务器交互之中。可`get`和`post`。

```javascript
var url = '/data.php'
var params = [
    'id = 934875',
    'limit = 20'
];
var req = new XMLHttpRequest();
req.onreadystatechange = function(){
    if(req.readyState === 4){
        var responseHeaders = req.getAllResponseHeaders();//获取响应头信息
        var data = req.responseText;//获取数据
        //todo 数据处理
    }
}
req.open('GET', url + '?' + params.join('&'), true);
req.setRequestHeader('X-Requested-With', 'XMLHttpRequest');//设置请求头信息
req.send(null);//发送一个请求
```

- 4.动态脚本注入：能够跨域请求，控制有限，只能使用`GET`方式。
- 5.Multipart XHR最新技术，客户端使用一个`HTTP`可以从服务端向客户端传送多个资源，将多个文件打包成双方约定的长字符串，然后进行解析。

```javascript
//将一个获取多张图片的请求发送到服务器
var req = new XMLHttpRequest();
req.open('GET','rollup_images.php',true);
req.onreadystatechage = function(){
    if(req.readystate == 4){
        splitImages(req.responseText);
    }
}
req.send(null);
//服务器读取图片并将它们转换为字符串
$images = array('kitten.jpg', 'sunset.jpg', 'baby.jpg');
foreach($images as $image){
    $image_fh = fopen($image, 'r');
    $image_data = fread($image_fh, filesize($image));
    fclose($image_fh);
    $payloads[] = base64_encode($image_data);
}
$newline = chr(1);
echo implode($newline, $payloads);

//数据传送到客户端并解析
function splitImages(imageString){
    var imageData = imageString.split("\u0001");
    var imageElement;

    for(var i = 0; len = imageData.length; i < len; i++){
        imageElement = document.createElement('img');
        imageElement.src = 'data:img/jpg;base64,' + imageData[i];
        document.getElementById('container').appendChild(imageElement);
    }
}
``` 
- 6.使用XHR发送数据到服务器，`GET`会更快，`GET`向服务器只发送一个数据包，而`POST`至少要发送两个数据包，post适合发送大量数据包到服务器。
- 7.xml优点：格式统一，符合标准，容易交互，数据共享方便。缺点：文件庞大，格式复杂，占用带宽，不同浏览器解析xml方式不一样，浪费资源和时间。
- 8.json优点：轻量级，易读写，占用宽带小，支持多种语言，易开发，易维护。其缺点：没有xml通用性好。

### 第8章：编程实践

- 1.通过避免使用`eval()`和`Function()`构造器来避免双重求值带来性能消耗。`setTimeout()`和'setInterval()'不建议传递字符串。
- 2.创建对象和数组使用直接量的效率会大大提升。


```javascript
//以下为传统方式非直接量
//创建一个对象
var myObject = new Object();
myObject.name = "Tom";
myObject.count = 50;
myObject.flag = true;
myObject.pointer = null;

//创建一个数组
var myArray = new Array();
myArray[0]  = "Tom";
myArray[1] = 50;
myArray[2] = true;
myArray [3] = null;

//------------------------
//使用直接量的方创建
var myObject = {
    name : "Tom",
    count : 50;
    flag = true,
    pointer = null
};                                                                                                                                      
var myArray = ["Tom"m 50, true, null];

```

- 3.避免重复工作，当需要检测到浏览器时，可以使用延迟加载和条件预加载。
- 4.在数学计算时，考虑使用直接操作数字的二进制形式的位运算。

> var num = 25; num.toString(2); //"110001"

- 5.使用js原生方法比任何代码都要快，特别是数学运算，使用`Math`对象。

### 第9章：构建并部署高性能JavaScript应用

- 1.合并javascript文件来减少HTTP请求。
- 2.使用YUI Compressor压缩javascript文件。
- 3.在服务端压缩javascript文件（GZip编码）。
- 4.设置HTTP响应头来缓存javascript文件，通过文件名增加时间戳来避免缓存问题。
- 5.使用CDN（分发网络）提高性能，同时可以文件管理和压缩。

---

注：完结，第十章工具省略，总体大概浏览，要更进一步的掌握需每个知识点进行研读和查阅相关资料，并测试coding。






