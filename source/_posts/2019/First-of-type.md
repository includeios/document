---
title: First-of-type
date: 2019-11-11
tag: css
---

今天在写一个css样式时，试图用first-of-type来修改第一个拥有特殊class标签的样式，发现并未生效，后来发现一直以来自己对first-of-type，last-of-type，nth-of-type这类标签的运用场景理解错了。

## 问题

当时的html代码结构是这样的

```html
<div class="card">
  <div class="title">标题</div>
  <div class="row">一行元素</div>
  <div class="row">一行元素</div>
  <div class="row">一行元素</div>
</div>
```

其中每个 `一行元素` 中间相隔10px，而 `标题` 与 `行内元素` 相隔大一些：15px，当时的想法是对第一个class是row的 `一行元素` 做特殊的样式处理，css代码为：

```scss
.card .row{
  margin: 10px 0;
  &:first-of-type{
    margin-top: 14px;
  }
}
```

伪类没有生效

## MDN解释：

贴上了中英文解释

> [CSS](https://developer.mozilla.org/zh-CN/docs/Web/CSS) [伪类](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Pseudo-classes) **:first-of-type** 表示一组兄弟元素中其类型的第一个元素。

> The :first-of-type [CSS](https://developer.mozilla.org/en-US/docs/Web/CSS) [pseudo-class](https://developer.mozilla.org/en-US/docs/Web/CSS/Pseudo-classes) represents the first element of its type among a group of sibling elements.

这里乍一眼看容易产生误解，first-of-type作用于满足筛选条件的第一个元素，实际上这个伪类首先作用与element上，前面只是基于选中的element的再次筛选。

也就是说，上面的html结构能被first-of-type选中的element只有 `<div class="title">标题</div>` ，而我又对可以被选中元素加上class="row"的条件筛选，最后的结果就是没有符合的元素了。

## 举个例子

```html
<div class="father">
  <p class="a">a1</p>
  <p class="a">a2</p>
  <p class="a">a3</p>
  <div class="b">b4</p>
  <div class="b">b5</p>
  <p class="c">c6</p>
</div>
```

```css
//不加筛选条件 a1和b4会变红 
//他两分别是p元素和div元素在father下的第一个子元素
.father :first-of-type{
  color:red
}

//加上筛选条件.a  a1会变红
//上一个例子里挑选出来的元素中符合class=a的只有a1
.father .a:first-of-type{
  color:red
}

//加上筛选条件.b b4会变红，原因同上
.father .b:first-of-type{
  color:red
}

//加上筛选条件.c 
//虽然<p class="c">c6</p>的class=c，但是c6已经不是father下第一个p元素了，因此没有符合这个条件的元素
.father .c:first-of-type{
  color:red
}
```

## 问题的解决方案

将html结构修改成如下就好：

```html
<div class="card">
  <p class="title">标题</p>
  <div class="row">一行元素</div>
  <div class="row">一行元素</div>
  <div class="row">一行元素</div>
</div>
```

## 总结 

- first-of-type / last-of-type / nth-of-type / nth-last-of-type ....等伪类都是首先筛选符合条件的element元素，再根据自定义的筛选条件过滤一遍已经筛选过一次的element元素。

