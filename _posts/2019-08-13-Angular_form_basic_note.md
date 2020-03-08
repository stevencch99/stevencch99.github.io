---
layout: post
title: "Angular Form 表單資料流筆記"
description: "Angular Form 表單資料流筆記"
crawlertitle: "Angular Form 表單資料流筆記"
date: 2019-08-13 01:15:46 +0800
categories: Angular
tags: Angular
comments: true
---

## 目錄
> 本文程式碼範例及圖片皆來自 [Angular.io](http://angular.io)

- toc
{:toc}

## Two type of Forms

Angular 提供兩種不同的方法透過表單處理使用者輸入：響應式(Reactive)表單和範本驅動(Template-driven)表單。
這兩種類型的表單都會從 view 中提供以下功能：

- 捕捉使用者的輸入事件
- 驗證使用者輸入
- 建立表單模型
- 修改資料模型
- 提供追縱這些更改的途徑

### Reactive forms 響應式表單

更穩固，而且擴充性、複用性和可測試性更強。如果表單是我們應用程式中的關鍵部分，建議使用響應式表單。

### Template-driven 範本驅動表單

建議使用在新增簡單的表單，適合非常基本的表單需求，以及能夠簡單地透過 template 管理的邏輯。

### 兩種表單的關鍵差異

|                       | Reactive 響應式表單                                | Template-driven 範本驅動表單 |
| --------------------- | -------------------------------------------------- | ---------------------------- |
| 表單建立              | 顯式，表單內容定義並建立在 Component 的 class (元件類)中。 | 隱式，由元件建立。           |
| Data model            | Structured                                         | Unstructured                 |
| 資料模式              | 結構化                                             | 非結構化                     |
| 可預測性              | 同步                                               | 非同步                       |
| 表單驗證              | 函式                                               | 透過指令(Directives)         |
| 可變性(Mutability)    | 不可變                                             | 可變                         |
| 可延展性(Scalability) | Low-level API access                               | Abstraction on top of APIs   |

### 共通的基礎

上述兩種表單同享以下的表單建構基礎元件。

- FormControl 從各自獨立的 FromControl 追蹤數值並驗證狀態（例如表單中的一個 checkbox）。
- FormGroup 追蹤整個表單控制組的 formContorls(包含各種欄位的整張表單)。
- FormArray 追蹤由 FormControls 組成的陣列（例如一張表單中包含多個根據使用者需求動態新增的寄件地址）。
- ControlValueAccessor 資料存取器建立 FormControl 實例和 DOM 元素的聯繫（將 html 中的元素和 Angular 的控制關聯在一起，藉此讀取值或者送出表單操作）。

## 建立表單模型

響應式和範本驅動表單都是透過表單模型(Form model)來追蹤 Angular 表單和輸入值的變化，透過以下例子展示如何建立表單模型。

### 響應式表單

下面是一個帶有輸入欄位的元件，使用響應式表單實現了單個控制元件。

```javascript
import { Component } from '@angular/core';
import { FormControl } from '@angular/forms';

@Component({
  selector: 'app-reactive-favorite-color',
  template: `
    Favorite Color: <input type="text" [formControl]="favoriteColorControl">
  `
})

export class FavoriteColorComponent {
  favoriteColorControl = new FormControl('');
}

```
- 響應式表單控制
![Reactive forms work flow from angular.io](https://i.imgur.com/Z71J318.png)
在響應式表單中，表單模型定義在**元件類**(component class)中。接著，響應式表單(FormControlDirective)會把這個現有的表單控制元件 instance 透過資料存取器(ControlValueAccessor instance)來指派給檢視中的表單元素。

### 範本驅動表單
```javascript
import { Component } from '@angular/core';

@Component({
  selector: 'app-template-favorite-color',
  template: `
    Favorite Color: <input type="text" [(ngModel)]="favoriteColor">
  `
})
export class FavoriteColorComponent {
  favoriteColor = '';
}
```
在範本驅動表單中，權威資料(the source of truth)的來源是在 template 中定義。
![Template-driven forms work flow from angular.io](https://i.imgur.com/fyi0Iv0.png)
這個例子中，將表單模型抽象化提取到 template 之中，有助於程式結構的簡化，範本中的 `[(ngModel)]` 指令負責建立和管理表單元素上的控制項 `favoriteColor`，這個表單元素因此變得隱晦，優點在於我們不必親自去個別操作表單模型。

## 響應式表單中的資料流
在響應式表單中，檢視中的每個表單元素都直接連結到一個表單控制模型的實例(FormControl instance)。從檢視到模型的操作以及從模型到檢視的修改都是同步的。

### 從檢視到表單模型
  ![The steps of the data flow from view to model](https://i.imgur.com/GrRlWP2.png)
  1. 使用者在輸入框中輸入了 'Blue'
  2. 輸入框會發出帶有最新值的 input 事件。
  3. 控制元件值存取器 ControlValueAccessor 會監聽表單輸入框元素上的事件，並立即把新值傳給 FormControl instance。
  4. FormControl instance 透過 valueChanges 可觀察物件(observable) 發出這個新值。
  5. 承上，valueChanges 的訂閱者都會收到這個新值，透過這個機制實現一些同步操作。

### 從表單模型到檢視
  ![The steps of the data flow from model to view](https://i.imgur.com/6hszF9Q.png)
  1. favoriteColorControl.setValue() 方法被呼叫，更新這個 FormControl 裡面的值。
  2. FormControl 透過 valueChanges observable 發出新值。 
  3. valueChanges 的訂閱者都會收到這個新值。
  4. 該表單輸入框元素上的控制元件值存取器會把控制元件更新為這個新值。

## 範本驅動表單中的資料流

在範本驅動表單中，每個表單元素都連結到一個指令上，該指令負責管理其內部表單模型。
![The steps of the data flow in template-driven forms](https://i.imgur.com/WWsrKFt.png)
1. 使用者在 input 元素輸入了 "Blue"。
2. input 元素會帶著 "Blue" 值，發出一個 "input event"。
3. 在這個 input 元素上的控制元件值存取器(control value accessor)會觸發 FormControl 的 `setValue()` 方法。
4. FormControl 透過 valueChanges observable 發出新值。
5. valueChanges 的訂閱者都會收到這個新值。
6. 控制元件值存取器還會呼叫 `ngModel.viewToModelUpdate()` 方法，發出一個 ngModelChange 事件。
7. 此元件範本資料透過[雙向資料繫結(Two-way data binding)]({{ site.baseurl }}{% link _posts/2019-08-4-Angular_component_basic_note.md %}#-資料繫結-Data-binding) 連結到 favoriteColor，元件(component)中的 favoriteColor 屬性就會修改為 ngModelChange 事件中攜帶的值 "Blue"。

## 參考

[Angular.io](https://angular.tw/guide/forms-overview)
