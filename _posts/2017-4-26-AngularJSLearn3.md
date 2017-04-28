---
layout: contents
title: AngularJs 4.0 学习 (三)
categories: [Web]
tages: [AngularJs, web, tech]
---
# AngularJS 学习（三）
> 本文翻译自[angularJS 官方文档](https://angular.io/docs/ts/latest/tutorial/toh-pt1.html)

### 双向绑定

当用户需要通过`<input>` 来对 `hero。name` 进行编辑的时候，我们就需要用到双向绑定了

首先，把template文件改成如下，或者新建一个template：

```html
<div>
  <label>name: </label>
  <input [(ngModel)]="myHero.name" placeholder="name">
  <p>Hero name is: {{myHero.name}}</p>
</div>
```

上述代码中 `[(ngModel)]` 是Angular语法，用来将hero.name 绑定到对应的textbox上， 这时数据是双向绑定的，可以从textbox修改hero.name, 也可以修改 hero.name来影响textbox的显示
尽管`[(ngModel)]` 是符合Angular语法的，但是想要其工作，必须导入FormsModel， 因为它是属于formsModel的。

### 导入FormsModule
打开 app.module.ts 文件，将FormsModule导入：
```javascript
import { NgModule }      from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { FormsModule }   from '@angular/forms'; // <-- NgModel lives here
import { AppComponent }  from './app.component';
@NgModule({
  imports: [
    BrowserModule,
    FormsModule // <-- import the FormsModule before binding with [(ngModel)]
  ],
  declarations: [
    AppComponent
  ],
  bootstrap: [ AppComponent ]
})
export class AppModule { }
```

然后你就可以对输入框进行编辑，查看效果：

![editHero]({{site.asseturl}}/AngularyjsLearn/edit-hero.png?raw=true)


### 加入样式

通过在Component 中的@Component 定义中使用styles 可以制定css 样式：

```javascript
/*app.component.ts*/
@Component({
  selector: 'my-app',
  templateUrl: './template/mainTemplate.html',
  styles: [`
    .selected {
      background-color: #CFD8DC !important;
      color: white;
    }
    .heroes {
      margin: 0 0 2em 0;
      list-style-type: none;
      padding: 0;
      width: 15em;
    }
  `]
})

```

如果想要引入css文件，可以使用styleUrls, 注意， styles 和 styleUrls 都是字符串数组类型的对象。

```javascript
/*app.component.ts*/
@Component({
  selector: 'my-app',
  templateUrl: './template/mainTemplate.html',
  styleUrls: ['./style/style.css'],
})
```

当你加入这些style 的时候，只会在当前Component 作用域下起作用，而不会影响到其他地方的HTML， 这有效解决了css 命名冲突的问题。

