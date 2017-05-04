---
layout: contents
title: AngularJs 4.0 学习 (五) --- 根模块(The Root Module)
categories: [Web]
tages: [AngularJs, web, tech]
---
# AngularJS 学习（五）---- 根模块

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

## *declarations* 数组

通过declarations 数组来告诉Angular 哪些组件是属于AppModule的，将它们放入 declarations数组中。 当你创建了更多的组件，把他们加入到 `declarations`中去。

你必须在`NgModule`类中声明所有的组件。如果你使用了一个没有声明的组件，你就会在浏览器的console终端里看到一个清楚的错误信息。

之后你会学到创建另外两种类，指令和管道， 你也必须添加到 `declarations`数组中

> 能够声明的类型：只有 组件，指令 和管道命令属于 `declarations`数组。不要把其他类型的类放到里面， 不能是`NgModule`类，服务类，以及模型类。

## *bootstrap* 数组

通过启动根模块 `AppModule`来启动应用。 其他方面， bootstapping 程序创建了列在 `bootstrap`数组中的组件并将每一个都差督导浏览器的DOM中去。

每一个启动的组件都是它的组件树的根组件。插入一个启动组件通常会出发级联创建组件。直到该组件树被填满。

尽管你可以在主页面放多个组件树，但是一般情况下不推荐。 大多数应用只有一个组件树，只启动一个根组件。

你可以叫这个根组件任意名称。但是通常大多数开发者称之为 `AppComponent`。 它将我们带入到bootstraping进程中.

## *main.ts*中的启动

有很多种启动应用的方式，关键在于你想如何编译应用并启动它。

一开始，你会通过 及时编译器 (Just-in-Time   JIT  Compile) 来动态编译应用并在浏览器中加载。稍后你会学到其他的选项。

一个推荐装载启动 JIT-compiled 浏览器应用的地方是一个单独的放在src文件夹下的文件： `src/main.ts`。

```javascript
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';
import { AppModule }              from './app/app.module';

platformBrowserDynamic().bootstrapModule(AppModule);
```

这段代码创建了一个浏览器平台的动态实时编译器并启动了 `AppModule`。

启动程序这只了执行环境，将 `AppComponent`从 模块的bootstrap 数组中取出，创建了该组件的一个实例并将其插入到组件的selector所代表的标签中。

在大多数文档和例子中，`AppComponent` 的选择器 是 `my-app` , 因此 Angular在`index.html`中找到`<my-app>`标签并将`AppComnponent`展现到这里

```html
<my-app><!-- content managed by Angular --></my-app>
```

这个文件十分稳定，一旦你设置完毕，你永远不会再改动它。

## 关于Angular Modules 的一些扩展

现在你的新应用中只包含一个跟模块。 当你的app越来越大，你会考虑将模块分割成不同的功能模块。其中有一些你可以延迟加载（当用户需要进入这些功能的时候再加载）
