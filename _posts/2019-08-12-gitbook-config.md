---
layout:     post
title:      GitBook本地搭建
subtitle:   GitBook的相关配置以及报错的解决方案
date:       2019-08-12
author:     Aro
header-img: img/post-bg-gitbook.jpg
catalog: true
tags:
    - GitBook
---

>GitBook 对我来说只是作写书，摘抄使用。在GitBook官网如果不去订阅的话，只能有一个public空间和一个private空间。对我而言，我会把本地写好的书build之后放到自己的服务器上供阅读。所以这篇文章只是关于gitbook的搭建以及控制台报错的解决。

## 安装与初始化

>我目前的版本为: CLI version: 2.3.2   GitBook version: 3.2.3

1. 首先是Node环境，这对前端开发来说无处不在，去官网下载一个安装即可;
2. 使用`Node.js`提供的包管理器`NPM`来安装`GitBook`脚手架工具`gitbook-cli`;

```
npm install gitbook-cli
```

3. 初始化GitBook;

```
gitbook init
```


## 配置

#### 目录的配置

>`SUMMARY.md` 文件中定义了书籍的目录信息，可在该文件中通过 Markdown 的 列表语法 来构建书籍章节的层级关系。

**截至目前我的Summary.md配置如下:**

```
# Summary

* [简介](README.md)

* [1. 概述](chapter1/README.md)
    * [1.1. 计算机网络在信息时代中的作用](chapter1/1/Effect.md)
    * [1.7. 计算机网络体系结构](chapter1/7/README.md)
        * [1.7.1. 计算机网络体系结构的形成](chapter1/7/计算机网络体系结构的形成.md)
        * [1.7.2. 协议与划分层次](chapter1/7/协议与划分层次.md)
        * [1.7.3. 具有五层协议的体系结构](chapter1/7/具有五层协议的体系结构.md)
        * [1.7.4. 实体、协议、服务和服务访问点](chapter1/7/实体、协议、服务和服务访问点.md)
        * [1.7.5. TCP IP的体系结构](chapter1/7/TCP IP的体系结构.md)
```


#### book.json配置

刚初始化的时候是没有book.json，搞得我第一次一脸懵逼，还以为哪里配置错了。然后索性就自己创建了一个json，大致就是下面那样，粘贴过去就能用，记得把`title`、`author`、`description`的参数改一下。插件都配置好了，对我来说够用。配置好后在根目录下运行`gitbook install`，否则下面启动服务会出错。

```
{
    "title": "计算机网络 摘要",
    "author": "Aro",
    "language": "zh-hans",
    "gitbook": "3.2.3",
    "root": "/",
    "description": "计算机网络（第7版） 谢希仁 编著",
    "structure": {
        "readme": "README.md"
    },
    "plugins": [
        "-sharing",
        "search",
        "github",
        "search-pro",
        "back-to-top-button",
        "custom-favicon"
    ],
    "pluginsConfig": {
        "github": {
            "url": "https://github.com/AroZhou"
        },
        "favicon": "assets/images/favicon.ico"
    }
}
```

至此我的gitbook目录结构如下图:

![gitbook目录](/img/20190812gitbook.png)


## 启动服务

通过 `gitbook serve` 启动一个本地服务，以预览书籍效果。默认开启一个4000的端口，在地址栏输入`http://localhost:4000/`便可以访问。

## 部署

通过 `gitbook build` 命令可以将内容 （Markdown）编译成网页（HTML），生成的网页将存放在目录下的 `_book` 文件夹中。
GitBook 将根据 `SUMMARY.md` 中的内容来决定被编译的内容，并将它们链接起来。


## 关于错误的解决

以上的流程走通后其实已经可以预览到书了，但是打开控制台后会发现滑动页面的时候会报错`theme.js:4 Uncaught TypeError: Cannot read property 'split' of undefined`

**下面是具体解决步骤:**

1. 进入 GitBook 默认主题所在的文件夹 

`.gitbook`文件如果默认安装的话，我的Mac上是在`‎⁨Macintosh HD⁩ ▸ ⁨用户⁩ ▸ ⁨aro⁩`，我的Windows上是在`C:\Users\Aro`，这个文件是被系统隐藏了，Mac下快捷键`command+shift+.`，Windows下显示隐藏的文件如下图

![20190812gitbook02](/img/20190812gitbook02.jpg)

`.gitbook` -> `versions` -> `3.2.3` -> `node_modules` -> `gitbook-plugin-theme-default` -> `src` -> `js` -> `theme` -> `navigation.js` 找到`getChapterHash`函数 

```js
function getChapterHash($chapter) {
    var $link = $chapter.children('a'),
        hash = $link.attr('href').split('#')[1];

    if (hash) hash = '#'+hash;
    return (!!hash)? hash : '';
}
```

2. 将该函数修改为下面的形式:

```js
function getChapterHash($chapter) {
    var $link = $chapter.children('a'),
        hash,
        href,
        parts;

    if ($link.length) {
        href = $link.attr('href')
        if (href) {
            parts = href.split('#');
            if (parts.length>1) {
                hash = parts[1];
            }
        }
    }

    if (hash) hash = '#'+hash;
    return (!!hash)? hash : '';
}
```

3. 回到 `gitbook-plugin-theme-default` 文件夹，运行 `npm install` 重新编译文件。

4. 至此问题解决