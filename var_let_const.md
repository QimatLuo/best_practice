# var? let? const?

從某個時代以後，可能會看到一些文章寫說不要再用 `var` 了，一律用 `let` 取代。  
確實用 `let` 可以避免掉一些神奇的問題，但我會說：一律用 `const`。

## 魔法般的 var

### 範例 1

```js
var a = "qimat";

function test() {
  console.log(a);
}

test(); // "qimat"
```

先看看這個例子，呼叫 `test()` 之後會印出字串 `qimat`，很正常。  
那如過這樣呢？

### 範例 2

```js
var a = "qimat";

function test() {
  console.log(a); // undefined
  var a = "luo";
  console.log(a); // "luo"
}

test();
```

會發現原本應該印 `qimat` 的地方卻變成了 `undefined`，導致程式運作不正常。  
其實是因為 `var` 實際運作時會長得像這樣。

```js
var a = "qimat";

function test() {
  var a;
  console.log(a); // undefined
  a = "luo";
  console.log(a); // "luo"
}

test();
```

要避免這種事只要用 `let` 就好了，這就是為什麼會有一律用 `let` 取代 `var` 的說法。

```js
let a = "qimat";

function test() {
  console.log(a); // ReferenceError: Cannot access 'a' before initialization
  let a = "luo";
  console.log(a);
}

test();
```

程式碼會報錯，逼得你一定要寫正確，藉此預防你運作不預期的程式。

## 嚴謹的 const

那為什麼我說一律用 `const` 呢？  
因為除了有跟上述一樣的效果以外，還多出了一些效果。

### 範例 3

```js
let a = "qimat";

function test() {
  a = "luo";
  return a;
}

console.log(test()); // "luo"
```

我改了一些小地方，讓 `test()` function 裡面"忘記"用 `let` 宣告，導致使用了 [closure] 的效果去用到了外層的同名變數 `a` 。  
雖然程式運作正常，但是如果隨著開發一直增加程式就有可能...

### 範例 4

```js
let a = "qimat";

function test() {
  a = "luo";
  return a;
}

function getQimat() {
  return a;
}

console.log(getQimat()); // "qimat"
console.log(test()); // "luo"
console.log(getQimat()); // "luo"
```

有個不尋常的地方是，為什麼同樣是 `getQimat()` 第一次是 `qimat` 第二次卻是 `luo` 呢？  
因為在執行 `test()` 時，這個 function 偷偷把外面的 `a` 改了。  
雖然如果你是故意設計成這樣的話，那當然沒錯，但我不建議這樣做。  
因為實際例子來講，程式碼通常會很長，不是一目瞭然。  
隨著程式碼慢慢的變多，我們沒辦法一瞬間就看出有個 function 會偷偷改大家共用的東西，導致不預期的結果發生。

實例上最常犯的錯誤就是最外面那個 `a` 是個像 config 的設定，各自的 function 會依賴於 config 的設定而有所變化。  
但由於某人因需求貪圖方便去改了它，或是如上述用到同名且忘記宣告而導致不小心竄改了 config 進而讓別人產生 bug 。  
要避免這種事只要使用 `const` 就可以了。

```js
const a = "qimat";

function test() {
  a = "luo"; // TypeError: Assignment to constant variable.
  return a;
}

function getQimat() {
  return a;
}

console.log(getQimat()); // "qimat"
console.log(test());
console.log(getQimat());
```

程式會報錯告訴你 `a` 是個 `const` 所以不能被竄改。  
這樣就能避免上述那些悲劇發生，所以用 `const` 理論上是多一層保護，有好無壞。

## 用相對好的方法去寫程式

除此之外還能幫你限縮可能性，明確方向。  
因為要透過程式完成一件事，有太多種可執行的方式了，你要如何評斷用哪一種方式來完成比較好？  
當你面試一個人請求完成一個有加總功能的 function，隨著面試的人越來越多，版本就越來越多。  
雖然每一種版本都可以達到需要的效果，但總是能從不同版本之間挑出相對好的版本去感受到這個人對寫程式比較有經驗。  
比如說...

### 範例 5

```js
function sum() {
  let total = 0;
  for (let i = 0; i <= 5; i++) {
    total = total + i;
  }
  return total;
}
console.log(sum()); // 15
```

如果你能認同盡量不要用 `let` 那該怎麼做呢？ `const` 也做不到吧？

### 範例 6

```js
function sum() {
  return Array(5)
    .fill()
    .reduce((a, _, i) => a + i + 1, 0);
}
console.log(sum()); // 15
```

沒見過這種寫法的人可能會很疑惑這是怎麼運作的，可以參考官方文件 [fill], [reduce]。  
因為 `reduce()` 幫你把"從多變一"的效果做出來了，你只要在裡面的 function 執行加總的邏輯即可，這樣就可以避免用到 `let` 的方式去竄改值去加總。

不要因為加總這個範例太簡單而使用 `let` 的方式去實作，因為這會養成習慣造成你思考方向都是以竄改值的方式去完成，導致以後遇到複雜的情境容易寫出容易產生問題的程式碼。  
所以只要要求變數宣告一率用 `const` ，你會發現在某些情況下無法去處理，就可以逼你去尋找不一樣的解決方式來應對。  
如果不逼自己去增廣見聞的話，可能永遠都在使用最基礎的方式去處理問題，儘管最終問題都能得到解決，但相對的也可能對未來改這份程式的人造成負擔。  
所以我喜歡把**一律用 const**作為一個寫程式的原則，只要把握這個原則就可以避免上述 `var` 跟 `let` 的問題，且在 `const` 做不到的情況下去學習不一樣的方式去寫程式。

## 附註

在範例 5 中，也是有用 `const` 的解法。

### 範例 7

```js
function sum() {
  const total = [0];
  for (let i = 0; i <= 5; i++) {
    total[0] = total[0] + i;
  }
  return total[0];
}
console.log(sum()); // 15
```

可以透過 array, object 或是任何奇技淫巧去完成。  
它可以不被 `const` 受限的原因是因為被竄改的是物件裡面的值，而不是變數本身。  
如果是改變變數本身就可以被偵測到。

```js
const a = {};
a = {}; // TypeError: Assignment to constant variable.
```

所謂"一律用 const"這個原則本意並不在 `const` 本身，而是希望你不要去寫出 **竄改** 的這個行為的程式碼。  
所以範例 7 中雖然用 `const` 去解題，卻是不被我接受的一種解法。  
以精確的方式應該是說：**不要竄改變數**。  
只是這句話對於一個沒有經驗的工程師來說可能只知其然而不知其所以然，所以"一律用 const"這是相對容易給初學者去遵從的原則。

另外上述明明是說"物件"，可是範例明明是 array，是因為 array 也是物件的一種，請參閱 [typeof] 。

```js
typeof []; // "object"
typeof {}; // "object"
```

若想讓物件有 `const` 的效果，可以使用 [Object.freeze]。

[closure]: https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Closures
[fill]: https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Array/fill
[reduce]: https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce
[typeof]: https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Operators/typeof
[object.freeze]: https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze
