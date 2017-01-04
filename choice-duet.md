# Choice Duet (Ember Engine)

- 日期：2017年01月04日
- 作者：余凡 albert@cform.io

## 概述

> 💁 **duet** *|d(y)o͞oˈet|* 由两人组成的交互行为，特别是对话、对唱、二重奏、双人舞等表现形式
>
> ❓ Ember Engine 是一种增强型的特化 Add-on，它可以实现路由挂载／惰性加载等高级特性
>
> ❗️ 这份 RFC 最终会成为 choice-duet 的 *README*（其中的一部分）

[Choice Duet](https://github.com/choice-form/choice-duet) 是一个 [Ember Engine](http://ember-engines.com/)（引擎），它专门用于将传统的答题式表单转变成更友好的对话式表单形式，并可以直接挂载于现在的 Q 端系统（使用独立的路由）。

## 背景

### 关于对话式表单

最初的灵感来源自一个很有意思的项目：[Conversational Form](https://space10-community.github.io/conversational-form)，但对于巧思问卷来说它还是过于基础和简单。Choice Duet 的使命就是探索这种交互形式应用于调研表单的无限可能性。

### 关于 Ember Engine

在这份 RFC 诞生的时候，Ember Engine 已经得到了 Ember 社区的广泛关注，开发进度和状况反馈也非常活跃。相比于普通的 Add-on，Ember Engine 的主要特点是：

1. 引擎可以开发独立的路由并挂载于宿主应用（Host Ember Application）之上。这使得可路由引擎能够独立管理属于自己的静态资源并随着路由激活而惰性加载这些静态资源
   1. 无路由引擎可以使用 `{{mount}}` 渲染 UI 组件至宿主应用的任何地方（可路由引擎也是一样）
2. 引擎可以共享宿主应用的依赖（主要是 services 和其他路由）

这些特性非常适合 Choice Duet 的需求，这也是我们选择 Ember Engine 来开发 Choice Duet 的原因。

### 关于 Promises Reducer

[@darkbaby123](https://github.com/darkbaby123) 跟我说起 promise 聚合在 Ember 框架（特别是 Computed Property）内的支持情况，在他的提示下我先做了一些[尝试](https://ember-twiddle.com/4ac626f60bbab5622f54dfa0350ad9dc)，并且很快意识到这些技巧对于 Choice Duet 的实现会非常有用，具体细节将在后文详细描述。这方面特别是 [@fenyiwudian](https://github.com/fenyiwudian) 需要好好想想。

## 细节



## 问题



## 其他

