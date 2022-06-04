# 設計可被組合的 function 們

## pipe

如果你有開始抽象你的實作的話，理論上會有很多小 function，但是組合的過程中並沒有想像中好閱讀。  
比如說我們來透過四則運算組合出梯形面積的公式好了。

### 範例 1

```js
const add = (a) => (b) => a + b;
const mul = (a) => (b) => a * b;
const div = (a) => (b) => a / b;

const f = (u) => (d) => (h) => div(
  mul(
    add(u)(d)
  )(h)
)(2);

console.log(f(2)(3)(4)); // 10
```

會看見所有 function 以巢狀的方式包在一起，閱讀上必須找到最裡層的 function 才有辦法讀出它是做"(上底 + 下底) \* 高 / 2"的運算。  
如果我們能把程式寫出跟公式一樣的閱讀順序就會好很多。

### 範例 2

```js
const add = (a) => (b) => a + b;
const mul = (a) => (b) => a * b;
const div = (a) => (b) => b / a;

const pipe = (...[x, ...fs]) => fs.reduce((v, f) => f(v), x);

const f = (u) => (d) => (h) => pipe(
  u,
  add(d),
  mul(h),
  div(2)
);

console.log(f(2)(3)(4)); // 10
```

只要我們生一個 `pipe()` 幫我們把帶入的一堆 function 依序執行就可以了。

這邊除了多了 `pipe()` 以外，我還偷偷改了 `div()` 讓它參數的順序互換。  
因為我最先知道的是除以 2 這個行為，如果不改的話則會變成是 2 除以 N 的狀況。

## flip

當出現需要順序反過來的情況，我們不會把所有 function 都寫出另一組一模一樣但順序調換的版本。  
這時候我們只要做一個專門幫你順序調換的 function 就好了。

### 範例 3

```js
const flip = (f) => (a) => (b) => f(b)(a);

const add = (a) => (b) => a + b;
const add2 = flip(add);

console.log(add("Qimat")("Luo")); // "QimatLuo"
console.log(add2("Qimat")("Luo")); // "LuoQimat"
```

另外也有 placeholder 的作法，詳情參閱 [Ramda.__]。

## Transducers

當我們學會了 FP 的概念，開始都用 function 處理所有事情，想必也很習慣用 array 天生支援的 function。  
這樣子雖然好閱讀，可是如果處理的 array 比較大的話就可能會發生效能問題。

### 範例 4

```js
const mul = (a) => (b) => console.log("mul") || b * a;
const gt = (a) => (b) => console.log("gt") || b > a;
const add = (a, b) => console.log("add") || b + a;

const xs = [1, 2, 3, 4, 5];

xs
  .map(mul(2))
  .filter(gt(5))
  .reduce(add, 0);

/*
mul
mul
mul
mul
mul
gt
gt
gt
gt
gt
add
add
add
*/
```

可以看見 `map()` 時跑了 5 次迴圈， `filter()` 也跑了 5 次迴圈，`reduce()`跑了 3 次迴圈，總共是 13 次迴圈。  
原因是每個 array function 都是把整個內容跑完後才進入下一個 function。  
有些 library 有支援 transducer 的功能來避免這樣的問題，比如 [Ramda.transduce]。

### 範例 5

```js
import { compose, filter, map, transduce } from "ramda";

const mul = (a) => (b) => console.log("mul") || b * a;
const gt = (a) => (b) => console.log("gt") || b > a;
const add = (a, b) => console.log("add") || b + a;

const xs = [1, 2, 3, 4, 5];

const f = compose(
  map(mul(2)),
  filter(gt(5))
);

transduce(f, add, 0, xs);

/*
mul
gt
mul
gt
mul
gt
add
mul
gt
add
mul
gt
add
*/
```

可以看見跑的順序跟寫 `for` 一樣，只會跑 5 次迴圈就可以結束了。

那這樣為什麼不寫 `for` 呢？ 只要能把動作一行一行寫，也是一樣很好閱讀啊。

### 範例 6

```js
const mul = (a) => (b) => console.log("mul") || b * a;
const gt = (a) => (b) => console.log("gt") || b > a;
const add = (a, b) => console.log("add") || b + a;

const xs = [1, 2, 3, 4, 5];

let sum = 0;
for (let n of xs) {
  const x = mul(2)(n);
  if (!gt(5)(x)) continue;
  sum = add(sum, x);
}
```

別忘了我們還是希望不要有竄改變數的寫法，因為程式一但複雜起來就容易不小心埋下陷阱。  
另外寫 `for` 或是 `if`，甚至是 `+`, `?`, `!`, `<` ... 等，都是語言本身的語法，無法被共用。  
所以我們才透過 function 去把這些功能包裝起來，然後再透過組合的方式去實現所需的功能。

## Observable

但 transducer 似乎不是很容易懂，有沒有更直覺的方案去處理這個問題呢？  
當然有，像是我目前愛用的 RxJS 內建這個效果。

### 範例 7

```js
import { from, map, filter, reduce } from "rxjs";

const mul = (a) => (b) => console.log("mul") || b * a;
const gt = (a) => (b) => console.log("gt") || b > a;
const add = (a, b) => console.log("add") || b + a;

const xs = [1, 2, 3, 4, 5];

from(xs)
  .pipe(
    map(mul(2)),
    filter(gt(5)),
    reduce(add, 0)
  )
  .subscribe();

/*
mul
gt
mul
gt
mul
gt
add
mul
gt
add
mul
gt
add
*/
```

RxJS 不但內建 transducer，而且還能同時處理 sync 跟 async 的資料，只要全部都轉換成 observable 就可以了。  
第一次接觸 RxJS 的人可以使用 [Operator Decision Tree] 幫助你快速了解個情境之間應該使用哪種 operator 來處理。

[ramda.__]: https://ramdajs.com/docs/#__
[ramda.transduce]: https://ramdajs.com/docs/#transduce
[operator decision tree]: https://rxjs.dev/operator-decision-tree
