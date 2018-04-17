---
title: ajax跨域
date: 2018-04-15 13:05:56
tags:
---
# 什么是跨域 #
前台在调用后台服务接口的时候，接口不是同一个域的时候，请求就会失败。
# 产生跨域的原因 #
1、浏览器限制
2、域名不同
3、请求类型是xhr
# 解决思路
## 解除浏览器限制
通过指定参数让浏览器不去校验，但是价值不大，因为这种需要对每个客户端进行处理
## 修改请求类型，保证发出的不是xhr
典型解决办法：jsonp；jsonp有很多弊端，所以现在使用较少
jsonp：通过插入script标签，请求获取回来一段js代码，最后执行回调函数来得到返回值
jsonp的缺点：1、服务器需要改动代码支持

2、只支持get请求，无法满足需求；
3、发送的不是xhr请求，xhr有很多新的特性，异步、各种事件。
## 域名不同
解决办法，支持跨域：被调用方调整，通过后端处理满足跨域需求，这样就可以实现支持跨域；调用方修改，通过代理的方式，实现隐藏跨域的效果。
这是最理想的解决方案
最常见javaee架构
![屏幕快照 2018-04-15 下午1.34.59.png](http://lxlimg.gz.bcebos.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-04-15%20%E4%B8%8B%E5%8D%881.34.59.png)

调用方解决跨域：请求从http服务器发出；原理是是从中间的http服务器转发过去的请求。
反向代理：访问同一域名两个url，最后会去到两个不同的服务器



被调用方解决跨域：请求从客户端发出，在被调用方那边进行设置，允许接口被调用。
原理是基于http协议关于跨域的一些规定，在响应头上加上一些字段（http服务器或者应用服务器中增加），告诉浏览器允许跨域
具体实现方式：1、服务器端实现
跨域请求头会多一个origin当前域的字段
access-control-allow-origin 
access-control-allow-methods 
access-control-allow-contentType
### 简单请求和非简单请求
是不是所有请求都是先请求后判断呢？
如果是简单请求，是先请求后判断
如果是非简单请求，会先发一个预检命令，通过后才会发出请求
简单请求：方法为get、head、post；请求头里无自定义头，content-type为以下几种
text/plain；multipart/form-data；application/x-www-form-urlencoded；

非简单请求：put、delete，发送json格式的ajax请求，带自定义头的ajax请求
由于预检命令导致跨域的非简单请求，每次会发出两条请求，解决办法，
预检命令的缓存：加响应头，access-control-max-age 参数是一个数字秒数，告诉浏览器在这个时间内可以缓存预检命令

### 带cookie的跨域
access-control-allow-origin：*能否满足所有的跨域呢？
不能满足带cookie的跨域请求，必须设置为cookie对应的域名
且需要加access-control-allow-credentials：true
那如何设置多个带cookie的跨域呢？
办法：后台动态获取请求头中的origin，然后传入access-control-allow-origin字段

### 带自定义头的跨域
需要在响应头中设置对应access-control-allow-Headers


2、nginx配置
被调用方nginx的设置，让请求先发送到http服务器，再转至应用服务器

3、apache配置
同nginx
4、spring框架的解决方案
配置crossoigin



两者都可能是对http服务器进行设置，但思路是不一样的
第一种是修改调用方的http服务器，第二种是修改被调用方的http服务器。






