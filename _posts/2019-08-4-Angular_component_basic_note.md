---
layout: post
title: "Angular Component 簡介筆記"
description: "Angular Component 簡介筆記"
crawlertitle: "Angular Component 簡介筆記"
date: 2019-08-04 23:50:46 +0800
categories: Angular
tags: ["Angular"]
comments: true
---

- toc
{:toc}

## 顯示畫面
元件(Component)定義了畫面, 是一組 Angular 可以根據屬性(propertie)、方法(method)和資料內容去操作的畫面中的元素。或也可說是畫面範圍的區塊，小從一個按鈕元件、大到整個畫面都能劃分到一個元件之中。

根據在元件類(export class)之中定義的邏輯、屬性和 API 來支援顯示端 View 的作動，在使用上可以根據複用性 reuseable 作考量，當我們發現一個按鈕、表單或者功能可以抽象提取出來提供其他地方使用，即是劃分元件的時機。

在應用程式中切換穿行時，Angular 就會建立、更新、銷毀一些元件，我們的應用程式可以透過 Lifecycle hooks 選擇在這些操作生命週期裡面特定的時機觸發動作。

- Components 使用各種 *services*, services 提供特定的功能性，其可以作為關聯性被注入 injected 到元件中，讓程式碼模組化、可複用並且更有效率。

## Component 元件的組成結構

- Component class  
  掌管了資料和功能控制。
- HTML template  
  決定要呈現的畫面內容。
- Component-specific styles  
  要取用的樣式表。

元件結構大概會看起來像這樣，以下範例取自[官網](https://angular.tw/guide/architecture-components)教程：  


In `src/app/heroes.component.ts`

```javascript
import { Component, OnInit } from '@angular/core';
// 引入的相關資料
import { Hero } from '../hero';
// 引入 HeroService service 功能
import { HeroService } from '../hero.service';

@Component({
  // selector 決定了在 html 中的標籤 <app-heroes></app-heroes>
  selector: 'app-heroes',
  // 此元件欲渲染的畫面
  templateUrl: './heroes.component.html',
  // 此元件畫面取用的樣式表
  styleUrls: ['./heroes.component.css']
})
export class HeroesComponent implements OnInit {
  heroes: Hero[];

  constructor(private heroService: HeroService) { }

  // Lifecycle hooks 決定在不同生命週期要執行的動作
  // ngOnInit() 在此元件初始化的時候呼叫 getHeroes() 方法
  ngOnInit() {
    this.getHeroes();
  }

  // 定義類方法取用了另一個位於 heroService 裡面的 getHeroes() 方法
  getHeroes(): void {
    this.heroService.getHeroes()
    // 用到非同步的概念訂閱了 heroService.getHeroes() 的回傳內容。
    .subscribe(heroes => this.heroes = heroes);
  }
  //...省略
}
```

Component 和 Service 都是單純的 class，只是配合不同的裝飾器(decorator)定義類型並提供 metadata。
- `Component` class 中的 metadata 將之與決定畫面呈現的 `template` 關聯在一起。
- `Service` class 中的 metadata 提供了 Angular 需要的資訊，令其可以透過 dependency injection 被注入給其他 `Component` 使用。
@Components 裝飾器會指出接在後面的 class 是個 component class，順便為其指定 metadata，把它和 template 關聯起來。
也就是有了這個裝飾器，其 class 才從普通的 javascript class 變成了 Angular component。

一個 Component 的 metadata 常見組成：  
In `src/app/hero-list.component.ts (metadata)`
```javascript
@Component({
  selector:    'app-hero-list',
  templateUrl: './hero-list.component.html',
  providers:  [ HeroService ]
})
export class HeroListComponent implements OnInit {
/* . . . */
}
```

- selector：是一個 CSS 選擇器，它會告訴 Angular，一旦在範本 HTML 中找到了這個選擇器對應的標籤，就建立並插入該元件的一個 instance。 如果 HTML 中包含 `<app-hero-list></app-hero-list>`，Angular 就會在這些標籤中插入一個 HeroListComponent instance 的檢視，也就是 `./hero-list.component.html` 的內容。

- templateUrl：此元件的 HTML 範本檔案相對於這個元件檔案的位置。在這邊也可以直接寫 inline HTML 作為 template 的值。
範例：
```javascript
@Component({
  selector: 'app-root',
  template: `
    <h1>Tour of Heroes</h1>
    <app-hero-main [hero]="hero"></app-hero-main>
  `,
  styles: ['h1 { font-weight: normal; }']
})
export class HeroAppComponent {
/* . . . */
}
```

- providers：當前元件所需的 services 的 Array。在這個例子中，它告訴 Angular 該如何提供一個 HeroService instance ，以獲取要顯示的英雄列表。

### 樹狀結構
一個 Angular 應用程式中的元件由樹狀結構組合，每個元件都有各自不同的目的和任務（圖片取自官網）：
![Angular component tree structure](https://angular.io/generated/images/guide/start/app-components.png)

- app-root(orange box) 最底層的父層元件
  應用程式的殼層結構，是第一個被載入的元件，也是其他所有元件的父層元件。
  - app-top-bar
  - app-product-list
    - app-prouct-alerts

## 範本與檢視
帶層次結構的檢視可以包含同一[模組（NgModule）]({{ sit.baseurl }}{% link _posts/2019-07-31-Angular_module_basic_note.md %})中元件的檢視，也可以（而且經常會）包含其它模組中定義的元件的檢視。
![A view hierarchy](https://i.imgur.com/DgscIRd.png)

### 範本語法 Template syntax
範本很像標準的 HTML，但是它還包含 Angular 的範本語法(template syntax)，這些範本語法可以根據你的應用邏輯、應用狀態和 DOM 資料修改 HTML。 你的範本可以使用資料繫結(data binding)來協調應用和 DOM 中的資料，使用管道(pipes)在顯示出來之前對其進行轉換，使用指令(directives)來把程式邏輯應用到要顯示的內容上。

`src/app/hero-list.component.html`
```javascript
<h2>Hero List</h2>

<p><i>Pick a hero from the list</i></p>
<ul>
  <li *ngFor="let hero of heroes" (click)="selectHero(hero)">
    {{ "{{ hero.name "}}}}
  </li>
</ul>

<app-hero-detail *ngIf="selectedHero" [hero]="selectedHero"></app-hero-detail>
```
- `*ngFor` 指令告訴 Angular 在一個列表上進行迭代。

- `{{"{{ hero.name "}}}} `、`(click)` 和 `[hero]` 把程式資料和 DOM binding 起來，以響應使用者的輸入。

範本中的 `<app-hero-detail>` 標籤代表渲染元件 HeroDetailComponent 的顯示元素。

## 資料繫結 Data binding

Template 結合了普通的 HTML、Angular directive) 和資料繫結標籤(binding markup)。

如果沒有框架，就要自己把資料推送到 HTML 中，然後監聽使用者的動作事件更新對應的值到指定的元素中，實現這間雙向資料繫結便是框架的厲害之處。

Angular 支援雙向資料繫結(two-way data binding)，是一種對範本中各個部分和元件做聯繫。

下圖來自官網，顯示了資料繫結標籤連結應用程式資料和 DOM 的四種形式和資料傳遞的方向：
![data binding from angular.io](https://i.imgur.com/3ZCPVjR.png)

  - **事件繫結 Event binding**: 在目標環境中更新應用程式資料以回應使用者的輸入。
  - **屬性繫結 Property binding**: 將計算結果插入 HTML 之中。

In `src/app/hero-list.component.html (binding)`
```html
<li>{{"{{ hero.name "}}}}</li>
<app-hero-detail [hero]="selectedHero"></app-hero-detail>
<li (click)="selectHero(hero)"></li>
```
- `{{"{{hero.name"}}}}`插值表示式(interpolation )在 `<li>` 標籤中顯示元件的 `hero.name` 屬性的值。
- `[hero]` 屬性繫結(property binding)把父元件 HeroListComponent 的 selectedHero 的值傳到子元件 HeroDetailComponent 的 hero 屬性中。
- 當用戶點選某個英雄的名字時，(click) 事件繫結(event binding)會呼叫元件的 selectHero 方法。

至於前面提到的雙向資料繫結主要用於範本驅動表單(template-driven forms)中，它會把屬性繫結和事件繫結組合成一種單獨的寫法。下面這個來自 HeroDetailComponent 範本中的例子透過 ngModel 指令使用了雙向資料繫結：
In `src/app/hero-detail.component.html (ngModel)`
```html
<input [(ngModel)]="hero.name">
```
在雙向繫結中，資料屬性值透過屬性繫結從元件流到輸入框。使用者的修改透過事件繫結流回元件，把屬性值設定為最新的值。

Angular 在每個 JavaScript 事件迴圈中處理所有的資料繫結，它會從元件樹的根部開始，遞迴處理全部子元件。
![data binding flow chart from angular.io](https://i.imgur.com/9Oh2zf9.png)

資料繫結在範本及其元件之間的通訊中扮演了非常重要的角色，它對於父元件和子元件之間的通訊也同樣重要。

## 管道 Pipes
Angular 的管道可以讓你在範本中宣告顯示值的轉換邏輯。帶有 @Pipe 裝飾器的 class 中會定義一個轉換函式，用來把輸入值轉換成供檢視顯示用的輸出值。Angular 有很多[**內建的管道**](https://angular.io/api?type=pipe)，也可以自訂新增管道。範例：

```html
<p>{{"{{ 'From zen to code' | titlecase "}}}}</p>

<!-- 結果會在網頁呈現 -->
<p>From Zen To Code</p>

<!-- Default format: output 'Jun 15, 2015'-->

<p>Today is {{"{{ today | date "}}}}</p>

<!-- fullDate format: output 'Monday, June 15, 2015'-->

<p>The date is {{"{{ today | date:'fullDate' "}}}}</p>

 <!-- shortTime format: output '9:43 AM'-->

 <p>The time is {{"{{ today | date:'shortTime' "}}}}</p>
```

## 參考
  [Angular.io](https://angular.io/guide/displaying-data)