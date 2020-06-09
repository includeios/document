---
title: git合并原理 (一) -- 内部原理
 ## ## 更新于2020年06月09日
date: 2020-06-09
tag: [git]
---
相信刚开始使用git的时候大家都会遇到这些困惑：为什么我本地有些文件合并完远程代码就消失了？我到底应该什么时候使用rebase什么时候使用merge？为什么我合并几个commit记录会有这么这么多冲突？

在我们搞清楚git的底层合并原理时，这些问题就迎刃而解了。

## Tree-Way Merge

假设有两个同学在各自的分支上对同一个文件进行修改，如下图

![image-20200603204547200](/images/2020/git1/image-20200603204547200.png)

这个时候我们需要合并两个分支成一个分支，如果我们只对这两个文件进行对比，那么在代码合并时，只知道这两个文件在第30行有差异，却不知道应该采纳谁的版本。

如果我知道这个文件的”原件“，那么通过和”原件“比对代码就能推算出应该采用谁的版本：

![image-20200603205137404](/images/2020/git1/image-20200603205137404.png)

图示可以看出，Mine中的代码和Base一样，说明Mine并没有对这个文件做修改，而Yours中的代码合Base不一样，说明Yours在这个文件的基础上修改了内容，那么Yours和Mine合并应该采用Yours的内容。

当然还有一种情况是三个文件的代码都不相同，这就需要我们自己手动去解决冲突了：

![image-20200603205528518](/images/2020/git1/image-20200603205528518.png)

从上面的例子可以看出采用Tree-Way-Marge原理来合并代码有个重要前提是可以找到两份代码的”原件”，而git因为记录了文件的提交历史，再通过自身的合并算法就可以找到两个commit的公共commit是哪个，从而通过比对代码来进行合并。

那么后面我们就来详细说一下git是如何记录提交历史和git的合并算法是怎么推算出公共commit的。

## git是如何记录提交历史的

我们在控制台执行`git init`，git会创建一个`.git`目录，这个目录包含了几乎所有git存储和操作的东西，结构如下：

```
- config  //项目特有的配置选项
- description  //仓库描述信息，主要给gitweb等git托管系统使用
- HEAD  //文件，记录目前正在使用的分支
- hooks/  // 目录，包含git的钩子脚本
- info/   // 目录，包含一个全局性排除文件， 用以放置那些不希望被记录在 .gitignore 文件中的忽略模式
- objects/   //目录，存储所有的数据内容，
- refs/   //目录，存储指向数据的提交对象指针
```

其中HEAD文件，refs和objects目录是git能够记录提交历史的关键所在。

### blob object

接下来我们创建两个文件，并通过`git add .`将修改提交到git暂存区中:

```sqlite
echo '111' > a.txt
echo '222' > b.txt
git add .
```

这个时候再去.git/object目录下，你会发现仓库里多了两个文件：

![image-20200604170403533](/images/2020/git1/image-20200604170403533.png)

> tree命令： 打印指定目录下的文件结构，mac需要自行安装：`brew install tree -g`

好奇的我们再来打印一下这两个文件里的内容：

```js
cat c2/00906
cat d5/234eb
```

![image-20200604170800351](/images/2020/git1/image-20200604170800351.png)

发现输出了一串乱码，这是因为git将信息压缩成二进制文件，不过没关系git提供`cat-file`命令来取回转码前的数据，`-t` 可以查看object文件的类型，`-p `可以查看object文件的具体内容。

> git cat-file [-t] [-p]  object filename   object的名字不用输入全，能唯一区分就好（一般都是6位）

```js
git cat-file -t c20090
git cat-file -p c20090
```

![image-20200604171246787](/images/2020/git1/image-20200604171246787.png)

可以发现这个object文件是一个blob类型的节点，他的内容是”222“，也就是说这个object存储着`b.txt`文件的内容。

这是我们遇到的第一种git object：blob，它只存储一个文件的内容，不包含文件名等其他信息。该内容加上特定头部信息一起的 SHA-1 校验和为文件命名，作为这个object在git仓库中的唯一身份证。 校验和的前两个字符用于命名子目录，余下的 38 个字符则用作文件名。

此时，我们的git仓库长这样：

![image-20200604211814186](/images/2020/git1/image-20200604211814186.png)

### tree object

我们继续探索，在控制台输入`git commit -m 'feat: 第一次commit'`，打印objects文件目录如下：

![image-20200604212656020](/images/2020/git1/image-20200604212656020.png)

继续`cat-file`查看新增的两个文件的类型和内容：

![image-20200604212748766](/images/2020/git1/image-20200604212748766.png)

这里我们遇到了第二个git object类型：tree，它将当前的目录结构打了个快照，从它的存储内容中可以发现它存储了一个目录结构，以及每个文件的模式编号，对应的blob object Hash值，文件名。

此刻我们的git仓库是这样的：

![image-20200604213736863](/images/2020/git1/image-20200604213736863.png)

### commit object

接着看另一个object文件的内容：

![image-20200604214050940](/images/2020/git1/image-20200604214050940.png)

至此我们得到了第三个git object类型：commit，它记录了当前项目tree object的Hash，作者/提交者的信息，以及这条commit的提交注释。

此刻我们的仓库是这样的:

![image-20200604214833936](/images/2020/git1/image-20200604214833936.png)

### HEAD and refs

我们再来看下HEAD和refs的内容：

![image-20200604215750564](/images/2020/git1/image-20200604215750564.png)

refs下存储着你所有分支的当前commit object Hash，HEAD相当于一个指针，指向refs中你当前的分支。

**概括来说整个引用关系就是：HEAD里面的内容是当前分支的ref，而当前ref的内容是commit hash，commit object内容是 tree hash，tree object的内容是blob hash，blob存储着文件的具体内容**

![image-20200604220840984](/images/2020/git1/image-20200604220840984.png)

为了更具象的看一下ref和HEAD指针代表值的含义，我们切换到一个新分支重复我们刚才的操作，控制台输入：

```sqlite
git checkout -b dev
echo '333' > a.txt
git add .
git commit -m 'feat: 第二次dev提交'
```

可以看到object下又多了三个文件：

![image-20200604220428873](/images/2020/git1/image-20200604220428873.png)

我们依次输出一下三个文件的内容和类型：

![image-20200604220413262](/images/2020/git1/image-20200604220413262.png)

依次多了blob object，tree object和commit object，因为我们并没有修改b.text文件的内容，所以仓库里和b.text有关的blob object依旧是之前的那一个。

再看一下此刻的HEAD和refs下的内容：

![image-20200604220529268](/images/2020/git1/image-20200604220529268.png)

refs目录下增加了对应的heads/dev文件，记录的是我们新生成的commit object Hash，HEAD里的内容也变成了当前分支：dev。

此时的仓库为：

![image-20200607103844138](/images/2020/git1/image-20200607103844138.png)

### commit 和 commit

到这一步，我们已经能弄清楚git是如何保存我们在代码仓库里写的代码，以及如何区分不同分支下的代码是什么样的。下面我们来看一下同一个分支的多个commit记录是怎么关联起来的？

实际上commit object除了记录当前项目tree object的Hash，还会记录前一次提价的commit object Hash，上面那个例子是因为我们是第一次提交，所以没有对应的记录值。

在控制台输入：

```sqlite
git checkout master
echo '444' > a.txt
git add .
git commit -m 'feat: 第二次提交'
```

继续打印objects目录下新增的object类容和类型：

![image-20200607104155632](/images/2020/git1/image-20200607104155632.png)

![image-20200607104918776](/images/2020/git1/image-20200607104918776.png)

可以看到commit的内容里多了一个parent字段，记录了上一次提交的commit object Hash。

此刻的仓库长这样：

![image-20200607105333667](/images/2020/git1/image-20200607105333667.png)

通过这种方法，一个分支下的所有提交就能像一个树一样串联起来了。
