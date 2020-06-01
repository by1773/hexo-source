---
title: PWA-Fetch
tags:
  - TypeScript
  - PWA
categories: 前端
top_img: 'https://www.typescriptlang.org/assets/images/foreground_bluprint.svg'
cover: >-
  https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1584595114319&di=34b083f9787eeaef1e031bba0468e8e5&imgtype=0&src=http%3A%2F%2Fwww.west.cn%2Fcms%2Fimages%2F2018-07-31%2Fny1xnm0i0op.jpg
abbrlink: 42769
date: 2020-03-19 09:29:12
---
# PWA-fetch API 
>Progressive Web App （PWA）是渐进增强 Web App，它能让我们在不可靠的网络上也能快速加载、能够接收桌面通知、具有桌面图标，并且可采用顶层全屏体验的方式加载。

![markdown](http://cdn.semlinker.com/pwa-fetch-api.png)
## 1. PWA 简介
Progressive Web App 具有的以下主要特点：
- 渐进式 - 适用于选用任何浏览器的所有用户，因为它是以渐进式增强作为核心宗旨来开发的。
- 自适应 - 适合任何机型：桌面设备、移动设备、平板电脑或任何未来设备。
- 持续更新 - 在服务工作线程更新进程的作用下时刻保持最新状态。
- 安全 - 通过 HTTPS 提供，以防止窥探和确保内容不被篡改。
- 可安装 - 用户可免去使用应用商店的麻烦，直接将对其最有用的应用“保留”在主屏幕上。
- 可链接 - 可通过网址轻松分享，无需复杂的安装。

&emsp;&emsp;PWA 基于很多新的 API 和新的技术，如 fetch API、CacheStorage API、Background Sync、Service Worker 和 IndexedDB 等。然而要想真正了解并掌握 PWA，就必须了解它背后基于的技术。因此后续的文章，我们将逐一介绍 PWA 的相关技术。有兴趣的小伙伴们赶紧上车，我们将从 fetch API 开始，开启 PWA 的学习旅程。
## 2.fetch API

&emsp;&emsp;fetch 中文的意思为获取，即通过它我们可以用来获取资源。在前端的日常工作中，我们通常需要从 API 获取数据，然后对数据进行处理或展示。在与服务器交互过程中使用的数据格式一般是 JSON，接下来我们先来体验一下，利用 fetch API 获取 angular 项目的团队的前五位成员，实现代码如下：
```
if("fetch" in this) {
  fetch("https://api.github.com/orgs/angular/members?page=1&per_page=5")
    .then(res => res.json())
    .then(console.dir)
}
```

&emsp;&emsp;要实现同样的功能，我们当然也可以使用 XMLHttpRequest 对象，实现代码如下：
```
var xhr = new XMLHttpRequest();
xhr.open('GET', "https://api.github.com/orgs/angular/members?page=1&per_page=5");
xhr.responseType = 'json';
xhr.onload = function() {
  console.dir(xhr.response);
};
xhr.send();
```
&emsp;&emsp;是不是感觉使用 fetch 简单很多，然而我们并不能随心所欲的使用它，因为它有兼容性问题，具体如下下图所示 （详细信息可浏览  [can i use](https://www.caniuse.com")- fetch）：

![markdown](http://cdn.semlinker.com/can-i-use-fetch-api.jpg)

&emsp;&emsp;对于大多数小伙伴来说，应该更熟悉 jQuery.ajax()，fetch 规范与 jQuery.ajax() 的主要区别如下：
&emsp;&emsp;当接收到一个代表错误的 HTTP 状态码时，从 fetch() 返回的 Promise不会被标记为 reject， 即使该 HTTP 响应的状态码是 404 或 500。相反，它会将 Promise 状态标记为 resolve （但是会将 resolve 的返回值的 ok 属性设置为 false ）， 仅当网络故障时或请求被阻止时，才会标记为 reject。
&emsp;&emsp;默认情况下, fetch 不会从服务端发送或接收任何 cookies，如果站点依赖于用户 session，则会导致未经认证的请求（要发送 cookies，必须设置credentials 选项）。
&emsp;&emsp;除了使用 fetch API 来获取 JSON 数据，我们也可以使用它来获取其它资源，比如普通文本、图片资源等。下面我们来看一下如何使用 fetch API 来获取图片资源，并在页面中显示。
```
if ("fetch" in this) {
  let myImage = document.querySelector('img');
  fetch('https://mdn.github.io/dom-examples/streams/grayscale-png/tortoise.png')
    .then(function(response) {
       return response.blob();
    })
    .then(function(myBlob) {
       let objectURL = URL.createObjectURL(myBlob);
       myImage.src = objectURL;
    });
}
```
&emsp;&emsp;前面的两个示例中，我们通过 fetch 方法分别实现 JSON 数据的读取和图片的资源的获取功能。fetch 方法是 Fetch API 的核心方法，同时定义在 window 和 WorkerGlobalScope 环境中，因此我们可以在 Service Worker 环境中使用它。
>在 Fetch 标准 中该方法的声明如下：

```
partial interface WindowOrWorkerGlobalScope {
  [NewObject] Promise<Response> fetch(RequestInfo input, optional RequestInit init);
};
```
&emsp;&emsp;通过观察上面的方法签名，我们可以知道 fetch 方法支持两个参数，调用后返回一个 Promise 对象。第二个参数是一个参数对象，用来初始化 Request。该参数对象有几个重要的属性：
- method：请求方法，可取 "GET", "POST" 等，默认为 "GET"。
- mode：请求模式，可取 "no-cors", "cors", "same-origin"。
- credentials：是否携带 Cookie，可取："omit", "same-origin","include"。
- cache: 缓存模式，可取: default, no-store, reload, no-cache, force-cache, only-if-cached。

>了解详细的信息，请阅读 https://developer.mozilla.org/zh-CN/docs/Web/API/GlobalFetch/fetch。

&emsp;&emsp;在 PWA 应用中，fetch API 的用武之地在于资源（比如图片、脚本文件或样式文件等）的获取。为了能够保证用户的离线体验，我们可以在获取资源时，对资源进行缓存。
具体示例如下：
```
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((response) => {
      // 若缓存不存在，则使用fetch API从网上获取，然后利用Cache API缓存资源。
      return response || fetch(event.request).then((response) => {
          return caches.open('v1').then((cache) => {
            cache.put(event.request, response.clone());
            return response;
          });
     }); 
   })
  );
});
```
&emsp;&emsp;对于 Cache API，我们下一篇会介绍。这里需要注意的是，在使用 cache.put() 保存资源时，我们调用 response 对象的 clone() 方法，而不是直接保存 response 对象。为什么需要克隆响应对象呢？这是因为 Request 和 Response 的 body （响应体）只能被读取一次！它们有一个属性叫 bodyUsed，读取一次之后设置为 true，就不能再读取了。

## 3.总结

本文只是简单介绍了 fetch API，其实 fetch API 还有很多东西需要进一步了解，如设置请求头、处理 Text/HTML 资源、表单提交、Cookies 与 CORS 处理等。这里就不再展开了，有兴趣的小伙伴，请自行查阅相关资料。