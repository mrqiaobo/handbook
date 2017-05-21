---
layout: contents
title: AngularJs 4.0 学习 (九) --- 依赖注入(Dependency Injection)
categories: [Web]
tages: [AngularJs, web, tech]
---
# AngularJs 4.0 学习 (九) --- 依赖注入(Dependency Injection)

> Angular 的依赖注入系统可以实时创建和发布以来的服务 本文翻译自[AngularJS官方文档](https://angular.io/docs/ts/latest/guide/dependency-injection.html)

依赖注入是应用设计模式的重要部分。 Angular 有自己的依赖注入框架，没有它就无法创建一个Angular应用。 依赖注入被广泛使用，又被称之为DI

本篇文档包含了DI的定义，重要性以及如何使用它来创建Angular 应用。

## 为什么要使用依赖注入

为了理解依赖注入的重要性，考虑如果自己手写下面的code:

```javascript
export class Car {
  public engine: Engine;
  public tires: Tires;
  public description = 'No DI';
  constructor() {
    this.engine = new Engine();
    this.tires = new Tires();
  }
  // Method using the engine and tires
  drive() {
    return `${this.description} car with ` +
      `${this.engine.cylinders} cylinders and ${this.tires.make} tires.`;
  }
}
```

`Car`类在构造函数中创造了他需要的所有的东西。那么问题来了， `Car` 类此刻是脆弱的，不灵活的，也不便于测试。

一个`Car`需要引擎和轮胎，这里`Car`没有请求，缺在构造函数中自己实例化了`Engine`和`Tire`的两个实例。

那么，如果此时`Engine`类改变了并且构造函数需要参数了怎么办？这样`Car`类就会被break, 除非你重写。 
当开始写`Car`类的时候甚至不知道`Engine`的构造函数会有参数。甚至现在都不会使用它。但是现在必须开始因为当
`Engine`改变了， `Car`类必须改变。 这使得`Car`是脆弱的。

如果我们想要在`Car`上使用不同的tire呢？ 不行，因为现在我们锁定了`Tires`类的创建实例，这使得`Car`类不灵活

现在所有的`car`都有自己的`engine`, 不能喝其他实例共享。 但是共享对于engine来说是合理的， `Car`类缺乏灵活性
来共享在前面为其他的客户创建的服务。

当写测试的时候，要祈求它的以来不要太过分，甚至在测试环境中能够创建`Engine`是否可能
也是个问题。我们不知道在测试的时候会发生什么。

我们逆序控制car的隐藏依赖。 当不能控制这些依赖的时候，类就变得很难测试。

那么如何做到这些？ 也简单，将它改成如下所示：

```javascript
public description = 'DI';

constructor(public engine: Engine, public tires: Tires) { }
```

那么现在如何？ 现在以来的定义在构造函数中。 car现在不在自己创建engine 和 tires, 只是使用它们。

现在可以通过在创建car的实例的时候传入engine和tires到它的构造函数了。

```javascript
// Simple car with 4 cylinders and Flintstone tires.
let car = new Car(new Engine(), new Tires());
```

厉害吧？engine 和 tire 的依赖和car类解耦了。你可以传入任意的engine和car，只要它们符合engine 和tires的标准API定义即可。
现在，如果有继承了Engine类的，Car也没有任何关系。 然而car的使用者却有关系，因为需要改变car的创建方法，如下所示：

```javascript
class Engine2 {
  constructor(public cylinders: number) { }
}
// Super car with 12 cylinders and Flintstone tires.
let bigCylinders = 12;
let car = new Car(new Engine2(bigCylinders), new Tires());
```

现在`Car`类也更好测试了。因为我们完全控制了它的依赖。我们可以传入的mock的数据

```javascript
class MockEngine extends Engine { cylinders = 8; }
class MockTires  extends Tires  { make = 'YokoGoodStone'; }

// Test car with 8 cylinders and YokoGoodStone tires.
let car = new Car(new MockEngine(), new MockTires());
```
**以上所示，就是依赖注入。**

依赖注入就是类从外部源接收依赖而不是自己创建他们。

那么对于简单的使用者如何？ 有人想使用`Car`但是却必须创建3个部分： `Car`, `Engine`, `Tires`。 Car将
它的问题转移到了使用者哪里。我们需要一些来装配这些的部分. 首先，可以尝试写下面的大类：

```javascript
import { Engine, Tires, Car } from './car';
// BAD pattern!
export class CarFactory {
  createCar() {
    let car = new Car(this.createEngine(), this.createTires());
    car.description = 'Factory';
    return car;
  }
  createEngine() {
    return new Engine();
  }
  createTires() {
    return new Tires();
  }
}
```
现在类中只有3个创建方法，还可以。 但是醉着应用不断变大，会变得很难维护。这个工厂会变得特别大。

如果我们能够仅仅列出我们需要的东西而不去定义如何注入呢？ 这就是依赖注入框架做的事情。 想象一下，
框架有一个叫注入器(*injector*)的东西，你在它里面注册了一些东西，他会找到真么创建它们。

当你需要一个car时，只需要让injector给你就可以了：

```javascript
let car = injector.get(Car);
```

完美。car类对如何创建engine和tires一无所知， 使用者对于如何创建car一无所知，我们不需要一个巨大的
工厂类去维护。 Car 和使用者仅仅向注入器要它们所需要的即可。

这就是**依赖注入框架**所做的事。

## Angular的依赖注入

Angular 使用自己的依赖注入框架。 该框架也可以作为单独的模块给其他的应用和框架来使用。

我们用一个例子来解释在Angular构建组件的时候做了什么。 下面是一个简单版本的`HeroesComponent`

```javascript
// src/app/heroes/heroes.component.ts
import { Component } from '@angular/core';
@Component({
  selector: 'my-heroes',
  template: `
  <h2>Heroes</h2>
  <hero-list></hero-list>
  `
})
export class HeroesComponent { }
```

```javascript
//src/app/heroes/hero-list.component.ts
import { Component }   from '@angular/core';
import { HEROES }      from './mock-heroes';
@Component({
  selector: 'hero-list',
  template: `
  <div *ngFor="let hero of heroes">
    {{hero.id}} - {{hero.name}}
  </div>
  `
})
export class HeroListComponent {
  heroes = HEROES;
}
```

```javascript
//src/app/heroes/hero.ts
export class Hero {
  id: number;
  name: string;
  isSecret = false;
}
```

```javascript
//src/app/heroes/mock-heroes.ts
import { Hero } from './hero';
export var HEROES: Hero[] = [
  { id: 11, isSecret: false, name: 'Mr. Nice' },
  { id: 12, isSecret: false, name: 'Narco' },
  { id: 13, isSecret: false, name: 'Bombasto' },
  { id: 14, isSecret: false, name: 'Celeritas' },
  { id: 15, isSecret: false, name: 'Magneta' },
  { id: 16, isSecret: false, name: 'RubberMan' },
  { id: 17, isSecret: false, name: 'Dynama' },
  { id: 18, isSecret: true,  name: 'Dr IQ' },
  { id: 19, isSecret: true,  name: 'Magma' },
  { id: 20, isSecret: true,  name: 'Tornado' }
];
```

`HeroComponent` 是*Hero*功能部分的根组件，包含了所有的子组件。这个例子中只有一个子组件`HeroListComponent`来展示英雄列表。

现在，`HeroListComponent`从`HEROES`中获得英雄。 这对早期的开发足够了，但是不够完美。如果要测试组建或者想要从其他服务获得英雄，就必须修改`heroes`的实现，并且修改`HEROES`

所以最好用一个service来包装app如何获得数据。

下面的`HeroService`暴露了一个`getHeroes`方法，返回一些mock数据，使用者不需要知道返回的是什么。

```javascript
import { Injectable } from '@angular/core';
import { HEROES }     from './mock-heroes';
@Injectable()
export class HeroService {
  getHeroes() { return HEROES; }
}
```

> 当然，这不是一个真正的service。如果app真的从远程服务器获取数据，API需要是异步的，可能会返回一个Promise类型的结果。 这样，还需要重写组件来使用service

一个service不过是Angular中的一个类而已。除非使用Angular注入器来注册它，否则就是一个类而已。

### 配置注册器

我们不用一定要自己创建一个注入器， Angular在启动的时候创建了一个应用级别的注册器：

```javascript
platformBrowserDynamic().bootstrapModule(AppModule);
```

我们必须通过在provider中注册应用需要的service来配置注入器。我们可以在NgModule或者应用的组件中注册provider。

### 在*NgModule*中注册providers

下面的`AppModule`注册了两个providers, `UserService` 和一个`APP_CONFIG`的provider， 在`providers` 数组中。

```javascript
@NgModule({
  imports: [
    BrowserModule
  ],
  declarations: [
    AppComponent,
    CarComponent,
    HeroesComponent,
    /* . . . */
  ],
  providers: [
    UserService,
    { provide: APP_CONFIG, useValue: HERO_DI_CONFIG }
  ],
  bootstrap: [ AppComponent ]
})
export class AppModule { }
```

由于`HeroService`只在`HeroesComponent`以及她的子组件中使用，因此，将其注册在顶层的`HeroesComponent`中是最合适的。

### 在组件中注册providers

下面的例子展示了在`HeroesComponent`的providers中注册了`HeroService`

```javascript
import { Component }          from '@angular/core';

import { HeroService }        from './hero.service';

@Component({
  selector: 'my-heroes',
  providers: [HeroService],
  template: `
  <h2>Heroes</h2>
  <hero-list></hero-list>
  `
})
export class HeroesComponent { }
```

### 使用NgModule还是使用组件来注册

在NgModule中注册是跟注册器，意味着每一个在这里注册的provider在整个应用中都可以使用(该module的应用), 而在组件中注册的只能在组建和它的子类中使用。

这里， `APP_CONFIG`服务需要在整个应用中使用，所以在`AppModule`的`@NgModule providers`中注册。 而`HeroService`仅仅在英雄功能部分使用，所以在`HeroComponent`
中注册比较合理。

### 为注入准备好*HeroListComponent*

`HeroListComponent`应该从注入的`HeroService`中获得英雄，每一个依赖注入的模型中，组建都必须在构造器中获取service。 这里做了一些小的改变：

```javascript
/*src/app/heroes/hero-list.component(with DI)*/
import { Component }   from '@angular/core';
import { Hero }        from './hero';
import { HeroService } from './hero.service';
@Component({
  selector: 'hero-list',
  template: `
  <div *ngFor="let hero of heroes">
    {{hero.id}} - {{hero.name}}
  </div>
  `
})
export class HeroListComponent {
  heroes: Hero[];
  constructor(heroService: HeroService) {
    this.heroes = heroService.getHeroes();
  }
}
```

```javascript
/*src/app/heroes/hero-list.component(without DI)*/
import { Component }   from '@angular/core';
import { HEROES }      from './mock-heroes';
@Component({
  selector: 'hero-list',
  template: `
  <div *ngFor="let hero of heroes">
    {{hero.id}} - {{hero.name}}
  </div>
  `
})
export class HeroListComponent {
  heroes = HEROES;
}
```

**注意构造函数**

添加一个参数到构造函数不是所有发生的事情。

```javascript
constructor(heroService: HeroService) {
  this.heroes = heroService.getHeroes();
}
```

注意，在构造器中有一个参数是`HeroServie`类型的，并且`HeroListComponent`类有`@Component`修饰， 并且父组件`HeroesComponent`中也有`HeroService`在providers的信息。

这些组合告诉了Anguler注入器将HeroService的实例在无论何时创建`HeroListComponent`的时候，注入一个`HeroService`进去。

### 隐式的创建注入器

通过下面的例子可以看到如何使用一个注入器创建一个新的`Car`, 这样是显式创建了一个注入器

```javascript
  injector = ReflectiveInjector.resolveAndCreate([Car, Engine, Tires]);
  let car = injector.get(Car);
```

我们在其他文档中没有找到这样的代码。 显式创建注入器不是最好的选择。 Angular在创建组件的时候已经做好了创建和调用注入器的工作，为ulunshi通过标签，还是在router中导航到一个组建。如果让Angular来做这些，将会十分方便。

### 单例服务

在注入器的作用域内，依赖是单例的。 在这个例子中，一个`HeroServie`的实例在`HeroesComponent`以及它的子组建`HeroListComponent`中是共享的。

但是，Angular的依赖注入是一个层级注入系统。这意味着在下层的注入器可以创建自己的service实例。

### 测试组件

依赖注入的类更加容易测试。只需要在构造函数中列出依赖即可。下面的例子中，创建一个HeroListComponent和一个模拟的service来进行测试

```javascript
let expectedHeroes = [{name: 'A'}, {name: 'B'}]
let mockService = <HeroService> {getHeroes: () => expectedHeroes }

it('should have heroes when HeroListComponent created', () => {
  let hlc = new HeroListComponent(mockService);
  expect(hlc.heroes.length).toEqual(expectedHeroes.length);
});
```

### 当service中需要service

HeroService是比较简单的，不需要其他的依赖。 但是当需要依赖的时候怎么办？例如需要一个log的服务。 我们也用同样的依赖注入形式即可。

```javascript
/*src/app/heroes/hero.service(v2)*/
import { Injectable } from '@angular/core';
import { HEROES }     from './mock-heroes';
import { Logger }     from '../logger.service';
@Injectable()
export class HeroService {
  constructor(private logger: Logger) {  }
  getHeroes() {
    this.logger.log('Getting heroes ...');
    return HEROES;
  }
}
```

```javascript
/*src/app/heroes/hero.service(v1)*/
import { Injectable } from '@angular/core';
import { HEROES }     from './mock-heroes';
@Injectable()
export class HeroService {
  getHeroes() { return HEROES; }
}
```

构造函数从构造器中获得Logger的实例并将其存储到私有属性中， 可以调用这个私有属性的方法。

### 为什么使用*@Injectable()*

`@Injectable()`标记一个类可以在注入器中进行实例化。也就是说，注入器实例化类的时候，如果类没有被标记，就会报错。

> 开始的时候，可以在第一个版本中省略`@Injectable()`因为没有要注入的参数。但是现在必须加上，因为服务有了依赖注入。 Angular需要构造函数参数元数据来注入Logger.

> 建议给所有的Service都加上`@Injectable()`

注入器也可以实例化组建的，但是为何没有给组建加`@Injectable()`呢？ 如果真的需要的时候可以加上。但是在这里不需要，因为组建已经有一个`@Component`， 这个装饰是`@Injectable`的一个子类型（包括`@Directory`和`@Pipe`）。 实际上，`@Injectable()`声明了一个类可以被注入器实例化。

在运行时，注册器可以在编译过的JS代码中读取类的元数据，并且使用构造器参数的类型去决定用哪一个来注入。 不是每一个JS类都有元数据的，TypeScript编译器默认取消了元数据。 如果在`tsconfig.json`中的`emitDecoratorMetadata`编译选项设置为true, 编译器会为每一个至少有一个装饰器的JS类添加元数据。

虽然任何的装饰器都能够达到注入的效果，给service使用`@Injectable()`会让代码更清晰。

注意在使用`@Injectable()`的时候，不要忘记了后面的小括号。

## 创建和注册一个logger service

注入一个logger到HeroServic分两步：
1. 创建一个logger service
2. 在应用中注册

logger service很简单：

```javascript
import { Injectable } from '@angular/core';
@Injectable()
export class Logger {
  logs: string[] = []; // capture logs for testing
  log(message: string) {
    this.logs.push(message);
    console.log(message);
  }
}
```

我们可能在许多地方使用到logger service, 因此将它放在app文件夹下并在appModule中注册它

如果忘记在provider中注册，Angular就会抛出一个异常。

### 注入器的providers

一个provider为依赖提供了容器以及运行时版本。 注入器根据providers创建需要注入到组建或者其他service的service的实例。 必须将给注入器注册一个service provider, 否则它不知道如何创建一个service。

### Provider 类和provide 对象迭代器

我们定义providers 是这样的:

```javascript
providers:[logger]
```

实际上这是一种简写的表达式，它实际上用由两个属性组成的provider迭代器创建的：

```javascript
[{provide: Logger, useClass: Logger}]
```
第一个是注册provider的token, 它提供可一个key 给依赖值和注册的provider。第二个是provider定义对象，可以认为是创建依赖值的秘方。有许多方法创建依赖值得原因是因为有许多方法来书写秘方。

### 可选类的providers

有些情况可能需要需要用一个不同的类来提供service。 下面的代码告诉注入器返回一个BetterLogger 

```javascript
[{ provide: Logger, useClass: BetterLogger }]
```

### 有依赖的类providers

也许一个`EventBetterLogger`可以在log中展示用户姓名， Logger 可以从注册器中获得UserService

```javascript
@Injectable()
class EvenBetterLogger extends Logger {
  constructor(private userService: UserService) { super(); }

  log(message: string) {
    let name = this.userService.user.name;
    super.log(`Message to ${name}: ${message}`);
  }
}
```

在BetterLogger中配置：

```javascript
[UserService, { provide: Logger, useClass: EventBetterLogger }]
```

### 别名的类providers

加入一个老的组件依赖于OldLogger类， OldLogger有一个同样的接口， NewLogger， 由于一些原因不能够更新老的组件去使用它。 当老组件使用OldLogger时，我们希望用NewLogger单例来替代它。 当组件要求新的或者老的组件的时候都应该注入它。 那么OldLogger应该成为NewLogger的一个别名。

当然我们不想要两个不同的NewLogger实例对象。但不幸的是，当时用useClass的时候会这样。

```javascript
[ NewLogger,
    //Not aliased! Creates two instance of `NewLogger`
    { provide: OldLogger, useClass: NewLogger}]
```

解决方案是： 使用useExisting

```javascript
[ NewLogger,
    //Not aliased! Creates two instance of `NewLogger`
    { provide: OldLogger, useExisting: NewLogger}]
```

### 值类型的providers

有的时候使用提供好值得对象比注入器创建类实例更加简单：

```javascript
// An object in the shape of the logger service
let silentLogger = {
  logs: ['Silent logger says "Shhhhh!". Provided via "useValue"'],
  log: () => {}
};
```

然后可以用userValue来注册
```javascript
[{ provide: Logger, useValue: silentLogger }]
```

### 工厂类型的providers

有些时候，我们需要根据一些只有在使用到的时候才能够获得的信息来动态创建依赖的值，并且这些信息可能随着浏览器session而变化。 假设这些可注入的服务不能够独立访问这些信息的源头， 这种情况叫做 Factory provider.

为了给这个情况举例，我们增加了一个新的需求: `HeroService` 在一般用户浏览的时候必须隐藏 secret Hero， 只有授权的用户才可以看到他们。 

因此`HeroService`需要知道用户的信息，要知道用户是否被授权查看 secret hero. 这种授权是会在session中改变的，比如当你登录了另一个user的时候。

我们不能讲`UserService`注入到`HeroService`， `HeroService`不能直接访问`UserServie`来获得用户授权的信息，只能在构造其中添加一个标识来标明是否获得授权。

```javascript
constructor(
  private logger: Logger,
  private isAuthorized: boolean) { }

getHeroes() {
  let auth = this.isAuthorized ? 'authorized ' : 'unauthorized';
  this.logger.log(`Getting heroes for ${auth} user.`);
  return HEROES.filter(hero => this.isAuthorized || !hero.isSecret);
}
```

我们可以注入`Logger`， 但是却不能够注入布尔值`isAuthorized`，因此我们需要使用factory provider 来接管对`isAuthorized` 的创建.

一个factory provider 需要一个工厂方法：

```javascript
let heroServiceFactory = (logger: Logger, userService: UserService) => {
  return new HeroService(logger, userService.user.isAuthorized);
};
```

虽然`HeroService`不能够访问`UserService`， 但是工厂方法可以可以. 我们用工厂方法将 Logger 和UserService一起注入到 heroService中去。

```javascript
export let heroServiceProvider =
  { provide: HeroService,
    useFactory: heroServiceFactory,
    deps: [Logger, UserService]
  };
```

上面代码中， useFactory 告诉Angular provider是一个由`HeroServiceFactory` 实现的工厂方法。 `deps` 属性是一个*provider tokens*数组， Logger和UserService作为token来服务于它们的provider。 注入器会根据这些token来将对应的Service注入到对应的工厂方法中。

将这个provider导出以便可以进行复用。 在之后，可以在任何需要使用的时候，利用这个provider 将HeroService注入。

在这个例子中，我们需要在`HeroComponent`中代替`HeroService`，更新后的代码如下所示：

```javascript
// inject with heroServiceProvider
import { Component }          from '@angular/core';
import { heroServiceProvider } from './hero.service.provider';
@Component({
  selector: 'my-heroes',
  template: `
  <h2>Heroes</h2>
  <hero-list></hero-list>
  `,
  providers: [heroServiceProvider]
})
export class HeroesComponent { }
```

```javascript
// inject with HeroService
import { Component }          from '@angular/core';
import { HeroService }        from './hero.service';
@Component({
  selector: 'my-heroes',
  providers: [HeroService],
  template: `
  <h2>Heroes</h2>
  <hero-list></hero-list>
  `
})
export class HeroesComponent { }
```

### 依赖注入的tokens

当注册一个provider到注册器的时候，需要给provider一个依赖注入的token. 注入器会维护一份内部的token-provider的匹配map, 当需要依赖的时候，会根据这个map来引用， token是这个map 的关键。

在之前的例子中，我们依赖的都是类的实例，而实例的类型（class type）会提供一个寻找它的key,  例如，直接通过提供实例的类型而提供了token:

```javascript
heroService: HeroService;
```

同样，当在构造函数中的需要class类型的依赖时候也是这样的， 当需要一个heroService类型的实例依赖的时候， Anguler会知道会将HeroService类作为token注入。

```javascript
constructor(heroService: HeroService)
```

因此，大部分的依赖注入是由class提供的时候是很方便的。


### 非class类型的依赖

当需要注入的不是一个class类型的时候如何呢？ 当需要注入的类型是一个字符串，布尔值，函数，或者一个对象呢？

应用通常有一个config 对象，通常包含许多个小的元素，例如标题或Web API的地址等等，但是这些配置对象很多情况下并不是一个class的实例。 通常是如下的object迭代器：

```javascript
export interface AppConfig {
  apiEndpoint: string;
  title: string;
}

export const HERO_DI_CONFIG: AppConfig = {
  apiEndpoint: 'api.heroes.com',
  title: 'Dependency Injection'
};
```

如果我们希望将这个配置对于注入器可用呢？ 我们可以通过value provider来注入一个object. 但是用什么作为token呢？ 因为object没有class作为token, 没有AppConfig这个class.

> 注意，TypeScript的intercace 并不是合法的token。 因为interface 仅仅是在TypeScript设计的时候提供的，当在产生JavaScript代码之后， interface已经不存在了。因此在运行时，没有interface留下的任何信息供Angular追踪。

### injection Token

解决非class类型的注入的方案之一是使用 injection token. 它的定义方式如下所示：

```javascript
import { InjectionToken } from '@angular/core';

export let APP_CONFIG = new InjectionToken<AppConfig>('app.config');
```

类型参数，是可选的，将依赖的类型转换为开发者的工具。 description 是另外一个开发者帮助工具。

使用InjectionToken的对象来注入：

```javascript
providers: [{ provide: APP_CONFIG, useValue: HERO_DI_CONFIG }]
```

现在，可以在构造器中使用@inject指令来注入配置对象了：

```javascript
constructor(@Inject(APP_CONFIG) config: AppConfig) {
  this.title = config.title;
}
```

## Optional 依赖

上面的例子中，`HeroService`依赖了`Logger`, 那么当我们获取的时候不一定需要Logger怎么办呢？ 我们可以使用@Optional()标记在构造器参数前面来告诉Angular这个参数是可选的。

```javascript
import { Optional } from '@angular/core';
constructor(@Optional() private logger: Logger) {
  if (this.logger) {
    this.logger.log(some_message);
  }
}
```

以上是Angular 依赖注入的基本用法。 但是依赖注入还远不止如此。需要学习更多，可以参考 [Hierarchial Dependency injection](https://angular.io/docs/ts/latest/guide/hierarchical-dependency-injection.html)
`