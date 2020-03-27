---
title: 蓝屏导致的 git 文件损坏解决方法
abbrlink: db39c52e
date: 2020-03-27 09:59:30
tags:
categories:
thumbnail:
---



首先运行

```
git fsck --al
```

可以看到有许多文件直接报错 ，这时我们需要删除所有报 error 的文件



```
git fetch --all
```

 删除错误文件之后就可以再次从远程仓库获取对象以及引用 ，在这一步分支列表被修复



```
git reset --hard master
```

重设分支到 master 分支上



```
git batch
```

这时候就可以看到所有的可用分支了 ，直接切换到需要的分支上



```
git pull
```

再次拉取分支即可