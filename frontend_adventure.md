# 從 JS Framework 、Rust WebAssembly 到 HTMX - 我對前端開發的嘗試與思考

> 本文原名：從 React 、 Leptos 到 HTMX - 新部落格開發之旅

我對於前端開發又愛又恨：我愛它帶給我的成就感，我恨它越來越高的複雜度。直到某天我遇見了 HTMX......原來，前端開發可以如此輕鬆自在。

## 我的前端開發歷程

我第一次接受網頁開發，其實是在小學時，我用 Dreamweaver （天啊，說出這程式的名字讓我感覺我老了 😂）設計出我的第一版個人網頁，並上傳到當時還算好找的 Free Hosting 。當時連 HTML5 都才剛開始發展，我用 HTML4 + CSS + JS 寫出的靜態網頁既單純又簡單，而且就已經可以達到我當時想達到的目標了。

數年前，我進入全端開發公司工作，使用的 stack 是 Angular + Spring 。當時以為是我自己努力不夠，才會難以理解 RxJS 。TypeScript 和 Java 一起寫也讓我非常的混亂：

-   為什麼在 Java 用的好好的邏輯，在 TypeScript 型別會亂跑導致錯誤呢？（當時公司把 TypeScript 當 AnyScript 寫，所以型別超級混亂）
-   不可避免的 Asynchronous Code 滿天飛，到底要怎麼讓我的邏輯正常運作呢？
-   說到正常運作，為什麼明明看起來一切正常的邏輯，上線之後會瘋狂的噴錯呢？

最後在我和我同事熬夜了半年，將已經逾期的案子開發並修復完後，我帶著對於前端開發的壞印象，以及得了腸躁症的身體，離職了。

離職後，我去了另一間公司擔任後端工程師，但在正式開始寫後端前，我先接下了公司的試驗性專案，前後端被限制要用 React + Express 寫，結果問題更多，不停的踩到 React 的 footgun ：

-   改一個 Object 的 Property 不重新渲染
-   Dev Mode `useEffect` 跑兩次（X/Twitter 前端熱門話題）
-   記憶體用量超高最後把瀏覽器炸了（那個專案需要高頻率更新資料）
-   等等⋯⋯不勝枚舉

我跑去看了 React 的文件，跑去問公司的前端，甚至晚上熬夜解 Bug 。最後趕在 deadline 前把前端穩定住了，壓線交差。

在這兩次讓我痛苦的回憶之後，讓我不禁思考，前端開發究竟是如何從以往的輕鬆開發，變成現在這樣的？

這時，有個新語言的崛起，讓我似乎看見了

## 接觸 Rust 與 WebAssembly

對於開發 JS 系語言的痛點，我在我的另一篇文章「我選擇寫 Rust ，因為我知道我自己並不聰明」中有提到。簡而言之就是 Rust 的型別系統、安全保證與編譯器提供的有效提示，讓我在寫程式時變得放心許多。

<div
    class="card bg-base-200 shadow-xl mb-5 lg:ml-20 lg:mr-20 rounded-lg select-none cursor-pointer hover:bg-base-300 transition-colors">
    <a
        href="/blog?filename=using-rust-because-im-stupid.md"
        hx-get="/article?filename=using-rust-because-im-stupid.md"
        hx-swap="transition:true show:window:top"
        hx-target="#content"
        hx-push-url="/blog?filename=using-rust-because-im-stupid.md">
        <div class="card-body z-30" style="color: black">
            <div class="flex lg:flex-row flex-col gap-2">
                <h1 class="card-title grow transition-colors" style="font-size: 1.25rem; font-weight: 600; margin: 0">我選擇寫 Rust ，因為我知道我自己並不聰明</h1>
                <h2 class="justify-end" style="font-size: .875rem; font-weight: normal; margin: 0">2023/07/15</h2>
            </div>
            <p style="font-size: revert; margin: 0"> 一年前，我 commit 了我的第一行 Rust code ，截至 2023/5/20 ， Rust Code 佔了我自己寫的程式的 73.28 %（現在應該佔比更高），現在的我已經可以很自豪的說我可以用 Rust 寫幾乎所有程式。這篇文章我會分享自己寫程式的經驗，以及為什麼我選擇了 Rust ，可以給別人在選擇該不該學習某種語言時，一點不同於他人的觀點。 </p>
        </div>
    </a>
</div>

由於原生 Rust 是沒有 Runtime 的，對於要求檔案大小的前端開發非常合適，在學習的過程中，我就發現了有人拿 Rust 寫前端。這個消息對我這個已經被前端開發搞到頭痛不已的人來說，實在是個好消息 - 至少解決了 TypeScript 型別定義和 unsound 的問題。 （Static Hermes 簡報截圖，記得附來源）

當時已經基本了解 React 開發的我，決定從號稱「Rust 版 React」 的 Yew 開始，以更新我的個人網頁為目標，嘗試學習 Rust 前端開發。

<div
    class="card bg-base-200 shadow-xl mb-5 lg:ml-20 lg:mr-20 rounded-lg select-none cursor-pointer hover:bg-base-300 transition-colors">
    <a
        href="/blog?filename=Personal-Page-Update.md"
        hx-get="/article?filename=Personal-Page-Update.md"
        hx-swap="transition:true show:window:top"
        hx-target="#content"
        hx-push-url="/blog?filename=Personal-Page-Update.md">
        <div class="card-body z-30" style="color: black">
            <div class="flex lg:flex-row flex-col gap-2">
                <h1 class="card-title grow transition-colors" style="font-size: 1.25rem; font-weight: 600; margin: 0">使用 Yew 框架與 Server Side Rendering，為個人網頁改版</h1>
                <h2 class="justify-end" style="font-size: .875rem; font-weight: normal; margin: 0">2023/01/29</h2>
            </div>
            <p style="font-size: revert; margin: 0"> 這篇文章，將會盡可能的記錄我改版個人網頁的過程。 </p>
        </div>
    </a>
</div>

學習的過程中，我挺享受在完全靜態強型別的環境下進行資料的處理與渲染， Yew 在和 React 保持盡可能 API 相同的同時，也透過 Rust 的語言特性，避免了許多 footgun ：譬如「改 Object Property 不會 Rerender」這件事，在 Yew 這邊根本不會發生，讓我開發起來比 React 自然且方便很多。我甚至在外接了案子，直接使用 Yew 進行網頁開發，對我而言其穩定性已經足夠，可以在生產環境進行使用。

然而 Yew 並不是完全沒有缺點：

-   語法複雜：由於 Rust 沒有 GC ，開發者需要自己處理記憶體的配置，再加上對於 Zero Cost Abstraction 的要求，語法比起 JavaScript 更為複雜。
-   `web_sys` crate 使用困難：由於生態系不完全，且暫時還沒有辦法不透過 JavaScript 進行 DOM 操作，目前 Rust WebAssembly 專案都只能通過 `web_sys` crate 進行和 DOM 的交互。它提供的 API 幾乎就和 Vanilla JS 一樣，但由於需要用 Rust 的語法去適配 JavaScript 的語言特性，該 crate 的開發體驗並不是很好。

<blockquote class="flex flex-col">
    <img
        style="max-width: 75%; align-self:center"
        srcset="
            https://mingchang.tw/images/25f02651-3666-405a-5983-d2ac531e5400/sm  360w,
            https://mingchang.tw/images/25f02651-3666-405a-5983-d2ac531e5400/md  432w,
            https://mingchang.tw/images/25f02651-3666-405a-5983-d2ac531e5400/lg  576w,
            https://mingchang.tw/images/25f02651-3666-405a-5983-d2ac531e5400/xl  720w
            https://mingchang.tw/images/25f02651-3666-405a-5983-d2ac531e5400/2xl 864w
            https://mingchang.tw/images/25f02651-3666-405a-5983-d2ac531e5400/max 1080w
        "
        sizes="(max-width: 75%) 100vw, 640px"
        src="https://mingchang.tw/images/25f02651-3666-405a-5983-d2ac531e5400/lg"
    />
    <br>
    <p style="align-self: center">
        圖片中呈現的是 web_sys 中常出現的 API 形式
        <br>
        由於 JavaScript 動態與沒有強迫處理錯誤的特性，使得很多 API 需要進行非常多的前期處理，導致開發者難以運用。
        <br>
        <div style="align-self: center">
        以此 API 的回傳值為例：
        <ul>
        <li>該 API 將有可能失敗，回傳值為任一合法 JavaScript 數值</li>
        <li>該 API 如執行成功，回傳值有可能為空值</li>
        <li>該 API 如執行成功且不為空值，回傳值有可能是任一 JavaScript 物件</li>
        </ul>
        </div>
    </p>
</blockquote>

在做完現有的所有專案後，我開始尋找其他 Rust 前端框架：當時除了 Yew ，還有 Dioxus 與 Leptos 是比較熱門的框架，不過由於 Dioxus 寫起來不像 JSX ，而且它的主要目標是要作為類似 Flutter 的 Cross-platform UI Framework 去發展，於是最後，我選擇了 Leptos 作為我下一款前端框架。

會知道 Leptos ，其實是因為看了 [ThePrimeagen 的訪談](https://youtu.be/UrMHPrumJEs?si=OaIoVhgYrT57x5XB)，他邀請 Leptos 的開發者 Greg Johnston 上直播，聽了內容後，我挺認同 Greg 對於目前前端界的看法，也對於他所開發的 Leptos 框架感興趣。

當時正值我想要大改部落格的架構，於是在經過研究與測試後，我決定採用 Leptos 的 Axum SSR 模板，對既有的 Yew CSR 專案進行改寫。

> 新部落格 Leptos 版：[https://github.com/ming900518/new_blog/tree/main](https://github.com/ming900518/new_blog/tree/main)

改寫的過程可說是驚喜不斷：

-   由於 Leptos 採用類似 SolidJS 的 Signal ，而非 React/Yew 的 Hooks ，運作方式不同，渲染的方式也不同，沒有 VDOM Diff 讓性能得以進一步提升。
-   汲取了 Yew 的經驗，Leptos 的 Signal 實現了 `Copy` trait ，在使用上更為方便。
-   文件完善度很高，可以靠官方提供的文件進行框架的學習，而非看著 API Docs 摸瞎。
-   框架開發者與用戶互動頗多，且在進行更動時，會盡可能的說明功能異動的目標、內容及結果。開發者可以更理解用戶的需求，而用戶可以從中理解更多的資訊。

但開發時的痛點也依然存在：由於 Leptos SSR 需要搭配 Server 才能運作，所以通常會和 Rust 的後端框架一起搭配使用。官方雖然有提供 Axum 和 Actix-web 的範本，但由於複雜度提高很多，對初學者其實不太友好。

在開發中隨著對於 WebAssembly 的了解加深，我發現其本身也有一些缺點尚待優化：

-   （表面上的）記憶體用量大：由於目前 WebAssembly 僅能透過 JavaScript 的 `ArrayBuffer` 取得記憶體空間，且目前沒辦法讓 JS 回收已經未使用的 `ArrayBuffer` ，所以雖然 WebAssembly 性能高效率快，記憶體用量都會相較於 JS 來得高。

<blockquote class="flex flex-col">
    <img
        style="max-width: 75%; align-self:center"
        srcset="
            https://mingchang.tw/images/8a82dbf3-62d9-4cfe-1604-860902025000/sm  360w,
            https://mingchang.tw/images/8a82dbf3-62d9-4cfe-1604-860902025000/md  432w,
            https://mingchang.tw/images/8a82dbf3-62d9-4cfe-1604-860902025000/lg  576w,
            https://mingchang.tw/images/8a82dbf3-62d9-4cfe-1604-860902025000/xl  720w
            https://mingchang.tw/images/8a82dbf3-62d9-4cfe-1604-860902025000/2xl 864w
            https://mingchang.tw/images/8a82dbf3-62d9-4cfe-1604-860902025000/max 1080w
        "
        sizes="(max-width: 75%) 100vw, 640px"
        src="https://mingchang.tw/images/8a82dbf3-62d9-4cfe-1604-860902025000/lg"
    />
    <br>
    <p style="align-self: center">
        常見前端框架的記憶體用量，可以看到無論是 Yew 還是 Leptos 都非常高。
        <br>
        取自 <a href="https://github.com/krausest/js-framework-benchmark">js-framework-benchmark</a> 的<a href="https://krausest.github.io/js-framework-benchmark/current.html">官方測試結果</a>
    </p>
</blockquote>

-   檔案大小：WebAssembly 的運作原理有點像 JVM 語言，當選擇 WebAssembly 作為編譯目標時，編譯器就會將程式編譯成 `wasm` 檔，在執行時藉由各種 WebAssembly Runtime 讀取檔案中的 Byte Code 執行，目前編譯出來的檔案預設是靜態鏈結、必須包含原始語言的所有 Runtime 且無法對檔案進行拆分，這導致編譯出來的檔案相較於 JS 會大許多，這個問題在電腦端可能不是太大的問題，但在移動端就有可能讓用戶感受到連線緩慢或載入不順的問題。

<blockquote class="flex flex-col">
    <img
        style="max-width: 75%; align-self:center"
        srcset="
            https://mingchang.tw/images/d3321969-0fa1-400b-c44b-31b9864bd800/sm  360w,
            https://mingchang.tw/images/d3321969-0fa1-400b-c44b-31b9864bd800/md  432w,
            https://mingchang.tw/images/d3321969-0fa1-400b-c44b-31b9864bd800/lg  576w,
            https://mingchang.tw/images/d3321969-0fa1-400b-c44b-31b9864bd800/xl  720w
            https://mingchang.tw/images/d3321969-0fa1-400b-c44b-31b9864bd800/2xl 864w
            https://mingchang.tw/images/d3321969-0fa1-400b-c44b-31b9864bd800/max 1080w
        "
        sizes="(max-width: 75%) 100vw, 640px"
        src="https://mingchang.tw/images/d3321969-0fa1-400b-c44b-31b9864bd800/lg"
    />
    <br>
    <p style="align-self: center">
        常見前端框架的傳輸大小，可以看到所有 WASM 框架，表現都不是很理想。
        <br>
        取自 <a href="https://github.com/krausest/js-framework-benchmark">js-framework-benchmark</a> 的<a href="https://krausest.github.io/js-framework-benchmark/current.html">官方測試結果</a> 
    </p>
</blockquote>

-   Debug 體驗不好：這個問題雖然沒有太影響我（Rust 的 `dbg!` macro 跟永恆不變的 `println!` macro 大法好），但如果會想要在 Runtime 時做 Breakpoint ，或者逐行執行......我覺得還是先不要吧

<blockquote class="flex flex-col">
    <img
        style="max-width: 75%; align-self:center"
        srcset="
            https://mingchang.tw/images/ffac8fb4-a5c9-48c5-79b9-d05920d8fc00/sm  360w,
            https://mingchang.tw/images/ffac8fb4-a5c9-48c5-79b9-d05920d8fc00/md  432w,
            https://mingchang.tw/images/ffac8fb4-a5c9-48c5-79b9-d05920d8fc00/lg  576w,
            https://mingchang.tw/images/ffac8fb4-a5c9-48c5-79b9-d05920d8fc00/xl  720w
            https://mingchang.tw/images/ffac8fb4-a5c9-48c5-79b9-d05920d8fc00/2xl 864w
            https://mingchang.tw/images/ffac8fb4-a5c9-48c5-79b9-d05920d8fc00/max 1080w
        "
        sizes="(max-width: 75%) 100vw, 640px"
        src="https://mingchang.tw/images/ffac8fb4-a5c9-48c5-79b9-d05920d8fc00/lg"
    />
    <br>
    <p style="align-self: center">
        這是我用 Yew 框架寫出來的網頁，編譯出來的 WebAssembly 檔（副檔名 <code>.wasm</code>），經過工具轉換成 <a href="https://developer.mozilla.org/en-US/docs/WebAssembly/Understanding_the_text_format">WebAssembly text format</a> 後的樣子
        <br>
        除了檔案很長（20 多萬行）之外，裡面的內容也難以閱讀，更別說拿去 debug 了。
    </p>
</blockquote>

總之，我認為 WebAssembly 的出現，讓諸如 Rust 的語言得以加入前端生態是件好事，但以其現在的發展與生態系，我覺得其火候未到，尚待未來繼續發展才有可能改變前端的大環境。

## 反思

## HTMX

## 結語
