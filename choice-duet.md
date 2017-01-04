# Choice Duet (Ember Engine)

- 日期：2017年01月04日
- 作者：余凡 albert@cform.io

## 概述

> 💁 **duet** *|d(y)o͞oˈet|* 由两人组成的交互行为，特别是对话、对唱、二重奏、双人舞等表现形式
>
> ❓ Ember Engine 是一种增强型的特化 Add-on，它可以实现路由挂载／惰性加载等高级特性
>
> ❗️ 这份 RFC 最终会成为 choice-duet 的 *README*（其中的一部分）

[Choice Duet](https://github.com/choice-form/choice-duet) 是一个 [Ember Engine](http://ember-engines.com/)（引擎），它专门用于将传统的答题式表单转变成对话式的问答交互形式，并可以直接挂载于现在的 Q 端系统（使用独立的路由）。

## 背景

最初的灵感来源自一个很有意思的项目：[Conversational Form](https://space10-community.github.io/conversational-form)，但对于巧思问卷来说它还是过于基础和简单。Choice Duet 的使命就是探索这种交互形式应用于调研表单的无限可能性。

在这份 RFC 诞生的时候，Ember Engine 已经得到了 Ember 社区的广泛关注，开发进度和状况反馈也非常活跃。相比于普通的 Add-on，Ember Engine 的主要特点是：

1. 引擎可以开发独立的路由并挂载于宿主应用（Host Ember Application）之上。这使得可路由引擎能够独立管理属于自己的静态资源并随着路由激活而惰性加载这些静态资源
2. 无路由引擎可以使用 `{{mount}}` 渲染 UI 组件至宿主应用的任何地方（可路由引擎也是一样）
3. 引擎可以共享宿主应用的依赖（主要是 services 和其他路由）

这些特性非常适合 Choice Duet 的需求，这也是我们选择 Ember Engine 来开发 Choice Duet 的原因。

## 细节



## 问题



## 其他

