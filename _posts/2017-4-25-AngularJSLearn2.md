---
layout: contents
title: AngularJs 4.0 学习 (二)
categories: [Web]
tages: [AngularJs, web, tech]
---
# AngularJS 学习（二）数据显示
> 本文翻译自[angularJS 官方文档](https://angular.io/docs/ts/latest/guide/displaying-data.html)

你可以通过将显示数据绑定到HTML模板中Angular组件的属性中来显示数据。

## 主要内容：

* 使用插入法(interpolation)显示组件属性值
* 通过使用NgFor来显示一个数组属性
* 通过使用NgIf来根据条件显示数据

### 使用插入法(interpolation)显示组件属性值

显示组件属性值的最简单的方法就是使用插入法来绑定数据到属性名， 通过用两个大括号包括属性名来实现： `{{myHero}}`

以下是一个例子：

```javascript
/*app/app.component.ts*/
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

以上的例子显示了两个属性， title 和 myHero.
通过使用反引号 ` 来包括行字符串， 这样可以使得HTML更具有可读性。

AngularJS 自动将title 和 myHero的值从组件中拿出病插入到浏览器中。 当这些值改变时，Angular会 更新显示的值。

注意， 你并不需要使用 new 关键字来创建一个AppComponent的实例， Angular会帮你做这些。
在@Component装饰中的CSS选择器中制定了一个元素 <my-app>. 这个元素其实就是在index.html中控件的一个占位符。

```html
<body>
  <my-app>loading...</my-app>
</body>
```

当启动AppComponent类的时候(在main.ts中)，Angular 会在index.html中查找<my-app>元素， 然后生成一个AppComponent的实例并在<my-app>中加载

#### 使用模板文件

除了可以直接在template 属性中定义模板，还可以使用templateUrl来导入模板文件，模板文件可以是额外的html文件。 通过文件导入的方式可以使html与组件定义分离。

```javascript
/*app/app.component.ts*/
import { Component } from '@angular/core';

@Component({
  selector: 'my-app',
  templateUrl: './template/heroTemplate.html',
})
export class AppComponent  { 
  title = 'Tour of Heroes';
  hero = 'Windstorm';
}
```

```html
<!-- app/template/heroTemplate.html -->
<h1>{{title}}</h1>
<h2>My favorate hero is {{hero}}</h2>
```

效果和直接使用template 属性是一样的。

除了使用变量来指定属性之外，还可以使用构造函数：

```javascript
export class AppCtorComponent {
  title: string;
  hero: string;

  constructor() {
    this.title = 'Tour of Heroes';
    this.hero = 'Windstorm';
  }
}
```

### 通过使用NgFor来显示一个数组属性

如果需要显示多个属性值，可以使用数组来存储，然后在模板中使用ngFor来将数组中的值显示出来。 首先，我们添加一个数组作为属性：
```javascript
export class AppComponent {
  title = 'Tour of Heroes';
  heroes = ['Windstorm', 'Bombasto', 'Magneta', 'Tornado'];
  myHero = this.heroes[0];
}
```

然后，在模板中使用ngFor来循环输出数组中的值： 
```html
<h1>{{title}}</h1>
<h2>My favorate hero is {{myHero}}</h2>
<p>Heroes:</p>
<ul>
    <li *ngFor="let hero of heroes">
        {{hero}}
    </li>
</ul>
```

于是，就会有下面的结果：

![ngfor 显示数据]({site.asseturl}/AngularyjsLearn/using-ngfor.png)

`*ngFor` 在`<li>`中相当于一个重复器(repeater)的命令，它使得<li>元素以及它的子元素成为了一个重复模板. ngFor 可以迭代任意可迭代的对象(iterable object)
> `*ngFor` 的具体使用语法可以参考[AngularJS官方文档](https://angular.io/docs/ts/latest/guide/template-syntax.html#!#ngFor)

### 给数据创建一个类
尽管在小的demo 中，直接给属性赋值来作为数据是很方便的，但是这样并不是很规范的做法，我们需要把数据分离开来。
因此，我们需要创建一个类来作为数据的model:

```javascript
export class Hero {
  constructor(public id: number,  public name: string) { }
}
```
这里使用TypeScript中的构造函数声明方式：

```javascript
public id: number,
```

这段代码做以下事情：
* 声明了一个构造函数的参数和类型
* 声明了一个同名的 public 属性
* 当创建这个实例的时候用对应的参数初始化这个了这个属性

当导入 Hero 类后， AppComponent.heroes 应该要返回一个Hero对象组成的组，因此，修改component 中的数据：

```javascript
import { Hero } from './hero';  /* 注意，这里导入的时候，不能带.ts, 否则会编译失败*/

heroes = [
  new Hero(1, 'Windstorm'),
  new Hero(13, 'Bombasto'),
  new Hero(15, 'Magneta'),
  new Hero(20, 'Tornado')
];
myHero = this.heroes[0];
```

然后我们更新模板中的内容：

```html
<h1>{{title}}</h1>
  <h2>My favorite hero is: {{myHero.name}}</h2>
  <p>Heroes:</p>
  <ul>
    <li *ngFor="let hero of heroes">
      {{ hero.name }}
    </li>
  </ul>
```

### 使用NgIf 根据条件显示元素

Angular `ngIf` 指令 会根据条件的 *truthy/falsy* 来插入或删除元素
例如：

```html
<p *ngIf="heroes.length > 3">There are many heroes!</p>
```

注意，Angular 并没有隐藏和显示元素，这里做的操作是根据条件来 **添加** 或者 **删除** 元素. 这在一定程度上是提升性能的。

