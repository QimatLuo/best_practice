# JavaScript 最重要的 this

## 其實一點也不重要

當初學 JavaScript 的時候有一種不搞懂 `this` 在 JS 界就活不下去一樣。  
確實這東西很重要，因為在很多情況下 `this` 的行為會讓整個程式天差地別。  
但要講到 `this` 就會跟著講到 `apply`, `call`, `bind` ...etc.

本系列文章中很明顯就是推崇 FP，也可以看到所有事情就是 function 而已，不用扯到什麼 `this`。  
就算是逼不得已 framework 要讓我用到，也是放在最邊邊用最小的程度去使用而已，不需要深究 `this` 到底是怎麼運作的，所以不是很重要。  
當然也不是說 OOP 不好，只是 FP 真的比較好就是了。  
比如 OOP 最常一開始教的就是繼承，讓你有一個樣板然後類似的東西就可以從這個 class 去繼承過來。  
引用官方範例： [Sub classing with extends]
換成 FP 其實就是這樣而已：

### 範例 1

```js
function Animal(init = {}) {
  const o = {
    speak(x) {
      console.log(`${x} makes a noise.`);
    },
    ...init,
  };
  return (name) => ({
    speak: () => o.speak(name),
  });
}

const Dog = Animal({
  speak(name) {
    console.log(`${name} barks.`);
  },
});

let d = Dog("Mitzie");
d.speak(); // Mitzie barks.

const e = Animal()("Default");
e.speak(); // Default makes a noise.
```

先看結論，要繼承的 Dog 必須實作自己的 `speak()`，而生出 Dog 之後就可以用自己的 `speak()` 了。  
可以看見語言專用的繼承語法 class extends 不見了，語言專用的物件起始語法 new 不見了。  
這邊呈現出避開語言專用語法來達到程式可以被重用的效果，不管是同語言間的重複使用，或是不同語言之間的概念也可以重複使用。  
也可以看出 FP 版本做出來的效果，用法基本上沒差太多，那實際差在哪裡？

首先就是看不見 `this`，因為我們透過 closure 去處理這一塊的部分。  
接著我們不用 class，因為 object 本身就可以做到繼承的效果。  
因為不會憑空去得到 `this` 或者 `name`，所以 function 本身會有介面把需要的東西傳進去。

所以說其實不管是 OOP 或是 FP 都可以做到差不多類似的東西，所以基本上沒有誰特別好或誰特別差。  
一切比較都是有前提的，而本系列文章的前提就是不要有萬惡的 **竄改** 行為。  
OOP 為了避免不好的寫法當然也有一系列對應的處理方式： [OOP vs FP]  
但從比較表可以看到 OOP 透過各式各樣的設計模式去處理各式各樣的狀況，但 FP 只要用 function 就可以處理一切事情。  
唯一的條件就是 function 不能亂寫，而不能亂寫的條件就是讓你的 function 是 pure function 就好了，是不是很簡單？  
至於 pure function 要怎麼處理像上例的 `console.log()` 各種 side effect，只要再包一層 function 就好了，還是這麼簡單。  
詳情還是請另外找 FP 專門文章，這邊偏向實例範例的概念性闡述。

## 不可避免的 this

像是目前較流行的 Vue 或是 Angualr，大體上還是得透過 `this` 去處理事情。  
也不是說因為崇尚 FP 就一定得寫 React，畢竟公司用什麼 framework 不是我們能決定的。  
甚至某些 library 設計就是以竄改自身物件來達到方便的效果，所以實戰上免不了跟 `this` 打交道。

我們能做到的就是像 FP 那樣把 side effect 推到最邊邊，最後那一刻才讓 side effect 觸發。  
所以我們就把 `this` 相關的程式碼一樣縮到最小，過程盡量用 FP 去解決。  
這樣的好處是因為 FP 是 portalbe，哪天 framework 要換都不會是個問題，只要把最邊邊接 `this` 的部分換掉就好了。  
如果你的所有 utiliy 都綁死 Angular 的 module style，那麼搬移後的東西就要生出一個有跟 Angular 一樣的 DI 功能架構才行。  
如果你的做法都綁在 Vue 的 methods, computed，那麼搬移後的 function 大部分都得翻過一次。  
唯有 pure function 才是保證無痛轉移的設計。  
所以不管在什麼工作環境下，就算是在 OOP 裡面工作，FP 的概念還是可以持續使用的。

## 附註

雖然換 framework 可能不常有，可是 framework 升級如果有重大改變也是不可避免的。  
而上述的情境只是其中的假設，主要還是在於寫程式的概念本身。  
function 是各種語言基本上都會有的東西，所以理當 FP 學好就同等於萬用。  
如果學一種概念就可以萬用，那為什麼要學其他不能萬用的概念呢？

[oop vs fp]: https://blog.cleancoder.com/uncle-bob/images/fpvsoo.jpg
[sub classing with extends]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes#sub_classing_with_extends
