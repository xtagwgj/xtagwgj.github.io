---
title: 使用Hexo在Github上搭建个人博客
date: 2017-05-05 10:21:17
tags: 
- Hexo
- Github 
- NexT
categories:
- 教程
---

## 什么是 Hexo？
Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

## 安装的前提
安装 Hexo 相当简单，然而在安装前，你必须检查电脑中是否已经安装了下列应用程序
>  Node
>  Git

## 快速安装

### 安装 Hexo

- 所以的必备程序都安装后，即可使用 ``npm`` 安装 ``Hexo``
``` bash
$ npm install -g hexo-cli
```

### 建站

- 安装 Hexo 完成后，请执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件
``` bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```

### 配置 Github

- 在Github上建立与用户名相对应的仓库，仓库名必须为【your_user_name.github.io】这种形式
- 创立仓库成功后打开``<folder>``文件夹，找到``_config.yml``
- 移动到文本的最下方，修改内容为(使用https时会要求输入Github的用户名和密码)
``` bash
deploy:
  type: git
  repo: <仓库的https或ssh地址>
  branch: master
```
- 执行命令保存配置
``` bash
$ npm install hexo-deployer-git --save
```
- 执行配置命令
``` bash
$ hexo deploy
```

- 然后在浏览器中输入 http://your_user_name.github.io/ 就行了

## 部署的步骤

- 清除缓存
``` bash
$ hexo clean
```

- 生成静态网页
``` bash
$ hexo generate
```

- 部署
``` bash
$ hexo deploy
```

## 常用的命令

- 新建文章：您可以在命令中指定文章的布局（layout），默认为 ``post``，可以通过修改 ``_config.yml`` 中的 ``default_layout`` 参数来指定默认布局。
``` bash
$ hexo new [layout] <title>
```

- 新建页面：新建一个新的比如分类，关于等的页面
``` bash
$ hexo new page "pageName"
```

- 生成静态页面至public目录
``` bash
$ hexo generate
```

- 开启本地服务器
``` bash
$ hexo server
```

- 部署到GitHub
``` bash
$ hexo deploy
```

- 帮助
``` bash
$ hexo help
```

## 其他说明

> * 具体的配置查看 [NexT](http://theme-next.iissnan.com/getting-started.html) 和 [Hexo](https://hexo.io/zh-cn/docs/index.html)
> * 部分内容参考这篇文章 [HEXO搭建个人博客](http://baixin.io/2015/08/HEXO%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2/)
> * 最开始使用 ``SourceTree`` 上传文件，但都出现Page build failure错误，后来使用 ``HEXO搭建个人博客`` 的方法成功创建博客
> * 本文只记录了 ``Mac`` 下布置博客的步骤，其他平台未验证