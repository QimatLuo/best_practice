# 原子

原子是構成化學元素的普通物質的最小單位。  
這句話是從 Wiki 抄過來的，我們要講的跟化學沒關係，我們是要講程式設計。  
這個概念跟 FP 有點像，就像我們會生出 `add()` 這種最簡單的操作，然後再給用的人去隨意組合。  
RxJS 的思路也是一樣，我們做產品也是從最小單位開始。  
比如說我們透過 Highcharts 畫線圖，應該怎麼做？

## Highcharts

以 [Exosite Tsdb] 回應格式為例：

```json
{
  "tags": {},
  "columns": ["time", "temp"],
  "values": [
    ["2022-05-29T04:40:26+00:00", 34.123],
    ["2022-05-29T04:40:28+00:00", 35.456],
    ["2022-05-29T04:40:30+00:00", 36.789]
  ],
  "metrics": ["temp"]
}
```

目標是組出 Highcharts 的格式：

```json
{
  "series": [
    {
      "name": "Temperature",
      "data": [
        [1653799226000, "34.12 °C"],
        [1653799228000, "35.46 °C"],
        [1653799230000, "36.79 °C"]
      ]
    }
  ]
}
```

### 範例 1

```js
api()
  .then((result) =>
    result.values.map(([time, temp]) => {
      const x = Date.parse(time);
      const y = `${temp.toFixed(2)} °C`;
      return [x, y];
    })
  )
  .then((data) => {
    const chartOptions = {
      series: [
        {
          name: "Temperature",
          data,
        },
      ],
    };
    return chartOptions;
  });
```

看起來沒什麼問題對吧，就是從 API 拿到資料後，把 x 軸的資料跟 y 軸的資料處理過，整理成 Highcharts data ，然後組成 Hicharts options。  
那現在我想同時畫兩條線在圖上會需要改多少？

### 範例 2

```js
api()
  .then((result) =>
    result.values.map(([time, temp, humi]) => {
      const x = Date.parse(time);
      const y = `${temp.toFixed(2)} °C`;
      const y2 = `${humi.toFixed(2)}%`;
      return [x, y, y2];
    })
  )
  .then((data) => {
    const temp = data.map(([x, y]) => [x, y]);
    const humi = data.map(([x, _, y]) => [x, y]);
    const chartOptions = {
      series: [
        {
          name: "Temperature",
          data: temp,
        },
        {
          name: "Humidity",
          data: humi,
        },
      ],
    };
    return chartOptions;
  });
```

這是我短時間能想到的最快解法，很明顯這種寫法很沒有設計層面，只因為多加一條線就要破壞原本流程上的程式碼，最差的是它改了原本 `temp` 的實作方式。  
因為 `temp` 原本好好的卻被強加了要多一層 loop 去做再整理，對我來說這就是設計不良導致的後果。

理想的設計是，當有新需求來臨，只要加上新需求的程式碼就好了。  
要避免這件事情就是得抽象很多層，利用 API 所回應的 columns 來動態組出 data 再衍伸成 series 的格式。要做到這點必須在一開始就預想到這樣的需求然後做出這些抽象的設計才可以，這必須要有一點經驗值才能做到的事情。

讓我們用 RxJS 讓世界變得美好吧。

```js
from(api()).pipe(
  map((result) =>
    result.values.map(([time, temp, humi]) => {
      const x = Date.parse(time);
      const y = `${temp.toFixed(2)} °C`;
      const y2 = `${humi.toFixed(2)}%`;
      return [x, y, y2];
    })
  ),
  map((data) => {
    const temp = data.map(([x, y]) => [x, y]);
    const humi = data.map(([x, _, y]) => [x, y]);
    const chartOptions = {
      series: [
        {
          name: "Temperature",
          data: temp,
        },
        {
          name: "Humidity",
          data: humi,
        },
      ],
    };
    return chartOptions;
  })
);
```

當然不是長這樣啊，這就是我所謂的把 RxJS 當成一個資料處理的 library 而已，想法上並沒有流的概念。  
如果你用 RxJS 跟沒用 RxJS 的寫法長得一樣，那就要嘗試換個思路寫程式。

## 流一般的思考

想法上要像流一樣，首先聽到需求是畫一條線圖，這時候不用管怎麼從 API 的結構轉成 Highchart 的線圖格式，而是只單純看線圖的最小單位是什麼。  
是 `x` 或 `y`。雖然在範例上要能成立一個點是 `[x, y]`，但其實某些情況是可以忽略 `x` 只寫 `y`。

### 範例 3

```js
const values = from(api()).pipe(
  mergeMap((x) => x.values),
  share()
);

const time = values.pipe(
  map((xs) => xs[0]),
  map((x) => Date.parse(x))
);

const temp = values.pipe(
  map((xs) => xs[1]),
  map((x) => `${x.toFixed(2)}%`)
);
const tempData = zip(time, temp).pipe(toArray());
const tempSeries = zip(of("Temperature"), tempData).pipe(
  map(([name, data]) => ({ name, data }))
);

const series = zip(tempSeries).pipe(map((series) => ({ series })));
```

這時候看見我們每一條流都是可以獨立運作的，只要訂閱 `time` 就可以得到每一筆 `time` 的資料，`temp` 也是同樣的道理。  
這時候如果你有其他地方想單獨顯示最後一筆溫度，直接在該畫面上的位置訂閱 `temp` 就可以了，如果另一個區塊想顯示最後一次更新時間，就直接訂閱 `time` 就可以了。  
當資料必須是兩兩配對相互依賴時，只要像 `tempData` 那樣把它 `zip()` 起來就可以給 Highcharts 使用，各個流都可以這樣被重複使用並組出新的形態。

這時候如果有新的一條線要加進來，只要按照模式加上 `humi`, `humiData`, `humiSeries` 就可以讓別人再 `zip()` 進去。

### 範例 4

```js
const values = from(api()).pipe(
  mergeMap((x) => x.values),
  share()
);

const time = values.pipe(
  map((xs) => xs[0]),
  map((x) => Date.parse(x))
);

const temp = values.pipe(
  map((xs) => xs[1]),
  map((x) => `${x.toFixed(2)}%`)
);
const tempData = zip(time, temp).pipe(toArray());
const tempSeries = zip(of("Temperature"), tempData).pipe(
  map(([name, data]) => ({ name, data }))
);

const humi = values.pipe(
  map((xs) => xs[2]),
  map((x) => `${x.toFixed(2)} °C`)
);
const humiData = zip(time, humi).pipe(toArray());
const humiSeries = zip(of("Humidity"), humiData).pipe(
  map(([name, data]) => ({ name, data }))
);

const series = zip(tempSeries, humiSeries).pipe(map((series) => ({ series })));
```

主要是整個設計模式是很簡單的，就是從最小單位 `time` 跟 `temp` 開始寫。  
發現兩個來自同樣的上游，所以抽出來變成 `values` 並 `share()`。  
接下來利用兩個最小單位組出一個資料點 data 的樣子。  
再利用 data 組出 series 的樣子，每一個 observable 都是每個階段的最小單位化。  
最後就由 Hicharts options 去整合這些 series 就可以畫圖了，整個實作方法不會因為要畫幾條線而改變，或甚至是只畫一個點都可以。  
就算是 temperature 跟 humidity 來自兩個不同的 API 都沒問題，儘管要配合 WebSocket 做實時顯示，實作的模式也是如此。

當然這範例裡面也有可以參數化的部分，像是透過 function 來產生 observable，這樣就可以帶 index 或是帶 column name 就可以生出對應的 metric 的資料流。  
本篇主要是引導所謂流的思考方式，讓每一條流都可以獨立運作去減少需求變化時的異動程度。而情境就是透過 operator 去自由組合最小單位來完成，跟 FP 去組合各個 function 是大同小異的概念。

## 附註

`zip()` 適用在兩兩數量相等配對的情境，通常就是個別最小單位來自同一個來源，所以可以保證 1 : 1 的配對。  
若是情境為不保證 1 :1，則可以用 `combineLatest()` 來做組合，不過要小心 `combineLatest()` 是只有當所有 observable 都有資料時才會開始流動。  
若情境式任意 observable 有資料就開始動的話，可以透過 `merge()` + `scan()` 來完成資料的組合。  
若你有情境不知道怎麼用流的想法去組合，可以開 Issue 來做討論。

[exosite tsdb]: https://docs.exosite.io/murano/reference/services/tsdb/#query
