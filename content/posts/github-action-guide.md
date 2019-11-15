---
title: "Github Action Guide"
date: 2019-09-05T09:26:43+08:00
draft: true
markup: "mmark"
tags : [github action guide]
---

### Actions & Workflows

`Github Actions`有两个组件:

1. **action**
2. 使用action的**workflow**

一个工作流(`workflow`)可以包括多个`actions`,没有个action有自己的职责.所以我们需要将文件关联到自己文件目录下的`action`中.

## Step 1: 创建一个`Dockerfile`

`Actions`有两种类型: `container actions` 和 `JavaScript actions`.





#### Actions

工作流由`jobs`组成,`jobs`由`steps`组成.



#### workfile属性介绍

* `jobs`: 一个工作流运行的基础组件
* `build`: 标志我们触发这个job
* `name`: job name
* `uses: actions/checkout@master`: 使用我们`repository`代码的拷贝
* `uses: ./action-a`: 使用这个目录下的`action`
* `env`:环境变量







