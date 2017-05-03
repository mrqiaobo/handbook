---
layout: contents
title: AngularJs 4.0 学习 (五) --- 根模块(The Root Module)
categories: [Web]
tages: [AngularJs, web, tech]
---
# AngularJS 学习（五）---- 跟模块

## 通过根模块 "AppModule" 告诉Angular 如何构建病启动应用。
> 本文翻译自[angularJS 官方文档](https://angular.io/docs/ts/latest/guide/appmodule.html)

一个Angular 模块描述了如何将应用的各个部分组合在一起。每个Angular应用至少有一个模块，你启动应用所需要的跟模块。你可以任意给它起名，但一般来说它叫做`AppModule`。

在之前的 setup 介绍中通过一个小的 `AppModule` 创建了一个新的项目，你可以在这个module的基础上添加你自己的东西：

```javascript
import { NgModule }      from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent }  from './app.component';

@NgModule({
  imports:      [ BrowserModule ],
  declarations: [ AppComponent ],
  bootstrap:    [ AppComponent ]
})
export class AppModule { }
```

在 import 语句后，你可以看见有一个被 `@NgModule` 指令修饰的类

`@NgModule`指令声明 `AppModule` 作为一个Angular模块类（也称为 `NgModule` 类）， `@NgModule`
带有一个元数据对象，告诉Angular 如何来编译和运行应用。

* *imports* ——传入 `BrowserModule`，所有需要在浏览器上运行的应用都需要这个模块
* *declarations* ——声明应用中唯一的一个组件
* *bootstrap* ——根组件， Angular创建并且将其插入到 `index.html`主页面中。

这个Angular模块很有示范作用。所有你需要了解的就是这三个属性。

## *imports* 数组

Angular模块是组合分散功能到一起的一个很好的途径。 许多Angular的功能都用Module来组织： HTTP服务在`HttpModule`, 路由在`RouterModule`, 你也可以自己创建功能模块。

当应用需要这些功能时，把模块放到 `imports`数组中。

像许多应用一样，该应用在浏览器中运行。 所有运行在浏览器中的应用都需要 `@angular/platform-browser`中的`BrowserModule`模块。所以，所有这些应用都需要将`BrowserModule`放到
它的根模块 `AppModule` 的 `imports` 数组中。

> 注意，只有 `NgModule` 类可以放在imports 中，其他的类不能放在imports中。
在文件顶部的 import语句和 Angular模块中的 imports数组没有任何联系，并且功能完全不同。
JavaScript import 语句使你能够访问和引用其他文件 export的部分，你可以在任何文件中添加 import 语句，你不需要Angular做任何事情， Angular对其也一无所知。
模块的 imports 数组仅仅在 `@NgModule` 的元数据对象中出现，它告诉Angular 其他模块的细节，所有这些模块都是`@NgModule`修饰的类，都被应用所需要。

