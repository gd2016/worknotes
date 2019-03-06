[TOC]

# 一、开发指南补充
## 1、UI设计要求
1.  需要有【UI标注】文件夹：字号、颜色、行高（只对段落有效）、关键区域的大小标注；
2.  设计稿需要对于文字的长度做最大的兼容设计；
3.  psd文件大小在10M以内；
4.  psd文件图层分组清晰；
5. 特殊的图层分离需和设计沟通协商。

## 2、前端组测试相关
1.【测试机子】锁屏密码`123456`，目前有的测试机型：华为荣耀edi-al10、iPhone 6s、红米、OPPO r9s；
2.【公用微信】账号`156-6902-6846`，密码`qazwsx123456789`；
3.【tcyapp安装包相关】：
```
FTP路径：
主机：192.168.101.103 
端口：12018
协议：FTP-文件传输协议
加密：要求隐式的FTP over TLS
登录类型：正常
用户：ceshi
密码：2J9#BU8V6uDm
目录：D:/www1
```
```
iOS同城游包：/www1/iOS_new/归档包/adhoc/
Android安装包：/www1/Android/版本归档/debug包_1507
  ● debug是测试包
  ● release 是正式包
  ● preform是预发包
```
```
对应的安装的apk的包是直接找【项目经理】提供的。
对应的ios包是需要先去ftp上下载包，然后搜索具体的游戏，进行安装。
如果是对接游戏方，需要找【测试】或者【游戏方】协助发布zip包。
```
4.【测试连接网络】：
```
账号jiechi，密码ctwlapptest
账号CTWL-7F-Public，密码ctwl0007
```
5.【兼容】：
```
版本要求：
iOS兼容的版本：最低8；
Android 兼容的版本：最低4.4；
其余按照产品的文档要求兼容具体的版本。
```
【手机兼容在线测试地址】暂时访问不了： http://doc.uc108.org:8002/index.php?s=/168&page_id=1564

6.【服务代号查询】： http://nuget.ct108.net:8086/ 
 服务列表-服务名称（如：邀请有礼）-点击搜索-得到服务代号
 
# 二、邀请有礼V2.2开发总结
## 1、简述
 html5页面由游戏分享链接或海报到微信好友或朋友圈，在微信浏览器和外部浏览器有不同的页面逻辑， 微信里面主要负责微信授权、获取用户信息、用户关系绑定以及领取奖励，外部浏览器则负责跳转下载页面或直接打开app。
 
## 2、技术说明
  
  - vue + vux
  - 微信授权（同城游小助手)
  - base64解码
 
## 3、问题整理
### (1) 微信授权
 
 页面上需增加授权的脚本如下：
 
 ```
 /*
         ** test: sso1.tcysys.tcy365.com
         ** release: sso.tcysys.tcy365.com
         */
        var constUrl = 'http://sso1.tcysys.tcy365.com/wxcommauthmvc/openForUnionId?url=';

        function isWechat() {
            var ua = navigator.userAgent.toLowerCase();
            if (ua.match(/MicroMessenger/i) == "micromessenger") {
                return true;
            } else {
                return false;
            };
        };

        function getQueryString(name) {
            var reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)");
            var r = window.location.search.substr(1).match(reg);
            if (r != null) return unescape(r[2]);
            return null;
        };

        if (getQueryString('unionId') === null) {
            var getUnionIdUrl = constUrl + encodeURIComponent(window.location.href);
            window.location.href = getUnionIdUrl;
        };
 ```
 ```
 说明：授权主要是后端来配置（constUrl这个常量需要与后端进行确认），我们这里的脚本处理的是利用链接重定向进行页面跳转。也就是说如果微信授权了就可以拿到unionId值进行页面跳转，其次如果未授权就获取不到unionId值页面就空白。其他情况，未授权就直接在浏览器打开的话，微信会自动判断并给与提示【请在微信客户端打开】。
```
 
### (2) 与游戏方对接关键
  解析url地址中的参数，获取约定的参数。
假设游戏方分享出来的连接示例为：` url?data=.. data .. &signkey= .. my.md5(123456 .. my.md5(data .. 123456))`
如何解析？按照以下步骤：
     - a.获取url中的data参数：`const urlParamData = getUrlParam('data')`
     - b.先解码再使用vux中的base64工具函数解码并将其转为对象：`const fromGameData =JSON.parse(base64.decode(decodeURIComponent(urlParamData)));`
	 

### (3) 复制到剪切板兼容性
  
  安装插件：`cnpm install vue-clipboard2 -D`
iOS9及其以下都不兼容此插件，Android4.4及其以上都支持该插件。
注意事项：
- a.对于不支持的可进行手动复制处理，这里需要注意是h5是调取不到系统中的剪切板中的内容的。
- b.不支持动态拼接的数据到剪切板

### (4) 数据埋点数据统计缺失
  注意两点：
- a.统计代码配置在页面上所有脚本之前

```
    <div style="display:none">
        <script src="https://s19.cnzz.com/z_stat.php?id=1274862953&web_id=1274862953" language="JavaScript"></script>
    </div>
```

- b.统计代码需要try，catch捕获下，以免发生错误

```
try {
        // _czc.push(['_trackEvent', name, action]);
        _czc.push(['_trackEvent', category, action, unionid]);
        console.log('数据埋点：' + category + '+' + action + '+' + unionid);
    } catch (error) {
        console.log('埋点error信息：');
        console.log(error);
    }
```

### (5) 需要整理的公用组件
  
- 环境判断，如微信、浏览器、Android、iOS
- 数据埋点
- 横屏判断通用处理
- 通用的toast、通用的弹窗二次封装
- 公用的错误判断

[TOC]

## 已解决
1. 洪丹-A035-iPhone se-9.3.2 手机对于 background-image属性支持不友好。
	- 解决：需要使用img进行替换。
2. 小米手机对于图片背景会出现一条缝隙。
	- 解决：需要设置 background-size:99% 或者 可以把后边的div的margin-top设一个很小的负数;
3. 所有手机对于部分元素（目前是p元素，其他元素还待确认）设置了height、和line-height后呈现不一样的效果。
	- 解决：不使用P元素，直接使用div元素且不设置height和line-height，直接对div元素设置padding值即可。
4. 华为自带浏览器不支持es6语法；
	- 解决：main.js 需要引入  import 'babel-polyfill';
5. ios8、9都不支持复制到剪切板的功能：
	- 解决：提示用户手动复制。

## 待解决
1. h5页面 底部出现一条白色的线；
2. 【vivo x9】进入测试版h5活动页面（需要微信授权的页面）偶发性出现无法打开；

## 不解决
1. 启动同城游相关的问题（浏览器自身的问题，这个没有办法处理的）
	- uc浏览器版本：V12.1.6.996（当前算是最新版本）无法启动同城游（偶发性能启动），低于这个版本的都能启动。看了新版uc浏览器官网对于这种主动唤醒应该做了严格的限制，具体说明可查看http://kf.uc.cn/self_service/web/faqdetails-8311413_8311645_20748356_3.html 。所以初步判定属于uc浏览器版本更新后自身的兼容性问题。
	- 【oppo A57】自带浏览器并不支持自动唤醒第三方app。
	- 唤醒启动同城游app数据统计存在误差。
2. 数据埋点相关的问题
	- 【vivo Y66】自带的浏览器会屏蔽测试版的数据埋点，正式版不会被其屏蔽。


# 一、双旦活动V1.0.0开发总结
## 1、简述
在同城游app内访问活动页面，点击【邀请】按钮去微信平台邀请新用户。分为邀请有礼、下载有奖、广告宣传（福利和游戏）三个子活动。
 
## 2、技术说明
  
  - vue + vux
 
## 3、问题整理
### (1) 分享相关
1、
io和安卓v5.1及其以下只支持分享链接。
安卓v5.1以上支持分享链接、海报。
2、
ios端分享api方法中图标参数必须传值，否则同城游icon不显示。
3、
安卓客户端分享海报时有大小和宽高的限制，仅支持最大高度是400，最大不超过了400，否则安卓客户端会进行压缩，分享到微信的图片会很模糊。

### (2) 唤醒app相关
ios端仅app唤醒使用：`hztcygametcyapp://?`
安卓端仅app唤醒使用：`tcyapp://?`

### (3) es6兼容问题
低端机对于es6的支持不友好，需要全部转为es5的语法。！！！特别注意：在node_modules里边的脚本需要额外配置才会进行语法转化。
<img src="http://doc.uc108.org:8002/Public/Uploads/2018-12-19/5c19ea4b3d6da.png" width="400px">

### (4) 广告位配置问题
活动的url地址末尾不能加【#】号，否则出现低端机型无法打开的bug。 

### (5) 打开同城游客服
安卓v5.3.0以上使用第一个唤醒客服，v5.3.0以下使用第二个唤醒客服：
```
window.HtmlInterface.openActivity('com.uc108.mobile.gamecenter.friendmodule.ui.FeedbackActivity'); // 安卓v5.3.0以上支持
 window.HtmlInterface.openActivity('com.uc108.mobile.gamecenter.ui.FeedbackActivity');// 安卓v5.3.0以下及其自身支持
```
由于安卓客户端的问题，两个方法需要同时使用，版本上不会冲突。

ios使用以下唤醒客服：
```
window.HtmlInterface.openVC('GCChatCustomerServiceViewController');
```

### (6) ios输入框部分机型存在很难定位的问题
ios受fastclick这个类库影响造成的，需要重写fastclick的foucs方法：
https://www.jianshu.com/p/5b578e656966

## 4、发布正式版出现的问题
### （1）ios8、ios9兼容
ios8、ios9在微信页面里边，不支持未编译的es6语法，无法识别端口号，导致报错，因此微信里边的`view/share/index.ejs`模板每次发布正式版，都要调整页面的授权域名以及统计代码；
### （2）域名被微信封掉
tcy365域名被微信封掉了、因此需要更换`双旦活动-微信邀请有礼页面`的域名解决；
### （3）oss页面缓存
发布正式版时，页面存在缓存10分钟到一个星期不等的缓存，导致页面无法访问，让运维手动清除缓存处理的；

## 已解决
1.ios唤醒软键盘，fixed定位失效的问题
- [解决方案](https://blog.csdn.net/gg464556/article/details/77949185)