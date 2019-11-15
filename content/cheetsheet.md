---
title: "cheatsheet"
date: 2019-08-15T13:19:18+08:00
draft: true

---

# 常用idea快捷键

- 大小写切换 `command+shift+u`
- surround with `option+command+t`
- 进入实现类 `option+command+b`
- 查找当前类中的方法和变量 `command+F12`

# 自定义快捷键

- 右键调出长下文菜单  `command+shift+L`

# git

git remote prune origin

同步远程仓库,删除本地仓库中无用的分支(保持与远程仓库分支一致,远程没有的就删掉)

# vim

## 移动

<C-o> 跳转到光标上次停留的位置(往后调)

<C-i> 同上(往前跳)



**zz**: 将当前行移动到屏幕中央

## 编辑

**A**: 在当前行末尾编辑

**I**:在当前行首编辑



分别更改这些配对标点符号中的文本内容
ci’、ci”、ci(、ci[、ci{、ci< -

分别删除这些配对标点符号中的文本内容 
di’、di”、di(或dib、di[、di{或diB、di< -

分别复制这些配对标点符号中的文本内容 
yi’、yi”、yi(、yi[、yi{、yi< -

分别选中这些配对标点符号中的文本内容
vi’、vi”、vi(、vi[、vi{、vi< -



## 配置

**vs ~/.vimrc** 热更新配置
