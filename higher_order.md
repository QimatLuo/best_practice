# Higher-order Observables

## 需要 loop 生出很多 observable 時會發生的事情

RxJS 的 operator 會有類似的名字但是用於不同情境，比如 `retry()` 是給錯誤時用的，`repeat()`是給完成時用的，會有一系列錯誤的後處理跟完成的後處理。  
還有一些像是 `merge()`, `mergeMap()`, `mergeAll()` 或是 `concat()`, `concatMap()`, `concatAll()` 之類的，你可能發現 xxAll 這種 operator 很少用到。  
如果是的話，那本篇就要帶你進入用 xxAll 來處理 [higher-order observables] (之後簡稱 HOO) 的世界。

假設我們透過 API 去要一個清單只能要到一堆 id ，細節必須拿 id 再去打別的 API 來取得的時候就會長這樣：

### 範例 1

```js
from(listApi()).pipe(
  mergeMap((xs) => from(xs).pipe(
    mergeMap((x) => detailApi(x))
  ))
);
```

透過 `mergeAll()` 我們可以把程式碼變得更簡潔。
因為 array 也是 [Observable Input]，所以可以被 `mergeAll()` 攤平。

```js
from(listApi()).pipe(
  mergeAll(),
  mergeMap((x) => detailApi(x))
);
```

再簡潔的話就是把 `(x) => detailApi(x)` 做 [pointfree]。

```js
from(listApi()).pipe(
  mergeAll(),
  mergeMap(detailApi)
);
```

如果要呈現出 HOO 的話就要變成：

### 範例 2

```js
const hoo = from(listApi()).pipe(
  mergeAll(),
  map(detailApi)
);

hoo.pipe(
  mergeAll()
);
```

變成這樣有什麼用呢？就是讓可重複使用的流做前處理。  
因為我們把 `pipe()` 裡面的步驟抽掉像[上一篇的範例 3] 的 `values` 給別人使用的話，那麼只能拿到流動後的資料，所以只能做後處理。  
如果你有需求要做開始流動前的處理，就得學會 HOO 的技巧，比如說 API 發送時 loading 的狀態。

## loader

先來定義我們希望的 loading 呈現的方式。  
第一次發 `listApi()` 時必須呈現 loading，API 完成後解除。  
接下來把所有 id 都發送 `detailAPI(id)` 時必須呈現 loading，全部都完成後解除。

簡單講就是任意 API 發送過程就是要 loading，沒有在等待 API 的情況可以解除。  
當然做法有很多，我這邊選擇用數字加減的作法去處理：

1. 起始值為 0。
1. 任意 API 發送時 +1。
1. 任意 API 完成時 -1。
1. loading 狀態等於現在的數字轉成布林值，也就是只有歸 0 時 loading 才會消失。

這時候我們就需要做到前處理的效果：

### 範例 3

```js
const hoListApi = of(listApi().pipe(share()));
const hoDetailApi = hoListApi.pipe(
  mergeAll(),
  mergeAll(),
  map(detailApi),
  map((x) => x.pipe(share())),
  share()
);

const hoRequests = merge(hoListApi, hoDetailApi);

const loading = hoRequests.pipe(
  mergeMap((obs) => merge(of(1), obs.pipe(map(() => -1)))),
  scan((a, b) => a + b, 0),
  map(Boolean)
);

const items = hoDetailApi.pipe(
  mergeAll(),
  scan((a, b) => a.concat(b), [])
);
```

這時候我們會先分流出 list 跟 detail 的 HOO。  
裡面會有 `share()` 是因為 `loading` 跟 `items` 都會用到這些 HOO，所以必須透過 `share()` 來避免發出重複的 API request。

而前處理的部分就是 `loading` 的實作方式。  
它在收到 `obs` 時會先 `merge()` 一個 `of(1)` 來做 API 發送時的 +1 效果。  
且定義 `obs.pipe(map(() => -1))` 來讓 API 完成時做 -1 的效果。  
最後就是透過 `scan()` 來做加總去實時呈現 loading 的狀態。  
所以如果 loading UI 想要更潮還可以設計成顯示正在等多少數量的 API 也沒問題。

這邊可以把 loader 的實作抽成 [custom operators]，這樣所有 component 都可以按照這個模式去實作。

1. 先定義這個 component 會用到那些 API。
1. 把這堆 API 用 `const hoRequests = merge(...);` 蒐集起來。
1. 使用 loader 去處理 loading 的狀態 `const loading = hoRequests.pipe(loader());`。

完全是可以應付大部分情況的設計，甚至是錯誤處理也可以按照同樣的模式去做，只要任意 API 發生錯誤就會被偵測出來。  
所以 HOO 的觀念可以讓你的流更有彈性，是一個必學的技巧。

## 附註

HOO 有個不習慣的地方就是會看到連續 `mergeAll()` 的情況，其實這跟 higher-order function 的 curry function 的連續兩組小括號是同樣的道理，不用把它當成異類。

本範例的 loader 實作僅用於傳達 HOO 的概念，並沒有實作錯誤以及取消的處理。  
有興趣的人可以依照這個概念去做一些強化。若你還沒有能力的話，可以到小工具(TODO:補連結)區去取用。

[higher-order observables]: https://rxjs.dev/guide/operators#higher-order-observables
[observable input]: https://rxjs.dev/api/index/type-alias/ObservableInput
[pointfree]: https://mostly-adequate.gitbook.io/mostly-adequate-guide/ch05#pointfree
[上一篇的範例 3]: https://github.com/QimatLuo/best_practice/blob/main/atom.md#%E7%AF%84%E4%BE%8B-3
[custom operators]: https://rxjs.dev/guide/operators#creating-custom-operators
