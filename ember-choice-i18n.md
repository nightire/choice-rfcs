# Ember Choice I18n

- 日期 2016年12月28日
- 作者：余凡 albert@cform.io

## 概述

这篇文档将配合 [I18n Service API RFC](https://github.com/choice-form/rfcs/blob/i18n-service-api/i18n-service-api.md) 来描述客户端如何使用 I18n Service，在本文写作的最初阶段 Client 将特指 Ember 应用程序，等到我们满足了内部的需求之后才会考虑扩展到 Ember 以外的技术栈。

## 背景

I18n Service API 在设计之初就考虑了同客户端之间的紧密结合，这也是整个 I18n 服务区别于既有第三方服务的最大不同之处。目前存在的 I18n 方案都没能满足“最后一公里”的需求，也就是在前端 UI 开发的早期阶段无法简单方便的创建 I18n 的词条。

## 细节

### 代理 `{{t "namespace.key"}}` 到 I18n Service API

第一个问题是解决 ember-intl 的 t helper 处理 promise as key 的问题。ember intl 的处理过程如下：

1. `FormatMessageHelper`（`t` 的 factory）的 `compute` [方法](https://github.com/jasonmit/ember-intl/blob/v2.16.1/packages/ember-intl/addon/helpers/-format-base.js#L33)调用 `this.getValue` [方法](https://github.com/jasonmit/ember-intl/blob/v2.16.1/packages/ember-intl/addon/helpers/format-message.js#L13)
2. 接着调用 `intlService.findTranslationByKey` 方法并[代理](https://github.com/jasonmit/ember-intl/blob/v2.16.1/packages/ember-intl/addon/services/intl.js#L193) `defaultAdapter.findTranslationByKey` [方法](https://github.com/jasonmit/ember-intl/blob/v2.16.1/packages/ember-intl/addon/adapters/default.js#L48)
3. 在此阶段用自定义的 `intlService` 代理给自定义的 `adapter` 从而通过改写 `findTranslationByKey` 来发起向 I18n Service API 的请求——此时返回的不再是本地查找的 translation，而是一个 promise
4. 因此，1. 里的 `this.getValue` 方法的返回值就不能继续向下处理了，因为 ember-intl 默认不处理返回值为 promise 的情况，最终将会在 [intl-messageformat -> core.js#L24](https://github.com/yahoo/intl-messageformat/blob/v1.3.0/src/core.js#L24) 抛出异常

最好的处理策略应是重写 `FormatMessageHelper`，用自定义的 `compute` 方法来处理 `this.getValue` 返回的 promise，大致的思路可以参见 [issue #414](https://github.com/jasonmit/ember-intl/issues/414#issuecomment-270565447)

>💡 **Promise 最终走向异常的过程**
>
>当返回 promise，`compute` 方法会一路走到 [L53](https://github.com/jasonmit/ember-intl/blob/v2.16.1/packages/ember-intl/addon/helpers/-format-base.js#L53) 调用 `FormatMessage.format` [方法](https://github.com/jasonmit/ember-intl/blob/v2.16.1/packages/ember-intl/addon/formatters/format-message.js#L21)，之后便代理给 [intl-format-cache](https://github.com/yahoo/intl-format-cache) 了