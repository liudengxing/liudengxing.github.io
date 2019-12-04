---
title: Python StringPool 存在吗 ？
thumbnail: 'https://piccdn.freejishu.com/images/2019/08/09/PnYKVc.jpg'
abbrlink: 5a6e9021
date: 2019-08-09 10:42:12
tags:
categories:
---

*这是 六等星 的第 30 篇文章* 



开篇给想直接知道答案的朋友们答案 ：Python 存在 interned 用于存储 String ，再次生成相同 String 时 Python 会将变量链接至原有 String 位置 。



在学习 *《Learn Python Hard Way》* 的时候遇到了这么一个问题 ，判断 “Test” == “Test” 是否为 True ，我想了想 ，这取决于 Python 在新建字符串时的策略 ，就进行了简单的测试 。测试如下 ：

```
# Stringpool.py

str1 = "test"
str2 = "test"

print(str1 == str2) # 检测 str1 str2 值是否相等 : True
print(str1 is str2) # 检测 str1 str2 是否指向同一地址 : True


str3 = "testA"
str4 = str1 + "A"

print(str3 == str4) # 检测 str3 str4 值是否相等 : True
print(str3 is str4) # 检测 str3 str4 是否指向同一地址 : False

str5 = str("test")
str6 = str("test")

print(str6 == str5) # 检测 str5 str6 值是否相等 : True
print(str6 is str5) # 检测 str5 str6 是否指向同一地址 : True

str7 = str("test" * 10000)
str8 = str("test" * 10000)

print(str7 == str8) # 检测 str7 str8 值是否相等 : True
print(str7 is str8)  # 检测 str7 str8 是否指向同一地址 : False
```



在结合网上（指 Google）的一些资料后总结如下 ：



在 Python 中 String 为不可变对象 ，以便于 实现（Implementation） 决定是否 存储（intern - C# 术语，指将 String 储存于 pool 中）String 。在新建 String 时， Python 会创建一个临时的字符串对象 ，再将该字符串对象与 interned 里已存在的 String 进行对比 ，如果已存在 ，就销毁临时对象 ，将引用指向 interned 中的字符串对象 。但这一过程并不是 100% 适用所有情况 ，拼接生成的字符串不会自动存入 interned （str3 与 str4）、在字符串长度足够长时将不会生效 。（参考 str7 与 str8 ，对于长字符串过于低效）



我们在上面使用的都是固定的内容生成的 String ，如果是使用参数传递方式生成 String 呢 ？

这次我们直接看源码 ，参考 [Are strings pooled in Python](https://stackoverflow.com/questions/2519580/are-strings-pooled-in-python)

```
static PyStringObject *characters[UCHAR_MAX + 1];

...

PyObject *
PyString_FromStringAndSize(const char *str, Py_ssize_t size)
{

...

    if (size == 1 && str != NULL &&
    (op = characters[*str & UCHAR_MAX]) != NULL)
    {
        #ifdef COUNT_ALLOCS
            one_strings++;
        #endif

        Py_INCREF(op);
        return (PyObject *)op;
    }

...
```



可以看到 ，大部分情况下 ，使用参数传递的方式构造 String 都会新建一个 String 对象 ：

```
a = str(num)
b = str(num)
print (a is b) # False
```

只有在 String 长度为 1 时会在 interned 中匹配对应值 。



