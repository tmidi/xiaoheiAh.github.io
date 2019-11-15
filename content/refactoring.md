---
title: "refactor"
date: 2019-08-15T13:19:18+08:00
draft: true
---



# 「重构笔记」

## 代码坏味道

1. 重复代码(duplicated code)
   * 单纯的重复代码 —> idea就可以将其抽出来，多个地方同时引用
   * 互为兄弟的子类含相同的表达式，也可以抽出来，再拆一个模板方法来连起来
   * 不同的逻辑拥有相同一部分逻辑，就直接拆出来放到一个地方公用。如公司的`trandfer` 包
2. 过长函数(Long Method)
   * 就需要分解函数把函数变小
3. 过大的类(Large Class)
   * 单个类做了太多的事儿
4. 过长的参数列(Long Parameter List)

