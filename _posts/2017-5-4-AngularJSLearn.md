---
layout: contents
title: AngularJs 4.0 学习 (六) --- 数据展示
categories: [Web]
tages: [AngularJs, web, tech]
---
# AngularJS 学习（五）---- 数据展示

> 通过属性绑定帮助应用在UI上展示数据。本文翻译自[angularJS 官方文档](https://angular.io/docs/ts/latest/guide/displaying-data.html)

你可以通过将HTML模板中的控件绑定到Angular组件来显示数据。

在这片文档中，我们会创建一个hero list的控件，在列表中显示英雄的名称并且根据不同情况显示不同的消息。

大概的UI如图所示：

![](https://angular.io/resources/images/devguide/displaying-data/final.png)

## 使用嵌入绑定(interpolation)显示组件属性值
通过嵌入绑定(interpolation)属性的名称是显示组件属性最简单的一种方式。 把属性名放在模板中，用双花括号包括，就完成了嵌入式绑定：
`{{myHero}}`。

首先，参考设置教程，创建一个app.component.ts:

```javascript
import { Component } from '@angular/core';
@Component({
  selector: 'my-app',
  template: `
    <h1>{{title}}</h1>
    <h2>My favorite hero is: {{myHero}}</h2>
    `
})
export class AppComponent {
  title = 'Tour of Heroes';
  myHero = 'Windstorm';
}
```

这里，我们添加了两个属性： `title` 和 `myHero`。里面的template 使用花括号嵌入来展示了这两个属性值：

```javascript
template: `
  <h1>{{title}}</h1>
  <h2>My favorite hero is: {{myHero}}</h2>
  `
```

> 注意，这个模板使用了ES2015 中的反引号(\`\`)。 它允许你组合换行的字符串，这使得HTML更有可读性。

Angular 会走动将 title 和 myHero 属性的值从组建中取出并插入到浏览器中，当属性值发生变化时，Angular会更新它们。

> 更精确地说，重新显示会在异步加载事件完成后发生，例如按键，计时器完成，或者HTTP请求的回应 等事件。

注意，你并不需要使用 `new`关键字来创建一个 `AppComponent` 的实例， Angular已经为你创建好了。如何做到的呢？

`@Component`指令中的CSS选择器 `selector` 定义了一个元素叫做`<my-app>`。这个元素是在 index.html中的body里面的一个占位符：

```html
<body>
  <my-app>loading...</my-app>
</body>
```

当启动AppComponent类的时候， Angular会找到它，创建一个 AppComponent的实例并将其插入到该标签中。

现在启动项目， 应该如下所示：

![](https://angular.io/resources/images/devguide/displaying-data/title-and-hero.png)

## 行内模板还是模板文件？

我们可以在两个地方存储组件的模板，一种是在template属性中定义，另一种是在单独的HTML文件中定义，然后通过组件元数据`@Component`
的`templateUrl`属性来把它引用过来。

这两种选择根据情况，或者公司或个人的喜好而定。我们的例子中采用的是行内模板，因为这个demo比较小并且很简单。 数据绑定对于两种绑定效果是相同的。

## 使用构造函数还是在变量中初始化？

除了上面的例子使用的是变量来初始化组件的属性值，我们还可以用构造函数来初始化它们。

```javascript
export class AppCtorComponent {
  title: string;
  myHero: string;

  constructor() {
    this.title = 'Tour of Heroes';
    this.myHero = 'Windstorm';
  }
}
```

这样应用就更加简洁明了。

## 通过使用 *ngFor来显示数组属性

要想显示一列英雄，我们首先要添加一个英雄名称的数组：

```javascript
export class AppComponent {
  title = 'Tour of Heroes';
  heroes = ['Windstorm', 'Bombasto', 'Magneta', 'Tornado'];
  myHero = this.heroes[0];
}
```

然后使用 Angular 的 `ngFor`指令在模板中显示：

```javascript
  template: `
    <h1>{{title}}</h1>
    <h2>My favorite hero is: {{myHero}}</h2>
    <p>Heroes:</p>
    <ul>
      <li *ngFor="let hero of heroes">
        {{ hero }}
      </li>
    </ul>
  `
```

其他内容，在教程中可以看到。  


