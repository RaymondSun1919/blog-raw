---
toc:
    depth_from: 1
    depth_to: 4
title: 错排问题 - 算法 - huangfeihu.xyz
---
@import "/mystyle.less"

## 错排问题
> 返回:house:[首页](../../index.html)，:rocket:[算法](../index.html)

[TOC]
---
### 介绍
> 参考：[http://oi-wiki.com/math/combination/#_11](http://oi-wiki.com/math/combination/#_11)

我们把错位排列问题具体化，考虑这样一个问题：

$n$ 封不同的信，编号分别是 $1,2,3,4,5$ ，现在要把这五封信放在编号 $1,2,3,4,5$ 的信封中，要求信封的编号与信的编号不一样。问有多少种不同的放置方法？
假设我们考虑到第 $n$ 个信封，初始时我们暂时把第 $n$ 封信放在第 $n$ 个信封中，然后考虑两种情况的递推：
- 前面 $n-1$ 个信封全部装错；
- 前面 $n-1$ 个信封有一个没有装错其余全部装错。

对于第一种情况，前面 $n-1$ 个信封全部装错：因为前面 $n-1$ 个已经全部装错了，所以第 $n$ 封只需要与前面任一一个位置交换即可，总共有 $f(n-1)\times(n-1)$ 种情况。

对于第二种情况，前面 $n-1$ 个信封有一个没有装错其余全部装错：考虑这种情况的目的在于，若 $n-1$ 个信封中如果有一个没装错，那么我们把那个没装错的与 $n$ 交换，即可得到一个全错位排列情况。

其他情况，我们不可能通过一次操作来把它变成一个长度为 $n$ 的错排。于是可得错位排列的递推式为 $f(n)=(n-1)(f(n-1)+f(n-2))$ 。
错位排列数列的前几项为 $0, 1, 2, 9, 44, 265$ 。