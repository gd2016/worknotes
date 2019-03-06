###倒计时问题
####出现环境
IOS客户端Webview中
####错误写法
````
<body>
	倒计时容器
</body>
````
#### 错误的表现形式
1.在滚动body的时候会阻塞js代码的执行，一旦滚动停止，计时器会迅速跳转到相应的位置，例如：
当滚动开始的时候定时器显示时间为00:00:11, 滚动操作耗时了20秒，那么在滚动过程中定时器将始终停留在11秒的时候，一旦滚动停止，定时器会迅速跳转到31秒的时刻

###解决方案

````
<body style="overflow:hidden">
	<div style="overlow: scroll"> //滚动容器
		内容
		定时器
		内容
	</div>
</body>
````
####解释
由于在webview中 事件的执行顺序是 优先响应touch事件，此时会暂停所有的js代码的执行，这是苹果系统的机制，所以在我们做滚动的时候最好是禁止body的滚动，而去滚动内部的元素，这样就避免了webview的滚动，从而避免了Ios系统在滚动性能上的考虑

## iOS中使用iframe后子页面fixed失效

### 存在问题
在iOS中使用iframe后，如果子页面高度远远大于父页面高度，子页面使用`fixed定位`的弹窗会出现`定位失效`的情况，弹框的位置只是相对于子页面来定位，并没有相对于视口定位。

### 解决方案
子页面的`样式`需要对应的`进行调整`:
```Javascript
html {
	position: relative;
	width: 100%;
	height: 100%;
	overflow: hidden;
}
body {
	position: fixed;
	top: 0;
	left: 0;
	width: 100%;
	height: 100%;
	overflow: scroll;
}
```

## iOS中使用iframe后子页面无法进行滚动

## 基础

cros跨域 需要服务端配置相应属性

## 普通模式
```
Access-Control-Allow-Origin :* 
```
## 自定义headers模式
```
Access-Control-Allow-Origin :* 
Access-Control-Allow-Methods :GET, POST, PUT, DELETE, OPTIONS 
Access-Control-Allow-Headers :key  //自定义无法使用'*'需要添加完全
```
## 疑问
什么时候会有 OPTIONS 请求？
原来，产生 OPTIOINS 请求的原因是：自定义 Headers 头信息导致的。设置完自定义 header 字段后，问题就出现了：原来的简单请求会变成预检请求。

## 结果

禁止使用 getheaderinfo 直接发给后端 ，使用循环赋值的形式

[TOC]
###  background-image
1. 洪丹-A035-iPhone se-9.3.2 手机对于 background-image属性支持不友好。
	![](http://doc.uc108.org:8002/Public/Uploads/2018-11-13/5bea3999e68bd.png)
	- 解决：需要使用img进行替换。

2. 小米手机对于图片背景会出现一条缝隙。
	- 解决：需要设置 background-size:99% 或者 可以把后边的div的margin-top设一个很小的负数;

###  height、line-height
1. 所有手机对于部分元素（目前是p元素，其他元素还待确认）设置了height、和line-height后呈现不一样的效果。
	- 解决：不使用P元素，直接使用div元素且不设置height和line-height，直接对div元素设置padding值即可。

### es6语法兼容
1. 华为自带浏览器不支持es6语法；
	- 解决：main.js 需要引入  import 'babel-polyfill';

### 复制到剪切板
5. ios8、9都不支持复制到剪切板的功能：
	- 解决：提示用户手动复制。

### 数据埋点被屏蔽（不解决）
1. 【vivo Y66】自带的浏览器会屏蔽测试版的数据埋点，正式版不会被其屏蔽。

### 无法启动同城游（不解决）
1. 启动同城游相关的问题（浏览器自身的问题，这个没有办法处理的）
	- uc浏览器版本：V12.1.6.996（当前算是最新版本）无法启动同城游（偶发性能启动），低于这个版本的都能启动。看了新版uc浏览器官网对于这种主动唤醒应该做了严格的限制，具体说明可查看http://kf.uc.cn/self_service/web/faqdetails-8311413_8311645_20748356_3.html 。所以初步判定属于uc浏览器版本更新后自身的兼容性问题。
	- 【oppo A57】自带浏览器并不支持自动唤醒第三方app。
	- 唤醒启动同城游app数据统计存在误差。
  
  
  ios受fastclick这个类库影响造成的，需要重写fastclick的foucs方法：
https://www.jianshu.com/p/5b578e656966

