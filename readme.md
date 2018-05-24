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





