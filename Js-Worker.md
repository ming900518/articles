# JavaScript Worker (NodeJS Worker Threads) 實作記錄

## 緣起

由於敝人目前任職的公司用 Express
做後端，所以我也只能入境隨俗，結果才沒多久我就感受到 JavaScript/TypeScript
的最大限制：單線程

後端掛了一堆業務邏輯要用到的 EventListener，API
也全都放在同個線程處理，時間一久除了性能上不去之外，還出現過莫名奇妙的 Race
Condition（業務邏輯不便展示，總之就是有出現過就是了）

而且買了多核心 CPU
不用多線程真的浪費啊（笑），指望單核性能像十多年前一樣一代進步個 30~40%
更是想都不用想

那 JS 真的就只能單線程慢慢跑了嗎？

經過一段時間的 Google ，我在 MDN 找到了 Web Workers API 跟 NodeJS 內建的
[Worker Threads](https://nodejs.org/api/worker_threads.html)，宣稱可以開出另一個線程，避免佔用主線程

看起來能解決問題啊，也不用換語言（事後覺得不如換語言 😂）

剛好手邊有個專案，就想說嘗試一下，~~看 JS 還有沒有救~~ 看能不能讓其他划水中的
CPU 核心有點事情幹

> 在這邊要先友情勸退一些人：以下的東西較為複雜，且會碰到很多由於 JS 跟 V8
> 內部實現導致的限制<br>如果有性能需求，我會建議換個語言或用 WebAssembly
> 處理會比較方便

> <blockquote class="twitter-tweet"><p lang="zh" dir="ltr">多核/多執行緒帶來的性能進步，需要軟體的配合才能體現出來<br><br>JS 寫後端，不要求性能的情況下很簡單<br>如果要求性能，那我真的覺得不如換個語言</p>&mdash; Ming Chang (@mingchang137) <a href="https://twitter.com/mingchang137/status/1645635590719959041?ref_src=twsrc%5Etfw">April 11, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## 背景知識補充

Worker 怎麼實現的，可以參考
[MDN 的文件](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API)

由於 NodeJS 沒有 `window` ，所以 NodeJS 改成利用 V8 Isolates ，建立不同的 V8
Instance 將 Code
執行在不同線程，想了解更多，可以參考[這篇文章](https://zhuanlan.zhihu.com/p/167920353)

至於使用上有什麼限制呢？有

第一：沒有線程池，所以比起其他語言實現的線程更昂貴一點

第二：可以在主線程與 Worker Thread
之間傳輸或共享的資料類型有限制，可以參考[這篇文章](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Transferable_objects)
跟
[ES8 的 SharedArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer)

> 我寫 WebAssembly 時都能傳輸
> [JsValue](https://rustwasm.github.io/wasm-bindgen/api/wasm_bindgen/struct.JsValue.html)
> 了，在 JS 的環境下限制居然比其他語言還多（？）

第三：一些 API 在 Worker 不能被取用，如果打算使用 Worker
，請參考[這篇文章](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API#worker_global_contexts_and_functions)確認可用的
API

## 實作 Worker

以下 Code 均為 TypeScript

### 開啓 Worker

首先，我們先 `import` NodeJS 的 worker_threads，並 `new` 一個新的 Worker 物件

```typescript
import { Worker } from "worker_threads";
const worker = new Worker("./src/worker.ts");
```

接著就可以前往 Worker 物件指定的文件位置，寫一些需要在其他線程處理的工作了

### 傳入資料

如果需要傳輸資料到 Worker ，可以在主線程中透過以下程式達成：

> 注意！只能用 `ArrayBuffer` 定義的記憶體空間，而且只能用
> [Typed Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Typed_arrays)<br>
> 想傳物件或字串？往下翻到爬坑的地方，我想出了一個方法解決這件事

```typescript
const arrayBuffer = new ArrayBuffer(2097152);
const array = new Uint16Array(arrayBuffer);
worker.postMessage(array, [arrayBuffer]);
```

> `2097152` 是我自己定義的，2MB 空間如果太大可以自己改數字

如果需要從 Worker 傳輸資料回主線程，可以在 Workers 透過以下程式達成：

```typescript
import { parentPort } from "worker_threads";

const arrayBuffer = new ArrayBuffer(2097152);
const array = new Uint16Array(arrayBuffer);
parentPort!.postMessage(array, [arrayBuffer]);
```

由於 `parentPort` 有可能為 `null` （只有 Worker 才有），這邊可以額外透過
[`worker.isMainThread`](https://nodejs.org/api/worker_threads.html#workerismainthread)
API
去判斷是不是在主線程，不在主線程才用，~~或者像我一樣自信的認為不會爆直接填`!`~~

> 需要注意的是，如果採用這種做法，當前的線程就會無法使用 `arrayBuffer` 與
> `array` 兩個物件了

### 共享資料

如果不想要直接將資料移動至 Workers/主線程 ，可以將 `ArrayBuffer` 換成
`SharedArrayBuffer` ，就能做到共享記憶體空間了

```typescript
const arrayBuffer = new SharedArrayBuffer(2097152);
const array = new Uint16Array(arrayBuffer);
worker.postMessage(array);
```

透過這種方式，就不需要再將資料回傳回來了

> 這邊有一點要注意，由於 `Uint16Array` 用了 `SharedArrayBuffer` 的空間
> initialize ，資料有可能在奇怪的地方被寫入，導致資料不正確/不完整<br>
> 如果有這種情形發生，可以考慮改用 Atomic Type 或將資料加鎖

### 利用資料

我們將資料傳輸/共享出去後，可以在目標上掛上 `EventListener`
來監聽有沒有訊息，有訊息的話就能拿到資料，並進行處理

假設我們傳輸的資料都是 `Uint16Array` ，主線程可以寫成這樣：

```typescript
worker.on("message", (array: Uint16Array) => {
  // 對 array 進行處理
});
```

而在 Workers 則可以寫成這樣：

```typescript
parentPort!.on("message", (array: Uint16Array) => {
  // 對 array 進行處理
});
```

## 坑：如果想要傳輸物件/字串呢？

嗯，這就有點尷尬了，由於 JS
的限制，我們只能在線程之間傳指定類型的物件，大幅降低了 Worker 的易用性

但路是人走出來的，方法是人想出來的

我下班搭車時，腦洞出了一個解法，後來測試了一下還真的可行：

<blockquote class="twitter-tweet"><p lang="zh" dir="ltr">在 JS Worker 與主線程之間共享 Map 物件<br><br>不知道有沒有更快的方法<br><br>- 主線程<br><br>new SharedArrayBuffer<br>new Uint8Array<br><br>- Worker<br><br>目標 Map 先轉成 Array<br>把 Array 拿去 stringify<br>把 string Encode 成 UTF-8 array<br>將 UTF-8 array 塞進主線程傳過來的 Uint8Array</p>&mdash; Ming Chang (@mingchang137) <a href="https://twitter.com/mingchang137/status/1645379833042718720?ref_src=twsrc%5Etfw">April 10, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

你問我跟誰學的？跟
[`wasm-bindgen`](https://docs.rs/wasm-bindgen/latest/wasm_bindgen/) 學的啊（笑）

最後在優化流程後，從 Worker 傳輸物件回主線程的流程如下：

1. 當 `EventListener`
   接收到來自主線程要求資料的訊息時，執行以下程式建立必要的物件

   ```typescript
   const arrayBuffer = new ArrayBuffer(2097152);
   const array = new Uint16Array(arrayBuffer);
   ```

   > 之所以使用 `Uint16Array` 是因為在 JavaScript 的世界，string 是用 UTF-16
   > 存放的，這樣可以避免多餘的 Encode/Decode

   > 不用 `SharedArrayBuffer` ，因為我發現沒有鎖的情況下，資料太容易出錯了

2. 將資料轉為 string，塞到 `Buffer` 去

   ```typescript
   const buffer = Buffer.from(JSON.stringify(data));
   ```

   > `Buffer` 會把內容都轉成能夠塞到 `Uint16Array` 的樣子

   > 有些 object （如 `Map`）是無法被 `JSON.stringify()` 序列化成 JSON string
   > 的，那就要再自己想辦法轉換成能被序列化的物件了

3. 把資料傳回主線程！

   ```typescript
   parentPort!.postMessage(array);
   ```

4. 接著在主線程接收物件即可

   ```typescript
   const receivedObject = JSON.parse(Buffer.from(mqttReturnedData).toString());
   ```

## 結語

我剛試成功的時候，開心的推了一則貼文：

<blockquote class="twitter-tweet"><p lang="zh" dir="ltr">Worker threads 搭配 SharedArrayBuffer 讓敝司的 API 能夠加速，不用等 Listener 把資料處理完<br><br>雖然語法跟易用性相較於 Tokio 就是依託答辯，但有得用已經很感激了</p>&mdash; Ming Chang (@mingchang137) <a href="https://twitter.com/mingchang137/status/1645376340106022915?ref_src=twsrc%5Etfw">April 10, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

大概就是我這邊原本要擺的文字吧

但今天寫這篇文章前幾個小時，我把新程式拿去跟舊程式進行壓測，卻只看到了 10%
不到的性能進步

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">發現就算把 Listener 移到 Worker Threads ，性能提升也有限……</p>&mdash; Ming Chang (@mingchang137) <a href="https://twitter.com/mingchang137/status/1645634737145540608?ref_src=twsrc%5Etfw">April 11, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

好吧，我覺得沒救

看起來性能瓶頸應該是在 Express 這塊，哪天等到 Express 支援 Worker ，可能才有辦法真正提高性能了。

> 或許有人會說：「寫 JavaScript
> 嘛，還要啥~~自行車~~性能，而且我看現在寫的程式也沒多慢啊」<br>
> 對，現在看起來確實是這樣沒錯，但性能問題會隨著程式量的增加而愈發明顯<br>
> 等到不得不改的時候，面對成千上萬行的 code ，根本不知道要如何改起<br>
> 那為什麼不是寫的當下就把程式寫好呢？
