# Ember Choice I18n

- 日期 2016年12月28日
- 作者：余凡 albert@cform.io

## 概述

这篇文档将配合 [I18n Service API RFC](https://github.com/choice-form/rfcs/blob/i18n-service-api/i18n-service-api.md) 来描述客户端如何使用 I18n Service，在本文写作的最初阶段 Client 将特指 Ember 应用程序，等到我们满足了内部的需求之后才会考虑扩展到 Ember 以外的技术栈。

## 背景

I18n Service API 在设计之初就考虑了同客户端之间的紧密结合，这也是整个 I18n 服务区别于既有第三方服务的最大不同之处。目前存在的 I18n 方案都没能满足“最后一公里”的需求，也就是在前端 UI 开发的早期阶段无法简单方便的创建 I18n 的词条。