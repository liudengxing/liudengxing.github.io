---
title: Mysql Decimal 格式数据探索
abbrlink: 8cfb7bd8
date: 2019-12-24 16:49:37
tags:
categories:
thumbnail:
---

Decimal 是 Mysql 的一种高精度数据类型 ，最大可达 128 bit ，在大部分使用方式下不存在精度损失 。

同样带有小数位的数据类型还有 float 和 double ，其中

- float 浮点型 字节数为 4 ，32 bit 大小 ，数据范围为 -1.7E38 - 1.7E38

- Double  双精度整形 字节数为 8 ，64 bit 大小 ，数据范围为 -1.7E308 - 1.7E308
- decimal 数字型字节数 16 ，128 bit 大小 



对数据的声明语法为 DECIMAL(M,D) , 取值如下 :

- M (Maximum number) 是最大位数，范围为 1 - 65 ，默认为 10
- D (Digits) 是小数位 ，范围为 0 - 30 ，默认为 0 ，并且值要小于 M

其中 M 包含有 D ，也就是 整数的大小是 M - D ，例如 DECIMAL(6,2) 代表最大存储位数为 6 ，其中包含 4 位整数位与 2 位小数位 ，取值范围是 -9999.99 - 9999.99 ，超出取值范围会报错 “ Out of range”

存储数值为 Decima 时，会对首部进行去 0 操作 ，同时对小数位不足的进行补 0 