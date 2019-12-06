---
title: Intersection observer
date: 2019-09-26
tag: js
---

拥有“交叉观察者“小宝贝，从此妈妈再也不用担心我的滚动交互了。

## 通过js计算属性来实现

记得以前写懒加载时，用的是一个（名字不重要）的第三方库，基本思想是：通过在onScroll事件里计算图片的offsetTop和clientHeight来判断图片是否到达了可展示区域，再替换img的src来加载正确的图片地址。

其实大部分和滚动相关交互的实现思路都和上述懒加载的实现差不多：**通过在onScroll里对一些计算属性值的判断来做一些操作**。这种方法不好的地方是代码里充斥着大量的offsetTop，clientHeight等取值计算，一来看着不舒服，二来这种计算属性的计算会造成浏览器的重绘，对性能上有一定的影响。

## Intersection observer小利器

Intersection observer ：交叉观察者，顾名思义，他是一个判断两个元素是否发生交叉的观察者！ 

这里是详细[MDN][1]介绍：Intersection Observer API 允许你配置一个回调函数，每当目标(**target**)元素与设备视窗或者其他指定元素发生交集的时候执行。设备视窗或者其他元素我们称它为根元素或根(**root**)。通常，您需要关注文档最接近的可滚动祖先元素的交集更改，如果元素不是可滚动元素的后代，则默认为设备视窗。如果要观察相对于根(**root**)元素的交集，请指定根(**root**)元素为 `null` 。

![image](https://user-images.githubusercontent.com/18004081/65669455-2d29fd00-e076-11e9-88b3-dbba44aee474.png)


请先忍忍，例子里有动图介绍！

## 具体用法

### 1.构造函数

```js
const observer = new IntersectionObserver(callback, options)
```

### 2.options

传递到 `IntersectionObserver()` 构造函数的 `options` 对象，允许你控制调用观察者的回调环境，具体配置有：

- **root**

  接受一个dom，作为发生交叉的视图窗口，必须是目标元素的父级元素。如果未指定或者为null，则默认为浏览器的视窗。

- **rootMargin**

  root元素的外边距，可以扩大/缩小视窗的交集范围。用法和css中的margin属性一样，比如： `“10px 120px 30px -60px”(top, right, bottom, left)` ，正数时是往外扩张，负数时是往里缩放。

- **threshold**

  接受一个number或者一个number数组（number的值在0 - 1中间）。当目标元素和root元素相交程度的百分比达到指定值时，构造函数里注册的回调函数将会被执行。比如设置threshold：0.5，那么当目标元素在root中的可视区域占自身50%时，触发回调函数，如果你想可见程度每增加/减少25%时就触发一次回调，那么应该设置成：threshold：[0, 0.25, 0.5, 0.75, 1]。默认是0。

### 3.目标（target）元素

构建后为每个观察者配置一个对象

```js
const target = document.querySelector('#target')
observer.observe(target)
```

### 4.callback

传递到 `IntersectionObserver()` 构造函数的回调函数，当达到IntersectionObserver指定的threshold时，触发回调。回调接受 `IntersectionObserverEntery` 对象和观察者列表。

> 请留意，你注册的回调函数将会在主线程中被执行。所以该函数执行速度要尽可能的快。如果有一些耗时的操作需要执行，建议使用 [`Window.requestIdleCallback()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestIdleCallback) 方法。

```js
const callback = function(entries, observer) { 
  entries.forEach(entry => {
    // Each entry describes an intersection change for one observed
    // target element
    //   entry.boundingClientRect 空间信息
    //   entry.intersectionRatio  目标元素可视区域占比
    //   entry.intersectionRect
    //   entry.isIntersecting   是否正在交叉，可判断元素是否可见
    //   entry.rootBounds
    //   entry.target
    //   entry.time
  })
}
```

### 5.常用方法

- observe(dom)：开始监听dom元素作为目标元素
- unobserve(dom)：结束监听dom元素
- takRecords()：返回所有监听的目标元素集合
- disconnect()：停止所有监听



## 例子

### 监听一个class="box"的元素是否进入视窗

```js
const box = document.querySelector('.box')

const observer = new IntersectionObserver(entries => {
    entries.forEach(item => {
        console.log(item.isIntersecting? "进入了视窗的内部":"离开了视窗的内部")
    })
})

observer.observe(box)
```

效果如下：

![QQ20190926-141057-HD](https://user-images.githubusercontent.com/18004081/65669562-5e0a3200-e076-11e9-9a7f-3f5ce04ec0b0.gif)

### 监听多个class="box"的元素是否进入视窗

```js
const box = document.querySelectorAll('.box')

const observer = new IntersectionObserver(entries => {
    console.log(`此时一共有${entries.length}个元素发生交叉行为`)
})

box.forEach(item=>observer.observe(item))
```

![QQ20190926-144225-HD](https://user-images.githubusercontent.com/18004081/65669640-83973b80-e076-11e9-97ee-e847445da391.gif)
![QQ20190926-144457-HD](https://user-images.githubusercontent.com/18004081/65669657-898d1c80-e076-11e9-94c3-3f00233e731b.gif)

为什么下面打印出来的length是1呢？entries返回的是**当前正在发生交叉**的目标集合。上面是大家一起发生交叉，每次返回的集合长度都是3，下面是大家轮流发生交叉，每次返回的集合长度都是1。

## 实际应用

相关例子的代码已上传至[intersectionDemo][2]

### 1.懒加载

用Intersection observer实现懒加载代码优雅多了，你只需将图片设置为目标函数。

假设html为：

```html
<!-- 例子一：懒加载 -->
<div class="contain lazy-load-contain">
    <div class="img-contain">
        <img src="" data-origin="./img/pic1.jpg" />
        <img src="" data-origin="./img/pic2.jpg" />
        <img src="" data-origin="./img/pic3.jpg" />
        <img src="" data-origin="./img/pic4.jpg" />
        <img src="" data-origin="./img/pic5.jpg" />
        <img src="" data-origin="./img/pic6.jpg" />
        <img src="" data-origin="./img/pic7.jpg" />
    </div>     
</div>
```

js相关代码为：

```js
// 懒加载
const lazyLoading = ()=>{
    const images = document.querySelectorAll('.lazy-load-contain img')

    const observer = new IntersectionObserver(entries => {
        entries.forEach(item => {
            if(item.isIntersecting){
                console.log('开始懒加载：',item.target.dataset.origin)
                item.target.src = item.target.dataset.origin //开始加载图片
                observer.unobserve(item.target)  //停止监听已经开始加载的图片
            }
        })
    }, {
        root:document.querySelector('.lazy-load-contain'),
        rootMargin:"0px 0px 50px 0px" //在底部距离50px时就开始加载图片
    })

    images.forEach(item=>observer.observe(item))
}
```

效果为：
（啊，文件太大被吞了，可通过运行demo里的例子自己看下哈，不过我拍的照片可真好看呀。）

### 2.列表加载数据

移动端滚动列表加载数据常见case是：下拉到页尾时继续加载下一页数据，这个也可以用Intersection observer来实现：

```js
<!-- 例子二：列表滚动加载数据 -->
<div class="contain list-scroll-load">
    <div class="img-contain" />  
	<div class="footer-reference">加载中...</div>  
</div>
const listLoading = ()=>{
    let listDate = []

    const addDate = ()=>{
        listDate = listDate.concat(["./img/pic1.jpg","./img/pic2.jpg","./img/pic3.jpg", "./img/pic4.jpg","./img/pic5.jpg","./img/pic6.jpg","./img/pic7.jpg"])
    }

    const convertHtml = ()=>{
        let dom = '' 
        listDate.forEach(item=>{
            dom += `<img src="${item}"/>`
        })
        document.querySelector('.list-scroll-load .img-contain').innerHTML = dom
    }

    const observer = new IntersectionObserver(entries => {
        //如果底部标识进入交叉区，开始请求下一页数据
        if(entries[0].isIntersecting){
            console.log('开始请求数据')
            addDate()
            convertHtml()
        }
    }, {
        root:document.querySelector('.list-scroll-load'),
        rootMargin:"0px 0px 100px 0px" //在底部距离100px的时候就开始加载数据
    })

    observer.observe(document.querySelector('.footer-reference'))
}
```

效果如下：
只是个简单的例子，每次到底部补充数据替换dom，所以替换dom时页面会有点晃动，如果你用react/vue这种牛逼哄哄的框架就不会有这种问题啦
（啊，文件依旧太大被吞了，可通过运行demo里的例子自己看下哈，不过我拍的照片确实好看呢。）
当然你可以把1和2的例子结合在一起，做一个底部请求数据同时懒加载的列表页。

### 3.模拟position: sticky实现吸顶效果

```js
    <!-- 例子四：吸顶 -->
    <div class="contain stick-top-contain">
      <div class="title">标题XXXXX</div>
      <div class="top-reference"></div>
      <div class="stick-bar">筛选框等需要吸顶的元素</div>
      <div class="img-contain">
        <img src="./img/pic1.jpg" />
        <img src="./img/pic2.jpg" />
        <img src="./img/pic3.jpg" />
        <img src="./img/pic4.jpg" />
        <img src="./img/pic5.jpg" />
        <img src="./img/pic6.jpg" />
        <img src="./img/pic7.jpg" />
      </div>  
    </div>

const stickTop = ()=>{
    const observer = new IntersectionObserver(entries => {
        if(entries[0].isIntersecting){
            document.querySelector('.stick-bar').classList.remove('fix-top')
        } else {
            document.querySelector('.stick-bar').classList.add('fix-top')
        }
    }, {
        root:document.querySelector('.stick-top-contain'),
    })

    observer.observe(document.querySelector('.top-reference'))
}
```

效果如下：
（终于有个可以上传的例子了！，我拍的照片是不是很好看!）
![QQ20190926-161255-HD](https://user-images.githubusercontent.com/18004081/65670779-9f9bdc80-e078-11e9-8e38-94747f2119c8.gif)


## 兼容性

IE不兼容，不过有[官方的polyfill](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fw3c%2FIntersectionObserver%2Ftree%2Fmaster%2Fpolyfill)



## 最后

更多的一些例子请移步参考：[intersectionDemo][2]，感觉还能做很多其他好玩的事情，欢迎大家补充~



[1]: https://developer.mozilla.org/zh-CN/docs/Web/API/Intersection_Observer_API
[2]: https://github.com/includeios/document/tree/master/demo/intersectionDemo