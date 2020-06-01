---
title: 初识微信公众号开发
tags:
  - 微信
categories: 微信开发
keywords:
  - 微信开发
top_img: >-
  https://goss1.cfp.cn/creative/vcg/veer/400/new/VCG41N680280554.jpg?x-oss-process=image/format,webp
cover: 'https://www.xmtzxm.com/uploads/allimg/181224/5-1Q22419514Q18.jpg'
abbrlink: 57746
date: 2020-03-21 08:46:52
---

现如今，微信已经不再只承担着交流沟通、娱乐大众的功能，微信公众号的推出将微信逐渐转变成个人、商家、企业单位用来营销的重要工具。而微信推出的公众号开发功能，为我们码农带来很大的方便，让我们创造出更多的可能性。

## **第一章：开始开发(准备阶段)**

### **1、 接入指南**

接入微信公众平台开发，开发者需要按照如下步骤完成：

**·**填写服务器配置

![img](./wechat-01.png)

*说明：URL是开发者用来接收微信消息和事件的接口URL，该接口尽量写成两个请求方式，1:get请求，用于验证如下地址有效性，2:post请求, 用于接收消息和事件，Token 可以随便定义用于验证接口签名有效性, EncodingAESKey是加密的密钥，下面加密方式选兼容模式或者安全模式的时候开发者可根据该密钥对数据进行加解密*

**·**验证如上URL服务器地址的有效性

根据官方文档的说明，微信验证接口会带下面几个参数

![img](./wechat-02.png)

端接口接收到这些参数后进行签名验证，如下代码：

```js
exports.check = function (req, res, next) {
// 在这里验证签名
var signature = req.query['signature'],
timestamp = req.query['timestamp'],
nonce = req.query['nonce'],
echostr = req.query['echostr'];
var sha1 = crypto.createHash('sha1'),
sha1Str = sha1.update([config.weixin.token, timestamp, nonce].sort().join('')).digest('hex');
res.writeHead(200, {'Content-Type': 'text/plain'});
res.end((sha1Str === signature) ? echostr : '');
return res;
};
```

**·**依据接口文档实现业务逻辑

这里就是根据业务需求，进行接口调用的编程了，下面我会一一介绍

### **2、获取access_token**

 access_token是公众号的全局唯一接口调用凭据，公众号调用各接口时都需使用access_token。该接口一天只能请求2000次，开发者需要进行妥善保存。access_token的存储至少要保留512个字符空间。access_token的有效期目前为2个小时，需定时刷新，重复获取将导致上次获取的access_token失效。

如下代码事例：

```js
exports.get_token = function (fn) {
redis.get(weixin_token, function (err, token_str) {
if (token_str) {
return fn(err, JSON.parse(token_str));
} else {
request.get("https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=" + app_id + "&secret=" + app_secret, function (err, response, body) {
if (JSON.parse(body).errcode == 45009) {
return fn(err)
} else {
redis.set(weixin_token, body, function (err) {
redis.expire(weixin_token, 7000, function () {
return fn(err, JSON.parse(body));
};
```

上面的事例代码中，首先我会从redis中获取到access_token，因为我最初获取access_token的时候写入到redis中了，官方给的有效时间是7200秒，我放在redis中的有效时间是7000秒，所以我这里的token不会过期，过期后会重新调用接口获取并写入redis



## **第二章：自定义菜单**

实例代码中只写入创建接口的调用，查询和删除就不举例了。

### ·**1、自定义菜单创建接口**

如下代码事例以及说明：

```js
get_token(function(err, obj){
var access_token = obj.access_token;
request.post({
url: "https://api.weixin.qq.com/cgi-bin/menu/create?access_token=" + access_token,
json: {
"button":[
{
"type":"view",
"name":"工作台",
"url":"http://worktile.com"
},
{
"name":"解决方案",
"sub_button": [
{
"type":"view",
"name":"研发",
"url":"https://pro.worktile.com/solution/dev"
}
{
"type":"view",
"name":"最佳实践",
"url":"https://worktile.com/can"
}
{
"name":"更多",
"sub_button":[
{
"type":"view",
"name":"下载应用",
"url":"http://a.app.qq.com/o/simple.jsp?pkgname=com.worktile"
},
{
"type":"click",
"name":"合作",
"key":"work_together"
}]
}, function(err, res, body){
console.log(body)
})


```

这里get_token方法就是上面第一章中＃获取access_token ，而且我这里是作为脚本执行的，这样方便以后随便修改自定义菜单内容

### 2、自定义菜单查询接口

http请求方式：GET

https://api.weixin.qq.com/cgi-bin/menu/get?access_token=ACCESS_TOKEN

### **3、自定义菜单删除接口

http请求方式：GET

https://api.weixin.qq.com/cgi-bin/menu/delete?access_token=ACCESS_TOKEN



## **第三章：消息管理**

### 1、接收消息

接收的消息分为普通消息和事件消息，统一有第一章中接入指南填写的RUL接口来接收处理

> 微信消息里面消息的接收和返回都是已XMl格式交互的

```js
exports.receive = function (req, res, next) {
// 在这接收消息
var xml = '';
req.setEncoding('utf8');
req.on('data', function (chunk) {
xml += chunk;
});
req.on('end', function () {
toJSON(xml, res);
});

};
```

说明：receive方法就是接收用户发给公众号的消息，内容格式是xml，toJSON就是我解析xml为json的方法，如下

> xml转换json可以运用第三方工具库“xml2json”快捷进行转换

```js
//解析器
var toJSON = function (xml, res) {
var msg = {};
xml2js.parseString(xml, function (err, result) {
var data = result.xml;
msg.ToUserName = data.ToUserName[0];
msg.FromUserName = data.FromUserName[0];
msg.CreateTime = data.CreateTime[0];
msg.MsgType = data.MsgType[0];
switch (msg.MsgType) {
case 'text' :
    msg.Content = data.Content[0];
    msg.MsgId = data.MsgId[0];
    res.setHeader("Content-Type", "text/plain");
    res.send("");
    return handle_text(msg, res);
break;
case 'image' :
    msg.PicUrl = data.PicUrl[0];
    msg.MsgId = data.MsgId[0];
    msg.MediaId = data.MediaId[0];
    res.setHeader("Content-Type", "text/plain");
    res.send("");
break;
case 'voice' :
    msg.MediaId = data.MediaId[0];
    msg.Format = data.Format[0];
    msg.MsgId = data.MsgId[0];
    res.setHeader("Content-Type", "text/plain");
    res.send("");
break;

case 'video' :
    msg.MediaId = data.MediaId[0];
    msg.ThumbMediaId = data.ThumbMediaId[0];
    msg.MsgId = data.MsgId[0];
    res.setHeader("Content-Type", "text/plain");
    res.send("");
break;
case 'location' :
    msg.Location_X = data.Location_X[0];
    msg.Location_Y = data.Location_Y[0];
    msg.Scale = data.Scale[0];
    msg.Label = data.Label[0];
    msg.MsgId = data.MsgId[0];
    res.setHeader("Content-Type", "text/plain");
    res.send("");
break;
case 'link' :
    msg.Title = data.Title[0];
    msg.Description = data.Description[0];
    msg.Url = data.Url[0];
    msg.MsgId = data.MsgId[0];
    res.setHeader("Content-Type", "text/plain");
    res.send("");
break;
case 'event' :
    msg.Event = data.Event[0];
    if (data.EventKey && _.isArray(data.EventKey) && data.EventKey.length > 0) {
        msg.EventKey = data.EventKey[0];
        return handle_event(msg, res);
    }
    res.setHeader("Content-Type", "text/plain");
    res.send("");
break;
}
});
};
```

*说明：这里我用户发过来的消息xml解释称json数据msg, 根据不同的类型做不同的处理，如上文本，图片，音频，视频，链接，事件等消息*

```js
var handle_text = function (msg, res) {
var text = msg.Content;
if(text.trim() == "研发"){
    var data = {
    "touser":msg.FromUserName,
    "msgtype":"news",
    "news":{
    "articles": [
    {
    "title":"重磅！Worktile 推出研发管理解决方案",
    "description":"项目进度清晰掌握，快速跟进产品Bug，多维度统计报表，文件文档有序管理",
    "url":"https://pro.worktile.com/solution/dev",
    "picurl":"https://wt-prj.oss.aliyuncs.com/b327e3a5666048279583e8e026ac6b87/4bb6e53c-8516-4466-b278-4f3b596e46db.png"
    sendMessageToUser(data);
}else if(text.trim() == "电商"){
    var data = {
    "touser":msg.FromUserName,
    "msgtype":"news",
    "news":{},
    "articles": [
    {
    "title":"Worktile 『电商解决方案』上线！",
    "description":"降低运营成本，提高团队效率。日常运营、大促筹备、售后跟踪、研发管理……尽在掌握。",
    "url":"https://pro.worktile.com/solution/ecommerce",
    "picurl":"https://cdn.worktile.com/solution/ecommerce.png"
    }]
    sendMessageToUser(data);
}
*说明：这是对文本消息的处理，如上，如果接收到 研发 字样的消息，公众号会给该用户发送一条新闻消息*

var handle_event = function (msg, res) {
	console.log("weixin receive message ===", msg)
if (msg.Event == 'CLICK' && msg.EventKey == 'work_together') {
    var text = "Hello，谢谢对 Worktile 的关注啦，请访问worktile官方网站了解。。。。。";
    var data = {
        touser : msg.FromUserName,
        msgtype: "text",
        text  : {
        content: text
sendMessageToUser(data);
```

*说明：这是对事件消息的处理，如上，如果接收到 msg.Event == 'CLICK' && msg.EventKey == 'work_together' 事件的消息，公众号会给该用户发送一条文本消息，当然事件消息有很多，如：subscribe关注公众号消息，unsubscribe取消关注，扫描带参数二维码事件，还有如上说的自定义菜单事件， 上报地理位置事件等*

```js
var sendMessageToUser = function ( data) {
	get_token(function (err, obj) {
        request.post({
            url  : "https://api.weixin.qq.com/cgi-bin/message/custom/send?access_token="             +obj.access_token,
            headers: {"Content-Type": "application/json"},
            json  : data
        }, function (err, res, body) {
        console.log("sendmessage....", body)
        })
    }
    
```

*这个发放就是调用微信接口给用户发送消息，那接下来咱们就看下发送消息*

### **2、发送消息**

发送消息分为，发送被动消息，发送客服消息，发送模版消息

- 被动消息如上接收消息后根据消息判断发送给用户的消息即是被动消息
- 客服消息，是公众号收到用户来的消息客服根据内容回复给用户的消息

如果用户并没有给公众号发消息，此时客服是无法给用户发送消息的，这是微信做的一个限制

如下代码：

```js

wtutil.get_token(function (err, obj) {
//var text = "你好，这是一条消息，多谢支持...";
//var data = {
//  touser : "oy4hbwbd0MOMmn8aUtQWMcNxs8PI",
//  msgtype: "text",
//  text  : {
//    content: text
//  }
//};
var data = {
touser : "oy4hbwbd0MOMmn8aUtQWMcNxs8PI",
msgtype: "image",
"image":
{
	"media_id":"ZqQGrsR6ivb273zLApNfkEdAP3UI8nHJTJ9ekelfJ8OhKUF6UG-o6YbOBv4uWf4R"
}
};
request.post({
    url  : "https://api.weixin.qq.com/cgi-bin/message/custom/send?access_token=" + obj.access_token,
    headers: {"Content-Type": "application/json"},
    json  : data
}, function (err, res, body) {
    console.log(body);
    })
});
```

msgtype: 是消息类型，上面注释掉的是文本消息，下面是个图片消息，touser是用户的openid，这里我只是取过来直接使用的，这里跟上面接收消息后处理给用户发消息有写重复，就不多介绍了

### **3、模版消息**

模版消息大家肯定很熟悉，比如Worktile的微信公众号接收任务消息通知，这样的消息就是模版消息

![img](./wechat-03.png)

模版消息相对来说复杂一下，首先要从公众号添加或者申请消息模版，如下图

![img](./wechat-04.png)



那么有了消息模版之后就可以拿到模版ID，然后给用户发送模版消息了

```js
get_token(function (err, obj) {
    var data = {
        "touser"   : "oy4hbwbd0MOMmn8aUtQWMcNxs8PI",
        "template_id": "z6yV_lOIAM-LQbsrG-B3hTQvwt8_4Y3wVU2PH9UW16c",
        "url"    : "https://worktile.com",
        "topcolor"  : "#FF00FF",
        "data"    : {
            "first"  : {
                "value": "测试哈哈哈，颜色可以自定义",
                "color": "#33FF00"
            },
            "one": {
                "value": "one",
                "color": "#173177"
            },
            "two": {
                "value": "two",
                "color": "#FF0033"
            },
            "three": {
                "value": "three",
                "color": "#173177"
            },
			"remark" : {
                "value": "remark，了解更多详情，关注我。。。。",
                "color": "#33FF00"
            }
        }
request.post({
    url  : "https://api.weixin.qq.com/cgi-bin/message/template/send?access_token=" + obj.access_token,
    headers: {"Content-Type": "application/json"},
    json  : data
    }, function (err, res, body) {
    	console.log(body, "----")
    })
});

```

需要说明的是参数中data里面的key(first, one, two,three, remark)是模版中定义的，这里需要根据模版内容来写，还有可以设置每个字段的颜色值等属性

## **第四章：微信网页开发**

这个章节跟前面几章不同，前面几章介绍的是公众号开发的一些东西，这个章节介绍的是网页开发，主要针对h5应用或者是页面的开发，Worktile微信版就是微信网页开发完成的，下面咱们一步步的介绍。

### **·微信网页授权**

微信网页授权采用的是Oauth2.0的授权方式：

**第一步：访问如下链接获取code**

https://open.weixin.qq.com/connect/oauth2/authorize?appid=APPID&redirect_uri=REDIRECT_URI&response_type=code&scope=SCOPE&state=STATE#wechat_redirect

redirect_uri是你h5地址，授权成功后会把code加入到地址上，类似于：https://weixin.worktile.com?code=xxx这样

**第二步：通过code换取网页授权access_token**

请求接口（get请求）

https://api.weixin.qq.com/sns/oauth2/access_token?appid=APPID&secret=SECRET&code=CODE&grant_type=authorization_code

返回的结果如下

```json
{ 
    "access_token":"ACCESS_TOKEN",
    "expires_in":7200,
    "refresh_token":"REFRESH_TOKEN",
    "openid":"OPENID",
    "scope":"SCOPE" 
}
```

*access_token是用户授权的token，openid是用户对于该公众号的唯一标示，refresh_token：可以调用刷新token的接口获取最新的token*

**第三步：获取用户信息(需scope为 snsapi_userinfo)**

请求接口(get请求)

https://api.weixin.qq.com/sns/userinfo?access_token=ACCESS_TOKEN&openid=OPENID&lang=zh_CN

通过以上3个步骤就可以获取用户的信息，进行用户的管理操作了

### **·微信JS-SDK**

网页开发中，有时候我们会自定义分享的内容，图片，音频，视频的上传，下载，地理位置，摇一摇周边，扫码，支付等的功能，这时候就需要js-sdk的开发了，下面简单介绍下js-sdk的使用，读者还可以查看官方开发文档：https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140842&token=&lang=zh_CN

### **·绑定域名**

先登录微信公众平台进入“公众号设置”的“功能设置”里填写“JS接口安全域名”。

### **·引入JS文件**

在需要调用JS接口的页面引入如下JS文件，（支持https）：http://res.wx.qq.com/open/js/jweixin-1.0.0.js

如需使用摇一摇周边功能，请引入http://res.wx.qq.com/open/js/jweixin-1.1.0.js

### **·通过config接口注入权限验证配置**

如下代码需要在网页中配置

```js
wx.config({
debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打				开，参数信息会通过log打出，仅在pc端时才会打印。
appId: '', // 必填，公众号的唯一标识
timestamp: , // 必填，生成签名的时间戳
nonceStr: '', // 必填，生成签名的随机串
signature: '',// 必填，签名，见附录1
jsApiList: [] // 必填，需要使用的JS接口列表
});
```



> 如上代码中的timestamp, nonceStr, signature需要服务端做好签名返回给页面，这里可以使用异步调用的方式，如下为服务端签名代码

```js

var sign = function (jsapi_ticket, url) {
var ret = {
jsapi_ticket: jsapi_ticket,
nonceStr: createNonceStr(),
timestamp: createTimestamp(),
url: url
};
var string = raw(ret);
jsSHA = require('jssha');
shaObj = new jsSHA(string, 'TEXT');
ret.signature = shaObj.getHash('SHA-1', 'HEX');
return ret;
};
```



*ret就是我们需要的签名结果，其中jsapi_ticket是调用ticket接口获取的，官方文档中也有说明https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token= access_token&type=jsapi, rul是获取签名的当前网页地址, nonceStr:随机字符串，timestamp：当前10位时间戳*

```js
var raw = function (args) {
var keys = Object.keys(args);
keys = keys.sort()
var newArgs = {};
keys.forEach(function (key) {
newArgs[key.toLowerCase()] = args[key];
});
var string = '';
for (var k in newArgs) {
string += '&' + k + '=' + newArgs[k];
}
string = string.substr(1);
return string;
};
```

*这个方法是对签名对象的一个字符串格式化算法*

### **·通过ready接口处理成功验证**

> wx.ready(function(){
>
> // config信息验证后会执行ready方法，所有接口调用都必须在config接口获得结果之后，config是一个客户端的异步操作，所以如果需要在页面加载时就调用相关接口，则须把相关接口放在ready函数中调用来确保正确执行。对于用户触发时才调用的接口，则可以直接调用，不需要放在ready函数中。

});

### **·调用接口**

根据config里面 jsApiList的配置可以调用js-sdk的接口，如分享朋友圈，分享微信好友等。

``` js
wx.onMenuShareTimeline({
title: '', // 分享标题
link: '', // 分享链接
imgUrl: '', // 分享图标
success: function () {
// 用户确认分享后执行的回调函数
},
cancel: function () {
// 用户取消分享后执行的回调函数
}
});
```



这是一个分享到朋友圈的接口，可以自定义标题，自定义链接，自定义图标

如果想调用更多的js-sdk的接口可以参考官方文档进行开发：https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141115&token=&lang=zh_CN

### **·微信网页开发样式库**

https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1455784134&token=&lang=zh_CN

### **·微信web开发者工具**

https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1455784140&token=&lang=zh_CN

以上就是微信公众号开发的一些内容，算是入门篇。其实关于微信的开发还有很多可以做的事情，而且随着需求越来越多，技术越来越完善.