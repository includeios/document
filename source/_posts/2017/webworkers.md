---
title: Web Workers
date: 2017-12-22
tags: js
---
12月20号发布的Safri浏览器内核webkit表示自己开始支持service workers，这就意味的主流浏览器（window下的chorme和Firefox，ios下的Safri，ie.....嗯，先不管他）都支持service workers，web app离线缓存页面将不是梦~

Service workers 其实是web workers中的一种类型，所以在学习使用service workers之前，我们先来看看web workers是啥，有哪些类型，都能干些什么吧

## 整体介绍
**Web Workers**使script能够单独地运行在一个独立的线程中，这意味着我们可以将一些有复杂的逻辑计算的js放在其他线程下工作，而不去阻塞主线程（一般是ui渲染）的运行

## 概念和使用
worker是通过构造函数【Worker(filterName.js) 】生出来的Object，这个js文件里写的代码会单独运行在一个新的执行环境（context）中而不是当前的window，这个工作环境其实是一个[DedicatedWorkerGlobalScope][1]对象，*（对于dedicated类型的worker是他，对于shared类型的worker是[SharedWorkerGlobalScope][2])*.

理论上在这个worker线程下你想干啥就干啥，但是还是有一些例外，比如，你没有办法在worker中直接操作dom，也没办法使用window对象一些默认的方法和属性，不过呢，window下还是有很多功能你可以直接使用的，像webSockets，一些存储工具indexedDB啊，还有一个火狐浏览器搞的Data Store啊~，具体请参考 [Functions and classes available to workers][3]

workers与主线程间通过messages渠道来传递数据，发送message的方法都是**postMessage**,接受数据都是通过监听**onmessage**事件，值得注意的是数据在传输时是通过**复制**的方法而不是共享~

有的时候，workers可能会生成新的workers....，只要这些构建这些workers的js文件在同一源下就可以了。worker 可以通过XMLHttpRequest来访问网络，只是[XMLHttpRequest[4]的responseXML和channel这两个属性将总是null。

## workers类型
除了dedicated(专用）类型的workers,还有下面几种类型

* [SharedWorker][4]
共享类型的workers可以被好几个scripts运行在同一个源中不同的windows下，IFrames下，等等....，相比较dedicated workers，shared workers更复杂一点是因为在进行传输时需要启动**port**，具体请参考SharedWorkers

* [ServiceWorkers][5]
ServiceWorkers一般作为web应用程序、浏览器和网络（如果可用）之前的代理服务器。主要的作用是：

            1.后台消息传递

            2.网络代理，转发请求，伪造响应

            3.离线缓存，并且会在网络是否合适的情况下采取合适的行动更新驻留在服务器上的资源

            4.消息推送

* [Chrome Workers][6]
ChromeWorker 是火狐浏览器自己弄出来的东西，如果您正在开发附加组件，并且希望在扩展程序中使用worker，同时在你的worker中有访问[js-ctypes][7]的权限，那么你可以使用它。

* [Audio Workers][8]
Audio Workers可以直接在webworker的上下文中完成脚本化音频处理。



## 简单示例链接
dedicatedWorkesr: [github地址][9]，[在线实例][10]
SharedWorkers:[github地址][11]github地址，[在线实例][12]在线实例
ServiceWorkers:[github地址][13]，[在线实例][14]


## 兼容性
![](https://upload-images.jianshu.io/upload_images/9673192-9c514f4c5cbe8181.png)
![image](https://user-images.githubusercontent.com/18004081/47764460-40b2d380-dd00-11e8-9e0b-3900adcbfb33.png)




## 源文章地址和参考资料：
https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API

http://web.jobbole.com/84792/


  [1]: https://link.jianshu.com/?t=https://developer.mozilla.org/en-US/docs/Web/API/DedicatedWorkerGlobalScope
  [2]: https://developer.mozilla.org/en-US/docs/Web/API/SharedWorkerGlobalScope
  [3]: https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Functions_and_classes_available_to_workers
  [4]: https://developer.mozilla.org/en-US/docs/Web/API/SharedWorker
  [5]: https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API
  [6]: https://developer.mozilla.org/en-US/docs/Mozilla/Gecko/Chrome/API/ChromeWorker
  [7]: https://developer.mozilla.org/en-US/docs/Mozilla/js-ctypes
  [8]: https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API#Audio_Workers
  [9]: https://github.com/mdn/simple-web-worker
  [10]: http://mdn.github.io/simple-web-worker/
  [11]: https://github.com/mdn/simple-shared-worker
  [12]: http://mdn.github.io/simple-shared-worker/
  [13]: https://github.com/mdn/sw-test/
  [14]: https://mdn.github.io/sw-test/





