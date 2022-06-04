# 你以為抽象就是把一樣的程式抽成 function？

## 是這樣沒錯，但那只是第一步

常見的就是**新增**跟**編輯**這兩個功能幾乎長得一模一樣，我們這次用 UI 的 component 當成 function 來解釋比較容易懂。  
所以現在我們發現有兩個 component 都很像，只差在：

1. title 不一樣，一個是新增，一個是編輯。
1. 欄位不一樣，某些不能改的欄位只有新增時可用。
1. 背後打的 API 不一樣。

### 範例 1

```html
<header>新增</header>
<form>
  <input name="id" />
  <input name="name" />
  <button type="submit">Submit</button>
</form>
```

```html
<header>編輯</header>
<form>
  <input name="name" />
  <button type="submit">Submit</button>
</form>
```

所以這時候我們抽出了一個 share component (就像 share function 一樣)，但這個抽象要怎麼設計呢？  
如果你抽出來後給一個介面叫做 mode，然後傳 add 或 edit 進去，那這只會讓程式變得更難管理而已，而這也是最常見的錯誤。  
因為當你的抽象設計成這子，代表著那個 share component 裡面要寫 `if`，所以上述 3 點都要用 if mode 去處理，也就是事實上你要在一個 component 裡面寫 2 份邏輯。  
接著隨著需求越來越廣，我希望送出的按鈕不要叫 submit 或 ok 之類的，要明確說是 add, edit, update...etc 之類的明確字眼，這時候又多了一個寫 2 份邏輯的地方。  
最慘的是如果 UI 設計師說我們要有個**唯讀**的功能，也要長一樣，只要讓欄位都是不可編輯就好了。  
這時候你的 mode 就多了一個，原本到處都是 2 份邏輯的地方變成了 3 份邏輯。  
然後這個原本立意良好的 share component 就會變成後人最不想去碰的可怕 component。

```html
<header>{{ title }}</header>
<form>
  <v-if>
    <input />
    ...
  <v-else-if>
    <input />
    ...
  <v-else>
    <input />
    ...
  <button type="submit">Submit</button>
</form>
```

```js
{
  computed: {
    title() {
      switch(this.mode) {
        case 'add':
            return '新增';
          case 'edit':
            return '編輯';
          case 'read'：
            return '唯讀';
      }
    }
  }
}
```

## 抽象也是要設計的

上述的原因是因為 `if` 真是一個太好用太直覺的方法去處理事情了，但那也是萬惡的根源。  
被設計後的抽象是可以不用寫 `if` 的，比如上述的例子，有 3 處不一樣的地方，就應該設計 3 個介面給人家帶入所謂的不一樣。  
所以這時候**新增**要用這個 share component 只要把 title、欄位、API 帶進去就好了，**編輯**亦然。  
而 share component 幫忙處理的實際上是資料檢查，送 API 的動作，loading 擋 UI 等等，那些真的被共用的處理邏輯。
這樣一來就算新需求來了，就是按照新需求抽介面就好了，就算是**唯讀**要來也沒有問題。

### 範例 2

```html
<header>{{ title }}</header>
<form>
  <slot />
  <button type="submit">{{ btnText }}</button>
</form>
```

如果 UI 設計上不是不同欄位，而是同樣欄位只做 enable/disable，那麼重複性就會很高。  
這時候既然重複性本質是在 enable/disable，那麼抽象的就不是欄位，而是屬性值。  
經過上述的學習，我們不應該在抽象屬性值時讓 share component 用 `if` 去判斷是否該 enable/disable，而是讓用 component 的地方決定該屬性是什麼。

## function 的例子

上面是以 component 為例，但如果你還不能懂，我只能再舉一個單純 function 的例子。  
首先你會先發現有兩個 function 做了幾乎一模一樣的事情，比如說 SQL 的 update，然後大多都一樣，就是中間那個 `set number = n` 不一樣。  
這是一個 counter 的功能，所以 `n` 可能會是 +1 或 -1。

### 範例 3

```js
function increase() {
  return sql.get().then((x) => sql.set(x + 1));
}

function decrease() {
  return sql.get().then((x) => sql.set(x - 1));
}
```

所以當 function 抽出去時，不能在裡面寫 `if` 然後決定是 +1 或 -1，不然會發生上面假設過的悲劇。  
但是如果直接把加完或減完後的結果傳進來也有困難，因為所謂 +1 就是要先拿原本的值然後再加上去，所以會先有個 SQL query 拿到原本的值才能做加或減。  
這時候就可以把介面設計成讓別人帶 function 進來，然後這個 share function 會負責呼叫別人帶入的 function 並且帶入 SQL query 拿到的原本的值。  
被帶入的 function 會實際做 +1 或 -1 的動作，這樣我們就可以不用寫 `if` 來完成我們抽象的功能。

```js
function set(f) {
  return sql.get().then((x) => sql.set(f(x)));
}
```

這只是舉例，因為這例子的 counter 依賴於先 Get 後 Set，這兩個 async 的時間差會造成 race condition 的問題。  
所以實際上是不應該這樣實作的，SQL 相關的問題就得透過 SQL 處理，本篇就不詳述那一部分了。

## 附註

如果你舊有的程式是用 `if` 去處理的，不要氣餒。  
因為這絕對是第一步，只是你面對這一步的方式是用 `if` 而已。  
接下來你要做的是把 `if` 那一整塊設計成介面抽出去，讓別人把 `if` 裡面做的動作帶進來，這樣 share function 裡面就會很乾淨。  
在 component 的概念就是透過 slot 來完成這件事情。

所以不要太常依賴 `if`，就跟第一章不要依賴 `let` 一樣。  
儘管你這樣設計沒問題，但這習慣會造成你的思路習慣於用舊有的想法去處理事情，導致實際發生時難以應變。  
可以像不要寫 `let` 那樣當成一個原則去定義讓自己不要去寫 `if`，當你發現沒有自己寫 `if` 會困難時，那就是開始設計的時候了。  
這樣才能一步一步的成為程式設計師，而不是年復一年的碼農而已。
