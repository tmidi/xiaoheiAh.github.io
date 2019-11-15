---
title: "exception"
date: 2019-08-15T13:19:18+08:00
draft: true
---

# 异常

> 参考:http://www.cnblogs.com/swiftma/p/5651377.html
> https://blog.csdn.net/huhui_cs/article/details/38817791
## checked/unchecked
异常中需要正确区分的是`受检查异常`和`不受检查异常`
1. checkedException编译器会检查,并让你做出处理,否则编译不会通过,主要包括:
    1. IOException
    2. SQLException
    3. 用户自定义异常
2. uncheckedException主要是指`运行时异常 RuntimeException`和`Error及其子类`,是在程序运行中可能出现的异常,需要开发自己去注意并处理,主要包括:
    1. NullPointerException
    2. IndexOutOfBoundsException
    3. OutOfMemoryError
    4. ClassCastException
## 异常规范
[阿里异常规范](https://www.jianshu.com/p/40323840bf08)
