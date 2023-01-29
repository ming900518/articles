# 使用 Yew 框架與 Server Side Rendering，為個人網頁改版

這篇文章，將會盡可能的 ~~從我過年後混亂的記憶中回想這兩個月~~ 記錄我改版個人網頁的過程。

> 本文篇幅長，可善用 Ctrl+F 或 Command+F 直接跳到您想了解的部分

本文用到的一些框架與技術的連結：

- [Yew 框架](https://yew.rs)
- [Axum 框架](https://github.com/tokio-rs/axum)
- [CSR、SSR 的介紹與差別比較](https://shubo.io/rendering-patterns/)

> 如果你現在用的是電腦或 iPad 瀏覽本頁，你所看到的畫面是利用 Client Side Rendering，瀏覽器直接執行由 Yew 框架編譯出的 WebAssembly 所產生的。
> 
> 如果你現在用的是手機瀏覽本頁，你所看到的畫面是利用 Server Side Rendering，由我自架的 Server 執行 Axum 框架，搭配 Yew 框架的 `ServerRenderer`  所渲染出來的。

## 需求確認
最初這個網站，只是為了要交大學的網頁設計課作業，上網找模板修改之後做出來的，模板本身並沒有提供太多功能。

然而最近看到越來越多人使用 Medium 或 GitHub Pages 搭建自己的 Blog 作為技術分享的平臺，我也有些心動，想要將我個人寫在 GitHub Repository 的 Markdown 放到網站上讓人直接查看。

以下是我自己想要實現的一些功能與目標：

1. 將個人網頁從單頁式的 HTML + JS ，利用前端框架轉換成 SPA。
2. 將我寫在[這裡](https://github.com/ming900518/articles)的文章整合到這個網頁中，作為 Blog 與技術文的分享區。
3. 避免重寫文章內容或變更格式。
4. 不架後端與資料庫，壓低運維成本。

## 選擇技術

### 框架
我的前一份工作是在一間做全端系統的公司，利用 Angular 跟 Spring 框架做前後分離的系統給企業或政府單位，所以自己對前後端架構與系統架設都有一定的了解。

在評估要使用什麼框架的時候，當時正在學習 Rust 的我偶然間找到了 Yew 框架，熟悉的語法和幾乎不需要用 JavaScript 這兩點，深深的打動了我。

> 我對 JavaScript 可說是又愛又恨，用 JS 寫一些小專案或 Prototype 可以快速成型。但自從接觸 Rust 之後，JS 的 coercion 等特殊特性帶來的 Runtime Error，以及 Error 後的天書 Error Message 真的讓我難以忍受。
> 
> 「那 TypeScript 呢？」
>
> 首先是 TS 雖然是 JS w/ Type，但其底層依然是那個噁心的 JS，我自己就曾在 Angular 遇到過「看起來型別都沒問題，實際 run 起來卻噴型別有錯」的狀況。
> 
> 還有 `any` 這個型別真的是被濫用到不可思議，我之前的工作與其說用 TypeScript 寫前端，不如說用 AnyScript 寫前端，完全喪失利用 TypeScript 的意義。
>
> 最近有計劃要改用 strict mode TypeScript，我新工作也都是用 NodeJS（我好想換 Deno 😂），可能未來等我學的差不多了，會再寫一篇爬坑記錄吧。
> 
> （這段的 TL;DR：我不喜歡 & 不熟 JavaScript，但我並不排斥使用它）

反正這個網頁並沒有時程壓力也沒有語言要求，剛好也藉由這個機會學習更多知識。

### Markdown
我原本以為可以用 GitHub Pages 產生出來的畫面，套個 iframe 來實現，但經過我多次實驗，外層的網頁似乎是沒辦法對內層的網頁進行太多的設定與美化。

就當我準備開始閱讀[ GitHub Flavored Markdown Spec ](https://github.github.com/gfm/)寫 Parser 時，逛 crate.io 時看到了[ comrak](https://crates.io/crates/comrak)。

詳細的讀過說明文件與範例之後，發現使用方式還算簡單，所以就確定使用這個 crate 了。

### 平台
舊版的個人網頁放在 GitHub Pages 上，網址則是我自己跟中華電信買的，並利用Cloudflare 作為 DNS 與 Proxy，為了就是要盡可能降低成本。

（GitHub Pages 跟 Cloudflare 不用錢，網址一年 NT$800 但一次多買幾年有打折）

當時的想法是成品編譯出來之後，應該還是可以像傳統網頁一樣上傳，所以想要繼續沿用。

後來我才發現坑有點多......

## 將舊網頁轉換成 SPA

如果想要看原始碼的話，這邊有[ Before ](https://github.com/ming900518/ming900518.github.io/tree/old_master)跟[ After ](https://github.com/ming900518/ming900518.github.io/tree/main)可以參考，但我不推薦直接複製裡面的 Code 來用（自學 Rust 新手上路 - 指上路半年多 - 寫的不怎麼好，歡迎 PR）。

在轉換的過程中，當然不可能一帆風順，尤其 Yew 又是一個非常新的框架，在功能還沒有完全 Stable 的情況下，常常有 Breaking Changes 導致 StackOverflow 的回答無法用在新版 Yew 上。

我用的 Yew 版本是 0.20，首先照著 Yew 的文件把專案架好，接著開始爬坑。

### 第一個坑：既有的 JavaScript 套件沒有被包進成品中

如果原有的網頁有利用 JavaScript 套件（如 Bootstrap 之類的排版類套件），那麼你有以下幾種做法可以用：

1. 去找套件有沒有 Rust 版，然後改寫。  
很不幸的是，用 Rust 寫前端的生態還不夠完善，幾乎沒有套件會專門為 Rust 寫 crate。
2. 利用 Trunk（Yew 推薦採用的工具，用來編譯與開發用）的  `data-trunk` 標籤，提示 Trunk 在編譯成品的時候要順便把 JS 文件複製進去。  
詳細的參數可以參考[這裡](https://trunkrs.dev/assets/)。

在處理 JavaScript 的時候，需要特別注意 JS 執行的時間，有些套件需要等到畫面完全渲染完之後才被載入，遇到這種狀況，可以直接將其從 `index.html` 中移除，並在程式中新增一個 function component，讓其跟著 WebAssembly 一起載入：

```rust
#[function_component(MainScript)]
fn main_script() -> Html {
    return html! {
        <>
        <script src="/assets/js/main.js"></script>
        </>
    };
}
```

### 第二個坑：我要怎麼實現 onclick 事件之類的操作呢？

由於 Yew 是透過編譯成 WebAssembly 來實現 Web API 的存取，如果要做與 API 相關的操作，會需要多安裝兩個套件：[`wasm-bindgen`](https://crates.io/crates/wasm-bindgen)  跟 [`web-sys`](https://crates.io/crates/web-sys)。

這邊以實作 `<button>` 的  `onclick` 事件為例：

```html
<div class="col-md-12 text-center" style="padding-top: 2em;">
    <button type="submit" class="button button-a button-big button-rouded">
        <span class="ion-social-twitter">Twitter</span>
    </button>
</div>
```

1. 先把 HTML 改寫成 Yew 的 `html!` macro 支援的格式

    ```rust
    html! {
        <div class="col-md-12 text-center" style="padding-top: 2em;">
            <button type="submit" class="button button-a button-big button-rouded">
                <span class="ion-social-twitter">{"Twitter"}</span>
            </button>
        </div>
    }
    ```

2. 接著把 `onclick` DOM 事件透過 [Callback](https://docs.rs/yew/latest/yew/callback/struct.Callback.html) 加入 HTML 中，這邊接受一個 closure。

    ```rust
    html! {
        <div class="col-md-12 text-center" style="padding-top: 2em;">
            <button 
                type="submit"
                class="button button-a button-big button-rouded"
                onclick={ move |_| {
                    link_click("https://twitter.com/mingchang137");
                }}
            >
                <span class="ion-social-twitter">{"Twitter"}</span>
            </button>
        </div>
    }
    ```

3. 實作 closure 中用到的 function

    ```rust
    fn link_click(link: &str) {
        let document = window()
            .expect("Window not found.")
            .document()
            .expect("Document not found.");
        Location::set_href(&document.location().unwrap(), link)
            .unwrap_or_else(|_| error!("Can't set_href."));
    }
    ```

4. 這邊用到了 Window, Document 跟 Location 三個 Web API，所以我們需要在 `web-sys` crate 開啓相關的 feature

    ```toml
    web-sys = { version = "0.3.60", features = ["Window", "Document", "Location"] }
    ```

以上兩個坑爬完之後，花了一點時間，我就成功的把原本 500 多行的 HTML 成功拆成多個區塊了，整體流程還算簡單。

由於之前有看過 React 的專案，所以對於這種類似 JSX 的寫法也不會感到太陌生。

接著就要挑戰把 GitHub 的文章搬到網頁裡了！我事先先利用套件中有的元素搭出了一個簡單的 UI（就是你現在看這篇文章的 UI），正式開始本次改版的重頭戲。

## Yew Router

> 如果你想知道怎麼做 Server Side Rendering，可以直接跳過這一段，Yew Router 沒辦法用在 SSR 環境下。

由於我們會需要區分頁面需要顯示的內容，在選擇文章到顯示文章內容的頁面之間也會需要傳遞參數

所以我們需要利用Router進行路徑的指定

1. 把 `yew-router` 加到 `Cargo.toml` 中

    ```toml
    yew-router = "0.17"
    ```

2. 在 `main.rs` 中建立一個 enum

    ```rust
    #[derive(Clone, Routable, PartialEq)]
    enum Route {
        #[at("/")]
        Home,
        #[at("/blog/:article_filename")]
        BlogArticle { article_filename: String },
    }
    ```

    `BlogArticle` 的網址中有指定參數article_filename，用來傳遞 Makdown 檔案名稱。

3. 接著建立一個新的 `function_component`，並將上一步建立的 `Route` enum 加入

    ```rust
    #[function_component(App)]
    fn app() -> Html {
        return html! {
            <BrowserRouter>
                <Switch<Route> render={switch} />
            </BrowserRouter>
        };
    }
    ```

4. 指定 Yew 的 Render 渲染 Router

    ```rust
    fn main() {
        yew::Renderer::<App>::new().render();
    }
    ```

5. 建立 `switch` method，搭配 match-case expression 就能根據網址動態切換頁面了

    ```rust
    fn switch(routes: Route) -> Html {
        return match routes {
            Route::Home => {
                    todo!();
            }
            Route::BlogArticle { article_filename } => {
                    todo!();
            }
        };
    }
    ```

## 實現 Blog 文章列表

Blog 必不可少的元素，除了文章之外，還會需要文章列表方便用戶點選文章閱讀。

文章列表中，我認為至少需要出現文章名稱和文章更新時間，而要真的拿到 Markdown 檔案，並轉換成 HTML，則還會需要用到原本的 Markdown 檔名。

原本的想法是利用 GitHub 的公有 API 進行解析，但由於文章名稱在 Markdown 的第一行，要單靠 GitHub API 拿到所有文章的標題，就算不去考慮用戶端設備的的性能，在尚未選擇文章名稱的時候，就把所有文章下載回來解析也不是個好做法。

最後我透過在文章那個 Repository 多放一個 JSON 存放這些訊息，只要需要取得文章列表就過去撈那個檔案就行了。JSON 格式是這樣的：

```json
[
  {
    "name": "文章名稱",
    "date": "1970-01-01T00:00:00Z",
    "url": "FILENAME.md"
  }
]
```

在 GitHub 要拿到原檔，如果是在 Public Repository 中，只需要運用下面這個網址就行了：

```
https://raw.githubusercontent.com/把我替換成用戶名/把我替換成 Repo 名稱/把我替換為 Branch 名稱/把我替換成文件名稱（含副檔名）
```

那麼就開始實作吧！

1. 把所有會用到的 crate 加到 `Cargo.toml` 中

    > 在選用其他 crate 以前要先確認是否能正常編譯到 `wasm32-unknown-unknown` target，避免寫了 Code 之後發生無法正常編譯的慘劇

    ```toml
    serde = { version = "1.0.151", features = ["derive"]} # 用於反序列化，要加上derive feature才能在struct上直接利用Serde的derive macro
    serde_json = "1.0.91" # Serde的JSON實現
    time = { version = "0.3.17", features = ["macros", "serde", "formatting",   "parsing"]  } # 用於解析JSON裡的時間
    wasm-bindgen-futures = "0.4" # 由於我們不希望block住畫面的渲染，在取用API的時候會使用async進行處理，這個crate提供了在WASM中執行Future的方法
    gloo-net = "0.2" # 用於發送Request
    ```

2. 接著將 JSON 要反序列化的目標 struct 做出來

    這邊利用了 Serde 的 derive macro，並指定利用 time crate 內建的 ISO 8601 進行時間的反序列化

    ```rust
    #[derive(Deserialize)]
    struct ArticleData {
        name: String,
        #[serde(with = "time::serde::iso8601")]
        date: OffsetDateTime,
        url: String,
    }
    ```

3. 從 GitHub 取得文章列表 JSON

    ```rust
    // 利用Yew的State，當被取用的時候執行
    let article_data = use_state(|| vec![]);
    {
        let article_data = article_data.clone();
        use_effect_with_deps(
            move |_| {
                let article_data = article_data.clone();
                // 透過wasm_bindgen_futures的spawn_local function執行Rust的Future
                wasm_bindgen_futures::spawn_local(async move {
                    // 使用gloo_net的get method取得JSON
                    let mut fetched_data: Vec<ArticleData> = Request::get(
                        "https://raw.githubusercontent.com/USER/REPOSITORY/BRANCH/JSON_FILE.json",
                    )
                    .send()
                    .await
                    .unwrap()
                    // 由於內容是JSON，利用json method反序列化
                    .json()
                    .await
                    .unwrap();
                    // 這邊是我的需求，我怕我JSON填錯順序😂，所以在反序列化後將Vec重新排序
                    fetched_data.sort_by_key(|x| x.date.clone());
                    // 改成降冪排序，比較符合使用邏輯
                    fetched_data.reverse();
                    // 將整理好的Vec塞回
                    article_data.set(fetched_data)
                })
            },
            (),
        )
    }
    ```

4. 將取得的資料做成 `Html` 物件（因為有多筆，所以最後會是`Vec<Html>`），透過將 Markdown 檔名加到網址中，作為超連結的參數傳給下一頁進行解析

    ```rust
    // 由於資料不止一筆，所以對article_data進行迭代
    let blocks = &article_data.iter().map(|data| {
        return html! {
            <div class="col-md-12">
                <div class="work-box">
                    // 組合網址，這邊透過Yew Router實現參數的取得
                    <a href={format!("/blog/{}", &data.url)}>
                        <div class="work-content">
                            <div class="row">
                                <div class="col-sm-12">
                                    <h2 class="w-title">{&data.name}</h2>
                                    <div class="w-more">
                                        <span class="w-ctegory">{"文章"}</span>{" / "}<span class="w-date">{&data.date.date()}</span>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </a>
                </div>
            </div>
        };
    }).collect::<Html>();
    ```

5. 最後在外層 HTML 中把 `{blocks.clone()}` 塞進文章選項的空位中就完成了。

## GitHub Flavored Markdown to HTML

實現文章列表後，開始實作 Markdown 轉 HTML 的部分

1. 把必要的 crate 加到 `Cargo.toml` 去

    ```toml
    comrak = "0.16" # GitHub Flavored Markdown to HTML
    web-sys = { version = "0.3.60", features = ["Window", "Document"] } # 用於變更網頁title
    ```

2. 建立 struct，一個用於接收參數，另一個用於存放文章標題與內文

    ```rust
    #[derive(Clone, PartialEq, Properties)]
    pub struct Props {
        pub article_filename: String
    }

    #[derive(Clone, PartialEq)]
    struct BlogArticleContent {
        title: String,
        content: String,
    }

    // 內容要等到檔案從GitHub下載回來之後轉換成HTML才會被顯示出來
    // 這邊先為文章標題跟內文做個預設值，避免用戶看到完全空白的畫面
    impl Default for BlogArticleContent {
        fn default() -> Self {
            BlogArticleContent {
                title: String::from("文章載入中，請稍候......"),
                content: String::from("<p>無法載入？請檢查網路環境，或<a href=\"mailto:test@test.com\">與我聯繫</a></p>"),
            }
        }
    }
    ```

3. 將文章從 GitHub 下載回來，並將 Markdown 轉換為 HTML

    ```rust
    // 一樣使用use_state，當被取用的時候執行
    let blog_article_content = use_state(|| BlogArticleContent::default());
    {
        let blog_article_content = blog_article_content.clone();
        let id = props.id.clone();

        use_effect_with_deps(
            move |_| {
                let blog_article_content = blog_article_content.clone();
                let id = id.clone();
                // 透過wasm_bindgen_futures的spawn_local function執行Rust的Future
                wasm_bindgen_futures::spawn_local(async move {
                    // 使用gloo_net的get method取得JSON
                    let response = Request::get(&*format!("https://raw.githubusercontent.com/USER/REPOSITORY/BRANCH/{}", id))
                            .send()
                            .await
                            .unwrap();
                    let fetched_data = response
                        // 由於我們這邊接收的是Markdown，不需要反序列化
                        .text()
                        .await
                        .unwrap();
                    // 判斷是否成功（HTTP Status 200 OK）
                    if response.status() == 200 {
                        // 將拿到的資料根據換行符號做成迭代器後收集成Vec<&str>
                        let collected_data = fetched_data.as_str().lines().collect::<Vec<&str>>();
                        // 這邊我是預設將第一行作為文章的標題，所以將第一行拆開，如果失敗的話顯示載入失敗的訊息
                        let split_data = collected_data.split_first().unwrap_or((&"載入失敗", &["請回上一頁"]));
                        // 把Markdown的標籤去掉
                        let title = &split_data.0[2..];
                        // 透過Web API取得Document，並將文章標題放進Title
                        let document = window()
                            .expect("Window not found.")
                            .document()
                            .expect("Document not found.");
                        // 將文章標題放進Title
                        Document::set_title(&document, &[title, Document::title(&document).as_str()].join(" - "));
                        // 將資料寫進新的BlogArticleContent中
                        blog_article_content.set(BlogArticleContent {
                            title: title.parse().unwrap(),
                            // 利用comrak提供的markdown_to_html_with_plugins function，將Markdown轉成HTML
                            content: markdown_to_html_with_plugins(
                                // 由於前面把Markdown拆成Vec<&str>了，需要先把文章組合回來
                                split_data.1.join("\n").trim(),
                                // 這邊指定轉換的選項，可以根據需求自行選擇，連結：https://docs.rs/comrak/0.16.0/comrak/struct.ComrakOptions.html
                                &ComrakOptions {
                                    extension: ComrakExtensionOptions {
                                        strikethrough: true,
                                        table: true,
                                        tasklist: true,
                                        superscript: true,
                                        ..ComrakExtensionOptions::default()
                                    },
                                    parse: ComrakParseOptions {
                                        smart: true,
                                        ..ComrakParseOptions::default()
                                    },
                                    render: ComrakRenderOptions {
                                        github_pre_lang: true,
                                        unsafe_: true,
                                        ..ComrakRenderOptions::default()
                                    },
                                },
                                // 這邊指定轉換時需要使用的插件，這邊我選擇使用自帶的Syntect Syntax Highlight，連結：https://docs.rs/comrak/0.16.0/comrak/struct.ComrakRenderPlugins.html
                                &ComrakPlugins {
                                    render: ComrakRenderPlugins {
                                        codefence_syntax_highlighter: Some(&SyntectAdapter::new(
                                            "base16-ocean.dark",
                                        )),
                                    },
                                },
                            ),
                        })
                    } else {
                        // 當文件沒有被正常的取回，顯示錯誤碼跟錯誤訊息
                        blog_article_content.set(BlogArticleContent {
                            title: String::from(format!("錯誤代碼：{}", response.status())),
                            content: String::from(format!("<p>請確認網址是否正確，網路環境是否暢通<br>如有疑問請<a href=\"mailto:test@test.com\">與我聯繫</a></p><p>{}</p>", fetched_data)),
                        })
                    }
                });
                || ()
            },
            (),
        );
    }
    ```

4. 在標題空位插入 `{&blog_article_content.title}`

5. 在文章空位插入 `{Html::from_html_unchecked(blog_article_content.content.clone().into())}` 

    > 取得的文章內文雖然是 HTML，但型別是 `String` ，我們在塞入文章內文的空位時需要使用 `Html::from_html_unchecked()` function 進行轉換

## 發佈到網際網路上

寫到這邊，所有想要實現的功能都已經實現了，把所有 Rustc 輸出的問題都修正好之後，利用 `trunk build --release` 成功的包出了網頁（在 `dist` 資料夾中），接著就能像一般前端框架包出來的成品一樣，上傳到伺服器或網頁代管服務去了。

當我上傳到 GitHub Pages 時，卻發現......

由於 GitHub Pages 會根據網址的路徑分配給不同 Repository，所以會先行攔截我組合給 Yew componment 的參數，導致 Routing 出錯，GitHub 回傳 404。

這個問題雖然有 [Workaround](https://github.com/rafgraph/spa-github-pages)，但我自己測試時發現，這會導致一些App沒辦法正常取得網頁的內容，最後考量到使用者體驗跟管理的方便性（還有 [SSL 的問題](https://twitter.com/mingchang137/status/1618951012693446657?s=20)），我就將網頁直接移到 Cloudflare Pages 去了。

就當我以為這件事圓滿結束的時候，預期外的狀況發生了：這支 iPhone 12 mini，就卡在這個頁面長達 10 分鐘載不進去......

## 性能不夠 & 網路不穩的救世主 - Server Side Rendering

還記得前面為文章的標題跟內文設定預設值嗎，還好有設定，要不然估計又會被當 Bug（嘆

~~吃瓜群眾：Rust 不是很安全，bug free 嗎？啊哈哈哈哈~~

有鑑於 Apple M1 的小老弟 Apple A14 都發生了卡住的問題（這好歹是兩年前的旗艦機啊......），這問題實在是非常的嚴重，必須儘快解決。

其實我之前在舊公司寫前端時就常常遇到這問題，明明程式都沒問題，local run 也都沒測試出問題，一給客戶在網路不穩的環境下測試就噴一堆莫名奇妙的錯。

根據我的測試，只有我從我老爸接收的 Asus Zenfone 2（CPU 是 Intel Atom 那支）是因為未知的相容或性能原因而無法載入頁面，其他都是網路不穩導致的。

> 好笑的是，我直接用 M1 MacBook Air 跑 QEMU 模擬出來的 amd64 Windows 10，性能還比某些手機好，我真的是沒看懂為什麼。

在我們寫前端的時候，通常都會採用 CSR （Client Side Rendering）模式進行設計，將所有的前端程式編譯成 JavaScript 或 WebAssembly，再交由客戶端瀏覽器裡的引擎（V8、JavaScriptCore 或 SpiderMonkey）執行，進行 API 的呼叫或資源的下載。

這種做法相較於普通的網頁有著「不易形成用戶體驗割裂」的好處，但壞處也很明顯：太吃用戶的環境了。

程式邏輯越來越長，需要傳輸的程式大小就會越大，執行時需要接收的資料也會越來越多，就以我自己寫出來的個人網頁，WebAssembly 就有 12.4 MB，再加上其他圖片、字體等檔案，加一加也有高達 25 MB 的文件需要傳輸，在網路環境正常的情況下 25 MB 不需要多久就能全部載下來，但如果是移動式設備就不好說了。

這時候，有些腦筋動得快的人就想到了：那如果網頁在伺服器端就已經渲染好，傳到客戶端的資料是不是就可以變少/簡單呢？

Server Side Rendering，應需求而生。

>  SSR 有一個好朋友叫 Static Site Generation（SSG），其做法是在編譯成品時就先把靜態網頁編譯好，這樣就不需要再做運算了，這個比較適合用在靜態網頁上，不在本文的範疇中。

## 實作 Yew 的 Server Side Rendering

如果想要看原始碼的話，這邊有[ SSR 版 ](https://github.com/ming900518/ming900518.github.io/tree/ssr-test)可以參考

首先要先說明，由於 Server Side Rendering 本質上就只是把原本要透過 JavaScript 或 WebAssembly 在客戶端執行的程式改成在伺服器端執行，對於伺服器端的硬體需求會比 Client Side Rendering 還要高，不適合用在需要大吞吐量的場景中。

Yew 在 0.20 的時候發佈了 Server Side Rendering 功能，[這邊](https://yew.rs/docs/advanced-topics/server-side-rendering)是他們提供的文件。

要提供 SSR 服務的話，需要滿足以下幾個條件：

1. 自己的伺服器，我去 [AWS Lightsail](https://lightsail.aws.amazon.com) 開了一個 US$3.5/month 的 instance
2. 會用 HTTP Server Framework，這邊我使用的是 [Axum](https://crates.io/crates/axum)
3. 如果有用到 Web API，要想辦法找到替代方案，或者利用 `use_state()` 強制 Yew 把邏輯放到前端執行

滿足以上條件之後，我們就可以開始把 CSR 程式轉換成 SSR 了。

1. 把 crate 加到 `Cargo.toml` 去，將 Yew 的 feature 改為 `ssr`並 **移除yew-router**。

    ```toml
    yew = { version = "0.20.0", features = ["ssr"] }
    axum = { version = "0.6.4", features = ["query"] } # Axum Web Framework，這邊可以換用自己喜歡的框架，Yew提供的範例是使用Warp
    axum-extra = { version = "0.4.2", features = ["spa"] } # Axum的補充套件，由於我們要提供SPA網頁服務，這邊需要用到特殊的SPA Router提供靜態文件的存取
    tokio = { version = "1.24.2", features = ["full"] } # Tokio非同步
    reqwest = { version = "0.11.14", features = ["json"] } # HTTP Client，用於呼叫API
    ```

    > Yew Router 本身使用 WebAssembly 存取 Web API，在 SSR 環境下是不能被使用的，一執行就會Panic，所以必須將其移除。

2. 在 `main.rs` 中將 main function 替換成 Axum 的 Server

    ```rust
    // 由於Axum基於Tokio實現，所以我們需要透過Tokio的macro來標註main function
    #[tokio::main]
    // 注意這邊要加上async
    async fn main() {
        // 設定IP與Port
        let addr = SocketAddr::from(([0, 0, 0, 0], 80));
        // 由於我們需要讓前端網頁可以存取靜態資源，所以利用SpaRouter建立assets的Router
        let spa = SpaRouter::new("/assets", "assets");
        // 開始設定Axum的Server，將IP與Port bind進去
        axum::Server::bind(&addr)
                // 建立一個新的Router
                .serve(axum::Router::new()
                // 由於無法使用Yew Router，這邊我們利用Axum的Router重新實作routing
                .route("/", get(home_page))
                .route("/blog/:article_filename", get(blog_page))
                // 別忘了把剛剛設定好的SpaRouter加進來
                .merge(spa)
                .into_make_service()
            )
            .await
            .expect("Server startup failed.");
    }
    ```
3. 由於 Yew 透過 SSR 方式渲染後，並不會將 `index.html` 的內容一起渲染出來，我們需要手動製作一個 method 處理

    ```rust
    // 由於Axum也有Html這個struct，我們直接在這邊指定crate
    // 當然，也可以使用as keyword把Axum的同名struct bind成其他名字
    async fn page_assembler(content: String) -> axum::response::Html<String> {
        // 透過Tokio提供的function取得文件
        let index_html = tokio::fs::read_to_string("index.html")
            .await
            .expect("failed to read index.html");
        // 以body標籤作為目標，把index.html切成index_html_before和index_html_after兩部分
        let (index_html_before, index_html_after) = index_html.split_once("<body id=\"page-top\">").unwrap();
        // 這邊將參數中的String push進index_html_before後，再將index_html_after push進去，組合成完整的網頁
        let mut index_html_before = index_html_before.to_owned();
        index_html_before.push_str(content.as_str());
        index_html_before.push_str(index_html_after);
        // 組合完，回傳Html物件
        axum::response::Html::from(index_html_before)
    }
    ```

4. 建立兩個頁面（home 跟 blog）的獨立 function
    ```rust
    // 這個是主頁的GET API會使用的function
    // 將Yew ServerRenderer產生出來的HTML，放到上一步製作的page_assembler製作成完整的HTML
    async fn home_page() -> impl IntoResponse {
        page_assembler(ServerRenderer::<MainApp>::new().render().await).await
    }

    // 這邊和上面的做法一樣，多了參數
    async fn blog_page(Path(article_filename): Path<String>) -> impl IntoResponse {
        // 由於我們需要將參數傳入componment中，所以需要用with_props()搭配closure將參數傳入
        page_assembler(ServerRenderer::<BlogApp>::with_props(|| Props{article_filename}).render().await).await
    }
    ```

到這邊，Server Side Rendering 就設定的差不多了，由於我們要直接在 Server 端執行， 所以可以不需要利用 Trunk 進行編譯，直接 `cargo run` 即可執行。

這邊要特別注意，要將所有使用 Web API 的邏輯都換成替代，或利用 `use_state` 將其強制改為在 Client 端執行。

這邊以上面的「從 GitHub 取得文章列表 JSON」為例，以下是改寫為 SSR 可用的版本：

```rust
async fn fetch_article_data() -> Vec<ArticleData> {
    // 透過reqwest crate，取得JSON
    let resp = reqwest::get("https://raw.githubusercontent.com/ming900518/articles/main/article.json").await.unwrap();
    let mut fetched_data = resp.json::<Vec<ArticleData>>().await.unwrap();
    fetched_data.sort_by_key(|x| x.date.clone());
    fetched_data.reverse();
    fetched_data
}
```

```rust
// 改為使用use_prepared_state，並改為使用async的closure
let article_data = use_prepared_state!(async move |_| -> Vec<ArticleData> { fetch_article_data().await }, ())?.unwrap();
```

要注意的是，由於我們這邊使用非同步的 function 進行 Html 的產生，Yew 提供了 [`HtmlResult`](https://docs.rs/yew/latest/yew/html/type.HtmlResult.html) type

利用這個 Type 才能正確的對渲染時出現的問題進行錯誤處理。

接著，我們就將處理好的網頁部屬到伺服器去，並測試一下結果吧！

<blockquote class="twitter-tweet"><p lang="zh" dir="ltr">Yew SSR（左）v.s. CSR（右）<br>這次不在localhost測，Lighthouse Performance分數居然差這麼多<br>扯<br><br>SSR：<a href="https://t.co/O2XzgAuQB2">https://t.co/O2XzgAuQB2</a><br>CSR：<a href="https://t.co/OyVAI8Vrg6">https://t.co/OyVAI8Vrg6</a> <a href="https://t.co/R7aGs0cy6A">pic.twitter.com/R7aGs0cy6A</a></p>&mdash; Ming Chang (@mingchang@hachyderm.io) (@mingchang137) <a href="https://twitter.com/mingchang137/status/1618885264927260672?ref_src=twsrc%5Etfw">January 27, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## 後記
如果你成功的看到了這邊，恭喜你已經把這篇文章看到最後一部分了。

在學習 Rust 的過程中，我對 Rust 的特性感到興奮，並想繼續使用下去的理由如下：

 - 在其他語言寫 code 容易遇到的各種問題，因為 Rust contributor 對於記憶體安全的追求，而在 Safe Rust 的世界中幾乎不存在。因為這些特性，我也省下了許多維護成本。

 - Rustc 與 Clippy 的錯誤提示雖然容易勸退初學者（笑），但幾乎都是有用的提示，可以靠著 Compile Error 作為學習語言的方式（也就是「編譯器教你寫Code」）。

- Rust 是一種系統語言，本身足夠底層但也不會過於複雜，為其帶來了更多運用場景。在這篇文章中作為前端語言，也是我接觸 Rust 之前從未想過的用途。

在這次將 Rust 用於網頁前端開發的過程中，雖然坑真的挺多（主要是生態不完全，文件數量也不多），但在作出成品的當下，也讓我深刻的感受到自己所使用的程式語言是多麼的強大與萬用。

雖然我目前在職的公司並沒有採用 Rust 作為開發語言，在業界也非常難找 Rust 的工作（Cloudflare 大概是我看過開最多 Rust 職缺的非區塊鏈相關產業企業，等我國軍 Online 結束之後想去試試看 😂 ），但我也有我能做到的事情：將我的經驗與開發過程分享給更多的人知道。

期許未來 Rust 能繼續發展，成為被大眾所廣泛運用的程式語言。
