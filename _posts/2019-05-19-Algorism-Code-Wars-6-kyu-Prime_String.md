---
layout: post
title: "[Algorism] 找出規律的藝術：Codewars 6 kyu - Simple Fun 116: Prime String"
crawlertitle: "[Algorism][CodeWars]：6 kyu - Simple Fun 116: Prime String"
description: "[Algorism][CodeWars]：6 kyu - Simple Fun 116: Prime String 找出規律的藝術"
date: 2019-05-19 16:56:46 +0800
categories: Algorism
tags: ["Algorism"]
comments: true
---

> 本文同步收錄在 [Medium](https://medium.com/space-cat/codewars-%E8%A7%A3%E9%A1%8C-simple-fun-116-prime-string-6-kyu-7327f9fc90cd)

[CodeWars](https://www.codewars.com) 是我在走跳的刷題網站，相關介紹請參考
『[透過 codewars 修練 Swift 解題技巧](https://medium.com/%E5%BD%BC%E5%BE%97%E6%BD%98%E7%9A%84-swift-ios-app-%E9%96%8B%E7%99%BC%E5%95%8F%E9%A1%8C%E8%A7%A3%E7%AD%94%E9%9B%86/%E9%80%8F%E9%81%8E-codewars-%E4%BF%AE%E7%B7%B4-swift-%E8%A7%A3%E9%A1%8C%E6%8A%80%E5%B7%A7-50890eac0bec)』 by [彼得潘的 iOS App Neverland](https://medium.com/@apppeterpan)

Codewars 上的題目根據不同程度的定義和要求，分為最簡單的 8 kyu 到最難的 1 kyu，送交自己的答案之後就可以看到其他 Code Warriors 的解法和各種奇思妙想。

![題目分為最簡單的 8 kyu 到最難的 1 kyu](https://cdn-images-1.medium.com/max/2000/1*cFIitg1PKIHHNIUpmmiC4Q.jpeg)_題目分為最簡單的 8 kyu 到最難的 1 kyu_

## 今天來談談這讓我徹夜難眠的一題：難度 6 kyu 的

“[Simple Fun #116: Prime String](https://www.codewars.com/kata/simple-fun-number-116-prime-string)”

為了練習初學的 JavaScript 語法，解這題大約花了我 4 個小時，可以說是嚴重卡關，食不下嚥，寢不成寐。

熬到好不容易解完，才開心了幾秒，看見大家票選的 Best practice，
那優雅的解法一字一字映入眼簾，彼時我的內心震動，良久不能自已 ⋯⋯

> 『深深感受到我和世界的差距』

> 劇透警告！
> 本帖含有解答說明，如果你看過題目後希望自己先嘗試挑戰，請謹慎服用。  
> Spoiler alert!
> Do not read down if you want to solve this kata by yourself.

Prime 代表質數的意思，除了 1 以外不能被任何數整除，以此延伸來看題目的 “Prime String” 就很容易理解題意。

我們的任務是要定義一個方法 primeString() 檢查輸入的字串是否由**連續的重複字串**組合而成（意即能被某字串”整除“）：

例如 “steven” 就是個『質數字串』，而 “xyxy” 則否（“xyxy” = “xy” + “xy”）。

待測試的輸入是英文小寫字母，輸出要求回傳布林值，如果是質數字串則回傳 true。

簡單測試長得像這樣：

### JavaScript

```js
console.log(primeString("asdf")); // should be true
console.log(primeString("abac")); // should be true
console.log(primeString("qiuefgqiuefgqiuefg")); // should be false
console.log(primeString("zkvjhuj")); // should be true
```

### 我的方法

我的邏輯是將輸入字串中的某個字元作為分割陣列的符號，將其拆成具有某種規律的陣列，例：

```js
"qiuefgqiuefgqiuefg".split("q"); // [ '', 'iuefg', 'iuefg', 'iuefg' ]
```

如此將第一個空元素去掉即可得到整齊的陣列，接著統計陣列中每個元素的數量，如果每個元素出現的次數都完全相同，就代表此輸入字串可以被整除。

```js
function primeString(s) {
  // 將字串重複的部份切成一個規律的陣列
  let spArr = s.split(s[0]);
  // 切掉不要的首個空元素
  spArr.shift();

  // 宣告 不重複計數器
  let falseCount = 0;

  //  將各個元素和它們出現的次數存成 key-value pair
  let tCounter = {};
  spArr.forEach(x => {
    tCounter[x] = (tCounter[x] || 0) + 1;
  });

  // 將每個元素的重複出現次數清點過後比較，如果出現的次數相同表示有重複
  for (let e of spArr) {
    // 如果重複次數小於等於 1 表示開頭字母只出現一次，其餘都不重複 falseCount + 1
    // 或者有任何元素的出現次數和陣列第一個元素不同 falseCount + 1
    tCounter[spArr[0]] <= 1 || tCounter[spArr[0]] !== tCounter[e]
      ? (falseCount += 1)
      : e;
  }
  // 將不重複計數器 falseCount 轉為布林值輸出，大於 0 就會轉成 true
  return !!falseCount;
}
```

以上是我的作法，將近 10 行，接著來看看大家選出的 Best practices：

## Best practice & The most Clever

### JavaScript

```js
function primeString(s) {
  return (s + s).indexOf(s, 1) == s.length;
}
```

### Ruby

```ruby
def primeString(s)
  (s + s).index(s, 1) == s.size
end
```

舉個例子解釋：
"steven" 字串長度為 6，其中各個字母的 index 則是 0 ~ 5

|  s  |  t  |  e  |  v  |  e  |  n  |
| :-: | :-: | :-: | :-: | :-: | :-: |
|  0  |  1  |  2  |  3  |  4  |  5  |

這個解法先將字串變成兩倍長，並從 index 1 號位置開始找尋比對原始的字串：

|  s  |  t  |  e  |  v  |  e  |  n  |  s  |  t  |  e  |  v  |  e  |  n  |
| :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
|  0  |  1  |  2  |  3  |  4  |  5  |  6  |  7  |  8  |  9  | 10  | 11  |
|     |  ⇧  |  ~  |  ~  |  ~  |  ~  |  s  |  t  |  e  |  v  |  e  |  n  |

可以看到如果以『質數字串』下去比對，一定會在正好等於原始字串長度的位置(index 6)比對出來。

如下所示，輸入 “xyxy” 則會在 index = 2 的位置找到：

|  x  |  y  |  x  |  y  |  x  |  y  |  x  |  y  |
| :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
|  0  |  1  |  2  |  3  |  4  |  5  |  6  |  7  |
|     |  ⇧  |  x  |  y  |  x  |  y  |  -  |  -  |

## Regular expression

如果熟悉使用正規表達式，用群組的方式比對也是很快：

### JavaScript

```js
function primeString(s) {
  return !/^(.+)\1+$/.test(s);
}
```

### Ruby

```ruby
def prime_string(s)
  s !~ /^(.+)(\1+)$/
end
```

## Recursive

由 [Tai An Su](https://medium.com/@taiansu) 老師提出的遞迴解法，這邊我盡可能地將自己的理解寫在備註中。

### JavaScript

```js
function primeString(s, count = 1) {
  // 如果字串中有重複的模式，最少也會重複 1 次，故迭代超過字串長度的一半還沒找到就不用找了
  if (count > s.length / 2) {
    return true;
  }

  // 每次迭代將字串的一小塊，設定為分割陣列的符號 chunk
  let chunk = s.slice(0, count);
  // 被 chunk 切過的部分會變成陣列中的空元素，此行判斷是不是能夠將陣列切成重複的小塊
  // 如果切完後全部的陣列元素都是空的，代表字串被 chunk 整除，恭喜找到規律。
  let grouped = s.split(chunk).every(i => i === "");

  // 若上一行的結果為否表示沒有找到重複的規律，則呼叫自己，用更大的單位(count + 1)繼續切，
  // 直到超過字串長度的一半都沒找到，代表輸入為 prime string
  return grouped ? false : primeString(s, count + 1);
}
```

就這樣，和世界的差距又縮小了，覺得非常開心！

> ### Keep Coding 🙌
