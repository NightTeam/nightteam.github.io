# 发布文章流程

本流程简单介绍从本地发布文章到博客并部署的流程。

## 安装 Node.js

首先安装 Node.js，下载地址：[https://nodejs.org/zh-cn/download/](https://nodejs.org/zh-cn/download/)。

安装完毕之后确保可以使用 `npm` 命令。

## 安装 Hexo

使用 `npm` 全局安装 `hexo-cli` 工具，命令如下：

```
npm install -g hexo-cli
```

安装完毕之后确保可以使用 `hexo` 命令。

## 创建文章

使用如下命令创建一篇文章，如创建一篇「HelloWorld」的文章，命令如下：

```
hexo new hello-world
```

创建的文章会出现在 `source/_posts` 文件夹下，是 MarkDown 格式。

在文章开头通过如下格式添加必要信息：

```
---
title: 标题 # 自动创建，如 hello-world
date: 日期 # 自动创建，如2019-09-22 01:47:21
tags: 
- 标签1
- 标签2
- 标签3
categories:
- 分类1
- 分类2
---
```

开头下方撰写正文，MarkDown 格式书写即可。

## 发布

使用项目内脚本部署即可：

```
sh deploy.sh
```

或手动执行：

```
hexo clean
hexo generate
hexo deploy
```

如运行结果无报错，且提示已经部署到 master 分支，则证明部署成功。