---
title: 阿里2018面试题总结
date: 2018-04-17 20:18:46
tags:
---
## 动画实现方式
HTML5/CSS3时代，我们要在web里做动画选择其实已经很多了:

### 你可以用CSS3的animattion+keyframes;
例子：

`

	animation:mymove 5s infinite;
	@keyframe mymove {
  		from {top:0;}
  		to{top:200px;}
 	}
`

### 你也可以用css3的transition;
例子：
`

	div{
		width:100px;
		transition: width 2s;
		-moz-transition: width 2s; /* Firefox 4 */
		-webkit-transition: width 2s; /* Safari 和 Chrome */
		-o-transition: width 2s; /* Opera */
	}
`

#### 你还可以用通过在canvas上作图来实现动画，也可以借助jQuery动画相关的API方便地实现;

#### 当然最原始的你还可以使用window.setTimout()或者window.setInterval()通过不断更新元素的状态位置等来实现动画，前提是画面的更新频率要达到每秒60次才能让肉眼看到流畅的动画效果。

### window.requestAnimationFrame()方法。
MDN上关于requestAnimationFrame的定义：
The window.requestAnimationFrame() method tells the browser that you wish to perform an animation and requests that the browser call a specified function to update an animation before the next repaint. The method takes as an argument a callback to be invoked before the repaint.

window.requestAnimationFrame() 将告知浏览器你马上要开始动画效果了，后者需要在下次动画前调用相应方法来更新画面。这个方法就是传递给window.requestAnimationFrame()的回调函数。
也可理解这个方法原理其实也就跟setTimeout/setInterval差不多，通过递归调用同一方法来不断更新画面以达到动起来的效果
#### 基本用法
可以直接调用，也可以通过window来调用，接收一个函数作为回调，返回一个ID值，通过把这个ID值传给window.cancelAnimationFrame()可以取消该次动画。

    `requestAnimationFrame(callback)//callback为回调函数`
例子：模拟一个进度条动画，初始div宽度为1px,在step函数中将进度加1然后再更新到div宽度上，在进度达到100之前，一直重复这一过程。

`

   	<div id="test" style="width:1px;height:17px;background:#0f0;">0%</div>
	   <input type="button" value="Run" id="run"/>
		//复制代码
	window.requestAnimationFrame = window.requestAnimationFrame || window.mozRequestAnimationFrame || window.webkitRequestAnimationFrame || window.msRequestAnimationFrame;
	var start = null;
	var ele = document.getElementById("test");
	var progress = 0;

	function step(timestamp) {
    progress += 1;
    ele.style.width = progress + "%";
    ele.innerHTML=progress + "%";
    if (progress < 100) {
        requestAnimationFrame(step);
    }
	}
	requestAnimationFrame(step);
	document.getElementById("run").addEventListener("click", function() {
    ele.style.width = "1px";
    progress = 0;
    requestAnimationFrame(step);
	}, false);
  `
目前主流浏览器均支持，ie10+

优点：
  浏览器可以优化并行的动画动作，更合理的重新排列动作序列，并把能够合并的动作放在一个渲染周期内完成，从而呈现出更流畅的动画效果
  解决毫秒的不准确性
  避免过度渲染（渲染频率太高、tab不可见暂停等等）
  优于setTimeout/setInterval的地方在于它是由浏览器专门为动画提供的API，在运行时浏览器会自动优化方法的调用，并且如果页面不是激活状态下的话，动画会自动暂停，有效节省了CPU开销。

## 常用布局实现
### 右边宽度固定，左边自适应
第一种实现方式：flex定位
    ` 
        
        <style>
          body{
          display: flex;
          }
        .left{
          background-color: rebeccapurple;
          height: 200px;
          flex: 1;
        }
        .right{
          background-color: red;
          height: 200px;
          width: 100px;
        }
      </style>
      <body>
        <div class="left"></div>
        <div class="right"></div>
      </body>`
第二种实现方式：浮动定位实现

 ` 
        
       <style>
          div {
            height: 200px;
          }
          .left {
            float: right;
            width: 200px;
            background-color: rebeccapurple;
          }
          .right {
            margin-right: 200px;
            background-color: red;
          }
      </style>
      <body>
        <div class="left"></div>
        <div class="right"></div>
      </body>`

### 水平垂直居中
第一种实现方式：绝对定位+transform
` 
        
       #container{
          position:relative;
          }
       #center{
          width:100px;
          height:100px;
          position:absolute;
          top:50%;
          left:50%;
          transform: translate(-50%,-50%);
        }`
第二种实现方式：绝对定位+margin
` 
        
       #container{
          position:relative;
        }
       #center{
          width:100px;
          height:100px;
          position:absolute;
          top:50%;
          left:50%;
          margin:-50px 0 0 -50px;
      }`
      
第三种实现方式：绝对定位
` 
        
       #container{
          position:relative;
        }
       #center{
          position:absolute;
          margin:auto;
          top:0;
          bottom:0;
          left:0;
          right:0;
       }`
第四种实现方式：flex定位
` 
        
       #container{
          display:flex;
          justify-content:center;
          align-items: center;
        }`
        
### 四种地位方式的区别
static 是默认值；
relative 相对定位 相对于自身原有位置进行偏移，仍处于标准文档流中；
absolute 绝对定位 相对于最近的已定位的祖先元素, 有已定位(指 position不是 static的元素)祖先元素, 以最近的祖先元素为参考标准。如果无已定位祖先元素, 以 body元素为偏移参照基准, 完全脱离了标准文档流；
fixed 固定定位的元素会相对于视窗来定位,这意味着即便页面滚动，它还是会停留在相同的位置。一个固定定位元素不会保留它原本在页面应有的空隙。
## es6小问题
### let和var的区别
var 声明的变量，其作用域为该语句所在的函数内，且存在变量提升现象
let 声明的变量，其作用域为该语句所在的代码块内，不存在变量提升，且不允许重复声明
### 为什么var可以重复声明
当我们执行代码时，我们可以简单的理解为新变量分配一块儿内存，命名为 a，并赋值为 2，但在运行的时候编译器与引擎还会进行两项额外的操作：判断变量是否已经声明：

首先编译器对代码进行分析拆解，从左至右遇见 var a，则编译器会询问作用域是否已经存在叫 a 的变量了，如果不存在，则招呼作用域声明一个新的变量 a，若已经存在，则忽略 var 继续向下编译，这时 a = 2被编译成可执行的代码供引擎使用。
引擎遇见 a=2时同样会询问在当前的作用域下是否有变量 a，若存在，则将 a赋值为
2（由于第一步编译器忽略了重复声明的var，且作用域中已经有 a，所以重复声明会发生值的覆盖而并不会报错）。若不存在，则顺着作用域链向上查找，若最终找到了变量 a则将其赋值 2，若没有找到，则招呼作用域声明一个变量 a并赋值为 2。
### 封装一个函数，参数是定时器的时间，.then执行回掉函数
` 
        
       function sleep (time) {
        return new Promise((resolve) => setTimeout(resolve, time));
      }`
      
### 关于this指向的问题
` 
        
      obj = {
        name: 'a',
        getName : function () {
          console.log(this.name);
        }
      }
      var fn = obj.getName
      obj.getName()//a
      var fn2 = obj.getName()//a
      fn()//undefined
      fn2()`//报错
### CommonJS 中的 require/exports 和 ES6 中的 import/export 区别？
CommonJS 模块的重要特性是加载时执行，即脚本代码在 require 的时候，就会全部执行。一旦出现某个模块被”循环加载”，就只输出已经执行的部分，还未执行的部分不会输出；

ES6 模块是动态引用，如果使用 import 从一个模块加载变量，那些变量不会被缓存，而是成为一个指向被加载模块的引用，需要开发者自己保证，真正取值的时候能够取到值；
import/export 最终都是编译为 require/exports 来执行的；

CommonJS 规范规定，每个模块内部， module 变量代表当前模块。这个变量是一个对象，它的 exports 属性（即module.exports ）是对外的接口。加载某个模块，其实是加载该模块的 module.exports 属性；

export 命令规定的是对外的接口，必须与模块内部的变量建立一一对应关系。
### 一行代码实现数组去重
` 
    [...new Set([1,2,3,1,'a',1,'a'])]`
### 使用addEventListener点击li弹出内容，并且动态添加li之后有效
` 
        
        <ul>
          <li>1</li>
          <li>2</li>
          <li>3</li>
          <li>4</li>
        </ul>
        var ulNode = document.getElementById("ul");
        ulNode.addEventListener('click', function (e) {
          if (e.target && e.target.nodeName.toUpperCase() == "LI") {
            alert(e.target.innerHTML);
          }
        }, false);`
        
### 怎么判断两个对象相等
可以通过JSON.stringify将两个对象专成字符串再进行比较
### 项目中性能优化的常用方式
减少 HTTP 请求数
减少 DNS 查询
使用 CDN
避免重定向
图片懒加载
减少 DOM 元素数量
减少 DOM 操作
使用外部 JavaScript 和 CSS
压缩 JavaScript 、 CSS 、字体、图片等
优化 CSS Sprite
使用 iconfont
字体裁剪
多域名分发划分内容到不同域名
尽量减少 iframe 使用
避免图片 src 为空
把脚本放在页面底部 
### 模块化开发是怎么做的
 使用命名空间
### Set和Map的数据结构
ES6 提供了新的数据结构 Set 它类似于数组，但是成员的值都是唯一的，没有重复的值。
ES6 提供了 Map 数据结构。它类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键。也就是说， Object 结构提供了“字符串—值”的对应， Map 结构提供了“值—值”的对应，是一种更完善的 Hash结构实现。
### weakMap和Map的区别
WeakMap 结构与 Map 结构基本类似，唯一的区别是它只接受对象作为键名（ null 除外），不接受其他类型的值作为键名，而且键名所指向的对象，不计入垃圾回收机制。
WeakMap 最大的好处是可以避免内存泄漏。一个仅被 WeakMap 作为 key 而引用的对象，会被垃圾回收器回收掉。
WeakMap 拥有和 Map 类似的 set(key, value) 、 get(key)、has(key)、 delete(key) 和 clear() 方法, 没有任何与迭代有关的属性和方法。
### 重排和重绘
部分渲染树（或者整个渲染树）需要重新分析并且节点尺寸需要重新计算。这被称为重排。注意这里至少会有一次重排-初始化页面布局。
由于节点的几何属性发生改变或者由于样式发生改变，例如改变元素背景色时，屏幕上的部分内容需要更新。这样的更新被称为重绘。
### 什么情况会触发重排和重绘？
添加、删除、更新 DOM 节点
通过 display: none 隐藏一个 DOM 节点-触发重排和重绘
通过 visibility: hidden 隐藏一个 DOM 节点-只触发重绘，因为没有几何变化
移动或者给页面中的 DOM 节点添加动画
添加一个样式表，调整样式属性
用户行为，例如调整窗口大小，改变字号，或者滚动。
## 前端框架的小问题
### webpack和gulp的基本了解
### vue router跳转和location.href 有什么区别
router是hash改变，location.href是页面跳转，刷新页面
### vue双向绑定实现原理
通过object.defineProperty实现的
### 你能实现一下双向绑定吗？
` 
        
        <body>
          <div id="app">
            <input type="text" id="txt">
            <p id="show-txt"></p>
          </div>
          <script>
            var obj = {}
            Object.defineProperty(obj, 'txt', {
              get: function () {
                return obj
              },
              set: function (newValue) {
                document.getElementById('txt').value = newValue
                document.getElementById('show-txt').innerHTML = newValue
              }
            })
            document.addEventListener('keyup', function (e) {
              obj.txt = e.target.value
            })
          </script>
        </body>`
## 浏览器缓存
浏览器缓存分为强缓存和协商缓存。当客户端请求某个资源时，获取缓存的流程如下：

先根据这个资源的一些 http header 判断它是否命中强缓存，如果命中，则直接从本地获取缓存资源，不会发请求到服务器；
当强缓存没有命中时，客户端会发送请求到服务器，服务器通过另一些 request header验证这个资源是否命中协商缓存，称为 http再验证，如果命中，服务器将请求返回，但不返回资源，而是告诉客户端直接从缓存中获取，客户端收到返回后就会从缓存中获取资源；
强缓存和协商缓存共同之处在于，如果命中缓存，服务器都不会返回资源；
区别是，强缓存不对发送请求到服务器，但协商缓存会。
当协商缓存也没命中时，服务器就会将资源发送回客户端。
当 ctrl+f5 强制刷新网页时，直接从服务器加载，跳过强缓存和协商缓存；
当 f5 刷新网页时，跳过强缓存，但是会检查协商缓存；
强缓存

Expires（该字段是 http1.0 时的规范，值为一个绝对时间的 GMT 格式的时间字符串，代表缓存资源的过期时间）
Cache-Control:max-age（该字段是 http1.1 的规范，强缓存利用其 max-age 值来判断缓存资源的最大生命周期，它的值单位为秒）
协商缓存

Last-Modified（值为资源最后更新时间，随服务器response返回）
If-Modified-Since（通过比较两个时间来判断资源在两次请求期间是否有过修改，如果没有修改，则命中协商缓存）
ETag（表示资源内容的唯一标识，随服务器response返回）
If-None-Match（服务器通过比较请求头部的If-None-Match与当前资源的ETag是否一致来判断资源是否在两次请求之间有过修改，如果没有修改，则命中协商缓存）  