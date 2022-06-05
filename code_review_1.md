# 新功能不應該影響舊功能

## 前提是程式有被設計過

例子是一個後台操作的功能，比如點擊修改圖示，可以跳出修改框。改完表單後可以送出以完成修改。  
所以流程上會是：

1. 點擊修改圖示。
1. 顯示修改框。
1. 修改完後表單送出。
1. API 發送。

### 範例 1

```js
const request = onDialogSubmit.pipe(
  map((x) => ({
    a: x.a,
    b: x.b,
  })),
  // ... more opertors
  map((x) => api(x))
);
```

中間 `map()` 的行為可以想像是驗證後並取必要欄位作為 API 的 body，實際的程式碼可多可少。  
其他細節就不贅述，此區塊足以表達本文主旨。

## 新需求

新加的功能是 Audit Log，就是做任何修改都要附上原因。  
該設計者決定在表單送出按下時再跳出一個對話框，要求輸入修改原因。  
所以流程上會是：

1. 點擊修改圖示。
1. 顯示修改框。
1. 修改完後表單送出。
1. 顯示原因框。
1. 輸入完原因後表單送出。
1. API 發送。

且該原因會被透過 headers 的方式送出，而不是 body，所以會使用 `api()` 的第二個參數。

```js
api(body, headers);
```

## Code Review 前

### 範例 2

```js
const request = onAuditSubmit.pipe(
  map(([x, reason]) => {
    const headers = {
      "audit-log": reason,
    };
    return [
      {
        a: x.a,
        b: x.b,
      },
      headers,
    ];
  }),
  // ... all opertors need to be change
  map(([x, h]) => api(x, h))
);
```

因為 Audit 的對話框是在表單之後，而送出時除了要含自己的原因以外，還要含原本表單的資料，所以被設計成 array 含兩種資料的格式。  
因為格式改變了，所以原本表單送出後的處理流程也要跟著變。  
所以如果實際情況如果處理流程複雜，那麼這個新需求帶來的改變就會相對巨大。

## Code Review

首先原本功能好好的，為什麼要去改它？  
新的需求是多一個對話框，所以資料上就是多了 headers，不應該去影響 body 被組出來的流程。  
唯一合理會改到原本的程式的部分理論上只有 `api()` 的部分，因為原本只有 body，現在多了 headers。

## Code Review 後

### 範例 3

```js
const headers = onAuditSubmit.pipe(
  map(([, x]) => x),
  map((x) => ({
    "audit-log": x,
  }))
);

const request = onAuditSubmit.pipe(
  map(([x]) => x),
  map((x) => ({
    a: x.a,
    b: x.b,
  })),
  // ... all opertors no need to change
  withLatestFrom(headers),
  map(([x, h]) => api(x, h))
);
```

藉著[一切從最小單位出發]的心法，我們做一個只會拿到 headers 的 observable。  
而原本 body 因為依賴於對話框關閉時的結果，所以我們在前頭只取自己需要的部分，不要把新加的原因混進去，這樣就不用改到原本 body 的處理流程。  
接著我們在準備 `api()` 前透過 `withLatestFrom()` 或是 `combineLatestWith()` 來從中插入 headers。  
這樣就可以不用改到原本就有的程式邏輯，只要在組合的流程上動手腳就可以了。

## 附註

這邊有個前提是程式本身有被設計過。  
如果你發現你有這樣的想法卻執行不了，有可能是舊有程式碼設計的不夠好，導致後來需求的增減會有所困難。  
所以平常寫程式時就要習慣把單位抽好，比如上述的 body 就是一個獨立的資料，應該抽成獨立的 observable。  
那麼 request 就可以很明確的知道它的依賴性。

### 範例 4

```js
const request = zip(body, headers).pipe(
  map(([x, h]) => api(x, h))
);
```

[一切從最小單位出發]: https://github.com/QimatLuo/best_practice/blob/main/atom.md
