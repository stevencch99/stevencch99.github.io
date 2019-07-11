---
layout: post
title:  "什麼是 AJAX？"
description: "一種網頁的互動方式，在用戶無需重新載入整個網頁的情況下，就能夠更新網頁部分的內容。"
crawlertitle: "什麼是 AJAX？"
date:   2019-07-10 12:56:46 +0800
categories: JavaScript
tags: ['JavaScript']
comments: true
---

> 一種網頁的互動方式，在用戶無需重新載入整個網頁的情況下，就能夠更新網頁部分的內容。

## Asynchronous JavaScript and XML, AJAX

AJAX 這個詞誕生於 2005 年 2 月，係由 [Jesse James Garrett](https://zh.wikipedia.org/wiki/%E5%82%91%E8%A5%BF%C2%B7%E8%A9%B9%E5%A7%86%E5%A3%AB%C2%B7%E8%B3%88%E7%91%9E%E7%89%B9) 發明的術語，描述一種使用數個技術的方法而非一種技術。

這些技術包含有： 
- **HTML/XHTML**：  
  呈現網頁主要的內容

- **DOM**(Document Object Model), JavaScript：  
  動態地修改已經載入的網頁、撰寫 AJAX 引擎

- **CSS**：  
  定義網頁外觀樣式

- **XML**(Extensible Markup Language)：  
  資料交換的格式(實務上也有採用 JavaScript 陣列而不一定要用 XML 格式)

- **XMLHttpRequest**：  
  主要的溝通中介，非同步地與伺服器交換資料

- 網頁後端的處理程式(Rails, JAVA, PHP, etc.)

## 傳統的網頁動作

當使用者填寫表單送出內容後，向網頁伺服器傳送請求，伺服器接收並處理收到的表單，然後回覆一個新的網頁，在使用者看來，即使只更新頁面的一小部分內容，在送出資料後整個頁面卻都會消失再重新載入，而在整個重組網頁、傳送新資料的期間，我們就**只能等待**。

而且送出資料前、後的網頁內容大部分是相同的，這種傳統的做法會浪費許多效能和頻寬，而且每次 Web App 和伺服器的『溝通』都要傳送需求，頁面更新的時間又依賴伺服器的回應和網速，如此便導致較差的使用者體驗，同時也給伺服器造成負擔。

使用 AJAX 最大的優點就在於不必重新載入整個頁面，就可以和伺服器交換資料並更新部分內容。

## 使用時機

- **註冊使用者**  

  常見的例子是註冊新的使用者，在填寫資料的過程就可以做到即時驗證信箱或用戶名稱是否有重複的狀況。

- **Google**  

  另外最出名的例子大概就是 Gmail, Google Map 和 Google Suggest（當你鍵入搜尋時，就開始和伺服器請求建議資料並即時顯示），這些特殊、威力強大的互動模式和使用體驗，展示了網頁應用系統漸漸地跨過了和桌面應用程式之間操作模式的鴻溝。

## 參考資料：
- [Professional Ajax](https://www.delightpress.com.tw/book.aspx?book_id=SKTP00011) - 悅知文化
- [RUNOOB.COM](http://www.runoob.com/ajax/ajax-intro.html)