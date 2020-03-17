---
title: 基于Hexo快速搭建个人博客(二) -- 部署篇
date: 2020-03-17
tag: [js]
---

上一篇里，你已经能在你的本地跑起一个localhost的项目了，那么，怎么让你的项目别人也能访问到呢，如何绑定自定义域名以及让各大搜索引擎能搜索到自己的博客呢？

## github pages / coding pages

>  感谢这些免费且便捷的托管工具

1. 首先，我们在github上新建一个项目，用来存储我们的博客代码：

![image-20200317181846915](/images/2020/image-20200317181846915.png)

2. 接下来在项目的_config.yml文件里，把我们新创建的项目地址配置好，这样后面运行hexo的命令行时就可以自动的把我们本地代码推到托管平台上了：

```
deploy:
  type: 'git'
  repository:
    github: 'https://github.com/includeios/includeios.github.io.git'
    coding: 'git@e.coding.net:atangge_tripod/atangge_tripod.git'
  branch: master
```

这里我分别配置了github和coding

3. 生成静态资源和推送网站

```bash
hexo clean 
hexo g 
hexo d
```

4. 在项目里开启github page选项

   ![image-20200317182450897](/images/2020/image-20200317182450897.png)

   往下滚，找到Github Pages选项，将Source改为master branch，最后点击Save按钮



![image-20200317182639804](/images/2020/image-20200317182639804.png)

Ok，此刻你在地址栏里输入：http://username.github.io（username是你github的名字），就能看到你的博客页面拉。

coding上的部署和github类似，不同的是他会生成一个随机字符作为你静态页面的访问地址，不过比起github，国内网访问coding的静态网站会更快点。

## 绑定自定义域名

假设你已经花了几百大洋拥有一个属于自己的域名，还没有买的小伙伴可以去腾讯云或者阿里云上买一个。

阿里云购买传送桶：https://wanwang.aliyun.com/domain/

> 这个文章的域名 tripod.fun 提取关键字**阿汤哥的鼎**中的**鼎**！！！翻译来的....
>
> 199，10年，199，你买不来吃亏，买不了上当

接下来在你的控制台输入

```
ping username.github.io（github上你的域名/coding上你的域名）
```

就能看到你的网站的ip地址了：

![image-20200317224700951](/images/2020/image-20200317224700951.png)

接下来我们回到购买域名的平台中心，在域名解析里面添加上我们刚刚得到的ip地址

![image-20200317224940477](/images/2020/image-20200317224940477.png)

接下来返回github page，在Custom domain里面添加上透露着金钱味道的域名就好啦~

![image-20200317225211367](/images/2020/image-20200317225211367.png)

## 在各大搜索引擎里添加站点

直接在百度里面搜索我们新绑定的域名，他会提醒你要不要收录这个地址

![image-20200317225448788](/images/2020/image-20200317225448788.png)

点击链接提交完后，这个网址百度就会记录啦

不过我们博客一般不会只有这一条链接，像我的博客http://www.tripod.fun/，每篇文章都会有属于自己的url地址，如果没写一篇文章都要手动添加一下岂不是很麻烦。

百度提供了认证主站后，后面的子站都可以自动收录的功能，有三种方法，这里推荐用文件验证：

![image-20200317225917043](/images/2020/image-20200317225917043.png)

按照提示下载好文件，放到source下面，重新运行一下构建命令行，就能看到构建好的代码根目录下有对应的html文件：

![image-20200317230554497](/images/2020/image-20200317230554497.png)

google与百度同理，

google的站点管理页面传送桶：https://search.google.com/search-console/links?resource_id=https%3A%2F%2Fincludeios.github.io%2F&hl=zh-CN

欧卡！push完代码后，再过几天让两大搜索引擎的爬虫亲密光顾你的博客内容一阵子，你就能在搜索引擎上搜索到你自己啦

![image-20200317230738177](/images/2020/image-20200317230738177.png)

从此以后，爷也是在江湖上留下过脚印的人...