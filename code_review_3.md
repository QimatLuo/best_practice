# RxJS 的遞迴

## 緣由

今天同事群組裡面分享了一個連結是 Facebook [Angular Taiwan] 的社團。  
當時點進去看見的文章是 [用 RxJS 翻轉你的 coding 人生 - 以 Timer 為例] 裡面提供了一個 [範例]。  
剛好我在這方面也有一些自己的見解可以分享一下。

## 定義需求

由於文章內的範例也是大概，避免大家看程式的重點失焦，先把重點放在**背景自動刷新頁面**，並配合範例來思考。  
所以我認為情況大概是：

- 有一個區塊顯示的內容需要依賴於 API。
- 該區塊內容希望每 N 秒更新，該範例為 30 秒。
- 可以手動更新，手動更新時會重置該 N 秒。

想像起來大概就是有個像小儀錶板一樣的東西在顯示，該儀表板有個小更新按鈕可以馬上更新，沒人動的話每 N 秒會自動更新。  
所以 counter 其實不是重點，只是達到目的的其中一種作法，以下提供不用 counter 的做法。

## 設計概念

我個人的設計概念是 UI 上的一個變數只會有一個對應的 observable。  
因為如果有多個地方會去影響該 UI 變數呈現的方式，那麼到時候 debug 需要掃全篇的程式碼找關鍵字看誰影響了這個變數，所以保持一對一的好處就是只要追著這個 observable 跑就是所有該 UI 變數的可能性。

先模擬一個打完 API 後把資料顯示在畫面上的樣子。  
因為 API 可能會有動態參數要帶，所以設計成 function，而不是直接一個 observable，如果你的 API 是固定參數的話就可以直接用 observable。

### 範例 1

```ts
this.api()
  .subscribe((x) => {
    this.ui = x;
  });
```

然後透過 expand() 做 N 秒遞迴。

### 範例 2

```ts
this.api()
  .pipe(
    expand(() =>
      timer(3000).pipe(
        switchMap(() => this.api())
      )
    )
  )
  .subscribe((x) => {
    this.ui = x;
  });
```

接著就是神奇的地方了，我們要用 race()。  
目標是手動更新時希望取消原本自動更新的行為，以程式碼來看自動更新就是 expand() 那個區塊。  
但事實上我們想取消的不是整個自動更新，因為手動完之後還是會回到自動模式，所以真相其實是我們想取消自動更新的**等待時間**，也就是 timer()。  
所以只要讓 timer() 跟手動的 event 比賽誰快就好了。

### 範例 3

```ts
this.api()
  .pipe(
    expand(() =>
      race(timer(3000), this.refresh).pipe(
        switchMap(() => this.api())
      )
    )
  )
  .subscribe((x) => {
    this.ui = x;
  });
```

用 expand() 有個重點要注意，裡面的 observable 是要能 complete，不然會在內層產生越來越多個 next 在這個流。  
而此範例的 this.refresh 就是一個聽 UI 點擊事件的 Subject，基本上是不會 complete，所以需要加上 take(1) 來避免溢流。

### 範例 4

```ts
this.api()
  .pipe(
    expand(() =>
      race(timer(3000), this.refresh).pipe(
        switchMap(() => this.api())
      )
    ),
    take(1)
  )
  .subscribe((x) => {
    this.ui = x;
  });
```

其實精確一點應該把 take(1) 掛在 this.refresh 上，因為真正不會 complete 是它。

### 範例 5

```ts
this.api()
  .pipe(
    expand(() =>
      race(timer(3000), this.refresh.pipe(take(1))).pipe(
        switchMap(() => this.api())
      )
    )
  )
  .subscribe((x) => {
    this.ui = x;
  });
```

## 為何不用 interval()？

因為 interval() 是一個不斷發送的事件，它其實不會管你 interval 後面做了什麼事情。

### 範例 6

```ts
interval(1000).pipe(
  tap((x) => console.log("interval", x)),
  switchMap(() => this.api())
);
```

如果 this.api() 超過 1 秒才會回來，那麼他就永遠不會動了，因為 switchMap() 會不斷地取消還沒完成的請求。  
換成 mergeMap() 的話確實就可以避免這個問題，但取而代之的問題就是 UI 會亂跳，因為 API 回應速度不固定，可是又固定每秒發送請求。  
那 concatMap() 總該可以了吧？沒錯，在這個情境下運作會很正常，但背後其實 interval() 已經多跑了好多東西，假設 API 要花 5 秒，那麼 interval() 會是在 0, 5, 10, 15 才會發送這 4 個請求，所以其實是對不上的，因為都被 concatMap() 佇列起來了。  
雖然佇列的請求行為都是正確的，基本上實際運用是不會有什麼問題，但如果請求跟 interval() 之間還會多做一些什麼的話，那有可能佇列在那裏的都是多做的，也是某種程度上的浪費。  
可以改用 exhaustMap()，這樣就不會有被浪費的佇列情為產生，運作也正常。唯一要考量的是如果流程上有依賴於實際 interval() 的 counter 就會有問題，所以要拿捏一下。

以上種種個人還是建議用 expand() 來實作，才不會被 interval() + xxMap() 所產生的特例暴雷。  
雖然 interval() 在想法上比較容易建構，透過 expand() 實作只要換個思考方式就可以完成，應該不會太難，且只要確保內部的流會完成就好了，不能確保的話無腦一律加上 take(1) 其實也沒差。

[angular taiwan]: https://www.facebook.com/groups/augularjs.tw/?multi_permalinks=5864813023529019
[用 rxjs 翻轉你的 coding 人生 - 以 timer 為例]: https://blog.leochen.dev/2022/09/29/timer-sample-in-rxjs/
[範例]: https://jsbin.com/sacikupapo/1/edit?html,js,console,output
