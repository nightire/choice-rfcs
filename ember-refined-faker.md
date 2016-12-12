- 日期：2016年12月12日
- 作者：余凡 <albert@cform.io>

## 概述

[ember-refined-faker](https://github.com/very-geek/ember-refined-faker) 是一个 Ember Addon，封装了 [faker.js](https://github.com/Marak/faker.js)，并提供可在模版上直接使用的 htmlbars helper，其目的是为 UI 开发提供便利的随机伪数据生成功能。

## 背景

目前我们在 ember-cform-ui 等 addons 中使用了 faker.js 生成伪数据，然而只在 route/controller 层面使用 JavaScript APIs 来做，能用但是繁琐。

我们需要的是在开发 UI 的过程中对于伪数据的依赖可以更加细致和直接了当一些，这就意味着我们需要在 template 层面上有直接可以调用的 component(s) / helper(s) 可用。

Ember 社区已经存在针对 faker.js 封装的 addon [ember-faker](github.com/johnotander/ember-faker)，但是它只能做到我们现在已经做到的事情，因此我将重新封装一个新的 addon 并按照我们的实际需求提供 template helpers。

## 细节

## 问题