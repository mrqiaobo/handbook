---
layout: contents
title: AngularJs 4.0 学习 (八) --- 表单(Forms)
categories: [Web]
tages: [AngularJs, web, tech]
---
# AngularJs 4.0 学习 (八) --- 表单(Forms)

> 表单可以使得用户输入紧密，有效，强力。 Angular 表单可以使用一系列数据绑定的用户控件，追踪改变值，验证输入以及显示错误。
本文翻译自[官方文档](https://angular.io/docs/ts/latest/guide/forms.html)

表单是商业应用的重要部分。 我们使用表单来登录，提交一个帮助请求，下订单，预定航班，安排会议一定做无数的数据输入任务。

在开发一个表单的时候，通过工作流来建立一个高效的，有引导性的数据输入用户体验十分重要。

开发一个表单需要设计能力，以及所使用的框架要支持数据拜大年过，数据追踪，验证，错误处理等。这一篇文档会告诉你这些。

本篇文档会教你如何建立一个简单的表单应用。在途中我们会学到：

* 使用组件和模板建立一个Angular表单
* 使用 `ngModel`创建一个双向绑定，用于读写输入控件的值
* 追踪改变数据，验证控件的输入
* 通过使用特殊的CSS类来追踪控件的状态，提供一个虚拟的回馈
* 展示验证错误信息以及禁用或启用控件
* 通过模板引用变量来共享HTML元素的信息


## 模板驱动的表单

我们可以通过在模板中使用本文档所描述的技巧和表单相关的指令以及模板语法来建立一个表单。

> 也可以通过使用反应式（或者模型驱动）的方法建立表单。本文档值专注于模板驱动的表单

我们可以用Angular模板建立任意的表单： 登录表单，联系表单以及很多的商业表单。你可以根据你的创造力来累出控件，然后将他们绑定到数据，然后验证规则业绩展示验证错误信息，根据条件禁用和激活控件，根据条件出发虚拟反馈等等。

相比你自己写，Anlugar处理一些重复的样板的任务会更加容易。

最终的例子会看起来如下所示：

![](https://angular.io/resources/images/devguide/forms/hero-form-1.png)

*英雄雇佣机构* 使用这个表单来会务英雄的个人信息。 每一个英雄都需要有一个工作。 这是一个让英雄去处理相应的危机为任务的公司。

这个表单中的三个字段中有两个是必填的。必填字段的左边会有一个绿色的边，以方便标识。

如果你删除了英雄的姓名，这个表单就会显示出一个验证错误信息，这个信息使用可以引起你注意的样式:

![](https://angular.io/resources/images/devguide/forms/hero-form-2.png)

注意，提交按钮是禁用的，并且此时必填字段的左边由绿色变成了红色。

> 你可以在CSS中自定义必填字段的左边的颜色

以下是建立这个表单的步骤：

1. 创建一个`Hero`模型类
2. 创建一个组件来控制表单
3. 创建一个模板来初始化表单的布局
4. 使用`ngModel`双向数据绑定方法来讲数据绑定到每个表单控件上
5. 给每一个表单输入控件添加 `name`属性
6. 通过添加CSS来提供虚拟反馈
7. 显示和隐藏雁阵错误信息
8. 通过使用`ngSubmit`来处理提交任务
9. 除非表单是验证合法的，否则禁用Submit按钮

项目的建立，参考[Setup](https://angular.io/docs/ts/latest/guide/setup.html)

### 创建Hero 模型类

当用户输入表单数据的时候，我们需要使用一个模型的实例来捕捉数据的变化，在我们了解模型是什么样子之前，无法画出这个表单。

一个模型其实就和一个属性包一样简单，这个属性报应该包含应用的重要的信息。 所以，Hero 类就包含了三个必填字段（`id`, `name`, `power`）以及一个选填字段`alterEgo`

在`app`文件夹下，创建一个文件并写下如下代码：

```javascript
export class Hero {
  constructor(
    public id: number,
    public name: string,
    public power: string,
    public alterEgo?: string
  ) {  }
}
```

这是一个简单的模型，仅仅有几个必要的字段，没有任何行为，这对demo来说是完美的。

TypeScript编译器为每一个 `public`构造器参数自动生成了一个公共字段，并在创建的时候给他们赋值。

`alterEgo`是可选的，所以构造器允许你省略它，注意`alterEgo?`的问号

你可以创建一个英雄，如下所示：

```javascript
let myHero =  new Hero(42, 'SkyDog',
                       'Fetch any object at any distance',
                       'Leslie Rollover');
console.log('My hero is called ' + myHero.name); // "My hero is called SkyDog"
```

### 创建一个表单组件

一个Angular 表单有两部分： 一个基于HTML的模板和一个处理数据和用户交互的组件。

从这个类开始因为它简要生命了这个永雄编辑器的作用。 根据下面的代码创建组件：

```javascript
import { Component } from '@angular/core';

import { Hero }    from './hero';

@Component({
  selector: 'hero-form',
  templateUrl: './hero-form.component.html'
})
export class HeroFormComponent {

  powers = ['Really Smart', 'Super Flexible',
            'Super Hot', 'Weather Changer'];

  model = new Hero(18, 'Dr IQ', this.powers[0], 'Chuck Overstreet');

  submitted = false;

  onSubmit() { this.submitted = true; }

  // TODO: Remove this when we're done
  get diagnostic() { return JSON.stringify(this.model); }
}es
```

这个组件和你之间缩写的任何组件没有任何区别。

理解这个组件仅仅需要之前你所学习的概念即可。

* 这段代码导入了Angular core类库以及你创建的`Hero`模型
* `@Component` 的选择器`hero-form`意味着你可以在模板中加入`<hero-form>`来讲组件插入
* `templateUrl`属性指定了一个独立文件作为模板文件
* 给`model`和`powers`定义了虚拟的数据，作为demo使用。

接下蓝，我们可以插入是个数据服务来获取和存储真正的数据，或者将属性暴露出来作为输入和输出到父组件。现在这些还没有关系，并且这些功能的改变不会影响到表单。

* 通过添加`diagostic`属性返回一个模型的JSON值可以使我们看到在开发的过程中发生了什么，我们会在后面添加一个注释提醒自己最后要删除它。

#### 为什么将模板独立出来

为什么不直接在组件文件中直接写模板呢？ 其实没有正确的方式， 行内模板在它十分端的情况下是十分方便的。 但是大多数的模板是很长的。 TypeScript 以及JavaScript 文件通常不是写入和放置长HTML语法的地方，并且很少的编辑器能够处理有HTML语法混合的代码。

表单模板趋向于大文件，尽管展示的字段很少。 所以通常最好将HTML模板作为一个单独的文件。 过一会我们才会写。 首先，我们修改`app.modue.ts`和`app.component.ts`来使用新的组件`HeroFormComponent`.

### 使用*app.module.ts*

`app.module.ts`定义了应用的跟模块。在里面我们定义了我们要在应用中使用的模块已经声明本模块的组件，如`HeroFormComponent`。

由于模板驱动的表单是属于自己的模块，所以我们需要添加`FormsModule`到`imports`数组，这样可以使用表单。

将文件改成如下所示：

```javascript
import { NgModule }      from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { FormsModule }   from '@angular/forms';
import { AppComponent }  from './app.component';
import { HeroFormComponent } from './hero-form.component';
@NgModule({
  imports: [
    BrowserModule,
    FormsModule
  ],
  declarations: [
    AppComponent,
    HeroFormComponent
  ],
  bootstrap: [ AppComponent ]
})
export class AppModule { }
```

>  这里有三点变化：
1. 导入了 `FormsModule`和新的组件`HeroFormComponent`.
2. 将`FormsModule`添加到了`ngModule`中定义的`imports`列表中。这样应用可以访问所有的模板驱动的表单功能，包括`ngModel`。
3. 添加`HeroFormComponent`到`ngModel`声明的`declarations`列表中，这使得`HeroComponent`组件可以在本模块中可见。

如果一个组件，指令，管道指令输入一个`imports`中的模块，则不要在`declarations`重复声明。 如果是你自己写的，应该属于本模块，则需要在`declarations`中声明。

### 使用*app.component.ts*

`AppComponent`是应用的根组件，它会包含新的组件`HeroFormComponent`，
将其改成下面代码：

```javascript
import { Component } from '@angular/core';
@Component({
  selector: 'my-app',
  template: '<hero-form></hero-form>'
})
export class AppComponent { }
```

### 创建一个初始化的HTML表单模板

创建一个模板文件如下所示：

```html
<div class="container">
    <h1>Hero Form</h1>
    <form>
      <div class="form-group">
        <label for="name">Name</label>
        <input type="text" class="form-control" id="name" required>
      </div>
      <div class="form-group">
        <label for="alterEgo">Alter Ego</label>
        <input type="text" class="form-control" id="alterEgo">
      </div>
      <button type="submit" class="btn btn-success">Submit</button>
    </form>
</div>
```

这是简单的HTML5， 将两个字段`name`和`alterEgo`放入，并对输入框开放。 Name的输入框有一个H5 的required属性而alterEgo没有，因为是可选的。 添加Submit按钮。这个文件仅仅是一个简单的布局。

> 在模板驱动的表单中，如果你导入了`FormsModule`，则不需要对`<form>`做任何事情，确保使用`FormsModule`.

`container`, `form-group`, `form-control`以及`btn`都是Bootstrp中的类。要加载这些，你需要在index.html中的`<head>`中加入下面的链接：

```html
<link rel="stylesheet"
      href="https://unpkg.com/bootstrap@3.3.7/dist/css/bootstrap.min.css">
```

### 使用 **ngFor* 添加powers

每个英雄必须从固定机构批准的的列表里选择一个超能力，我们在内部管理这个列表(在`HeroFormComponent`中)

在表单中添加一个`select`来使用`ngFor`绑定`power`, `ngFor` 可参考[数据绑定](https://angular.io/docs/ts/latest/guide/displaying-data.html)

在Alter Ego 组下添加下面的代码：

```html
<div class="form-group">
  <label for="power">Hero Power</label>
  <select class="form-control" id="power" required>
    <option *ngFor="let pow of powers" [value]="pow">{{pow}}</option>
  </select>
</div>
```

这段代码给列表中的每一个power都添加了一个`<option>`标签。 `pow`模板输入变量在每次迭代中都是不同的power， 通过嵌入式绑定来展示名字。

### 使用*ngModel*的双向绑定

现在运行app你会看到：

![](https://angular.io/resources/images/devguide/forms/hero-form-3.png)

你没有看到任何英雄，因为你没有绑定它。在之前的文档中你已经知道如何进行绑定了。
现在你需要在同一时间进行数据展示和监听。尽管你可以用之前的知识来做，但是在这里最好使用新技能：`[(ngModel)]` , 他可以使表单绑定更加方便。

找到Name`<input>`标签然后修改成如下：

```html
<input type="text" class="form-control" id="name"
       required
       [(ngModel)]="model.name" name="name">
TODO: remove this: {{model.name}}
```

注意绑定语法`[(ngModel)]="..."`, 我们需要在添加一些东西来展示数据。为表单声明一个模板变量。更新`<form>`标签，添加`#heroForm="ngForm"` 如下所示：

```html
<form #heroForm="ngForm">
```

变量 `heroForm`现在是`NgForm`的指令的一个引用。 它管理整个表单。

> **NgForm指令**
Angular帮我们添加了一个NgForm。它自动向`<form>`标签添加了这个指令。 `NgForm`指令为from元素补充了一些功能。它通过`ngModel`指令和`name`属性来管理和控制你创建的控件并监听他们的属性，包括验证， 它有一个`valid`属性，当所所有包含的控件都是合法时，为true.

现在如果运行app并在Name输入框中输入，添加或者删除字符，你会看见他们出现或者在文字中。 如下图所示：

![](https://angular.io/resources/images/devguide/forms/ng-model-in-action.png)

diagnostic是数据在输入框和模型中流动的证据。

> 双向绑定，详情见 [模板语法：双向绑定部分](https://angular.io/docs/ts/latest/guide/template-syntax.html#ngModel)

注意，你添加了一个`name`属性到`<input>`标签，并将其值设置为"name"这会对英雄的name属性产生影响、 任何单独的值都可以，但是使用有描述性的名字会更好。 当使用`[(ngModel)]`来和from联合的时候定义一个`name`属性是必要的。

> Angular会在内部隐式地创建`FormControl`实例并将他们通过`NgForm`指令注册到`<form>`标签上。 每一个`formControl`都注册在你赋值给`name`属性的名称上。 更多请参考[NgForm指令](https://angular.io/docs/ts/latest/guide/forms.html#ngForm)

给Alter Ego 和Hero Power的`name`属性半丁一个`[(ngModel)]`. 我们UI开始将input半丁信息，并添加新的绑定到`diagnostic`属性。 然后我们可以确定对于`hero`模板的整个双向绑定都是成功的。

```html
{{diagnostic}}
<div class="form-group">
  <label for="name">Name</label>
  <input type="text" class="form-control" id="name"
         required
         [(ngModel)]="model.name" name="name">
</div>

<div class="form-group">
  <label for="alterEgo">Alter Ego</label>
  <input type="text"  class="form-control" id="alterEgo"
         [(ngModel)]="model.alterEgo" name="alterEgo">
</div>

<div class="form-group">
  <label for="power">Hero Power</label>
  <select class="form-control"  id="power"
          required
          [(ngModel)]="model.power" name="power">
    <option *ngFor="let pow of powers" [value]="pow">{{pow}}</option>
  </select>
</div>
```

> * 每一个input元素的id属性是用来和label元素的for属性来匹配的
* 每个input元素的name属性是Angular表单注册控件所需要的

现在运行app会看到以下结果：

![](https://angular.io/resources/images/devguide/forms/ng-model-in-action-2.png)

顶部的diagnostic 会确定你所有的改变都印象了整个模型。达到目的后，我们可以删除`{{diagnostic}}`绑定。

### 使用*ngModel*追踪控件状态以及验证

在表单中使用`ngModel`不仅仅可以进行双向绑定。还告诉你用户是否触摸了控件，是否值改变了，以及是否值变成了非法的。

*ngModel* 指令不仅仅追踪状态，也可以用特殊的Angular CSS类来更新控件。你可以利用下面的这些类名来改变控件的表现形式：

| State | Class if true | Class if false |
| --- | --- | --- |
| 控件被访问过 | ng-touched  | ng-untouched |
| 控件值改变 | ng-dirty | ng-pristine |
| 控件值是合法的 | ng-valid | ng-invalid |

可以暂时给input添加一个模板引用变量spy 来显示当前input的CSS class

```html
<input type="text" class="form-control" id="name"
  required
  [(ngModel)]="model.name" name="name"
  #spy>
<br>TODO: remove this: {{spy.className}}
```

现在运行app并观察Name输入框， 按照以下步骤操作：
1. 看一下但不要触摸
2. 点击name输入框， 然后点击外面
3. 在name后面添加一些斜杠
4. 删除这些name

效果如下所示：

![](https://angular.io/resources/images/devguide/forms/control-state-transitions-anim.gif)

![](https://angular.io/resources/images/devguide/forms/ng-control-class-changes.png)

`ng-valid`/`ng-invalid`对 是最有趣的，因为当数据是非法的时候我们想有一个强烈的视觉效果， 我们想要标记这些字段。 若要达到这种效果，我们需要使用`ng-*`CSS类

现在删除#spy 因为它已经没用了。

## 添加自定义CSS进行虚拟反馈

我们通过在输入框的左边添加有颜色的边框来标记必填字段和非法数据：

![](https://angular.io/resources/images/devguide/forms/validity-required-indicator.png)

通过创建一个新的文件`forms.css`定义这些css类然后在index.html中引用:

```css
.ng-valid[required], .ng-valid.required  {
  border-left: 5px solid #42A948; /* green */
}
.ng-invalid:not(form)  {
  border-left: 5px solid #a94442; /* red */
}
```

```html
<link rel="stylesheet" href="styles.css">
<link rel="stylesheet" href="forms.css">
```

## 显示和隐藏验证错误信息

我们继续更新表单。 现在Name输入框是必填，清楚后边框会变为红色。这表示有错误， 但是用户不知道是什么错误也不知道要怎么办。 让我们根据控件的行为添加一些有用的信息。 当用户删除了名字，表单如下所示：

![](https://angular.io/resources/images/devguide/forms/name-required-error.png)

要实现这种鲜果， 扩展`<input>`标签：
* 添加一个模板引用变量
* “必填”的信息在`<div>`旁边，当控件非法时显示

下面是例子：

```html
    <label for="name">Name</label>
    <input type="text" class="form-control" id="name"
            required
            [(ngModel)]="model.name" name="name"
            #name="ngModel">
    <div [hidden]="name.valid || name.pristine"
            class="alert alert-danger">
        Name is required
    </div>
```

我们西药一个模板引用变量来获从模板中取输入框的Angular 控件。 这里创建了一个叫name的变量并复制为ngModel

> 为什么是"ngModel"? 一个指令的exportAs属性告诉Angular如何将变量连接到指令。 设置name为ngModel因为 ngModel 指令的 exportAs属性在ngModel中出现了。

我们通过绑定name属性到`<div>`的hidden属性来控制错误信息的显示：

```html
<div [hidden]="name.valid || name.pristine"
     class="alert alert-danger">
```

这个例子里，当控件是合法的时候或者初始化的时候，隐藏消息。

这种用户体验是根据开发者的选择开定的。有一些开发者希望消息一直显示。如果你不在乎初始化状态`pristine`, 你可以只在value是合法的时候隐藏消息。

有些开发者希望只在用户做了非法改变的时候显示消息。 那么在初始状态隐藏消息就可以了。

Hero的Alter Ego是可选的所以你不用管它。

Hero的Power selection是不虚的。 如果你想的话，添加同样的错误处理到`<select>`，但是没有必要，因为选择框已经包含了合法的值。

现在我们会添加一个新的英雄到这个表单。 将New Hero 按钮放在表单底部，然后将他的点击事件和newHero的方法绑定

```html
<button type="button" class="btn btn-default" (click)="newHero()">New Hero</button>
```

```javascript
newHero() {
    this.model = new Hero(42, '', '');
}
```

再次运行app, 点击New Hero按钮，然后表单清空了， 必填字段的控件左边变为红色，标识nae和power是不合法的。 这可以理解，因为他们是必填字段。错误信息隐藏了因为现在表单初始化了。你没有做任何改变。

填入一个姓名然后再次点击按钮。 app显示了*Name is required*的错误信息。 我们并无希望在创建一个新的英雄时显示错误信息，为何现在有一个错误消息呢？

通过浏览器工具检查来看，name输入框已经不是初始状态了。 表单记录了你曾经在点击new hero按钮前输入过姓名。 替换当前的hero 对象并没有使表单的控件恢复的初始状态。

所以我们要在调用`newHero()`方法后使用`rerset()`来清楚所有的标签。

```html
<button type="button" class="btn btn-default" (click)="newHero(); heroForm.reset()">New Hero</button>
```

现在，再次点击，发现错误消失了。

## 使用*ngSubmit*提交表单

用户在填完表单后应该能够提交它， 现在表单底部的Submit按钮没有做任何事， 但是大会提交表单，因为他的type: `(type="submit")`.

现在表单提交是无用的。为了使它有用，将表单的`ngSubmit`时间属性绑定到hero表单组件的`onSubmit()`方法：

```html
<form (ngSubmit)="onSubmit()" #heroForm="ngForm">
```

我们已经定义过一个模板引用变量`#heroForm`， 并且用`ngForm`初始化了它。现在使用这个变量来访问提交按钮。

通过事件绑定，我们可以讲所有表单中的验证通过`heroForm`的变量和按钮的`disable`属性绑定起来:

```html
<button type="submit" class="btn btn-success" [disabled]="!heroForm.form.valid">Submit</button>
```

现在运行app, 可以发现按钮是激活的，尽管现在没有做任何事情。

现在如果删除名字，你就破坏了必填的规则，会触发错误消息，Submit按钮也会变为禁用。

印象不深？ 那么想一下，如果没有Angular的帮助，你会如何实现按钮的enable/disable呢？

对于我们来说，可能是这样：
1. 在表单元素上定义一个模板引用变量
2. 在按钮的实践中引用这个变量很多次

## 栓紧两个表单区域（额外知识）

现在提交表单不是最有意思的。 微课强化虚拟效果，隐藏输入区域并展示其他东西。

将表单包括在一个`<div>`内并将其hidden属性绑定到`HeroFromComponent.submitted`属性。

```html
  <div [hidden]="submitted">
    <h1>Hero Form</h1>
    <form (ngSubmit)="onSubmit()" #heroForm="ngForm">

       <!-- ... all of the form ... -->

    </form>
  </div>
```

闲杂主要的表单是可见的表单，因为submitted属性是false除非提交表单。 `HeroFormComponent`的内容如下：

```javascript
submitted = false;

onSubmit() { this.submitted = true; }
```

当点击提交按钮， submitted标识变为true，然后表单如期消失了。

现在app需要在表单提交后显示一些其他的东西。 添加下面的HTML：

```html
<div [hidden]="!submitted">
  <h2>You submitted the following:</h2>
  <div class="row">
    <div class="col-xs-3">Name</div>
    <div class="col-xs-9  pull-left">{{ model.name }}</div>
  </div>
  <div class="row">
    <div class="col-xs-3">Alter Ego</div>
    <div class="col-xs-9 pull-left">{{ model.alterEgo }}</div>
  </div>
  <div class="row">
    <div class="col-xs-3">Power</div>
    <div class="col-xs-9 pull-left">{{ model.power }}</div>
  </div>
  <br>
  <button class="btn btn-primary" (click)="submitted=false">Edit</button>
</div>
```

这是另外一个英雄了，展示了只读的嵌入式绑定。`<div>`只在提交后出现。 这里有一个Edit 按钮绑定了一个清楚submitted标识的表达式。

当点击Edit按钮， 这个界面小时，可编辑的表单又回来了。

## 总结

本篇文档中使用 Angular表单做了一下功能：提供数据修改，验证以及其他的支持：

* 一个Angular HTML表单模板
* 一个带`@Component`修饰符的表单组件类
* 通过绑定`NgForm.ngSubmit`事件属性来处理表单提交
* 模板引用变量，如`#heroForm`, `#name`
* `[(ngModel)]`语法的双向绑定
* `name`属性验证的用法以及表单元素追踪
* 输入控件的引用变量valid属性检查控件是否合法并控制错误消息的现实和隐藏
* 通过绑定`NgForm`控制提交按钮的激活状态。
* 自定义css 类来提供虚拟回馈

最终的文件结构如下所示：

```
angular-forms
|  -  src
|      | -  app
|            | -  app.component.ts
|            | -  app.module.ts
|            | -  hero.ts
|            | -  hero-form.component.html
|            | -  hero-form.component.ts
|  -  main.ts
|  -  tsconfig.json
|  -  index.html
node_modules ...
package.json
```

最终版本的代码如下所示：

```javascript
// hero-form.component.ts
import { Component } from '@angular/core';
import { Hero }    from './hero';
@Component({
  selector: 'hero-form',
  templateUrl: './hero-form.component.html'
})
export class HeroFormComponent {
  powers = ['Really Smart', 'Super Flexible',
            'Super Hot', 'Weather Changer'];
  model = new Hero(18, 'Dr IQ', this.powers[0], 'Chuck Overstreet');
  submitted = false;
  onSubmit() { this.submitted = true; }
  newHero() {
    this.model = new Hero(42, '', '');
  }
}
```


```html
<!-- hero-form.component.html -->
<div class="container">
  <div [hidden]="submitted">
    <h1>Hero Form</h1>
    <form (ngSubmit)="onSubmit()" #heroForm="ngForm">
      <div class="form-group">
        <label for="name">Name</label>
        <input type="text" class="form-control" id="name"
               required
               [(ngModel)]="model.name" name="name"
               #name="ngModel">
        <div [hidden]="name.valid || name.pristine"
             class="alert alert-danger">
          Name is required
        </div>
      </div>
      <div class="form-group">
        <label for="alterEgo">Alter Ego</label>
        <input type="text" class="form-control" id="alterEgo"
               [(ngModel)]="model.alterEgo" name="alterEgo">
      </div>
      <div class="form-group">
        <label for="power">Hero Power</label>
        <select class="form-control" id="power"
                required
                [(ngModel)]="model.power" name="power"
                #power="ngModel">
          <option *ngFor="let pow of powers" [value]="pow">{{pow}}</option>
        </select>
        <div [hidden]="power.valid || power.pristine" class="alert alert-danger">
          Power is required
        </div>
      </div>
      <button type="submit" class="btn btn-success" [disabled]="!heroForm.form.valid">Submit</button>
      <button type="button" class="btn btn-default" (click)="newHero(); heroForm.reset()">New Hero</button>
    </form>
  </div>
  <div [hidden]="!submitted">
    <h2>You submitted the following:</h2>
    <div class="row">
      <div class="col-xs-3">Name</div>
      <div class="col-xs-9  pull-left">{{ model.name }}</div>
    </div>
    <div class="row">
      <div class="col-xs-3">Alter Ego</div>
      <div class="col-xs-9 pull-left">{{ model.alterEgo }}</div>
    </div>
    <div class="row">
      <div class="col-xs-3">Power</div>
      <div class="col-xs-9 pull-left">{{ model.power }}</div>
    </div>
    <br>
    <button class="btn btn-primary" (click)="submitted=false">Edit</button>
  </div>
</div>
```

```javascript
// hero.ts
export class Hero {
  constructor(
    public id: number,
    public name: string,
    public power: string,
    public alterEgo?: string
  ) {  }
}
```

```javascript
// app.module.ts
import { NgModule }      from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { FormsModule }   from '@angular/forms';
import { AppComponent }  from './app.component';
import { HeroFormComponent } from './hero-form.component';
@NgModule({
  imports: [
    BrowserModule,
    FormsModule
  ],
  declarations: [
    AppComponent,
    HeroFormComponent
  ],
  bootstrap: [ AppComponent ]
})
export class AppModule { }
```

```javascript
// app.component.ts
import { Component } from '@angular/core';
@Component({
  selector: 'my-app',
  template: '<hero-form></hero-form>'
})
export class AppComponent { }
```

```javascript
// main.ts
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';
import { AppModule } from './app/app.module';
platformBrowserDynamic().bootstrapModule(AppModule);
```

```html
<!--index.html-->
<html>
  <head>
    <title>Hero Form</title>
    <base href="/">
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet"
          href="https://unpkg.com/bootstrap@3.3.7/dist/css/bootstrap.min.css">
    <link rel="stylesheet" href="styles.css">
    <link rel="stylesheet" href="forms.css">
    <!-- Polyfills -->
    <script src="node_modules/core-js/client/shim.min.js"></script>
    <script src="node_modules/zone.js/dist/zone.js"></script>
    <script src="node_modules/systemjs/dist/system.src.js"></script>
    <script src="systemjs.config.js"></script>
    <script>
      System.import('main.js').catch(function(err){ console.error(err); });
    </script>
  </head>
  <body>
    <my-app>Loading...</my-app>
  </body>
</html>
```

```css
/*forms.css*/
.ng-valid[required], .ng-valid.required  {
  border-left: 5px solid #42A948; /* green */
}
.ng-invalid:not(form)  {
  border-left: 5px solid #a94442; /* red */
}
```