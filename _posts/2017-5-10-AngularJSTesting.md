---
layout: contents
title: AngularJs 4.0 学习 --- 测试
categories: [Web]
tages: [AngularJs, web, tech]
---
# AngularJs 4.0 学习 --- 依赖注入测试

> 本文翻译自[官方文档](https://angular.io/docs/ts/latest/guide/testing.html)

本篇文档提供了测试Angular应用的提示和技巧。 尽管包括一些基本的测试概念和技巧，但是重点还是在如何使用Angular进行测试。

## Angular 测试的介绍

本篇文档通过教导通过写测试来确认应用的行为正确性，按照以下步骤来测试：

1. 注意避免改变造成的code break (regressions)
2. 无论在预期的用途还是非正常的情况，都要声明代码做了什么
3. 在设计和实现的时候揭露错误。测试时是代码的救星。 当一部分应用很难测试的时候，根本原因往往是因为
设计的问题，应该尽早去改正以减少修改代价。

### 测试工具

有很多，例如： Jasmine, Angular testing utilities, Karma, Protractor等等，这里不再详细介绍这些工具

### 启动项目

收先是初始化一个项目，可以参考之前的文档。 本篇文档是基于之前文档中的项目例子。

### 独立单元测试和 Angular testing utilites

独立的单元测试不需要Angular和其他的注入值的依赖，独立完成对类的实例的测试。 测试者通过`new`来创建一个类的实例作为测试实例。
提供给测试的构造函数参数是必要的，然后尝试测试实例的API

*独立单元测试一般用于管道和服务*

也可以用独立测试来测试组件。但是独立单元测试不能够检测到组件是如何在Angular中相互影响的，也就是说，不能够测试覆盖到
组件对模板以及其他组件的影响。

这种情况需要使用Angular testing utilites. 它包含在`TestBed`类中并有一些从`@angular/core/testing`的帮助方法。
Angular testing utilites是本篇文档的主要主要关注点，在介绍如何写第一个组件测试的时候会介绍。

不过，写测试之前要首先设立测试环境以及了解一些基本测试技巧。

## 第一个Karma 测试

首先，安装Karma : `npm install karma --save-dev`

--TODO: