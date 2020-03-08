---
layout: post
title: "Angular Module 簡介筆記"
description: "Angular Module 簡介筆記"
crawlertitle: "Angular Module 簡介筆記"
date: 2019-07-31 23:50:46 +0800
categories: Angular
tags: Angular
comments: true
---

## Modules

Angular 的 Module 是透過 `@NgModule` 這個裝飾器 decorator 宣告的類別，`NgModule` 是 Angular 應用程式最基本的組成元素，有點像是類別(class)，把相互關聯的其他[元件(Components)]({{ site.baseurl }}{% link _posts/2019-08-4-Angular_component_basic_note.md %})、服務(Services)封裝、打包起來，成為一內聚的功能塊，指定宣告哪些編譯過的內容提供給外部使用。
每個 Angular app 最少都有一個根模組 root module 用來啟動此應用程式，按照慣例命名為 `AppModule`，除此之外通常還會包含有其他不同功能的模組。

透過 [Angular CLI](https://angular.tw/cli) 新建專案後，產生的檔案裡面就包含有最基本的模組 `app.module.ts`：
![new_angular_app_file_tree](https://i.imgur.com/LFQinwo.jpg)

`my-app/src/app/app.module.ts`

```javascript
// javascript 層級的 import
import { BrowserModule } from "@angular/platform-browser";
import { NgModule } from "@angular/core";

import { AppRoutingModule } from "./app-routing.module";
import { AppComponent } from "./app.component";
import { FooComponent } from "./foo/foo.component";
import { FooService } from "./foo.service";

// @NgModule 裝飾器 和其 metadata
@NgModule({
  // 宣告哪些是屬於此模組的內部元件，供 Angular 編譯用
  declarations: [AppComponent],
  // NgModule 的 import，引入其他的模組功能
  imports: [BrowserModule, AppRoutingModule],
  // exports: 列表決定哪些功能公開給外部使用
  exports: [],
  // providers: 注入 FooService 給旗下元件使用
  providers: [FooService],
  // bootstrap: 定義 root component
  bootstrap: [AppComponent]
})
export class AppModule {}
```

### @NgModule 參數

- `declarations: []`  

  一個宣示主權的動作：宣告和 View 相關的 component 及這個 module 麾下封裝的其他 component, directives 和 pipes。Module 用得到的東西都必須定義在這裏。
  每個元件都應該，且只能被宣告 declare 在一個 NgModule 之中，如果使用了未經宣告的元件就會跳錯誤，如果要用到該元件的功能，就必須匯入 import 包含他的模組。
  例如：

  ```javascript
  //app.module.ts

  import { FooDirective } from './Foo.directive';

  // ...省略
  // 新增到 @NgModule 的 declarations 陣列中：
  declarations: [
    AppComponent,
    FooDirective
  ],
  ```

- import/export  
  就像 JavaScript 的 module 一樣，可以 import / export 其他 NgModules 的功能。
  `imports: []` 的內容表示我們要匯入的其他 module 取用他們之中的函式、屬性等等。
  `exports: []` 決定哪些是要公開給外部使用的。
- `providers: []`  

  宣告在這個 module 層級注入的 Services，供 module 底下的組成元件使用。
- `bootstrap: []`  

  根元件，Angular 建立`<app-root>`標籤並置入 index.html 頁面。

## 組織你的程式碼

按照業務邏輯、工作流組織你的程式碼，將各種特性盡可能地規劃出明確的區別，在其中透過 import/export 作為轉接各種應用模組的出入口，整合為有清晰區別的模組功能區，可以幫助管理日益複雜的應用程式、增加複用性和可維護性，如此還有個好處：
透過 Angular 的非同步惰性載入 `lazy-loading` 功能，只在需要的時候載入相關的功能模組，最小化初始載入的程式碼。

## 參考
[Angular.io](https://angular.io/guide/ngmodules)
