# 成為程式"設計師"

## 程式界的咖哩

在上次的[範例 6]中，我們抽象了共用的 function。

```js
/* -- helper.js -- */
const getName = (x) => x.name;
const getNames = (xs) => xs.map(getName);

/* -- main.js -- */
const cats = [
  { name: "Luna", age: 3 },
  { name: "Milo", age: 1 },
];

const catNames = getNames(cats);
const dogNames = getNames(dogs);
```

是不是也可以把 `getName()` 再次抽象，讓它不再限制於只有 name 這個屬性，而是可以更動態地、更靈活地去使用呢？  
我們先單純把常數值換成變數，讓它可以動態。

### 範例 1

```js
/* -- library.js -- */
const prop = (x, k) => x[k];

/* -- helper.js -- */
const getName = (x) => prop(x, "name");
const getAge = (x) => prop(x, "age");

const getNames = (xs) => xs.map(getName);
const getAges = (xs) => xs.map(getAge);

/* -- main.js -- */
const cats = [
  { name: "Luna", age: 3 },
  { name: "Milo", age: 1 },
];

console.log(getNames(cats)); // ["luna", "Milo"]
console.log(getAges(cats)); // [3, 1]
```

雖然我們更抽象了 `prop()` 這個 function，但似乎好像沒有太多的幫助。  
因為我們依然要在自己的程式中寫很多自己的 helper function，真正被共用的地方好像沒很多。  
其實是因為 `prop()` 這個 function 設計的不好，導致用起來好像沒有幫助。

如果你有概念想做到範例 1 的樣子，恭喜你已經有想要讓自己所寫出來的程式變得更好的念頭了。  
接下來你將要提升一個等級，讓你從一個只會用程式解決問題的碼農，變成一個程式"設計師"。  
我們將使用 [currying] 的技巧把共用 function 設計得更活用。  
首先我們先將 function 的參數從一次全部帶，變成一次帶一個，並分次呼叫。

```js
const prop = (x) => (k) => x[k];

const getName = (x) => prop(x)("name");

console.log(getName({ name: "qimat" })); // qimat
```

如果你是第一次看見寫法，不要驚訝，以後會變成家常便飯。  
可以比較範例 1 的 `getName()` 的實作其實沒有差太多，就是把參數分開成兩次去帶，差不多是把逗號變成小掛號而已。  
但這並沒有改善到任何東西，是因為參數順序的問題。

可以從 `getName()`, `getAge()` 看到，其實我們常見的狀況是早就知道想要做的"動作"是什麼，就是取 name 或是 age 這個屬性。  
但我們還沒辦法預測"資料"是什麼時候會被帶進來，就是那個將要被帶進來的物件 `x`。  
以 `prop()` 為例，就是我們通常會先拿到 `k` 然後再拿到 `x`，那麼設計上的順序就應該這樣做。

```js
const prop = (k) => (x) => x[k];

const getName = (x) => prop("name")(x);

console.log(getName({ name: "qimat" })); // qimat
```

此時這個順序的翻轉就可以導致天差地別的結果，別看 `getName()`的實作也只是換呼叫的順序而已。  
可以看見 `getName()` 所需要的 `(x) =>` 就是 `prop("name")(x)` 所需要的 `x`，所以 `prop("name")` 其實就是 `getName()`。

> const getName = ~~(x) =>~~ prop("name")~~(x)~~ ;

```js
const prop = (k) => (x) => x[k];

const getName = prop("name");

console.log(getName({ name: "qimat" })); // qimat
```

那麼我們就不用一直去生出那些中間過程的 helper function，直接組出來用就好了。

```js
/* -- library.js -- */
const prop = (k) => (x) => x[k];

/* -- helper.js -- */
const getNames = (xs) => xs.map(prop("name"));
const getAges = (xs) => xs.map(prop("age"));
```

## [高階函數]

而且 `getNames()` 跟 `getAges()` 也可以做一樣的設計。  
先把 map 所需要的 callback 變成可以帶入的參數。

```js
(xs, f) => xs.map(f);
```

再把參數變成一次只帶一個的形式。

```js
(xs) => (f) => xs.map(f);
```

接著把參數的順序設計成常用情境的順序。

```js
(f) => (xs) => xs.map(f);
```

最後就能得到一個更好用的、可組合的共用 function。

```js
/* -- library.js -- */
const prop = (k) => (x) => x[k];
const map = (f) => (xs) => xs.map(f);

/* -- main.js -- */
const cats = [
  { name: "Luna", age: 3 },
  { name: "Milo", age: 1 },
];

console.log(map(prop("name"))(cats)); // ["luna", "Milo"]
console.log(map(prop("age"))(cats)); // [3, 1]
```

當然如果你不喜歡看到那麼多小掛號在同一行，也可以先宣告變數存起來，這樣也可以賦予意義，使其更好閱讀。  
甚至不用真的知道 `map(prop())` 到底做了些什麼，因為變數命名就說明了一切，不需要寫任何註解。

### 範例 2

```js
/* -- helper.js -- */
const getNames = map(prop("name"));
const getAges = map(prop("age"));

/* -- main.js -- */
const cats = [
  { name: "Luna", age: 3 },
  { name: "Milo", age: 1 },
];

console.log(getNames(cats)); // ["luna", "Milo"]
console.log(getAges(cats)); // [3, 1]
```

這樣設計完之後，其實最終要取值得那瞬間也都是一樣的使用方式，對於使用 `getNames()` 的人完全沒有影響。  
但是卻讓背後實作 `getNames()` 的人可以透過一些共用 function 的組合去產生新的 function 給別人使用，達到靈活的效果。

## 附註

本文中的 `map()` 之所以能稱為高階函數是因為它把 function 當成參數讓人帶入，並回傳新的 function 當成值傳出。  
所以只要你所寫的程式語言可以把 function 當成參數去傳遞的話，那麼這些觀念跟技巧都是可以通用的。  
也就是你只要在該語言會寫 function，剩下的都是同一套邏輯就可以組出你需要的東西，學一套就萬用的觀念：[Functional programming]

[範例 6]: https://github.com/QimatLuo/best_practice/blob/main/xyz.md#%E7%AF%84%E4%BE%8B-6
[currying]: https://zh.wikipedia.org/wiki/%E6%9F%AF%E9%87%8C%E5%8C%96
[高階函數]: https://zh.wikipedia.org/wiki/%E9%AB%98%E9%98%B6%E5%87%BD%E6%95%B0
[functional programming]: https://zh.wikipedia.org/zh-tw/%E5%87%BD%E6%95%B0%E5%BC%8F%E7%BC%96%E7%A8%8B
