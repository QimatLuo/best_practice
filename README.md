# 不負責任聲明

在這邊宣導一下我個人寫程式的偏好。  
很明顯就是"個人"偏好，有人可能覺得誤人子弟，但如果不介意就請耐心看下去。  
沒有一個規則是可以符合所有人想要的樣子，所以大概就只是個分享。  
如果有對文中感到疑慮之處歡迎開 Issue 討論。

本系列主打維護性，所以設計方向不會以優化或速度的方向去走，而是以可讀性，可擴充性為目的。  
所謂**可擴充性**就是解法應該是一層一層堆疊，沒被堆疊過的解法都可以拿來處理對應的事情。透過堆疊的方式就可以組出另一種解法來解決新的問題，而不是優化了一種解法只能針對某一種情境去處理。  
所謂**可讀性**就是該程式的結果是可被預測的，當你讀越少部分的程式碼就可以推敲出整個程式的執行結果，那就代表它有更好的可讀性。

其中可讀性可能被誤解成別的意思，就是"你的程式我看不懂"就代表可讀性不高，這是**錯誤**的定義。  
在我個人的觀點，看不懂的程式分兩種：

1. 每塊程式碼我都看得懂，但我就是看不懂你程式是在寫什麼。這就是程式設計的不好，流程複雜、到處竄改，需要在腦中記住運算的結果來接續其他塊的程式碼導致不易閱讀。
1. 沒概念，不知從何看起，像天書一樣。這就是道行不夠，可能是 library 不熟，沒接觸過這種程式設計的方式等。

如果是情況 1 基本上可以判定是設計不良，當然批評別人以前你要先能提出更好的設計來說服別人。  
如果是情況 2 則是必須去了解 library 或是該設計模式，一開始當然會不好讀，畢竟沒見過。了解後才能去評斷這樣的設計好或不好。
所以看不懂的程式碼不要急著評論它或是拒絕它，會這樣寫一定是有它的原因，至於原因是好是壞得先去理解對方再去評論會比較好。

# 程式設計

## 概念

1. [你還在寫 let？](https://github.com/QimatLuo/best_practice/blob/main/var_let_const.md)。(不要竄改變數)
1. [你還在寫 長的變數名？](https://github.com/QimatLuo/best_practice/blob/main/xyz.md)。(抽象化你的思考方式)
1. [你還在寫 註解？](https://github.com/QimatLuo/best_practice/blob/main/curry.md)。(程式本身就是註解)
1. [你還在寫 if？](https://github.com/QimatLuo/best_practice/blob/main/if.md)。(function 不是抽出去就好了)
1. [你還在寫 for？](https://github.com/QimatLuo/best_practice/blob/main/transducers.md)。(任何事都用 function 解決)
1. [你還在寫 this？](https://github.com/QimatLuo/best_practice/blob/main/this.md)。(學一套，到處用)
1. [我愛 RxJS](https://github.com/QimatLuo/best_practice/blob/main/rxjs.md)。(統一窗口)
1. [一切從最小單位出發](https://github.com/QimatLuo/best_practice/blob/main/atom.md)。(讓你的 observable 可被隨意組合)
1. [Rx 界的 curry](https://github.com/QimatLuo/best_practice/blob/main/higher_order.md)。(Observable 的前處理)

## 實例

- [避免改動原本邏輯](https://github.com/QimatLuo/best_practice/blob/main/code_review_1.md)。
- [避免竄改變數](https://github.com/QimatLuo/best_practice/blob/main/code_review_2.md)。

# Git

1. [你有沒有拆 commit 呢？](https://github.com/QimatLuo/best_practice/blob/main/commit.md)。(留後路)
1. [不必是當事人也可以正確地解衝突](https://github.com/QimatLuo/best_practice/blob/main/conflict.md)。(diff3)
