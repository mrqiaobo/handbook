---
layout: contents
title: AngularJs 4.0 学习 (七) --- 用户输入
categories: [Web]
tages: [AngularJs, web, tech]
---
# AngularJS 学习（六）---- 用户输入

用户输入会触发DOM事件。 我们通过事件绑定来监听这些事件并将这些变更的值更新到组件和模块中。

用户的一些动作，例如点击链接，点击按钮，输入文字等会触发DOM事件。 这片文档解释了如何通过Angular 事件
绑定的语法将这些事件绑定到组件事件处理器。

## 绑定用户输入事件

我们可以使用Angular事件绑定来响应任何的DOM事件。 许多的DOM事件是由用户输入触发的。通过绑定这些事件，提供
用户的输入内容。

通过使用小括号包括ODM事件名称，然后将模板表达式放在双引号内赋值给它来绑定DOM事件
如下面的例子所示：

```html
<button (click)="onClickMe()">Click me!</button>
```

等号左边的`(click)`指明绑定的目标是一个DOM的click事件， 等号右边双引号内的内容是一个模板表达式，它通过调用组件的
`onClickMe`方法来响应这个点击事件。

在写绑定的时候，一定要注意模板表达式的执行上下文。模板表达式中的标识符属于一个上下文环境对象，通常是控制这个模板的Angular组件。
下面的例子展示了一个单行的HTML，但是它属于一个更大的组件：

```javascript
@Component({
  selector: 'click-me',
  template: `
    <button (click)="onClickMe()">Click me!</button>
    {{clickMessage}}`
})
export class ClickMeComponent {
  clickMessage = '';

  onClickMe() {
    this.clickMessage = 'You are my hero!';
  }
}
```

当用户点击按钮的时候， Angular会调用`ClickMeComponent`中的`onClickMe`方法

## 从$event对象中获取用户输入

DOM事件装载了一些对组件有用的信息， 这里展示了如何在每个按键点击后将`keyup`事件绑定到input输入框

下面的例子监听了`keyup`事件，并将整个事件($event)传入到了组件的事件处理器中:

```javascript
template: `
  <input (keyup)="onKey($event)">
  <p>{{values}}</p>
`
```

当用户点击按键并抬起的时候，会触发`keyup`事件，这时，Angular会提供一个响应的DOM 事件的对象在$event对象中，$event对象将作为组件
中的`onKey()`方法的参数传入：

```javascript
export class KeyUpComponent_v1 {
  values = '';

  onKey(event: any) { // without type info
    this.values += event.target.value + ' | ';
  }
}
```

`$event`对象的属性根据DOM事件的类型而变化。例如，鼠标点击事件就包含不同的信息。

所有的标准DOM事件对象都包含一个`target`属性，它引用指向发起事件的那个元素。 在这个例子中，target指向`<input>`元素，`event.target.value`
返回它当前的内容。

上面的效果如下所示：

![](https://angular.io/resources/images/devguide/user-input/keyup1-anim.gif)

### 给$event制定类型

上面的例子将$event 转换成了`any`类型，这是一种简化写法。 这样没有属性可以显示事件对象的类型信息，这是一种愚蠢的写法。

下面的例子将方法重写为带类型的event:

```javascript
export class KeyUpComponent_v1 {
  values = '';

  onKey(event: KeyboardEvent) { // with type info
    this.values += (`<HTMLInputElement>`event.target).value + ' | ';
  }
}
```

> (注意，这里由于文章格式原因在HTMLInputElement使用了反引号，实际code 不应该有)

现在$event被指定为了 `KeyboardEvent`类型。 不是所有的元素都value属性，所有将其target转换为了一个input元素。
`OnKey`方法就更明确的表明了它期望从模板中传入什么样的事件。

### 传递$event可能不是明智的

给event对象制定类型暴露了整个DOMevent对象，组件对模板信息了解的太详细了。如果不知道HTML中的内容是无法很好的提取信息的
这样的行为打破了组件和模板分离的原则。 接下来会告诉你如何用模板引用来解决这个问题，

## 从模板中引用变量来获取用户输入

还有另外一种方式来获取用户输入的信息： 使用Angular模板引用变量。这些变量可以直接访问模板中的某一个元素。 通过使用`#`来声明这种变量
如下面的例子所示，模板引用了一个变量会送按键事件：

```javascript
@Component({
  selector: 'loop-back',
  template: `
    <input #box (keyup)="0">
    <p>{{box.value}}</p>
  `
})
export class LoopbackComponent { }
```

这个模板引用了一个`box`变量， 它在`<input>`元素中声明，引用了`<input>`元素自己，这段代码获取输入框的值并将其显示在`<p>`标签中。

这个模板是完全自封闭的，没有绑定到组件上。组件什么也没有做。

此时在输入框输入内容，结果如下：

![](https://angular.io/resources/images/devguide/user-input/keyup-loop-back-anim.gif)

> 注意，这种方式必须绑定一个事件才能用。 Angular只在app做出事件的异步回应后才会更新绑定。这个例子中将`keyup`事件绑定了一个数字0
这是最简短的模板表达式。尽管这个表达式没有什么实际意义，但是符合了Angular的需求，让Angular能够更新界面。

使用模板变量引用的方式比传递`$event`对象来获取输入更简单。 下面是使用这种方式对之前代码的重构：

```javascript
@Component({
  selector: 'key-up2',
  template: `
    <input #box (keyup)="onKey(box.value)">
    <p>{{values}}</p>
  `
})
export class KeyUpComponent_v2 {
  values = '';
  onKey(value: string) {
    this.values += value + ' | ';
  }
}
```

这段代码的好处是，组件从视图中获得的值很清晰，不再需要了解`$event`的结构和用法。

## Key事件的过滤(使用 `key.enter`事件)

`(keyup)`事件处理器监听了每一个按键事件。有时候需要用到*Enter*按键，因为它标志着用户停止输入。 有一种减少干扰的方式是检验每个 `$event.keyCode`，
只在输入*Enter*的时候才处理。

有一种简单的方法： 绑定Angular的静态事件 `keyup.enter`， 然后Angular就会只在输入*Enter*的时候才处理：

```javascript
@Component({
  selector: 'key-up3',
  template: `
    <input #box (keyup.enter)="onEnter(box.value)">
    <p>{{value}}</p>
  `
})
export class KeyUpComponent_v3 {
  value = '';
  onEnter(value: string) { this.value = value; }
}
```

效果如下：

![](https://angular.io/resources/images/devguide/user-input/keyup3-anim.gif)

### 聚焦事件

前面的例子中，当用户鼠标移动或者点击页面其他部分前没有输入*Enter*，当前的状态就会丢失。 为了解决这个问题，我们
需要监听Enter和blur事件：

```javascript
@Component({
  selector: 'key-up4',
  template: `
    <input #box
      (keyup.enter)="update(box.value)"
      (blur)="update(box.value)">

    <p>{{value}}</p>
  `
})
export class KeyUpComponent_v4 {
  value = '';
  update(value: string) { this.value = value; }
}
```

### 合并之前的代码

之前的教程讲过如何展示数据，这个文档演示了事件绑定，现在，把他们合在一起作为一个小程序来展示一列英雄
并加入新的英雄到这个别彪。 用户可以通过输入英雄的名称到输入框然后点击add来添加英雄。

![](https://angular.io/resources/images/devguide/user-input/little-tour-anim.gif)

下面是这个组件的代码：

```javascript
@Component({
  selector: 'little-tour',
  template: `
    <input #newHero
      (keyup.enter)="addHero(newHero.value)"
      (blur)="addHero(newHero.value); newHero.value='' ">

    <button (click)="addHero(newHero.value)">Add</button>

    <ul><li *ngFor="let hero of heroes">{{hero}}</li></ul>
  `
})
export class LittleTourComponent {
  heroes = ['Windstorm', 'Bombasto', 'Magneta', 'Tornado'];
  addHero(newHero: string) {
    if (newHero) {
      this.heroes.push(newHero);
    }
  }
}
```



