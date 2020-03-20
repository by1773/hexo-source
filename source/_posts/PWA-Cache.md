---
title: PWA-Cache
date: 2020-03-19 09:59:02
tags: [TypeScript,PWA]
categories: 前端
keywords: [Ts,PWA,js]
description:
top_img: 'https://www.typescriptlang.org/assets/images/foreground_bluprint.svg'
# comments  是否顯示評論(除非設置false,可以不寫)
cover:  'https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1584595114319&di=34b083f9787eeaef1e031bba0468e8e5&imgtype=0&src=http%3A%2F%2Fwww.west.cn%2Fcms%2Fimages%2F2018-07-31%2Fny1xnm0i0op.jpg'
# toc:  是否顯示toc (除非特定文章設置，可以不寫)
# toc_number: 是否顯示toc數字 (除非特定文章設置，可以不寫)
# copyright: 是否顯示版權 (除非特定文章設置，可以不寫)
mathjax:
katex:
hide:
---
# PWA-CacheStorage API 
>Progressive Web App （PWA）是渐进增强 Web App，它能让我们在不可靠的网络上也能快速加载、能够接收桌面通知、具有桌面图标，并且可采用顶层全屏体验的方式加载。

![markdown](http://cdn.semlinker.com/cache-storage-api.png)


&emsp;&emsp;在 PWA 学习笔记之 fetch API 这篇文章中，我们介绍 fetch API 相关的一些基础知识，在该文章末尾我们还介绍了它在 PWA 应用中的使用场景，在具体的使用示例中，我们应用了 CacheStorage API。CacheStorage API（缓存存储应用程序接口），顾名思义是用来实现资源存储。该接口提供缓存 Request /Response 对象对的存储机制，尽管它被定义在 service worker 的标准中，但它不一定要配合 service worker 使用，接下来我们就来会一会 CacheStorage API。
&emsp;&emsp;首先我们来回顾一下 fetch API 文章中使用的示例：

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
&emsp;&emsp;这个示例中，我们监听页面的 fetch 事件，对页面请求事件进行拦截，然后使用 event.respondWith 设置响应对象。在设置响应对象时，我们会优先从缓存中获取，若请求命中，则直接从缓存中获取已缓存的对象，直接返回。若请求未命中，则通过 fetch API 从远程获取对应的资源，当资源请求成功时，我们通过 open 方法获取 v1 版本的 CacheStorage 实例，然后调用该实例的 put 方法，进行资源缓存。
&emsp;&emsp;这里需要注意的是，在使用 cache对象的 put 方法保存资源时，我们调用 response 对象的 clone 方法，而不是直接保存 response 对象。为什么需要克隆响应对象呢？这是因为 Request 和 Response 的 body （响应体）只能被读取一次！它们有一个属性叫 bodyUsed，读取一次之后设置为 true，就不能再读取了。
&emsp;&emsp;通过上面的示例，我们简单介绍了 CacheStorage API 中的一些常用方法。接下来我们趁热打铁，来系统地了解一下目前 CacheStorage 已有的 API。
## 1.CacheStorage API 简介
>CacheStorage.match()：返回一个 Promise 对象，用于判断给定的请求对象是否已被缓存。若匹配则返回已缓存的对象。

```
caches.match(request, options).then(function(response) {
  // Do something with the response
});
```
>CacheStorage.has()：返回一个 Promise 对象，用于判断 cacheName 对应的 Cache 对象是否存在。若 cacheName 对应的 Cache 对象存在，则返回 true 否则返回 false。

````
caches.has(cacheName).then(function(boolean) {
  // true: 缓存存在
});
```

>CacheStorage.open()：返回一个 Promise 对象，用于获取 cacheName 对应的 Cache 对象。若 cacheName 对应的 Cache 存在则返回对应的 Cache 对象，若不存在的话，则会创建一个新的 Cache 对象。

```
caches.open(cacheName).then(function(cache) {
  // Do something with your cache
});
```

>CacheStorage.delete()：返回一个 Promise 对象，用于删除 cacheName 对应的 Cache 对象。若 cacheName 对应的 Cache 对象存在且被成功删除，则返回 true，否则返回 false。

```
caches.delete(cacheName).then(function(true) {
  //your cache is now deleted
});
```

>CacheStorage.keys()：返回一个 Promise 对象，用于获取 CacheStorage 对象中已存在的缓存名称列表。


```
caches.keys().then(function(keyList) {
  //do something with your keyList
});
```

>CacheStorage API 使用示例

```
CacheStorage.match()
caches.match(event.request).then(function(response) {
  return response || fetch(event.request).then(function(r) {
    caches.open('v1').then(function(cache) {
      cache.put(event.request, r);
    });
    return r.clone();
  });
}).catch(function() {
  return caches.match('/sw-test/gallery/myLittleVader.jpg');
});
CacheStorage.has()
caches.has('v1').then(function(hasCache) {
  if (!hasCache) { // v1对应的Cache对象不存在
    someCacheSetupfunction();
  } else {
    caches.open('v1').then(function(cache) {
      return cache.addAll(myAssets);
    });
  }
}).catch(function() {
  // 处理异常
});
CacheStorage.open()
var cachedResponse = caches.match(event.request)
 .catch(function() {
    return fetch(event.request);
  }).then(function(response) {
    caches.open('v1').then(function(cache) {
      cache.put(event.request, response);
  });
  return response.clone();
}).catch(function() {
  return caches.match('/sw-test/gallery/myLittleVader.jpg');
});
CacheStorage.delete() & caches.keys()
this.addEventListener('activate', function(event) {
  var cacheWhitelist = ['v2']; // 缓存的白名单
  event.waitUntil(
    caches.keys().then(function(keyList) { // 返回已缓存的缓存名称列表
      return Promise.all(keyList.map(function(key) {
        if (cacheWhitelist.indexOf(key) === -1) { // 删除不含有v2的缓存对象
          return caches.delete(key);
        }
      }));
    })
  );
});
```
&emsp;&emsp;以上 CacheStorage API 相关示例来源于 MDN - CacheStorage 缓存存储。一口气分析完了 CacheStorage 相关的 API，不知道小伙伴有没有疑问。感觉对刚了解 PWA 的小伙伴们来说，会对示例中所使用的 fetch 或 activate 事件感到疑惑，这个不用着急。后续的文章我们会来揭开它们神秘的面纱。

## 2.总结
&emsp;&emsp;本文介绍了 CacheStorage API 的相关基础知识，最后我们来介绍一个小知识 —— 如何查看已缓存的资源？其实这个很简单，容我偷个懒，这里只介绍 Chrome 浏览器如何查看。准备好了么？Follow Me！
>1. 打开 Chrome 浏览器；
2. 打开开发者工具；
3. 切换到 Application Tab 页；
4. 选择左侧 Cache 菜单下的 CacheStorage 选项。
5. 初探 CacheStorage API 到此结束，下一篇就是介绍我们的 Service Workers 了，它可是 PWA 的核心 “人物 ” 哟。目前刚开始学习 PWA，有误之处，请小伙伴们多多指教。