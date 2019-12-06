---
title: Service Workers
date: 2017-12-27
tags: js
---

## 简介
service是一个注册在一个指定源和路径下，由事件驱动[web worker][1] ，通过注册的js文件控制相关联的页面，拦截或修改访问和资源请求。赋予你很大很大的权限去控制你的app在某一个状态下的表现（最常使用在离线的情况下）

由于servive worker是web workers类型中的一种，同样也运行在一个worker上下文环境中，因此web worker有的特性他都会有：比如不能访问dom，运行在一个单独线程下而不会阻塞主线程，完全异步的设计致使同步的xhr和localStorage都不能被使用（但是可以使用IndexedDB）

出于安全考虑，service workers只能在https下运行，因为其逆天的可以伪造，过滤，修改网络请求的功能如果有人在中间恶意攻击情况会非常糟糕，在火狐浏览器中，service worker会不能使用如果用户打开了用户隐私模式。而service workers之所以优于以前的同类尝试（如AppCache），是因为它们无法支持当操作出错时终止操作。Service workers可以更细致地控制每一件事情。

## 生命周期
service workers拥有一个完全独立与web页面的生命周期

* Registration 注册 （准备）
service worker首次进入页面时会调用[ServiceWorkerContainer.register()][2]方法注册。如果注册成功，service worker就会被下载到客户端并尝试之后的安装和激活，作用于整个域内用户可访问的URL，或者其特定子集。

* Download 下载 
在用户第一次尝试打开由service worker控制的页面时，service worker就会立刻被下载。之后，service worker会每一段时间就会下载一次，隔的时间最长不会超过24小时----为了确保js代码都是相对比较有效的

* Install 安装
安装会在service worker第一次下载，或者新下载的文件与旧版本不一样时开始。安装时可以通过监听[InstallEvent][3]事件来做一些准备工作，比如使用storageAPI来做一些缓存，放置一些资源为离线时做准备。如果安装成功，进入激活状态，安装失败（文件加载或缓存失败）会在下次继续尝试安装~

* Activate 激活
如果已经有启动了的service worker，新版本会处于一种叫worker in waiting的状态*(在后台安装却不会立即被激活)*，直到所有的页面都不再使用旧版本的serviceworker，新版本才会被激活。我们可以使用ServiceWorkerGlobalScope.skipWaiting()跳过这个过程。active状态可以监听activate even，在这个事件里很适合去清除/替换那些在上一个版本的service worker时做的缓存。
![image](https://user-images.githubusercontent.com/18004081/47765479-1f081b00-dd05-11e8-84ee-065b9e5a3424.png)

## 监听事件
- **installEvent**一般用来做一些准备工作，比如使用storage API来做一些缓存，放置一些资源为离线时做准备
- **activateEven**很适合去清除/替换那些在上一个版本的service worker时做的缓存                  
- **FetchEvent**可以对你发起的http请求做处理，你可以把response任意蹂躏成你想要的样子，通过FetchEvent.respondWith 方法返回
- 由于oninstall和onactivate 需要一段时间后才能完成，service worker提供了一个叫**waitUntil**的方法，一旦oninstall或者onactivate完成就立刻被回调，接收一个promise,在这个promise没有成功的resolved前功能性的事件不会被分配到service worker下

## 总体来说Serive Workers适合做
1. 后台资源同步

2. 响应其他源的资源请求

3. 集中进行一些计算成本高的数据更新，同时多个页面可以使用同一套数据

4. 后台服务钩子
5. 自定义模板用于特定URL模式
6. 性能增强，比如预取用户可能需要的资源，比如相册中的后面数张图片

## 兼容性
![image](https://user-images.githubusercontent.com/18004081/47765488-2a5b4680-dd05-11e8-9ea4-ee8fcb792c03.png)
![image](https://user-images.githubusercontent.com/18004081/47765512-4363f780-dd05-11e8-82ce-b607045a9275.png)




## 源文章地址：
https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API


  [1]: https://github.com/includeios/document/issues/2
  [2]: https://developer.mozilla.org/zh-CN/docs/Web/API/ServiceWorkerContainer/register
  [3]: https://developer.mozilla.org/en-US/docs/Web/API/InstallEvent