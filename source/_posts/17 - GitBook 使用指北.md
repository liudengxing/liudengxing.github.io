---
title: GitBook使用指北
tags:
  - GitBook
categories:
  - 操作手册
abbrlink: ea08642a
date: 2019-05-20 14:11:31
thumbnail: https://piccdn.freejishu.com/images/2019/05/20/PnVog0.jpg
---
# GitBook使用指北

*这是六等星的小宇宙的第 17 篇文章  
写作时间 : 4小时  
总字数 : 3083字   
预计阅读时间 : 6分钟*


#### 什么是GitBook？

一进GitBook官网就能看到两行标语

> Document Everything!
>
> For you ,your users, and your team

简单来说，GitBook就是一个基于MarkDown语法的文档生成器，他可以生成Html网页文档便于你发布到Html容器中供自己、团队以及你的用户阅读。



#### 为什么我们要使用GitBook写文档？

在以前，我们使用最多的文档编写工具应该是 Microsoft Word，功能强大但臃肿，在80%的时间里我们只会用到20%的功能，同时二进制保存，格式，软件不兼容，版本控制复杂，无法实时分享预览，无法多人协作的问题还困扰着我们。在这一方面，我们可以有更好的选择，使用GitBook + Git + Typora可以解决上述问题。

###### Github支持

在程序员的世界里，我们离不开三个网站，面向Google编程，面向Github造轮子，面向Stack Overflow改Bug。

在Github中，每个项目都需要写一个Readme，大一些的Readme就等于一个Document，GitBook在优雅的实现Document上有着无与伦比的优势，原生基于MD，可以搭载于Github page，这些都是GitBook的优势所在，现在在Github上有许多的项目的Document是基于Gitbook实现的。

#### 怎么安装Gitbook

GitBook是基于Node.js的，我们首先需要安装[Node.js](https://nodejs.org/en/download/), 选择适合的版本下载安装即可。

最新的Node.js都会默认安装node包管理工具npm，所以我们只需要

```
npm install -g gitbook -cli
```

安装完成后我们就可以愉快的使用GitBook了，就这么简单。

不过如果我们想要优雅的使用GitBook，那我们需要的不仅仅是这些。

我们还需要

一个优雅的MD编辑器

一个分享文档的方式

#### 编译器的选择

我们在编写文档期间面向最多的并不是GitBook,而是我们的markDown编辑器，一个好的MarkDown编译器能带给你极佳的书写体验。

我使用过的MD编辑器有：

- Microsoft notepad（大家都使用过的巨硬家的文本编辑器，TXT转MD容易出现格式问题）
- Bear（编辑器中的瑞士军刀，ios与Macos双平台，用过的最舒适的编辑器）
- Atom（什么文档格式都能做，开心的时候还能编个程（笑），我用了很长时间的Atom，比微软自带的编辑器是好不少，但毕竟不是专业MD编辑器，想优雅的话需要自配许多的插件）
- 纯纯写作（安卓端纯文本编辑器，应该是安卓上极佳的编辑器之一了，在手机端的编辑体验很舒适）
- Typora（主打“所见即所得”的编辑器，界面简洁，MacOS，WIn双平台都有，Windows还在测试阶段，书写体验极佳，十分优雅）
- MarkdownPad（Windows端功能最全的编辑器，就是丑了点）

在以上这些编辑器中，我推荐Bear（Macos），Windows上如果你愿意折腾的话我推荐Atom，应为你可以把界面，书写体验等全都调成你喜欢的样式。如果你想即开即用，那么Typora就是你的不二选择，你可以在不用设置的情况下就体会到书写的快乐。



####  部署方式的选择

GitBook本身是自带空间的，可以实现无缝同步使用，但由于 404 的问题，我们在使用GitBook时需要科学手段，所以更方便的使用方式是 Git + GitHub，使用GitHub page来实现访问。


#### GitBook初始化

在以前，如果你需要写一本书，是不是新建一个文件夹，在文件夹中新建一个Word文件，“我的第一本书”，然后成为无情的码字机器，疯狂的敲打的键盘来输出你的思想。

在GitBook的体系下，我们起步的方式就完全不同了，我们首先要在新建的文件夹下执行命令

```
gitbook init
```

来执行GitBook的初始化，GitBook会为你在目录下创建两个初始文件，分别为 README.md 与 SUMMARY.md

![Readme](https://piccdn.freejishu.com/images/2019/05/20/PnVpb8.png)

README是以及被我修改过后的样子了，SUMMARY还是默认格式

![Summary](https://piccdn.freejishu.com/images/2019/05/20/PnV69O.png)

README没什么好说的，大家都知道这是做什么的。

SUMMARY使用的是MD的` - `缩进来判断目录层级，具体可以参考[MarkDown语法指北](https://nightmare233.top/posts/16ffb977.html)。

#### 新建文章

新建文章的话直接在目录下创建一个新的md文件即可，不过推荐通过文件夹层级体现文章层级，不然当文章过多的时候容易混乱。


GitBook是支持插件的，这使得GitBook可以按照你的想法进行定制。



#### 安装toggle-chapters插件实现目录折叠

npm安装

```
npm install gitbook-plugin-toggle-chapters --save-dev
```

在书籍根目录下新建Book.json配置插件

```
{
    "plugins":["toggle-chapters"]
}
```

再次部署后我们就可以看到目录已经被折叠了，在点开后才会展开


GitBook支持将缩写内容进行制定导出，例如导出静态网站，PDF，epub，mobi等。

#### 构建书籍

```
gitbook build [书籍路径（可选）] [输出路径（可选）]
```

#### 搭建本地服务

```
gitbook serve [--port 2333]
```

端口是可选的，默认端口是4000，可以直接访问[localhost](localhost:4000)查看书籍。

#### 导出不同格式的电子书

```
gitbook [书籍格式] [书籍路径] [导出路径]
```

例如导出PDF就是

```
gitbook pdf ./ ./mybook.pdf
```

在导出PDF等格式时可能会出现导出失败，这时候你需要安装ebook-convert或在Typora中安装Pandoc进行导出。



#### 使用Git + GitHub Page实现远程访问

上面的方法实现的都是本地的访问方式实现，在我们不在本地环境的时候，我们就需要将内容托管到服务器上，这就要求有简单的建站经验并且有自己的服务器。我们还有一个更简洁的解决方法，托管于GitHub，并借助GitHub Page实现网页访问。

我们首先运行一遍

```
gitbook build
```

生成静态网站页面

我们在把根目录下的_book文件夹内的内容push至你的github仓库中

再在仓库 Settiing 中启用 GitHub Pages，选择master分支。

或者你可以将网站导入到分支 gh-pages中，这是Github支持的文档分支。

![GithubPages](https://piccdn.freejishu.com/images/2019/05/20/PnV25B.png)
