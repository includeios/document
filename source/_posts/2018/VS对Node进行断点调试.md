---
title: 使用Visual Studio Code对Node.js进行断点调试
date: 2018-07-20
tags: [node,vscode]
---

## 1.进入vscode，前往虫子的那个页面
![image](https://user-images.githubusercontent.com/18004081/47766305-feda5b00-dd08-11e8-919e-ac5645383506.png)

## 2.点击调试旁边的那个下拉框，选择“添加配置”
![image](https://user-images.githubusercontent.com/18004081/47766309-039f0f00-dd09-11e8-8ca2-8cd0fe1f7f46.png)

## 3.选择Node.js:启动程序
![image](https://user-images.githubusercontent.com/18004081/47766317-0ef23a80-dd09-11e8-9be0-fb79fa49e115.png)

之后，项目的根目录会生成一个.vscode的目录，这个目录中存放了各种各样的VScode编译器的配置。现在这个文件夹下面就放了一个叫launch.json的文件，是我们debug的配置入口。
![image](https://user-images.githubusercontent.com/18004081/47766325-174a7580-dd09-11e8-9ada-1fde28918a00.png)

- 现在 launch.json已经自动为我们生成了一些配置项字段，具体的配置项字段可以参考https://go.microsoft.com/fwlink/?linkid=830387 的解释。
- 其中比较重要的是**program**字段，这个字段定义了整个调试器的入口，开启调试器的时候会从这个入口启动程序。
- 一般生成 lanch.json的时候这个字段已经有值了，这是因为VScode在初始化launch.json的时候会查看项目的**package.json**中是否有包含键名为**start**的scripts，如果有的话，就会把start配置的内容作为program字段的值。
- 我这个项目是通过 dev 命令来启动服务的，将dev后面的内容修改到 program 下：

![image](https://user-images.githubusercontent.com/18004081/47766332-203b4700-dd09-11e8-8ac0-84b744f6b610.png)

- 如果启动调试器的时候需要添加参数，可以通过**args**字段来设置，环境通过 env 字段来设置，例子如下：

![image](https://user-images.githubusercontent.com/18004081/47766346-27faeb80-dd09-11e8-99ad-6bffb3b20925.png)

## 4.点击调试旁边绿色运行箭头，就可以开始调试了，可以通过控制面板来查看调试的信息

![image](https://user-images.githubusercontent.com/18004081/47766355-2df0cc80-dd09-11e8-9b94-70b1f0b65b7c.png)

## 5.然后我们设置一个断点，重新启动调试服务 

![image](https://user-images.githubusercontent.com/18004081/47766365-35b07100-dd09-11e8-8236-02528f333d8c.png)

 运行的调试面板大概就如上图所示，调试控制台面板亮着的按钮从左到右依次是运行，单步跳过，单步调试，单步跳出，重启和停止调试。当鼠标放在断点前的变量或者参数上，可以查看该变量当前的数值，与Chrome开发者工具的调试一致。可以在面板左边那里查看上下文环境，在监视里面监听关注的变量当前值，控制台会显示console.log的打印值。还是很很很很方便的~






