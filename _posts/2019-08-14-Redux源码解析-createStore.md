---
layout:     post
title:      Redux 源码解析(二)  compose.js
subtitle:   逐行解析Redux的API
date:       2019-08-14
author:     Aro
header-img: img/post-bg-redux.png
catalog: true
tags:
    - Redux
    - 前端
---

>目前所分析的Redux版本为4.0.4。`compose`这个模块的代码十分简练，但是实现的作用却是十分强大。`reduce`正是这个模块的核心。





