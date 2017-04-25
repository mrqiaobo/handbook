---
layout: contents
title: AngularJs 4.0 学习
categories: [Web]
tages: [AngularJs, web, tech]
---
# AngularJs 学习（一）
> 本文翻译自[angularJS 官方文档](https://angular.io/docs/ts/latest/cli-quickstart.html)
## AngularJS的一个简单的小例子
AngularJs 由 component 组成， 每一个component由 HTML template 和 一个控制屏幕入口的component class 组成。

例如：

```javascript
import { Component } from '@angular/core';

@Component({
  selector: 'my-app',
  template: `<h1>Hello {{name}}</h1>`
})
export class AppComponent { name = 'Angular'; }
```
Component 由 @Component这个metadata 对象开始， metadata对象描述了如何将template 和class 组合起来工作的。
selector 属性告诉AungularJS component将在元素<my-app />中显示：
```html
<my-app>Loading AppComponent content here ...</my-app>
```
template 中的{{name}} 是AngularJS的插入式表达式(Interpolation exporession), 在运行时，AngularJS会用组件的name属性的值来替换掉{{name}}

## 使用命令行工具快速构建Angular项目 （Augular CLI tool）
Angular CLI 是一个命令行接口工具，可以创建项目，添加文件或者进行各种开发任务如测试，编译，部署等。
这是一个用Angular CLI编译和运行一个angular 应用的简单例子

1. 设置开发环境

安装nodeJS 和 npm
> 最低版本是node 6.9.x 以及 npm 3.x.x

然后在全局安装 Angular CLI
```CLI
npm install -g @angular/cli
```

2. 创建一个新的项目

打开终端，输入以下命令开创建一个项目的框架：
```CLI
ng new projectname
```

这可能需要一段时间（npm 需要根据配置文件来下载对应的依赖）

3. 启动这个项目

项目构建完毕后，进入这个项目的文件夹并启动这个项目：
```CLI
cd projectname
ng serve --open
```

`ng serve` 命令启动了一个服务来监控你的文件，如果文件有所改动则重新编译这些文件

使用--open 选项会自动打开 localhost:4200/

![](https://github.com/mrqiaobo/handbook/blob/gh-pages/assets/AngularyjsLearn/angular-serve.png?raw=true)

4. 开始你自己的第一个Component

CLI 已经为你创建了一个component,  ./src/app/app.component.ts
打开该文件，编辑自己的component:
```javascript
/* src/app/app.component.ts */
export class AppComponent {
  title = 'This is jacob\'s first component ';
}
```
以及样式文件：
```css
/* src/app/app.component.css */
h1 {
  color: #369;
  font-family: Arial, Helvetica, sans-serif;
  font-size: 250%;
}
```
此时观察浏览器，已经为你自动更新了展现效果：

![](https://github.com/mrqiaobo/handbook/blob/gh-pages/assets/AngularyjsLearn/first-component.png?raw=true)

## Angular 项目的文件架构说明

在上个项目中， Angular CLI 产生的项目结构里，所有的components, styles, images以及其他的项目文件都存储在src 文件夹下

> src 文件目录
```
.                                                                                                                                                                        
├── app                                                                                                                                                                  
│   ├── app.component.css                                                                                                                                                
│   ├── app.component.html                                                                                                                                               
│   ├── app.component.spec.ts                                                                                                                                            
│   ├── app.component.ts                                                                                                                                                 
│   └── app.module.ts                                                                                                                                                    
├── assets                                                                                                                                                               
├── environments                                                                                                                                                         
│   ├── environment.prod.ts                                                                                                                                              
│   └── environment.ts                                                                                                                                                   
├── favicon.ico                                                                                                                                                          
├── index.html                                                                                                                                                           
├── main.ts                                                                                                                                                              
├── polyfills.ts                                                                                                                                                         
├── styles.css                                                                                                                                                           
├── test.ts                                                                                                                                                              
├── tsconfig.app.json                                                                                                                                                    
├── tsconfig.spec.json                                                                                                                                                   
└── typings.d.ts   
```

| 文件 | 作用 |
| ---- | ---- |
| app/app.components.{ts,html.css,spec.ts} |  定义AppComponent ,包括 html, css样式和单元测试。 这是其他component的根 component|
| app/app.module.ts | 定义 AppModule, 这是一个根 Module, 它告诉Angular如何组合所有的Component |
| assets/* | 该文件夹可以放图片或者其他在部署的时候需要的文件。可以整个拷贝到部署环境 |
| environments/* | 该文件夹包含的文件，为每一个最终使用环境提供了一个配置文件, 这些文件会在环境运行时被替换 |
| favicon.icon | 网站的书签图标 |
| index.html | 项目的根网页，一般情况下不需要对它进行编辑。Angular CLI将所有的js和css引用加入了这个文件， 因此不需要再手动添加 |
| main.ts | app 的主要入口， 使用JIT compiler 对项目进行编译并启动AppModule 使项目在浏览器中运行， 如果想要使用自动编译功能，不需要改动code, 在使用命令 `ng build` 或  `ng serve` 时 传入 `--aot` 即可 |
| polyfills.ts | 不同的浏览器对web标准的支持不一样， Polyfills的作用是统一这些不同之处 |
| style.css | 全局css 在这里定义， 大多数情况下，需要在components中自定义local css， 但是影响全局的css 可以写在这里 |
| test.ts | 这是单元测试的主要入口 |
| tsconfig.{app 或 spec}.json | TypeScript编译器配置 for app(tsconfig.app.json) or for UT (tsconfig.spec.json)|

> 根目录
根目录下的其他文件有助于对项目的构建，测试，文档编写，维护以及部署
```
.                                                                                                                                                                        
├── README.md                                                                                                                                                            
├── e2e                                                                                                                                                                  
├── karma.conf.js                                                                                                                                                        
├── node_modules                                                                                                                                                         
├── package.json                                                                                                                                                         
├── protractor.conf.js                                                                                                                                                   
├── src                                                                                                                                                                  
├── tsconfig.json                                                                                                                                                        
└── tslint.json   
```
| 文件 | 作用 |
| ---- | ---- |
| e2e/ | e2e文件夹下防止端到端测试(end to end test), 这些测试不放在src下是因为端到端测试应该是和app 分离的， 只测试你的mainApp, 因此它有自己的 tsconfig.e2e.json配置文件 |
| node_modules/ | NodeJS 根据package.json 安装的依赖包放在此文件夹 |
| angular-cli.json | 配置Angular CLI， 这个文件可以设置默认值以及在项目启动的时候包含那些文件等等 |
| .editorconfig| 对编辑器的简单配置，保证所有使用你项目的人用编辑器打开的时候使用的是同样的配置 |
| .gitignore | git配置，确保自动产生的文件不会提交到git |
| karma.config.js | UT 的配置文件， 当使用ng test 的时候， 会根据该文件运行Karma test runner |
| package.json | npm 第三方依赖包配置列表， 可以在这里添加你自己的依赖 |
| protractor.conf.js | Protractor的端到端测试配置， 在使用 `ng e2e`时会用到 |
| README.md | 项目的基本文档，CLI预先添加了CLI命令信息。 使用该文档去维护项目，让其他人了解你的项目以及方便使用 |
| tsconfig.json | IDE 的 TypeScript编译器配置 |
| tslint.json | TSLint 和Codelyzer 的lint 配置， 当使用 `ng lint` 的时候， Linting 帮助你的code style 保持一致 |

以上是基本的Angular Js 使用 和项目的初始化。

