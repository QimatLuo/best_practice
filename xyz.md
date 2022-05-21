# 變數 a, b, x, y 其實沒有不好

## 賦予程式意義

在上次的[範例 6]中，我們開啟了新的寫程式的方式。

```js
function sum() {
  return Array(5)
    .fill()
    .reduce((a, _, i) => a + i + 1, 0);
}
console.log(sum()); // 15
```

但這版還是有令人困惑兩件事是 `a + i + 1` 跟 `reduce(..., 0)` 。  
原因是因為我們生出了長度 5 的 array ，而 array 的 index 是從 0 開始，所以 `i` 分別會是 0 ~ 4，若沒有 `i + 1` 的話實際會是 0 + 0 + 1 + 2 + 3 + 4 = 10 。  
前面的兩個 0 並不是多打，是因為 `reduce(..., 0)` 這個起始值也是必要的，不然就會變 `undefined` 加數字而最終變成 `NaN` 。

我們可以透過 [map] 來處理 `i + 1` 的問題。

### 範例 1

```js
function sum() {
  return Array(5)
    .fill()
    .map((_, i) => i + 1)
    .reduce((a, b) => a + b, 0);
}
console.log(sum()); // 15
```

雖然大致上基本沒有太大變化，但是意義上就有所不同。  
如果我們一行一行閱讀，讀到 `map()` 那行的時候可以理解 `i + 1` 的問題，就可以預期得到一組 1 ~ 5 的數字。  
而下一行的 `reduce()` 可以很輕鬆的理解就是單純兩兩相加 `a + b` 且起始值為 0。
這個行為可以避免 `a + i + 1` 這種一個程式區塊處理多個問題的現象，讓各區塊個程式碼各司其職使其容易閱讀。

只要你能賦予程式意義，那麼你就可以更知道你希望寫出什麼樣的程式，或是更能理解、推斷、抉擇你所看見的程式碼。  
比如要達到 `Array(5).fill()` 的效果也是有一些變化。

```js
Array(5).fill(); // [undefined x 5]
Array.from({ length: 5 }); // [undefined x 5]
```

這兩種寫法都可以產生出長度 5 的 array，差別在語意有些不同。  
`Array.from({ length: 5 })` 語意上就是單純產生出 array。  
因為沒有帶第二個參數所以預設值都是 `undefined` ，可參閱 [Array.from]。  
而 `Array(5).fill()` 其實是兩句話，產出預期有 5 個位置，並且把這些位置塞入 `undefined` (因為沒有帶值)。  
如果我們沒有執行 `fill()` 的話，則這個 array 其實是沒東西的，無法迭代。

```js
function sum() {
  return Array(5)
    .map((_, i) => i + 1)
    .reduce((a, b) => a + b, 0);
}
console.log(sum()); // 0
```

所以如果我們希望下一句話 `.map((_, i) => i + 1)` 不要有個隱晦的 `_` 無用變數去影響解讀的話。  
那我們可以用 `fill(1)` 塞入預設值，這樣就避免那個無用變數。

### 範例 2

```js
function sum() {
  return Array(5)
    .fill(1)
    .map((v, i) => v + i)
    .reduce((a, b) => a + b, 0);
}
console.log(sum()); // 15
```

當然也可以使用 `Array.from()` 把兩句話寫在一起。

### 範例 3

```js
function sum() {
  return Array
    .from({ length: 5 }, (_, i) => i + 1)
    .reduce((a, b) => a + b, 0);
}
console.log(sum()); // 15
```

一切就是看你想怎麼表達而已。

若你沒有偏好的表達方式，當然就是兩者都可以。  
但如果把其他事情考慮進來，在這個情境上你應該會跟我一樣選擇用 `fill(1)`。  
因為這樣可以產生出一模一樣的程式碼 `(v, i) => v + i`, `(a, b) => a + b`，進而生出共用的 function 。

### 範例 4

```js
function f(a, b) {
  return a + b;
}

function sum() {
  return Array(5)
    .fill(1)
    .map(f)
    .reduce(f, 0);
}
console.log(sum()); // 15
```

## 賦予正確的意義

如果一時之間你沒發現這兩個 function 是一模一樣的話，就代表你不夠抽象思考，或是賦予了程式錯誤的意義。  
之所以我們會寫成 `.map((v, i) => v + i)` 是因為我們知道 Array 的 map function 會帶給你三個參數分別是 value, index, array 以至於我們這樣命名。  
那是因為我們把外面的 map 跟裡面的 function 一起解讀，所以就理所當然的把 function 裡的 arguments 照著文件的方式去命名。

以文件的角度來看，它為了表示"我給你了什麼"當然要這樣明確的表達。  
但以我們自己寫的程式來看，應該只注重在裡面我們自己寫的部分 `(v, i) => v + i`，這時候就可以發現其實 `v` 是什麼、 `i` 是什麼並不是很重要，最重要的點是在我們做了 `+` 的這個動作。  
所以此時不應該用 i 這種會引導人去想像它可能會是 number 型態的值，而是用 a, b, c, x, y, z 這種比較沒有意義的方式去命名更好。  
因為語意上我們真實要表達的是 `+` 的行為，不應該被其他因素去讓人誤解你是要 number 相加或是 string 相加之類的語意。  
如果你要表達是 number 相加的話，那程式應該是這樣寫。

```js
function add(n1, n2) {
  if (typeof n1 !== "number" || typeof n2 !== "number")
    throw "arguments should be number";
  return n1 + n2;
}
```

這樣賦予程式的意義才是相對精確的。  
所以如果變數本身並不限制型態，那我們不應該引導別人去做型態限制的想像。  
如果你能認同上述的道理，那麼可以開始把你無型態的變數去賦予無型態的命名，或許會發現很多地方都是可以共用的程式碼。

實例上最常見的就是：

### 範例 5

```js
function getCatNames(cats) {
  return cats.map((cat) => cat.name);
}

function getDogNames(dogs) {
  return dogs.map((dog) => dog.name);
}
```

Cats 裡層取名 cat，dogs 裡層取名 dog，看起來非常正常，users, members ...etc 都依樣畫葫蘆。  
但由於這兩個是不一樣的東西，所以程式可能寫在不同地方，然後整個專案事實上都在重複一樣的事情都沒發現，只是因為命名限制了你的想像。  
除此之外也會讓一些工具幫不了你，像是 [SonarCloud] 可以幫你檢查出散落在不同檔案之間重複的程式碼。  
但因為你命名不同的關係，導致工具無法判別他們之間是一模一樣的，這時候如果你統一用 x, y, z 命名就不會有這種困擾。

## 但這樣都是一堆 x, y, z，我都看不懂了怎麼辦？

其實你不太需要去看那些程式碼，因為被抽象後的東西都會像 library 一樣藏在後面，不需要去看它。

```js
/* -- helper.js -- */
const getName = (x) => x.name;
const getNames = (xs) => xs.map(getName);

/* -- main.js -- */
const catNames = getNames(cats);
const dogNames = getNames(dogs);
```

所以事實上要了解一個專案裡的程式怎麼跑，你只要讀 main.js 就有足夠的語意讓你了解在做些什麼。  
而對底層有興趣的人可以再更深入地去看 `getNames()` 到底是什麼怎麼做的，且進去到 helper.js 看的時候不會被限制說這個 function 只能被用於 cat 或 dog。

[範例 6]: https://github.com/QimatLuo/best_practice/blob/main/var_let_const.md#%E7%AF%84%E4%BE%8B-6
[map]: https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Array/map
[array.from]: https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Array/from
[sonarcloud]: https://sonarcloud.io/
