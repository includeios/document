---
title: 基于vue搭建pwa应用（一）
date: 2019-11-22
tags: [js,vue]
---

本篇不做pwa相关的技术分享，只是记录一些自己搭建时遇到的问题供以后查阅。关于pwa的技术分享：https://juejin.im/post/5a6c86e451882573505174e7 这篇文章总结的挺好的，例子也很不错。

## 实现效果一览
1. #### 添加到主屏幕

![241574393822_ pic](https://user-images.githubusercontent.com/18004081/69401690-af5e3700-0d30-11ea-8b9f-59317aa8884e.jpg)
![231574393777_ pic](https://user-images.githubusercontent.com/18004081/69401716-bedd8000-0d30-11ea-96ad-80b4e1d92cff.jpg)

2. #### 离线应用 : [离线时也能正常访问页面][3]

3. #### 服务器推送消息

![201574393752_ pic](https://user-images.githubusercontent.com/18004081/69402146-eaad3580-0d31-11ea-8c3c-85503595ab3c.jpg)

## 1.模拟https环境

Service Worker 只能运行在localhost或者https环境下，你可以跑个本地服务试一试，或者代理个线上https环境

- 本地build，生成dist目录
- 根据dist路径起个nginx服务
- 将线上的静态资源代理到本地nginx服务

## 2.保存到主屏幕：manifest

manifest可以配置应用程序安装到设备主屏幕时的icon，开屏动画，第一次进入地址等，例子里说的很详细，不详细展开了，具体配置可参考[MDN][1]。

beforeinstallprompt的官方解释，他将会在一个**合适的时间提醒用户保存到主屏幕**时触发，这个**合适**的sao操作大家自行感受一下。

## 3.离线缓存：ServiceWorker

官网推荐使用：[Workbox][2]

#### **首先在项目中注册Service Worker：**

```js
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('/mobile/service-worker.js',{scope:'/mobile'})
    .then((reg) => {
      console.log('Service Worker registered! ', reg);
    })
    .catch((err) => {
      console.error('Service Worker register error: ', err);
    });
  });
}
```

- register第一个传参是Service Worker的脚本的url地址，注意是可访问的url地址，而不是相对路径
- Service Worker只会拦截在scope作用域下的请求，scope会在注册时生成（默认为Service Worker的location），你只能设置为Service Worker所在的根目录以及他子集，所以一般把生成的Service Worker放到根目录下

#### **配置workbox实现离线缓存**

目前比较粗糙，静态资源和接口都缓存下来了，Workbox有好几种缓存策略，比较方便配置。

```js
// 静态资源预缓存
workbox.precaching.precacheAndRoute(self.__precacheManifest || []);

//静态资源
workbox.routing.registerRoute(
  /(?:\/overview|\.js|\.css)$/,
  new workbox.strategies.StaleWhileRevalidate({
    cacheName: 'static-cache',
  })
);

// 图片缓存
workbox.routing.registerRoute(
  /\.(?:png|jpg|jpeg|svg|gif)$/,

  new workbox.strategies.CacheFirst({
    // Use a custom cache name.
    cacheName: 'image-cache',
    plugins: [
      new workbox.expiration.Plugin({
        // Cache for a maximum of a week.
        maxAgeSeconds: 7 * 24 * 60 * 60,
      }),
    ],
  })
);

// 接口缓存
workbox.routing.registerRoute(
  /\/v\/api/,
  new workbox.strategies.NetworkFirst({
    cacheName: 'api-cache',
  })
);
```

#### **Webpack的workbox插件**

```js
const WorkboxPlugin = require('workbox-webpack-plugin');

 new WorkboxPlugin.InjectManifest({
   swSrc: 'src/pages/author/service-worker.js',
   swDest: 'service-worker.js',
 }),
```

## 4.服务端主动推送消息

这篇文章解释的挺好的：https://www.jianshu.com/p/9970a9340a2d

#### **注册Service Worker时订阅push service**

```js
const applicationServerPublicKey = 'BMz9tUR-Iq3W2K0u1fy0qb5p1zD65s7N0laipwmuq7yefjASIkbrFUXKjmEmayOClvCdc0ytiLSblU1UGTnSmkY';

function urlB64ToUint8Array(base64String) {
  const padding = '='.repeat((4 - base64String.length % 4) % 4);
  const base64 = (base64String + padding)
    .replace(/\-/g, '+')
    .replace(/_/g, '/');

  const rawData = window.atob(base64);
  const outputArray = new Uint8Array(rawData.length);

  for (let i = 0; i < rawData.length; ++i) {
    outputArray[i] = rawData.charCodeAt(i);
  }
  return outputArray;
}

function subscribeUserToPush(registration) {
  const subscribeOptions = {
    userVisibleOnly: true,
    applicationServerKey: urlB64ToUint8Array(applicationServerPublicKey),
  };
  return registration.pushManager.subscribe(subscribeOptions).then(function (pushSubscription) {
    console.log('Received PushSubscription: ', JSON.stringify(pushSubscription));
    return pushSubscription;
  });
}

function sendSubscriptionToServer(subscription) {
  return axios.get('/subscription?subscription=' + JSON.stringify(subscription));
}

if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('/mobile/service-worker.js')
    .then((reg) => {
      console.log('Service Worker registered! ', reg);
      return subscribeUserToPush(reg);
    })
    .then((subscription) => {
      if (subscription) {
        console.log('User is subscribed');
        sendSubscriptionToServer(subscription);
      } else {
        console.error('User is not subscribed!');
      }
    })
    .catch((err) => {
      console.error('Service Worker register error: ', err);
    });
  });
}
```

#### **起一个node服务模拟服务器发送消息**

```js
const http = require('http');
const querystring = require('querystring');
const util = require('util');
const webpush = require('web-push');

const vapidKeys = {
  publicKey: 'BMz9tUR-Iq3W2K0u1fy0qb5p1zD65s7N0laipwmuq7yefjASIkbrFUXKjmEmayOClvCdc0ytiLSblU1UGTnSmkY',
  privateKey: 'WQlcnnMc6ehYztTTUcn12EI4sCPVtA8EG18yXDgZn5I',
};

webpush.setVapidDetails(
  'https://star.toutiao.com',
  vapidKeys.publicKey,
  vapidKeys.privateKey
);


const server = http.createServer((req, res) => {
  const url = req.url.split('?');
  const query = querystring.parse(url[1]);
  if (url[0] === '/subscription') {
    const subscription = JSON.parse(query.subscription) || {};

    setTimeout(() => pushMessage(subscription), 5000);
    res.end('success');
  }
});

function pushMessage(subscription) {
  //发送了个“heiheihei”给客户端
  webpush.sendNotification(subscription, 'heiheihei').then(data => {
    console.log('push service的相应数据:', JSON.stringify(data));
    return;
  }).catch(err => {
    // 判断状态码，440和410表示失效
    if (err.statusCode === 410 || err.statusCode === 404) {
      return util.remove(subscription);
    } else {
      console.log(subscription);
      console.log(err);
    }
  });
}

server.listen('9876', () => {
  console.log('listen 9876');
});

```

#### **service worker中监听服务器主动发送的消息**

```js
// 服务端主动推送
self.addEventListener('push', function (event) {
  console.log('[Service Worker] Push Received.');
  console.log(`[Service Worker] Push had this data: "${event.data.text()}"`);

  const title = '有新的任务啦';
  const options = {
    body: event.data.text(),
    icon: 'url',
    badge: 'url',
  };

  const notificationPromise = self.registration.showNotification(title, options);
  event.waitUntil(notificationPromise);
});

self.addEventListener('notificationclick', function (event) {
  console.log('[Service Worker] Notification click Received.');

  event.notification.close();

  event.waitUntil(
    clients.openWindow('/mobile/sup/task-center')
  );
});
```


[1]:https://developer.mozilla.org/zh-CN/docs/Web/Manifest
[2]:https://stackoverflow.com/questions/35780397/understanding-service-worker-scope
[3]:https://bytedance.feishu.cn/space/file/boxcnlFmk1n4P3GKjz6amKZtyLb


