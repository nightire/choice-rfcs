# Ember Baidu Map Addon

- 日期：2017年01月03日
- 作者：余凡 albert@cform.io

## 概述

[ember-baidu-map](https://github.com/very-geek/ember-baidu-map) 是一个 Ember Addon，封装了[百度地图 JavaScript API](http://lbsyun.baidu.com/index.php?title=jspopular)。

## 背景

做这个 addon 的初衷很简单，巧思的产品有几处都在使用百度地图，它们普遍存在一些问题诸如：

1. 不同时期的项目引用的百度地图 SDK 的版本各不相同，乱
2. 引入的方式不同，SDK 自身写入脚本的方式也不同，造成一些 deprecations，乱
3. 不同开发者使用百度地图 SDK API 时的风格各不相同，相同的功能会出现不同的写法，乱
4. ……

因此在开始阶段这个 addon 的主要目标就是在 Ember 框架内把这些琐碎的，容易造成不一致的隐患消除，提供最简单最直观的配置／接口给业务开发者。更长期的目标则是不断扩充 UI Components，封装来自百度地图 SDK 的不同类，为业务开发提供更友好的集成方式，提高业务开发效率。

## 细节

### 高可靠性、一致性并且允许灵活配置的 SDK 加载机制

- [ ] 百度地图 JavaScript API 已经全部支持 *https*，因此加载将使用 *https* 并且不提供回退至 *http* 的配置选项（除非遇到确实需要的场景）
- [ ] 百度地图 JavaScript API 自 *v1.5* 起加载需要提供 **secret key**，因此提供配置选项填写 **secret key**，如果没有则加载 *v1.4* 版本，反之则加载最新版本（当前是 *v2.0*）；另外在 development 环境下通过控制台提醒开发者加载的具体版本
- [ ] 百度地图 JavaScript API 应允许使用 *async* 模式加载（直接写入 `<head></head>` 里），否则应尝试使用 *instance initializer* 来加载（监听 `DOMContentLoaded` 事件）
- [ ] 根据[文档描述](http://lbsyun.baidu.com/index.php?title=jspopular/guide/introduction#.E5.BC.82.E6.AD.A5.E5.8A.A0.E8.BD.BD)，百度地图 JavaScript API 支持加载时的异步回调，问题是范例代码使用的是全局函数，不过这个应该不难变成模块形式让开发者来定义回调函数

### 将百度地图 JavaScript API 封装为 ES2015 Modules 格式

***TODO***

## 问题

- 尝试解决一下关于 `document.write()` 引起的 deprecations，这可能是百度地图 SDK 自身引起的

## 其他

暂无。