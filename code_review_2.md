# 如何不要竄改變數？

## Code Review 前

### 範例 1

```js
function f(o) {
  const r = {};

  if (o.a) {
    mutateA(r);
  }

  if (o.b) {
    mutateB(r);
  }

  return r;
}
```

這個 function 會接受一個 object。  
如果 object 有特定的 property `a`，那麼就會做某些事。  
如果 object 有特定的 property `b`，那麼就會做某些事。  
這就是一個不斷地在竄改變數 `r` 的一個實作方式。

問題就是你無法預測 `return r` 到底會給你什麼東西。  
因為那個竄改的行為你不知道是加是減，甚至是沒做事只是個 side effect 也有可能。  
實際它跑出來的結果大概是：

```js
console.log(f({ a: "a" })); // { a1: "a1", a2: "a2", a3: "a3" }
console.log(f({ b: "b" })); // { b1: "b1", b2: "b2", b3: "b3" }
console.log(f({ a: "a", b: "b" })); // { a1: "a1", a2: "a2", a3: "a3", b1: "b1", b2: "b2", b3: "b3" }
```

就是有 `a` 就會塞入跟 `a` 相關的結果，有 `b` 亦然。都有的話就都塞。

## 換成 pure function

其實不要竄改的寫法並沒有很難，所有 pure function 都會回傳一個全新的 output。  
那我們只要把兩個 output 合在一起就是我們要的答案了。

```js
function f(o) {
  const r = {};

  if (o.a) {
    const a = pureA();
  }

  if (o.b) {
    const b = pureB();
  }

  return r;
}
```

現在面臨到 `a` 跟 `b` 沒辦法合起來的問題，因為 scope 不一樣。  
我們可以透過預設空值來讓它們處在同一個 scope。

```js
function f(o) {
  const r = {};

  const a = o.a ? pureA() : {};

  const b = o.b ? pureB() : {};

  return r;
}
```

那麼 `r` 事實上就是兩個合起來的結果。

## Code Review 後

### 範例 2

```js
function f(o) {
  const a = o.a ? pureA() : {};

  const b = o.b ? pureB() : {};

  return { ...a, ...b };
}
```

可以看見我們可以預期這個 function 的 output 就是 `a` 跟 `b` 合起來的結果，完全不用猜測。  
唯一要考量的就是 `a` 跟 `b` 是否有同樣的 key 會被覆蓋而已。  
這樣程式讀起來非常容易，也減少了不預期的行為，bug 出現率就相對降低了，就是所謂更好管理的程式設計。

## 一對一

還有一種情境有點類似。

### 範例 3

```js
function f(x) {
  let color, icon;

  if (x === "a") {
    color = "green";
    icon = "a-icon";
  } else if (x === "b") {
    color = "red";
    icon = "b-icon";
  } else {
    color = "default-color";
    icon = "default-icon";
  }

  return { color, icon };
}
```

會這樣寫是因為不想要每個 `if` 裡面都回傳 `{ color, icon }` 的這樣的結構，因為如果東西多了，結構性的程式碼的重複率就高了。  
所以透過把屬性獨立出來，然後讓每個 `if` 裡面去單獨改變值，在最後回傳結果時才把結構組出來，就可以避免重複寫結構的部分。  
當然也可以在宣告時就可以定義預設值，這樣可能更好。

但我們有更好的寫法就是不要竄改變數，畢竟程式一旦變大，後果就難以預測。  
這種一對一的情境其實就是用 mapping table 就可以完成了。

### 範例 4

```js
function f(x) {
  const mColor = {
    a: "green",
    b: "red",
  };

  const mIcon = {
    a: "a-icon",
    b: "b-icon",
  };

  const color = mColor[x] || "default-color";
  const icon = mIcon[x] || "default-icon";

  return { color, icon };
}
```

## 多對一

但上述只能應付一對一的情況，如果我們情況是多對一其實就沒用了。

### 範例 5

```js
function f(x) {
  let color, icon;

  if (x > 50) {
    color = "green";
    icon = "a-icon";
  } else if (x > 0) {
    color = "red";
    icon = "b-icon";
  } else {
    color = "default-color";
    icon = "default-icon";
  }

  return { color, icon };
}
```

這時候我們要退回去思考，為什麼都在做一樣的事情，可是條件有一點小改變我們就得改變作法呢？  
其原因是我們不小心"優化"了我們的程式，因為 color 跟 icon 本來就是兩件獨立的東西，只是因為情境是在一起，我們就很習慣的寫在一起了。  
基於[一切從最小單位出發]的心法，color 跟 icon 其實是獨立可以被運作的。

### 範例 6

```js
function getColor(x) {
  if (x > 50) {
    return "green";
  } else if (x > 0) {
    return "red";
  } else {
    return "default-color";
  }
}

function f(x) {
  return {
    color: getColor(x),
    icon: getIcon(x),
  };
}
```

這樣上述所有情境瞬間就被解開，因為所有問題都被包在各自獨立的 function 裡面處理掉了。  
儘管底層 function 還是用 `if` 去處理事情，但是因為它已經去結構化了，所以沒有範例 3 的問題發生。

## 附註

可能會有人問，那麼 `getColor()` 跟 `getIcon()` 不就重工邏輯了嗎？  
這其實是另一層面的問題，如果我們把他們各自視為獨立的功能，那麼他們只是邏輯在這個情境下"剛好"一樣而已。  
誰知道明天會不會跟你說不管大於 0 或 50 都是綠色，但是 icon 不變，對吧？  
用這樣去想似乎沒有重工，就跟做 API 到處都要定義回傳的是 200, 201, 404 ...etc，只是剛好一樣，但事實上每個 API 都是獨立運作的。

當然只看程式面確實重工了，而我們也是有辦法去抽象這一部分。  
只要把 `if` 裡面的東西抽出來做為 `isGt50()` 或是語意一點 `isEnough()`, `isNormal()` 都可以。  
這樣一來就有達到邏輯重用的效果，就連 `if` 也可以被像是 [cond()] 去抽出來，沒有 function 不能解決的事情。

[一切從最小單位出發]: https://github.com/QimatLuo/best_practice/blob/main/atom.md
[cond()]: https://ramdajs.com/docs/#cond
