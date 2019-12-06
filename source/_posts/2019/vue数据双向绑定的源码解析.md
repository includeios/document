---
title: vue数据双向绑定的源码解析
 ## ## 更新于2019年1月14日
date: 2019-10-30
tag: [js,vue]
---

关于vue是观察者模式还是发布订阅模式网上一直众说纷纭。
我之前一直认为vue是发布订阅模式，dep对象作为发布订阅模式里的event channel来处理数据更新和通知，后来听了字节跳动银国徽老师的《Mvvm设计模式》课程，阅读完vue的源码后，对vue是如何实现双向绑定有个更清楚的认识。

 ## 观察者模式和发布订阅模式

首先简单介绍一下观察者模式和发布订阅模式的概念和他们的区别：

![image](https://user-images.githubusercontent.com/18004081/67863951-d07f9d80-fb5f-11e9-9e9b-40e35716cc6d.png)


**观察者模式**

> 建立一种对象与对象之间的依赖关系，一个对象发生改变时将自动通知其他对象，其他对象将相应做出反应。在此，发生改变的对象称为观察目标，而被通知的对象称为观察者，一个观察目标可以对应多个观察者，而且这些观察者之间没有相互联系，可以根据需要增加和删除观察者，使得系统更易于扩展，这就是观察者模式的模式动机。

以卖房子为例：一个房东想卖掉自己的房子，于是他发布了一个售卖价格，有好几个买家心仪这套房子，但是又觉得价格有点小贵而犹豫不决，于是他们都关注着这套房子的房价伺机行动。

在这个例子里。房东就是那个观察目标，买家就是观察者，一个房东可能和多个买家对接，一旦房价发生变化就会告诉他们，而几个买家之间是没有相互联系的。

**发布订阅模式**

> 发布订阅模式是观察者模式的实现，更利于系统的解耦和重用

发布订阅模式和观察者模式最大的不同就是存在一个“中介”来统一处理一些事件，使得订阅者（观察者）和发布者（观察目标）没有直接联系。

还是以上面的卖房子为例：房东想卖掉自己的房子，但是又觉得自己每次更改房价就要挨个打电话通知买家自己的价格变化了实在太麻烦，于是他找了一个中介贝壳，把自己的房子信息托管给他们，以后每次更改房价他就只和中介贝壳的工作人员说一声就好了，再由贝壳的工作人员来通知那些想买房子的人。

至此，房东不再和买家直接打交道，房东甚至不知道有哪些买家关注着自己的房子信息，房东也不关心贝壳的工作人员是如何通知买家的，可能是群发消息，避免了房东挨个打电话通知的重复劳动性。



 ## 为什么说vue是观察者模式

对vue有一定了解的朋友应该都知道，vue实现双向绑定有几个重要的对象：Observer，Watcher，Dep。

![image](https://user-images.githubusercontent.com/18004081/67863963-d9706f00-fb5f-11e9-9b34-a1311eff0439.png)


乍一眼看这张图，觉得这妥妥的是上面说的发布订阅者模式呀，Observer是观察目标，Watcher是观察者，Dep是中间那个中介Event Channel。

实际上阅读完源码后，你会发现Dep没有做任何事情，他只是一个简单的Watcher的数组集合，在每次Observer数据发生更新时，循环遍历数组集合来挨个触发每个Watcher的update方法。

下面就通过解析源码来详细说一下这个实现过程吧！



 ## vue源码实现双向绑定

github地址：https://github.com/vuejs/vue.git

我们重点阅读 `src/core` 下这几个文件的内容：

![image](https://user-images.githubusercontent.com/18004081/67864034-f147f300-fb5f-11e9-9f94-74adf0a7522c.png)


首先看**init.js**下的Vue初始化方法：

```js
//options就是我们每次new一个Vue对象时传进去的那个对象
Vue.prototype.__init = function(options){
  //...
  vm._self = vm
  initLifecycle(vm)
  initEvents(vm)
  initRender(vm)
  callHook(vm,'beforeCreate')
  initInjections(vm) // resolve injections before data/props
  initState(vm)
  initProvide(vm) // resolve provide after data/props
  callHook(vm,'created')
  //...
}
```

初始化里主要完成了初始化事件、渲染、参数、注入等过程，并按照生命周期里的规则调用事件钩子的回调函数。我们重点关注初始化数据:**state.js**下的**initState**

```js
export function initState (vm) {
  vm._watchers = []
  const opts = vm.$options
  if (opts.props) initProps(vm, opts.props)
  if (opts.methods) initMethods(vm, opts.methods)
  if (opts.data) {
    initData(vm)
  } else {
    observe(vm._data = {}, true /* asRootData */)
  }
  if (opts.computed) initComputed(vm, opts.computed)
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch)
  }
}

```

这里依次检测参数中包含的 `props/methods/data/computed/watch` 并进入不同的函数进行初始化，我们只关心 `initData` ：

```js
function initData (vm) {
  let data = vm.$options.data
  data = vm._data = typeof data === 'function'
    ? getData(data, vm)
    : data || {}
  if (!isPlainObject(data)) {
    data = {}
    process.env.NODE_ENV !== 'production' && warn(
      'data functions should return an object:\n' +
      'https://vuejs.org/v2/guide/components.html#data-Must-Be-a-Function',
      vm
    )
  }
  // ...
  // observe data
  observe(data, true /* asRootData */)
}
```

可以看到 `data` 参数支持对象和回调函数，最终返回一个对象。

接下来就是重头戏了，我们如何将data参数设置为响应式的，下面阅读最后调用的**observe**函数

```js
/**
 * Attempt to create an observer instance for a value,
 * returns the new observer if successfully observed,
 * or the existing observer if the value already has one.
 */
export function observe (value ,asRootData){
  if (!isObject(value) || value instanceof VNode) {
    return
  }
  let ob
  if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {
    ob = value.__ob__
  } else if (
    shouldObserve &&
    !isServerRendering() &&
    (Array.isArray(value) || isPlainObject(value)) &&
    Object.isExtensible(value) &&
    !value._isVue
  ) {
    ob = new Observer(value)
  }
  if (asRootData && ob) {
    ob.vmCount++
  }
  return ob
}
```

源码里的注释写的比较清楚：observe函数生成一个基于value的观察目标 `new Observer(value)` ，如果存在则返回已存在的。Observer累保存在value的 `__ob__` 属性下。

下面我们来详细看一下Observer类的初始化函数（**observer/index.js**）：

```js
/**
 * Observer class that is attached to each observed
 * object. Once attached, the observer converts the target
 * object's property keys into getter/setters that
 * collect dependencies and dispatch updates.
 */
constructor (value) {
  this.value = value
  this.dep = new Dep()
  this.vmCount = 0 // number of vms that have this object as root $data
  // def函数是defineProperty的简单封装
  def(value, '__ob__', this)
  if (Array.isArray(value)) {
    // 在es5及更低版本的js里，无法完美继承数组，这里检测并选取合适的函数
    // protoAugment函数使用原型链继承，copyAugment函数使用原型链定义（即对每个数组defineProperty）
    if (hasProto) {
      protoAugment(value, arrayMethods)
    } else {
      copyAugment(value, arrayMethods, arrayKeys)
    }
    this.observeArray(value)
  } else {
    this.walk(value)
  }
}
```

如注释所说：它会被关联到每一个被检测的对象，使用 `getter/setter` 修改其默认读写，用于收集依赖和发布更新。至此，我们能想到Vue是如何进行双向数据绑定的，改写data的访问器属性：get/set，在get的时候添加观察者，在set的时候告知观察者。

在这一步，我们给观察目标Observer挂上一个dep属性，属性值是一个Dep对象，我们来看一下Dep的数据结构（Dep下的源码后面再详细展开）：

```js
//Dep对象的结构
id: 每个观察目标的唯一标识。
subs: 观察者列表。
target: 全局唯一的观察者对象，因为只能同时计算和更新一个观察者的值。
addSub(): 使用`push()`方法添加一个观察者。
removeSub(): 使用`splice()`方法移除一个观察者。
depend(): 将自己添加到当前观察者对象的依赖列表。
notify(): 在数据被更新时，会遍历subs对象，触发每一个观察者的更新。
```

这里可以看出Vue对于数组类型的value和非数组类型的value分别做了两种处理，是因为我们观察的只能是一个对象，而不能是一个数组，数字或者字符串。对于数组的处理这里不做详细展开了，有兴趣的同学可以自行阅读源码

对于非数组的value，调用了walk方法：

```js
 /**
   * Walk through all properties and convert them into
   * getter/setters. This method should only be called when
   * value type is Object.
   */
  walk (obj) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i])
    }
  }
```

注释：遍历obj下的属性并将其转换成使用 `getter/setter` 修改其默认读写的属性。所以 `defineReactive` 就是那个关键的函数了！

```js
/**
 * Define a reactive property on an Object.
 */
export function defineReactive (obj,key,val,customSetter,shallow) {
  const dep = new Dep()

  const property = Object.getOwnPropertyDescriptor(obj, key)
  if (property && property.configurable === false) {
    return
  }

  // cater for pre-defined getter/setters
  const getter = property && property.get
  const setter = property && property.set
  if ((!getter || setter) && arguments.length === 2) {
    val = obj[key]
  }

  let childOb = !shallow && observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val
      if (Dep.target) {
        dep.depend()
        if (childOb) {
          childOb.dep.depend()
          if (Array.isArray(value)) {
            dependArray(value)
          }
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val 
      /* eslint-disable no-self-compare */
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter()
      }
      // #7981: for accessor properties without setter
      if (getter && !setter) return
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      childOb = !shallow && observe(newVal)
      dep.notify()
    }
  })
}
```

终于看到了关键的重头戏！Vue在这里重写了属性的访问器属性get/set。

对于get访问器属性，在执行完属性原本的getter函数后，执行了**dep.dependd()**方法

关于条件判断里的**Dep**，代码里是这么定义的：

```js
// The current target watcher being evaluated.
// This is globally unique because only one watcher
// can be evaluated at a time.
Dep.target = null
const targetStack = []
```

注释看的出来：**Dep.targe**t是全局唯一的**watcher**对象，也就是当前正在指令计算的观察者，它会在计算时赋值成一个watcher对象，计算完成后赋值为null。

```js
export default class Dep{
	//...
  depend () {
    if (Dep.target) {
      Dep.target.addDep(this) /
    }
  },
 	//...    
}
```

depend的函数只是调用了**Dep.target.addDep()**，那我们看一下Watch类下的addDep函数的定义

```js
export default class Watch{
  //...
  /**
   * Add a dependency to this directive.
   */
  addDep (dep) {
    const id = dep.id
    if (!this.newDepIds.has(id)) {
      this.newDepIds.add(id)
      this.newDeps.push(dep)
      if (!this.depIds.has(id)) {
        dep.addSub(this)
      }
    }
  }
  //...
}
```

这里根据dep的id简单的做了一次去重操作，确定没有重复依赖后执行了**dep.addSub(this)**，于是我们再回到dep class下阅读关于addSub的定义：

```js
export default class Dep{
  //...
   //为这个观察对象下的dep属性添加
  //仅仅只是把Wacher压入subs堆栈里，并没有做其他任何操作
  addSub (Watcher) {
    this.subs.push(Watcher)
  }，
  //...
}
```

看到了没有！这里没有做任何的操作，只是把这个Watcher对象放到了这个Observer.dep.subs数组里！

那我们再看看setter函数下执行的**dep.notify**函数的定义：

```js
export default class Dep{
  //...
  notify () {
    // stabilize the subscriber list first
    const subs = this.subs.slice()
    if (process.env.NODE_ENV !== 'production' && !config.async) {
      // subs aren't sorted in scheduler if not running async
      // we need to sort them now to make sure they fire in correct
      // order
      subs.sort((a, b) => a.id - b.id)
    }
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
  //...
}
```

看到了没有！这里也没有对Watcher做任何的操作，只是依次执行subs队列里Watcher对象下的update方法。Dep其实只是一个Wacher对象的集合，方便每次Observe更新时通知所有的Wacher ---- 调用他们的update方法，本身并没有做任何的封装处理：Observer和Wacher依旧是强耦合状态。

至此，看完源码后同学们能理解为什么说Vue的数据绑定是观察者模式了吧。

关于Vue里Dep的定位，接着观察者模式里提到的卖房子的例子：他有点像房东收集所有买家电话号码的电话薄，房东在通知买家房价发生变化时可以通过查阅本子更快更方便的通知到所有买家，但本质上还是房东直接和买家发生交集的。



 ## 为什么vue要用观察者模式

课堂上有个同学提到了一个很有意思的说法，解释vue为什么要用观察者模式。

发布订阅模式其中一个特点是订阅通知的处理逻辑都交给了Event Channel来处理，这要带来的好处是发布者和订阅者关系解耦，代价就是引入了Event Channel会使代码逻辑更加复杂。

比如一个典型的发布订阅模型就是我们的dom事件，一个dom可能有多种类型的事件：click事件，mousedown事件...，不同的dom可能有相同类型的事件：两个button按钮都用了click事件。那么对于dom来说我究竟该出发哪一种类型的什么事件dom本身是不关心的，这个触发机制是浏览器来完成的，浏览器就充当了我们说的Event Channel角色。

而对于Vue这种数据双向绑定的应用场景，他不像dom那样有click/mousedown各种各样的事件类型，他仅仅只有一个更新的订阅通知，即，他只需要告诉他的观察者他更新了，而不用告诉观察者我是A类型的更新还是B类型的更新。

在这种情况下，直接的告知可能是最方便的做法，引入Event Channel反而会增加代码的复杂度。所以有的时候，没有哪种模式更好哪种模式更高级的说法，还是要根据具体的应用场景来做选择。



## 小结
- 发布订阅模式是观察者模式的一种实现，他两区别的关键在于是否有个充当“中介”的Event Channel。
- Vue的dep对象只是一个Watcher的集合，本身没有做任何操作，所以Vue的双向数据绑定属于观察者模式。
- 观察者模式和发布订阅模式没有谁更优的说法，应该根据具体的应用场景采取更合适的方法。


以上就是我对vue的初步了解，有什么问题或不准确的地方欢迎各方大佬们留言指正~