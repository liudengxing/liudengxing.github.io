---
title: Git使用指南
tags:
  - Git
categories:
  - 操作手册
thumbnail: https://piccdn.freejishu.com/images/2019/04/19/PnxEkA.jpg
abbrlink: 74add858
date: 2019-04-12 14:30:07
---



# Git使用指南

*这是六等星的小宇宙的第 9 篇文章  
写作时间 : 30分钟  
总字数 : 293字  
预计阅读时间 : 1分钟*

<br/>

# Git常用指令

创建Git仓库  

```
git init
```

添加文件  

```
git add filename  
```

```
git add .
```

是提交所有修改  

提交文件到仓库  

```
git commit -m "new file"
```

-m指令为添加解释，即这个版本做了什么  

查看历史记录  

```
git log
```

版本回退  

```
git reset
```

查看修改状态  

```
git status
```

添加远程仓库  

```
git remote add origin address
```
之后就可以直接用  

```
git push
```
命令推送本地分支到远程  

```
git clone 
```

则是拉取远程分支到本地  

删除绑定的远程仓库 

```
git remote remove name
```



### Git 创建 、合并 、删除分支

查看所有分支 ：

```
$ git branch -a
  develop
* master
  remotes/origin/develop
  remotes/origin/master
  remotes/origin/patch-1
```

remotes/origin/master 代表的就是远程分支 ，其他为本地分支

#### 新建分支

```
git branch dev
```

#### 切换分支

```
git checkout dev
```

或者可以使用

```
git checkout -b dev
```

相当于执行了新建 dev 分支并切换到 dev 分支两条命令 。

#### 删除分支

```
git push origin --delete develop  # 删除远程分支 develop
git branch -D develop  # 删除本地分支 develop
```

#### 合并分支

```
git merge dev # 合并 dev 到当前分支
```



#### 查看当前用户

```
git config user.name # 查看当前用户名
git config user.email # 查看当前用户邮箱
```

