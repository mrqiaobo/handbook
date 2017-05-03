---
layout: contents
title: AngularJs 4.0 学习 (四) --- 结构概览
categories: [Web]
tages: [AngularJs, web, tech]
---
# AngularJS 学习（四）---- 结构概览
## Angular 应用的基本构成
> 本文翻译自[angularJS 官方文档](https://angular.io/docs/ts/latest/guide/architecture.html)

Angular 是一个构建客户端应用的框架(framework)， 由HTML和JavaScript 或者TypeScript编译成的JavaScript组成。这个框架由许多必要的或者不必要的类库(libraries)组成。

如何创建一个Angular应用：
1. 使用Angular标签来组合一个HTML模板(template)。
2. 创建组件(component)来管理这些HTML模板。
3. 在服务(services)中添加应用的逻辑。
4. 将组件和服务放到模块中(modules)。
5. 通过启动（bootstrapping）根模块(root module)来启动app.  Angular会接管你的应用，将内容放在浏览器中，并且当用户有交互的时候，根据你的设定进行回应。

如图所示：

![Angular运行图](https://angular.io/resources/images/devguide/architecture/overview2.png)

示例图中定义了8个主要的组成：
* Modules 模块
* Components   组件
* Templates  模板
* Metadata  元数据
* Data binding 数据绑定
* Directives  指令 
* Services  服务
* Dependency injection  依赖注入

我们依次来学习这些组成部分。 
> [示例代码](https://embed.plnkr.co/?show=preview)

## Modules(模块)

![](https://angular.io/resources/images/devguide/architecture/module.png)

Angular 应用是模块化的，Angular有自己的一套模块化系统，称为 *Angular Modules* 或者 *NgModules*

每一个Angular的应用至少有一个Angular module, 根模块 （the root module)。 一般来说叫 `AppModule`。 也许在晓得Angular 应用中，只需要一个root module 就足够了，
但是大多数的应用函数拥有很多功能模块(feature modules)的， 每一个内聚的代码块都贡献给应用一个主要的部分，一个工作流或者相关的功能集合。

一个Angular模块，无论跟模块还是功能模块，都是一个带有`@NgModule`指令的类

> 指令是修改JavaScript类的函数。 Angular有许多指令，它们把元数据添加到类中从而让Angular知道这些类的含义以及应该做什么。

`NgModule` 是一个指令函数， 它带入一个元数据对象，该对象的属性描述这个模块，其中最重要的属性有：
* declarations - 该模块的 `view class`， Angular有三种 view class: components, directives 和 pipes。
* exports - declarations的子集， 表明哪些子集可以被其他模块的组件和模板使用和可见。
* imports - 其他模块到处的类，这些类被本模块中的组件或模板所依赖。
* providers - 服务(services) 的创造者， 这些服务在该模块中提供全局的服务，它们可以被所有的部分所引用。
* bootstrap - 主要应用视图(The main application view)， 称为 *the root component*, 用来装在所有其他的应用视图。 只有主模块才设置bootstrap 属性。

一下是一个简单的例子：

```javascript
/* src/app/applmodule */
import { NgModule }      from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
@NgModule({
  imports:      [ BrowserModule ],
  providers:    [ Logger ],
  declarations: [ AppComponent ],
  exports:      [ AppComponent ],
  bootstrap:    [ AppComponent ]
})
export class AppModule { }
```

注意，根模块是不需要到处compoent的，因为没有其他模块调用跟模块。
运行一个应用，是通过启动它的跟模块， 当开发的时候，你在main.ts中启动AppModule, 如下所示：

```javascript
/*src/main.ts*/
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';
import { AppModule } from './app/app.module';

platformBrowserDynamic().bootstrapModule(AppModule);

```

## Angular modules 和 JavaScript modules 的对比

Angular module, 是一个被 `@NgModule` 修饰的类， 是Angular重要的功能。 JavaScript 也有自己的模块系统，用于管理JavaScript对象集合。 它和Angular 模块系统完全不同并且没有任何联系

在JavaScript中，每一个文件就是一个模块，所有定义在这个文件中的对象都属于这个模块。 这个模块使用export定义了一些公共的对象。其他的模块使用 import 来访问这些公共对象。

例如：

```javascript
import { NgModule }     from '@angular/core';
import { AppComponent } from './app.component';

export class AppModule { }
```

这是两种不同并且互补的模块系统，两种都应用在你的应用中。

## Angular 类库

![Angular library](https://angular.io/resources/images/devguide/architecture/library-module.png)

Angular 可以看做一个JavaScript 模块的集合。你可以认为他们是类库模块
每一个Angular 类库名字都以`@angular`前缀开头， 可以通过 `npm`包管理工具来安装，并使用 `import` 来导入使用。

例如：

```javascript
import { Component } from '@angular/core';
```

你也可以使用`import`从Angular类库中导入Anuglar模块:

```javascript
import { BrowserModule } from '@angular/platform-browser';
```

在root module的例子中，应用需要 `BrowserModule` 中的元数据，要的到元数据，则把模块加入 `@NgModule` metadata 的 `imports`属性中：

```javascript
imports: [ BrowserModule ],
```

这样，你就同时使用了Angular 和 JavaScript 模块化系统。

## 组件（Component）

一个组件(Component)控制了一个屏幕，叫做视图(View)。 例如，以下的视图由组件控制：
* 带有导航链接的 app root
* 英雄的列表 (the list of heroes, 教程中的例子)
* 英雄的编辑器界面

在类中定义了组件的逻辑： 如何支持视图。 该类通过属性和方法的API与视图进行交互。

例如，下面的 `HeroListComponent`组件拥有一个 `heroes`属性，它返回一个英雄数组，该数组从服务获得。
`HeroListComponent` 还有一个 `SelectHero()` 的方法，当用户点击选择列表中的英雄时，对selectedHero进行赋值。

```javascript
export class HeroListComponent implements OnInit {
  heroes: Hero[];
  selectedHero: Hero;

  constructor(private service: HeroService) { }

  ngOnInit() {
    this.heroes = this.service.getHeroes();
  }

  selectHero(hero: Hero) { this.selectedHero = hero; }
}
```

Angular 创建，更新，删除组件当用户使用应用的时候。 你可以通过生命周期钩子在每个生命周期采取必要的措施，例如前面用到的`ngOnInit()`.

## 模板(Templates)

通过组件的伴侣模板(template)来定义视图。一个模板就是一个HTML组成的表单，来告诉Angular怎样加载组件。
模板看起来很像普通的HTML，但是还是有一些不同的，一下是`HerolistComponent` 的模板：

```html
<h2>Hero List</h2>
<p><i>Pick a hero from the list</i></p>
<ul>
  <li *ngFor="let hero of heroes" (click)="selectHero(hero)">
    {{hero.name}}
  </li>
</ul>
<hero-detail *ngIf="selectedHero" [hero]="selectedHero"></hero-detail>
```

尽管看起来这个模板使用了原生的HTML元素如 `<h2>` 以及 `<p>`, 但是也用了一些不同的代码，例如： `*ngFor`, `{{hero.name}}`, `(click)`， `[hero]`以及 `<hero-detail>` 等， 这些使用了Angular 模板语法。
在最后一行， `<hero-detail>`标签是一个自定义元素，代表了一个新的组件， `HeroDetailComponent`。

`HeroDetailComponent`和`HeroListComponent`不同， 它代表一个具体的用户从当前`HeroListComponent` 选择的英雄， 它算是`HeroListComponent`的一个下属组件。
![](https://angular.io/resources/images/devguide/architecture/component-tree.png)

注意，`<hero-detail>` 在原生HTML中毫无违和感，组件和原生HTML实现了无缝混编。

## 元数据(Metadata)

![metadata](https://angular.io/resources/images/devguide/architecture/metadata.png) 元数据告诉Angular如何加载一个类。

回顾一下`HeroListComponent` 的代码，你发现那仅仅是一个类，你找不到任何 'Angular'存在的痕迹。

实际上， `HeroListComponent`的确只是一个类，除非你告诉Angular, 它不会变成一个组件。

想要告诉Angular 它是一个组件，就在类中加入元数据(metadata)

在**TypeScript** 中， 通过使用指令(decorator)加载元数据， 一下是`HeroListComponent`加载元数据的例子：

```javascript
@Component({
  selector:    'hero-list',
  templateUrl: './hero-list.component.html',
  providers:  [ HeroService ]
})
export class HeroListComponent implements OnInit {
/* . . . */
}
```

我们可以在这里看到，是 `@Component `指令定义了它下面的类是一个组件类。
`@Component`指令中定义了一个必填的配置对象，它定义了Angular需要创建组件和视图的属性：

* `selector`: CSS选择器，告诉Angular 当找到对应的标签时（例如`<hero-list>`），创建并插入一个组件的实例. 例如，当app的HTML包含`<hero-list></hero-list>`标签的时候，Angular 就会在这个地方插入组件`HeroListComponent`的实例。
* `templateUrl`: 组件的模板地址。
* `providers`: 包含有组件需要的依赖注入的服务的数组。这是告诉Angular组件的构造函数需要某个服务（例如`HeroService`）的方法。

![](https://angular.io/resources/images/devguide/architecture/template-metadata-component.png) `@Component`中的元数据告诉Angular如何去获得component主要组成部分

模板， 元数据 和 组件一起描述了一个视图。
应用其他的元数据指令可以Angular的行为。 `@Injectable`, `@Input`, `@OutPut` 都是一些比较流行的指令
简单来说，你需要添加元数据来告诉Angular怎么去做。

## 数据绑定(Data binding)

如果不适用框架，你需要亲自把数据放入到HTML控件中，然后把用户的响应转换为actions和数据更新。 手动写这些push/pull逻辑既单调乏味又容易出错，并且是JQuery 工作者已经证实的噩梦...

![](https://angular.io/resources/images/devguide/architecture/databinding.png) Angular 支持数据绑定，是一个模板部分和组件部分粘合起来的机制。 给HTML模板添加数据绑定，告诉Angular如何连接两边。

如图所示，有四种数据绑定语法，每种都有方向： 指向DOM, 从DOM来，以及双向绑定。

以下是`HeroListComponent` 模板的三种绑定方式：

```html
<li>{{hero.name}}</li>
<hero-detail [hero]="selectedHero"></hero-detail>
<li (click)="selectHero(hero)"></li>
```

* `{{hero.name}}` 嵌入绑定， 展示了 组件的`hero.name`属性值和`<li>`元素。
* `[hero]` 元素绑定 将`selectedHero`的值从父组件`HeroListComponent`传入到`HeroDetailComponent`的`hero`属性中
* `(click)` 事件绑定，当用户点击英雄的名字的时候，调用了组件的`selectHero`方法。

双向绑定(Two-way data binding) 是第四种，重要的一种绑定方式，它用`ngModel`指令组合了属性绑定和事件绑定在一起：

```html
<input [(ngModel)]="hero.name">
```

在上面的双向绑定中，数据属性值和输入框进行了绑定，当用户改变值时，反向改变组件中的值（此时正如事件绑定一样）。

Angular 在每一个事件循环中加载所有的数据绑定，从根应用的组件书一直到所有的子组件：

![](https://angular.io/resources/images/devguide/architecture/component-databinding.png)
数据绑定在模板和组件交互中扮演着重要的角色。同时， 在父组件和子组件的交互中也扮演着重要角色。

## 指令 (Directives)

![](https://angular.io/resources/images/devguide/architecture/directive.png) 

Angular 的模板是动态生成的。当Angular加载模板的时候，通过指令的描述和介绍来转换DOM.

一个指令是一个带有@Directive 的类。 一个组件是一个指令带一个模板(a directive-with-a-template); 一个`@Component` 指令实际上是一个有模板方向的继承`@Directive`指令的功能。

尽管组件从技术上来讲是一个指令，但是组件对于Angular应用来说是很特殊的，因此这里将它和指令单独分开来。

另外，还存在其他的两种指令： 结构型(stuctural)和属性型(attribute)指令
他们倾向于在元素标签内，像属性一样出现，通常是用名称，但更多的时候作为赋值或者绑定的目标出现。

结构型指令通过在DOM添加，删除或者替换元素来改变显示：

```html
<li *ngFor="let hero of heroes"></li>
<hero-detail *ngIf="selectedHero"></hero-detail>
```

* `*ngFor` 告诉Angular heroList的每一个hero都输出一个`<li>`
* `*ngIf` 如果选中的英雄存在则包含`HeroDetail`组件。

属性指令改变一个已经存在的元素的表现或者行为，在模板中看起来像原生HTML的属性，所以称为属性值令。

`ngModel`指令，就是一个典型的属性值令。 它改变了存在元素的行为（通过与组件中的属性双向绑定）：

```html
<input [(ngModel)]="hero.name">
```
Angular 还有很多指令可以改变结构(`ngSwitch`) 以及改变DOM的面貌 （ngStyle, ngClass）

当然，你也可以自定义指令， 组件 `HerolistComponent`就算是一个自定义指令。

## 服务（Service）

![](https://angular.io/resources/images/devguide/architecture/service.png)

服务的定义比较广泛，包括任何你的应用需要的值，函数以及功能。 几乎所有的都可以算是服务。
一个服务通常是一个精密的，有良好定义意图的类，它的功能应该是专一并且精通的。例如：
* 登录服务
* 数据服务
* 信息服务
* 税务计算
* 应用配置

Angular 中服务没有什么特别，对服务也没有定义。 没有服务基于类，也没有地方注册服务。

但是服务对于Angular的应用来说还是重要的，组件是服务最大的消费者。
一下是一个log的服务类：

```javascript
export class Logger {
  log(msg: any)   { console.log(msg); }
  error(msg: any) { console.error(msg); }
  warn(msg: any)  { console.warn(msg); }
}
```

还有一个`HeroService`服务类，用Promise来获得heroes. `HeroService`依赖于Logger服务和另外一个`BackendService`，它用来做服务器启动工作。

```javascript
export class HeroService {
  private heroes: Hero[] = [];

  constructor(
    private backend: BackendService,
    private logger: Logger) { }

  getHeroes() {
    this.backend.getAll(Hero).then( (heroes: Hero[]) => {
      this.logger.log(`Fetched ${heroes.length} heroes.`);
      this.heroes.push(...heroes); // fill cache
    });
    return this.heroes;
  }
}
```

服务无处不在。
组件类应该尽量小(Lean)。 他不需要从服务器中获取数据，验证输入或者打log， 他应该委托这些给服务。
组件的只能应该是去提高用户体验， 它调视图(在template中)和应用逻辑（在model中），一个好的组件提供属性和方法给数据绑定，将其他所有的事情委托给服务。

Angular尽管不强制这些规则。 Angular可以通过依赖注入帮助你很好的实现这些规则，将你的逻辑放到服务中去。

## 依赖注入(Dependency injection)

![](https://angular.io/resources/images/devguide/architecture/dependency-injection.png)

依赖注入是为一个类提供它所需要的实例的一个途径。 大多数注入都是服务。 Angular通过依赖注入来给新的组件提供他们需要的服务。

Angular 可以通过组件的构造函数来告诉组件那些服务使他们需要的。例如：

```javascript
constructor(private service: HeroService) { }
```

当Angular 创建了一个组件，首先做的就是注入一个组件需要的服务。

一个注入维护了一个它之前创建的服务的实例。 如果一个需要的服务不在容器内，注入器会创建一个并在返回Angular之前加入到容器中。 当所有依赖的服务都被解决和反悔了， Angular 会调用组件的构造函数并把这些服务作为参数传入。 这就是依赖注入的过程。

如图所示：

![](https://angular.io/resources/images/devguide/architecture/injector-injects.png)

如果一个注入器没有这个服务要怎么创建？ 简单来说， 你必须在provider中注册这个服务给注入器。provider 可以创建或者返回服务，通常是服务本身。
你可以在modules 或者 component 中注册provider

通常，往根模块中注册provider，那么这个服务就可以在所有的地方引用了（同一个实例）。
另外，在component级别注册provider, 是在`@component` 的元数据中写入providers 属性：

```javascript
@Component({
  selector:    'hero-list',
  templateUrl: './hero-list.component.html',
  providers:  [ HeroService ]
})
```

在组件级别注册provider意味着每次创建这个组件的实例你都会获得该服务的一个新的实例。

依赖注入需要注意一下几点：
* 依赖注入是Angular framework内的可以在任何地方使用。
* 注入器是主要的机制
  * 一个注入器维护了它创建的一个服务的容器
  * 一个注入器可以从provider创建一个新的服务实例。
* 一个provider是创建服务的“秘方”
* 通过indectors 来注册 providers.

到此， Angular的一些基本的重要概念覆盖了。
