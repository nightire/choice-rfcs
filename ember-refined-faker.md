# Ember Refined Faker Addon

- 日期：2016年12月12日
- 作者：余凡 <albert@cform.io>

## 概述

[ember-refined-faker](https://github.com/very-geek/ember-refined-faker) 是一个 Ember Addon，封装了 [faker.js](https://github.com/Marak/faker.js)，并提供可在模版上直接使用的 htmlbars helper，其目的是为 UI 开发提供便利的随机伪数据生成功能。

## 背景

目前我们在 ember-cform-ui 等 addons 中使用了 faker.js 生成伪数据，然而只在 route/controller 层面使用 JavaScript APIs 来做，能用但是繁琐。

我们需要的是在开发 UI 的过程中对于伪数据的依赖可以更加细致和直接了当一些，这就意味着我们需要在 template 层面上有直接可以调用的 component(s) / helper(s) 可用。

Ember 社区已经存在针对 faker.js 封装的 addon [ember-faker](github.com/johnotander/ember-faker)，但是它只能做到我们现在已经做到的事情，因此我将重新封装一个新的 addon 并按照我们的实际需求提供 template helpers。

## 细节

### 代理一切方法的 helper

faker.js 的 API 规律性很强，在顶级命名空间 `faker` 之下用了十四个二级命名空间来分类，接着每一级之下有若干个方法来生成不同类型的随机伪数据。这些方法大多都是不带参数的，因此这样一个命名空间即可适应大多数应用场景：

```htmlbars
{{fake "namespace.method"}}
```

比如说随机生成一个国家的名字：

```htmlbars
{{fake "address.country"}}
```

代理调用：

```javascript
function fake([signature]) {
  const [namespace, method] = signature.split('.')
  return faker[namespace][method]()
}
```

有个别方法是需要参数支持的，比如：

```htmlbars
{{fake "commerce.price" 0 100 2 "$"}}
```

代理调用改为：

```javascript
function fake([signature, ...args]) {
  const [namespace, method] = signature.split('.')
  return faker[namespace][method].apply(null, args)
}
```

> 如果参数要求为对象，可以用内置的 `{{hash k=v}}` 来替代
>
> 如果参数要求为数组，可以考虑直接集成 [ember-composable-helpers](https://github.com/DockYard/ember-composable-helpers)，或者为避变依赖单独创建一个 `{{array}}` helper（但容易和 ember-composable-helpers 冲突，是否改名叫 `arr` 或 `list` ？或者叫 `fake-array`？@darkbaby123

另外，`faker.image` 默认使用 http://lorempixel.com/ 这个图片服务，根据实际观察该服务在中国境内访问非常慢。可以考虑在 **`/config/environment.js` 里增加一个配置选项**，通过重写 [`faker.image.imageUrl`](https://github.com/Marak/faker.js/blob/master/lib/image.js#L38) 方法替换成别的图片服务。

### 支持复合表达式的 helper

尽管 faker.js 拥有[非常多的 API 方法](http://marak.github.io/faker.js/)，不过它也有一个非常实用的方法 [`faker.fake`](http://marak.github.io/faker.js/faker.html#-static-fake__anchor) 可以封装所有不需要入参的方法，通过字符串内插的方式进行组合调用：

```javascript
faker.fake('Hi, I\'m {{name.lastName}}, {{name.firstName}}.')
// "Hi, I'm John Doe."
```

这种方式很灵活，适合一些需要特定语句但允许随机替换其中指定部分的场合。为了适应这种需求，ember-refined-faker 同样允许这样来使用 helper：

```htmlbars
{{fake "Hi, I'm [name.lastName], [name.firstName]." parse=true}}
```

使用 `[]` 来替代 `{{}}` 以规避 htmlbars 语法冲突，并附加 `parse=true` 告知 helper 代理不同的方法调用，其具体实现变为：

```javascript
function fake([signature, ...args], {parse = false}) {
  if (parse) {
    return faker.fake(signature.replace(/\[/g, '{{').replace(/\]/g, '}}'))
  } else {
    const [namespace, method] = signature.split('.')
    return faker[namespace][method].apply(null, args)
  }
}
```

### 多语言支持

faker.js 内置了多语言支持，所以在 UI 开发阶段可以不需要依赖 I18n 服务（不过内置的中文支持比较弱罢了），默认情况下还是使用 `en_US`，不过允许在 `/config/environment.js` 里使用配置选项来更改默认语言。同时也允许在调用 helper 的时候动态更改语言，如：

```htmlbars
{{fake "address.country" locale="zh_CN"}}
```

方法实现可进一步改进为：

```javascript
// reset to default locale
faker.locale = config.faker.defaultLocale

// tracking locale temporary change
let localeHasBeenChanged = false;

function changeLocale(locale) {
  if (locale && locale !== config.faker.defaultLocale) {
    faker.locale = locale
    localeHasBeenChanged = true
  } else if (localeHasBeenChanged) {
    faker.locale = config.faker.defaultLocale
    localeHasBeenChanged = false
  }
}

function fake([signature, ...args], {parse = false, locale}) {
  changeLocale(locale)
  
  if (parse) {
    return faker.fake(signature.replace(/\[/g, '{{').replace(/\]/g, '}}'))
  } else {
    const [namespace, method] = signature.split('.')
    return faker[namespace][method].apply(null, args)
  }
}
```

> [这个页面](https://cdn.rawgit.com/Marak/faker.js/master/examples/browser/index.html)可以用来查看不同语言在 faker.js 中的表现情况

## 其他

### 环境

默认情况下，ember-refined-faker 应该只允许工作在 `development` 环境下。不过当作开源组件来考虑的话，说不定有在 `production` 环境下使用的需求，因此需要通过在 `/config/environment.js` 提供配置选项来支持。