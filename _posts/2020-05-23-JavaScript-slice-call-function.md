---
layout: post
title: "JavaScript - Have you met [].slice.call() ?"
description: "JavaScript - The usage of Array convert: Array.prototype.slice.call() and [].slice.call()"
crawlertitle: "JavaScript - The usage of Array convert: Array.prototype.slice.call() and [].slice.call()"
date: 2020-05-23 18:35:11 +0800
categories: JavaScript
tags: JavaScript
comments: true
---
Have you met `[].slice.call(arguments)` or `Array.prototype.slice.call(arguments)` in any JavaScript code?

This confusing code is used to convert the argument object which has `.length` property and **numeric indices** (so-called array-like object) to a real Array, that invokes two methods: `slice()` and `call()`.

- `slice()`

  The `slice()` method returns a **shallow copy** of a portion of an array into a new array object selected from `begin` to `end` (`end` not included).

  Syntax: `arr.slice([begin[, end]])`

  ```js
  const animals = ['ant', 'bison', 'camel', 'duck', 'elephant'];

  console.log(animals.slice()); // [ 'ant', 'bison', 'camel', 'duck', 'elephant' ]
  console.log(animals.slice(2)); // ["camel", "duck", "elephant"]
  console.log(animals.slice(2, 3)); // [ 'camel' ]
  ```
  > see [MDN web docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice)

- `call()`

  Both `call()` and `apply()` methods let us manually set the value of `this`.  
  The `call()` method calls a function with a given `this` value and arguments provided individually.

  Syntax: `func.call([thisArg[, arg1, arg2, ...argN]])`

  ```js
  function Product(name, price) {
    this.name = name;
    this.price = price;
  }

  function Food(name, price) {
    Product.call(this, name, price);
    this.category = 'food';
  }

  console.log(new Food('cheese', 5).name); // cheese"
  ```

![paragraph break](https://order-brother.s3-ap-northeast-1.amazonaws.com/paragraph+break/separator-1.png)

So, if we specify the value of `this` in `slice()` to an array-liked object,  
we make `slice()` method workable on the object, and if we call `slice()` without passing optional arguments, it will return the whole chunk as a new array.

```js
let listObj = {
  '0': 'zero',
  '1': 'one',
  '2': 'two',
  length: 3
};

let arr = [].slice.call(listObj);
console.log(arr); // [ 'zero', 'one', 'two' ]
```

As you can see, we could even convert the `listObj` because the object has property `length` and numeric indices.

In the end, there are other workarounds to convert an object to an array, here's for your references.

Let's say we have a NodeList object from a website, and we want to manipulate it like a real Array:

```js
let links = document.querySelectorAll('.link');

let arr1 = [...links]; // iterable argument
let arr2 = Array.from(links);
let arr3 = [].slice.call(links); // [].slice === Array.prototype.slice;
let arr4 = Array.prototype.slice(links);
```

## References

- [Function.prototype.call()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call) - MDN web docs
- [how does Array.prototype.slice.call() work?](https://stackoverflow.com/questions/7056925/how-does-array-prototype-slice-call-work) - Stack Overflow