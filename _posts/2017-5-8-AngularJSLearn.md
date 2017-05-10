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

--TODO:

