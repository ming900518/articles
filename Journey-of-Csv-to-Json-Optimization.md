# 83% 性能提升！CSV 至 JSON 轉換工具優化記錄

> <img width="1440" alt="bg" src="https://user-images.githubusercontent.com/15919723/229822229-f17c4c41-c412-4ed0-95c3-f06c8afe8d3d.png">
> 程式性能測試截圖
> 
> 背景圖 by [HitenKei](https://twitter.com/HitenKei)

## 前言

由於這幾天是台灣的春假（清明節連假），所以想說利用這個機會來
~~逃離工作時被逼著用的 JS 跟 TS~~ 做點工作之外的事

正好我有個案子不想架後端與資料庫，想要利用 JSON 來儲存大量的資料，就計劃寫個 CSV
轉 JSON 的小工具來用

> [CSV-to-JSON GitHub Repository 連結](https://github.com/ming900518/csv-to-json) 
> 
> Releases 裡有已經編譯好的程式，沒有特別標註的話都是 for aarch64 macOS 的 binary

由於 Rust 的相關 crate 我都還算熟悉，結果只花了一個小時就搓出來了 😂
正在思考還有什麼事情可以做的時候，突然想到自己好像很久沒有優化程式了

原本我還胸有成竹，自己都用上了 **BLAZLINGLY FAST 🚀🚀🚀** 的可愛 🦀
，應該不會太慢才對？我賭兩秒以內！

<img width="450" alt="meme1" src="https://user-images.githubusercontent.com/15919723/229833580-eb89967b-0780-4190-a6b3-1180d13e7fd5.jpg">

> - 硬體：M1 MacBook Air, macOS 13.3
> - 測試用 CSV 檔：就是一個全都是字， 573.5 MB 的 CSV
> - 測試工具：[`hyperfine`](https://github.com/sharkdp/hyperfine)

測出來，3.3 秒（四捨五入）

<img width="450" alt="meme2" src="https://user-images.githubusercontent.com/15919723/229834730-b04f12db-e96d-40b4-b9ff-817931620872.jpeg">

於是這個我花了一個小時寫出來的 CLI Binary ，就和我一起開始了為期兩天的優化之旅

> 以下的每個章節都會附上與**最初版**程式相比的「**總計**性能進步百分比」跟「執行平均時間（秒）」，供讀者參考

## 第一步（18% - 2.8s）：資料平行 [Rayon](https://crates.io/crates/rayon)

其實我很早就知道這個 crate 了，但由於我寫後端嘛， Tokio
就已經很香了，反而用不太到 Rayon

Rayon 是一個 data-parallelism library ，可以將傳統的迭代器
[`std::iter`](https://doc.rust-lang.org/stable/std/iter/index.html)
能做到的事情，改為利用平行處理的迭代器
[`rayon::iter`](https://docs.rs/rayon/latest/rayon/iter/index.html) 去處理

而且寫法非常簡單：只需要把傳統迭代器 `vec.iter()` 換成 `vec.par_iter()` 就可以了

> 這邊要提醒一下，由於平行處理的本質就是開一堆 Thread 來處理資料，如果 closure
> 中有用到 scope 外面的變數，需要 impl Sync trait 或者用
> [`Arc`](https://doc.rust-lang.org/stable/std/sync/struct.Arc.html) 包住才行。
>
> [Rayon 如何實現 Parallel Iterator](https://github.com/rayon-rs/rayon/blob/master/src/iter/plumbing/README.md)

改完，我們利用 hyperfine 進行測試

<details>
    <summary>測試結果</summary>
    
    Benchmark 1: ./csv-to-json -i test.csv -o output-1.json
      Time (mean ± σ):      3.286 s ±  0.021 s    [User: 2.616 s, System: 0.585 s]
      Range (min … max):    3.260 s …  3.314 s    5 runs

    Benchmark 2: ./csv-to-json-rayon -i test.csv -o output-2.json
      Time (mean ± σ):      2.794 s ±  0.014 s    [User: 4.537 s, System: 0.827 s]
      Range (min … max):    2.777 s …  2.814 s    5 runs

    Summary
      './csv-to-json-rayon -i test.csv -o output-2.json' ran 1.18 ± 0.01 times faster than './csv-to-json -i test.csv -o output-1.json'

</details>
<br>

取得了 18% 的性能進步。

> 到這邊的程式碼：[csv-to-json 0.1.0](https://github.com/ming900518/csv-to-json/tree/0.1.0)

## 第二步（24% - 2.6s）：用對的工具做對的事 [indexmap](https://crates.io/crates/indexmap)

由於需要保留原始 CSV 的欄位排序，所以無法採用 std 中的
[`HashMap`](https://doc.rust-lang.org/stable/std/collections/struct.HashMap.html)
，而是用了
[`BTreeMap`](https://doc.rust-lang.org/stable/std/collections/struct.BTreeMap.html)

不過如果仔細看下
[When Should You Use Which Collection?](https://doc.rust-lang.org/stable/std/collections/index.html)
，會發現 `BTreeMap` 根本不是幹這個的

所以我又去翻了萬能的 crates.io ，看到了
[indexmap](https://crates.io/crates/indexmap) ，文件中宣稱 Fast to iterate 和
Preserves insertion order as long as you don't call `.remove()`

啊呀，剛好符合我的需求啊！那就事不宜遲，改程式，測試！

<details>
    <summary>測試結果</summary>

    Benchmark 1: ./csv-to-json -i test.csv -o output-1.json
      Time (mean ± σ):      3.248 s ±  0.014 s    [User: 2.614 s, System: 0.573 s]
      Range (min … max):    3.224 s …  3.259 s    5 runs

    Benchmark 2: ./csv-to-json-rayon -i test.csv -o output-2.json
      Time (mean ± σ):      2.806 s ±  0.016 s    [User: 4.547 s, System: 0.842 s]
      Range (min … max):    2.786 s …  2.826 s    5 runs

    Benchmark 3: ./csv-to-json-indexmap -i test.csv -o output-3.json
      Time (mean ± σ):      2.621 s ±  0.029 s    [User: 4.574 s, System: 0.701 s]
      Range (min … max):    2.599 s …  2.671 s    5 runs

    Summary
      './csv-to-json-indexmap -i test.csv -o output-3.json' ran
        1.07 ± 0.01 times faster than './csv-to-json-rayon -i test.csv -o output-2.json'
        1.24 ± 0.01 times faster than './csv-to-json -i test.csv -o output-1.json'

</details>
<br>

性能進步從 18% 增加到 24% 。

> 到這邊的程式碼：[csv-to-json 0.2.0](https://github.com/ming900518/csv-to-json/tree/0.2.0)

## 第三步（55% - 2.1s）：無心插柳柳成蔭 [Polars](https://crates.io/crates/polars)

性能提升總是讓人開心，但......還能不能再更快一點呢？離兩秒內的目標還有蠻大的差距

其實我原本並沒有打算要把 CSV
轉成其他格式，但網際網路就是這麼的神奇，常常在亂晃的時候看到可能有價值的東西

這邊用了 [Polars](https://www.pola.rs) ，可以將 CSV 轉換為 DataFrame ，號稱
lightning fast

由於我本質上並沒有要對 CSV
中的資料做任何的查詢，所以我當初對這個套件，是抱持著「就算沒有性能進步也沒關係」的心情測試的

沒想到，竟然產生了意想不到的結果？

<details>
    <summary>測試結果</summary>

    Benchmark 1: ./csv-to-json -i test.csv -o output-1.json
      Time (mean ± σ):      3.258 s ±  0.024 s    [User: 2.630 s, System: 0.563 s]
      Range (min … max):    3.235 s …  3.292 s    5 runs

    Benchmark 2: ./csv-to-json-rayon -i test.csv -o output-2.json
      Time (mean ± σ):      2.757 s ±  0.015 s    [User: 4.480 s, System: 0.787 s]
      Range (min … max):    2.735 s …  2.774 s    5 runs

    Benchmark 3: ./csv-to-json-indexmap -i test.csv -o output-3.json
      Time (mean ± σ):      2.569 s ±  0.019 s    [User: 4.537 s, System: 0.650 s]
      Range (min … max):    2.547 s …  2.594 s    5 runs

    Benchmark 4: ./csv-to-json-polar -i test.csv -o output-4.json
      Time (mean ± σ):      2.106 s ±  0.013 s    [User: 2.552 s, System: 0.573 s]
      Range (min … max):    2.086 s …  2.118 s    5 runs

    Summary
    './csv-to-json-polar -i test.csv -o output-4.json' ran
        1.22 ± 0.01 times faster than './csv-to-json-indexmap -i test.csv -o output-3.json'
        1.31 ± 0.01 times faster than './csv-to-json-rayon -i test.csv -o output-2.json'
        1.55 ± 0.01 times faster than './csv-to-json -i test.csv -o output-1.json'

</details>
<br>

性能進步再次從 24% 增加到 55% 。

> 根據 Polars API 文件中標示的說明：
> [`get_row`](https://pola-rs.github.io/polars/polars/frame/struct.DataFrame.html#method.get_row)
> 這個 method 因性能不好而不建議使用，所以這邊應該還有改進的可能（？）

## 第四步（60% - 2.0s）：不如預期 SIMD

正當我滿意的準備 `git push` 時， Polars
的文件有個[小段落](https://pola-rs.github.io/polars/polars/index.html#simd)引起了我的注意

SIMD ？？？ SIMD 還能拿來加速這種運算？

> 下面沒有特別標註的話，都是在 M1 MacBook 上用 ARM NEON 指令集測出來的結果

> 我沒有支援 AVX-512 的 x86_64 PC 可以測試，要不然我還蠻想知道 AVX-512 到底有多快

裝上 nightly Rust toolchain ，features
裝好，`RUSTFLAGS="-C target-cpu=native" cargo +nightly build --release`！

<details>
    <summary>測試結果</summary>

    Benchmark 1: ./csv-to-json -i test.csv -o output-1.json
      Time (mean ± σ):      3.271 s ±  0.009 s    [User: 2.617 s, System: 0.566 s]
      Range (min … max):    3.260 s …  3.284 s    5 runs

    Benchmark 2: ./csv-to-json-rayon -i test.csv -o output-2.json
      Time (mean ± σ):      2.780 s ±  0.010 s    [User: 4.516 s, System: 0.789 s]
      Range (min … max):    2.770 s …  2.792 s    5 runs

    Benchmark 3: ./csv-to-json-indexmap -i test.csv -o output-3.json
      Time (mean ± σ):      2.582 s ±  0.013 s    [User: 4.558 s, System: 0.649 s]
      Range (min … max):    2.571 s …  2.598 s    5 runs

    Benchmark 4: ./csv-to-json-polar -i test.csv -o output-4.json
      Time (mean ± σ):      2.114 s ±  0.023 s    [User: 2.561 s, System: 0.574 s]
      Range (min … max):    2.083 s …  2.142 s    5 runs

    Benchmark 5: ./csv-to-json-simd -i test.csv -o output-5.json
      Time (mean ± σ):      2.046 s ±  0.066 s    [User: 2.483 s, System: 0.554 s]
      Range (min … max):    2.004 s …  2.163 s    5 runs

    Summary
      './csv-to-json-simd -i test.csv -o output-5.json' ran
        1.03 ± 0.04 times faster than './csv-to-json-polar -i test.csv -o output-4.json'
        1.26 ± 0.04 times faster than './csv-to-json-indexmap -i test.csv -o output-3.json'
        1.36 ± 0.04 times faster than './csv-to-json-rayon -i test.csv -o output-2.json'
        1.60 ± 0.05 times faster than './csv-to-json -i test.csv -o output-1.json'
    </code>

</details>
<br>

性能進步只從 55% 小幅增加到 60% ，遠遠不如我預期中的秒天秒地，有種被騙的感覺

但我覺得可能不能怪 SIMD ，第一是誰也說不準 Polars 用了哪些 SIMD
，第二是搞不好這種操作就不是 SIMD 的強項

誰知道呢？

<details>
    <summary>在 Intel Core i7 12700（支援 AVX2 指令集，AVX-512 不支援）測試的結果</summary>

    Benchmark 1: ./csv-to-json -i test.csv -o output-1.json
      Time (mean ± σ):      2.644 s ±  0.010 s    [User: 2.039 s, System: 0.605 s]
      Range (min … max):    2.637 s …  2.660 s    5 runs

    Benchmark 2: ./csv-to-json-simd -i test.csv -o output-2.json
      Time (mean ± σ):      1.978 s ±  0.007 s    [User: 2.702 s, System: 1.082 s]
      Range (min … max):    1.969 s …  1.989 s    5 runs
    Summary
      './csv-to-json-simd -i test.csv -o output-2.json' ran
        1.34 ± 0.01 times faster than './csv-to-json -i test.csv -o output-1.json'

    可以看到，即使是在支援更多 SIMD 指令集的電腦上，也沒有顯著的性能進步

</details>
<br>

> 到這邊的程式碼：[csv-to-json 0.3.0](https://github.com/ming900518/csv-to-json/tree/0.3.0)

## 第五步（83% - 1.8s）：看圖優化，對症下藥 [flamegraph](https://github.com/flamegraph-rs/flamegraph) （CLI 工具）

當我仔細的觀察程式的 CPU 佔用時，其實有個點一直讓我很不解：明明都用上了 Rayon
，怎麼在全核心跑完後，程式還會繼續用單線程執行一小段時間呢？

重新閱讀了一次程式，再跑了下
[flamegraph](https://github.com/flamegraph-rs/flamegraph) ，發現 serde_json 的
[`to_string`](https://docs.rs/serde_json/1.0.95/serde_json/fn.to_string.html)
似乎可以優化一下

> <img width="1440" alt="Screenshot 2023-04-04 at 3 58 09 PM" src="https://user-images.githubusercontent.com/15919723/229726662-5500b642-f33d-4022-8cf6-9df5a2eb93b8.png">
> 在 Linux PC 跑出來的 flamegraph ，可以發現 `to_string` 裡面用的是普通的 iterator

用單線程跑 570 幾 MB 的資料確實不是很理想，但我花了一整天都沒看出來，太尷尬了 😅

這時我想到了兩種解法：

1. 自己看看 serde_json 是如何實現這支程式的，然後寫多線程版
2. 在處理資料的時候就先把資料序列化，最後再將資料組合起來

解法二目前是比較好的解法： 第一是我可以直接利用資料處理的 parallel iterator
，不需要重新迭代 第二是這樣也比較簡單

所以就實作一下，跑個 hyperfine 吧

<details>
    <summary>測試結果</summary>

    Benchmark 1: ./csv-to-json -i test.csv -o output-1.json
      Time (mean ± σ):      3.223 s ±  0.019 s    [User: 2.605 s, System: 0.533 s]
      Range (min … max):    3.204 s …  3.245 s    5 runs

    Benchmark 2: ./csv-to-json-rayon -i test.csv -o output-2.json
      Time (mean ± σ):      2.738 s ±  0.015 s    [User: 4.500 s, System: 0.760 s]
      Range (min … max):    2.724 s …  2.757 s    5 runs

    Benchmark 3: ./csv-to-json-indexmap -i test.csv -o output-3.json
      Time (mean ± σ):      2.525 s ±  0.012 s    [User: 4.557 s, System: 0.613 s]
      Range (min … max):    2.512 s …  2.541 s    5 runs

    Benchmark 4: ./csv-to-json-polar -i test.csv -o output-4.json
      Time (mean ± σ):      2.070 s ±  0.021 s    [User: 2.550 s, System: 0.537 s]
      Range (min … max):    2.042 s …  2.089 s    5 runs

    Benchmark 5: ./csv-to-json-simd -i test.csv -o output-5.json
      Time (mean ± σ):      2.001 s ±  0.035 s    [User: 2.482 s, System: 0.540 s]
      Range (min … max):    1.972 s …  2.057 s    5 runs

    Benchmark 6: ./csv-to-json-parallel-serialize  -i test.csv -o output-6.json
      Time (mean ± σ):      1.761 s ±  0.010 s    [User: 3.689 s, System: 0.497 s]
      Range (min … max):    1.749 s …  1.771 s    5 runs

    Summary
      './csv-to-json-parallel-serialize  -i test.csv -o output-6.json' ran
        1.14 ± 0.02 times faster than './csv-to-json-simd -i test.csv -o output-5.json'
        1.18 ± 0.01 times faster than './csv-to-json-polar -i test.csv -o output-4.json'
        1.43 ± 0.01 times faster than './csv-to-json-indexmap -i test.csv -o output-3.json'
        1.56 ± 0.01 times faster than './csv-to-json-rayon -i test.csv -o output-2.json'
        1.83 ± 0.01 times faster than './csv-to-json -i test.csv -o output-1.json'

</details>
<br>

比起原始版快了 83% ，平均用時也壓到兩秒以內了！

再來看看 flamegraph

- Before

  <img width="1440" alt="1" src="https://user-images.githubusercontent.com/15919723/229730639-6488f3f1-d8ca-4d33-a056-d8e1b2100e1a.png">

- After

  <img width="1440" alt="2" src="https://user-images.githubusercontent.com/15919723/229730808-3ac27e2b-aa97-4a32-ac05-0f36da835e29.png">

同樣以 "serde"
做為關鍵字（紫色區塊是查詢結果），可以清楚的看到序列化的工作已經被分散開來了。

未來可能會找個時間嘗試實作 serde_json 搭配 Rayon
進行平行序列化吧，看起來提升挺大的。

> 到這邊的程式碼：[csv-to-json 0.3.1](https://github.com/ming900518/csv-to-json/tree/0.3.1)

## 後記

這幾天這麼寫下來，雖然程式碼不長（才 74
行），但讓我學到了很多平常寫程式時學不到的事：

1. Rayon 讓 Rust 開發平行運算變得簡單自然，未來會在生產環境中多用。
2. 每一個 struct
   都有自己擅長處理的內容，如果要以性能為優先考量的話，需要仔細閱讀文件中的內容，並按照內容選用最適合的那個。
3. 永遠不要因為麻煩而不去嘗試。（當然，如果有時程壓力，那還是以產出成品為主）
4. SIMD 不是萬靈丹。
5. flamegraph 是個好工具，不僅可以幫助自己找出程式的性能瓶頸，也能藉此理解每個
   function 下的實現方式。

從去年開始，我每個連假都學到了一點東西，未來也會繼續學習！

感謝您的閱讀！

