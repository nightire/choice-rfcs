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

### I18n Editor

Addon 提供嵌入式的对话窗口用于对词条进行编辑，此功能目前已经实现了最基本的形态（[private link](https://gitlab.cform.io/dev/ember-choice-i18n/commit/1d0acffae5fd7919e23d9a346210581293ed945d)）。

目前存在的问题是：

- [ ] 进入对话窗口获取的 `value` 是转换后的，对于变量内插等原始语法没有还原。解决办法有而且也不难（比如把 `rawValue` 也附着在 `<span></span>` 标签上，但是考虑到会出现比较大的 `value` 或者格式比较复杂这个方案未必漂亮。另外一个思路是从 `Translation` model 着手，看看它的实现有没有办法程序化的获取 `rawValue`

- [ ] 做这个功能第一反应是用组件来实现，但遗憾的是目前似乎没有方法在 runtime 通过 javascript 代码来实例化一个组件；我们需要这个能力，因为第一、我们不想让模板行为在 dev/prod 之下有区别，第二、t 是一个 helper，它自身没有模板，因此需要在 javascript 中实例化+渲染

      **更新：**貌似 [ember-dialog](http://wheely.github.io/ember-dialog/) 里面实现了这个需求，但是操蛋的文档是用俄文写的！慢慢爬源码吧…开源万岁！

### Local Cached Translations

这是一个 ember-intl 内建的机制，对接完成之后代理 I18n Service API 的大体流程将变更为：

- **Development:**
  1. `ApplicationRoute.model` 发两个请求，请求当前项目所有的 locales 和 entries
  2. locales 用于构建语言选择菜单——这个逻辑在 production 下等价
  3. entries 借由 ember-intl service 推入到本地缓存的集合
  4. 之后所有的 t helper 现检查本地缓存，找不到再代理 I18n Service API 请求
  5. entry 创建成功后推入本地缓存集合，捕获 *Missing Translations* 错误，显示 code 在模板上
  6. entry 更新成功后将 value 存入本地缓存集合（not sure if intl service has updateTranslation method）
- **Production**
  1. 除了 locales 的请求，其余的步骤都将跳过，除非——
  2. 用户选择异步加载 entries，那么：
     1. 用户需要先获取 current locale（这由 consuming app 决定如何获取）
     2. 发起请求获取 entries?locale_id
     3. 借由 ember-intl service
  3. 此后所有的动作走 ember-intl 的默认逻辑，ember-choice-i18n 将不再工作

上述过程即使通过配置也无法做到完全透明化，我们尽可能将主要逻辑封装成单一方法提供出来，用户可以通过依赖注入自行决定何时调用与处理。在本产品公开／商用化之时，会提供这方面的 tutorials 以供参考。