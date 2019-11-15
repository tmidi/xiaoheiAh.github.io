---
title: "npm"
date: 2019-08-15T13:19:18+08:00
draft: true
---

# NPM笔记

node包管理器。
## [nvm](https://github.com/creationix/nvm)
管理node版本的工具，其他管理node版本的还有`n`。特点就是当本地项目需要来回切换node版本进行兼容时，`nvm`或`n`可以很方便的来操作切换.
#### 安装
```bash
# github官网上有对应下载，官方提供脚本
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
# 安装完成后更新环境变量
# 修改 /etc/profile或者.bashrc .zshrc ,添加下列环境变量
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
```
#### nvm使用
```bash
nvm install <node版本> # 版本一般为4.2.2类似这样  可以简写 4或者4.2 此时默认下载该系列最新版
nvm use <node版本> # 切换版本 版本格式同上
```
