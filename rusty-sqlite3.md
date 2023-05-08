# rusty-sqlite3: Why and How

rusty-sqlite3 是我這幾天寫的一個 Node-API Library ，把 Rust 中非常有名且用於非常多專案的 SQL Library - [SQLx](https://github.com/launchbadge/sqlx) 搬到 Node.js 後端專案用

目前滿足了我原先想要做到的所有功能

本文將說明我為什麼要做這個 Library ，以及我如何是如何實作這個 Library 的

> rusty-sqlite3: A SQLite3 Client built with Rust Library SQLx and Neon.  
> npm: [https://www.npmjs.com/package/rusty-sqlite3](https://www.npmjs.com/package/rusty-sqlite3)  
> GitHub: [https://github.com/ming900518/rusty-sqlite3](https://github.com/ming900518/rusty-sqlite3)

## 「為什麼不用 node-sqlite3 ？」

TL;DR: 我有用過，它能做到我要做的事情，但我覺得我能做的更好

我目前的工作可以說是 JavaScript / TypeScript everywhere ，所以當我接到上司要求開發一個服務的任務時，我當然也別無選擇，只能選擇採用 TypeScript 進行開發，採用 Express 搭配 node-sqlite3 做後端

開發的前幾天倒還沒出現什麼嚴重的問題，但當我開始設計資料邏輯時， node-sqlite3 （或者說， JavaScript 本身？）有幾個點我覺得不太優秀

> [node-sqlite3 API Documentation](https://github.com/TryGhost/node-sqlite3/wiki/API)

### 一、語法

如果要用 node-sqlite3 做 Prepared Statement 進行查詢，Code 會長成這樣：

```javascript
const database = new sqlite3.Database("./database.sqlite");
const preparedStatement = database.prepare("select * from target_table where target_column = ?", ["value"], (err) => {
    console.log(err);
});
preparedStatement.all((err, result) => {
    /* 成功與失敗的邏輯 */
});
```

由於資料庫操作為非同步， node-sqlite3 所提供的 API 均需要 Callback ，當需要使用多個 Callback 串在一起時，程式就會變得非常雜亂

> 那如果不管 Callback 呢？  
> 在 API 中有提到 excluding a callback does not turn the API synchronous.，所以也不能不管他喔！

我目前的解決辦法是多寫了一個 utility ，把 Callback 包裝成 Promise

```javascript
const database: Database = new sqlite3.Database("./database.sqlite");

export const db = {
    execute<T>(sql: string, param: unknown[]): Promise<Result<T[], Error>> {
        const exec = new Promise((resolve, reject) => {
            const preparedStatement = database.prepare(sql, param, (err: Error) => {
                err
                    ? reject(err)
                    : preparedStatement.all<T>((err: Error, rows: T[]) => (err ? reject(err) : resolve(rows)));
            });
        });
        return Result.safe(exec) as Promise<Result<T[], Error>>;
    }
};
```

接著再使用包裝後的 method

```javascript
match(await db.execute<SomeType>("select * from target_table where target_column = ?", ["value"]), {
    Ok: (result: SomeType[]) => {
        /* 成功的邏輯 */
    },
    Err: (error: Error) => {
        /* 失敗的邏輯 */
    }
});
```

才成功將程式複雜度稍微降低一點。

### 二、Error Handling 和穩定性

上面那串 code 用到的 `Result<T, E>` 、 `Result.safe()` 跟 `match()` 是 [oxide.ts](https://github.com/traverse1984/oxide.ts) 的 API，把 Rust 的 Result / Option 搬到 JS / TS 來，避免了 try-catch hell，我覺得很好用，但它似乎有段時間沒更新了。

這下第二個問題出現了：這邊如果不使用 oxide.ts 或其他套件進行處理，就會需要先做兩個 Promise ，再套兩個 try-catch 分別 await 才行，根據生物的天性，只要沒有強制要求或強迫症發作，根本就不會有人主動去做 Error Handling（有點暴論，但我目前遇到的情況都是這樣）

更何況，SQLite3 本來就不是 native JS ，所以：

<blockquote class="twitter-tweet"><p lang="zh" dir="ltr">我年初進公司到現在<br>NodeJS 已經遇到兩次 V8 引擎崩潰<br>今天用 sqlite3 ，又遇到 NAPI 崩潰<br><br>以上全都只能去找 Workaround 繞過去<br>開發體驗真的很差</p>&mdash; Ming Chang (@mingchang137) <a href="https://twitter.com/mingchang137/status/1646432894779543555?ref_src=twsrc%5Etfw">April 13, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

如果各位有在關注我的 Twitter ，應該有聽過我使用 Rust 作為我的主力語言，使用 Rust 進行開發，雖然增加了開發時間，但我能在處理完所有 WARNING / ERROR 後自信的說出「我的 Code 沒問題」（除非我邏輯做錯了，那另當別論）

> 其實如果對 Rust 夠熟悉（指願意在假日花時間學習如何讓 rustc 不要嗆你 code 寫太爛），我自己體感開發時間其實不會比其他語言長太多

JavaScript ？不好意思我不敢下這種保證，我到現在還在改我兩年前在前一份工作寫的 Angular 前端

我對我自己寫出來的程式有一定的要求，易於維護、好用這兩點是基本，而我相信我自己能做的更好，反正有空嘛，沒有影響到正事就隨便我整活了

## 開始實作！

我用 Java 、 JavaScript 與 Rust 寫過後端，搭配過 MyBatis 、 MyBatis Plus 、 Spring Data JPA 、 Spring Data R2DBC 、 Diesel 、 SeaQuery 、 SeaORM 跟 SQLx ，每一種 Library 都有其優點與缺點

而我認為其中最好用，也最有可能被搬到 Node.js 専案中的資料庫工具，大概就是 Rust 的 [SQLx](https://github.com/launchbadge/sqlx) 了

> Java 聽說在 WASM-GC 支援後能透過 GraalVM 編譯成 WASM ，但根據我之前寫 Spring Native 的經驗，就算真的搬過去了，編譯時間跟編譯大小會是個嚴重的問題，未來等到 WASM-GC 落地了我可能會嘗試看看吧

SQLx 允許開發者在 Rust 程式中連接到 SQLite、SQL Server、MySQL（MariaDB） 與 PostgreSQL ，並在編譯時檢查 SQL 的正確性，且查詢預設為 Prepared Statement ，不需要自己額外操作，我自己就在我的兩個 Side Project 中用過 SQLx，體驗十分良好

```rust
let Ok(db) = SqlitePool::connect("sqlite://database.sqlite").await else {
    /* 無法連線上 SQLite3 的 Error Handling */
}
match query_as!(SomeType, "select * from target_table where target_column = ?", value).fetch_all(db).await { // 這是 Rust Macro，編譯時會檢查 SQL 語法
    Ok(result: Vec<SomeType>) => /* 成功的邏輯 */,
    Err(error: Error) => /* 失敗的邏輯 */
}
```

> 如果你有發現語法怎麼跟我包裝 node-sqlite3 API 後的語法有點像，沒錯，我就是從 SQLx 搬來的

如果要把其他語言的 Library 放到 Node.js 去用，我查了一下，有兩種做法

-   WebAssembly（https://webassembly.org/）：將 Library 編譯成類似於 Java Bytecode 的中間層語言，然後讓 V8 去執行

-   Node-API（https://nodejs.org/api/n-api.html）：利用 Node.js 提供的 C API ，直接跟 Node.js 底層交流

由於我有寫 WebAssembly 的經驗（參見[「使用 Yew 框架與 Server Side Rendering，為個人網頁改版」](https://mingchang.tw/blog/Personal-Page-Update.md)），所以第一版選擇了 WebAssembly 進行開發，但當我開心的開始寫 code 時，才發現事情並沒有想像中的這麼簡單......

1. SQLx 依賴 Rust 的非同步運作，至少需要開啓 [Tokio](https://tokio.rs/) 、 [async-std](https://async.rs/) 或 actix（其實就是特化版的 Tokio ，用於 [actix-web](https://actix.rs/) 後端框架）其中一種非同步功能才能正常使用，但經過我的測試， SQLx 依賴要求的功能無法被 [wasm-bindgen-future](https://rustwasm.github.io/wasm-bindgen/api/wasm_bindgen_futures/) 支援，所以根本無法被編譯成 WebAssembly

> WebAssembly 目前主流有兩種 build target ，一種是傳統的 WebAssembly: wasm32-unknown-unknown ，另一種是 WASI: wasm32-wasi ，差別在於 WASI (WebAssembly System Interface) 有系統接口，可以支援更多功能，我兩種 target 都試過了，都沒辦法正常編譯。

2. 我不知道要如何在 WASM 中儲存 Async Runtime 跟 Database Connection Pool ，所以當時我想了一個非常糟糕的方法：將 `SqliteConnection` 做成 Raw Pointer 傳回 JS ，需要的時候用戶再回傳回來

由於沒辦法編譯，所以我也沒辦法驗證這個做法是否真的能實現，不過 `cargo check` 是能正常通過的

```rust
// 建立連線成功，把連線做成 Raw Pointer 傳回 JS
Ok(Box::into_raw(Box::new(connection)))

// 查詢時把從 JS 傳回來的 Raw Pointer 轉換回來
let mut db = unsafe { Box::from_raw(connection) };
```

於是，第一次的嘗試就這麼以失敗收場了

<blockquote class="twitter-tweet"><p lang="zh" dir="ltr">好吧失敗了<br>SQLx 的 Tokio dependency 沒辦法包成 WASI</p>&mdash; Ming Chang (@mingchang137) <a href="https://twitter.com/mingchang137/status/1651465971511721990?ref_src=twsrc%5Etfw">April 27, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

> WebAssembly 版的 Code: [https://github.com/ming900518/rusty-sqlite3/commit/2e53a80394965a1d8e81ae070ecf3e95dd5b626d](https://github.com/ming900518/rusty-sqlite3/commit/2e53a80394965a1d8e81ae070ecf3e95dd5b626d)

## Node-API - 比想像中簡單？！

我真的推薦各位如果對 Rust 開發有興趣，一定要關注一下 Reddit 的 [r/rust](https://www.reddit.com/r/rust/) ，我的 Rust 與其說是 [The Rust Programming Language](https://doc.rust-lang.org/book/) 這本書教的，不如說是在 Subreddit 上的大佬們教的

我每天早上通勤時，都會固定滑個半小時 r/rust ，學到了不少東西

其中[這篇文章](https://johns.codes/blog/exposing-a-rust-library-to-node-with-napirs)讓我大開眼界，我還真的不知道寫 Node-API 搭配 [NAPI-rs](https://napi.rs/) 或 [NEON](https://neon-bindings.com/) 竟然可以這麼簡單自然，而且還有不少用到 Tokio Runtime 的例子，這下穩了！

透過別人寫的[範例](https://github.com/owenthereal/neon-tonic-example/blob/master/src/lib.rs)，我才知道原來是可以透過 [once_cell](https://docs.rs/once_cell/1.17.1/once_cell/) 開個空間塞 Tokio Runtime 和資料庫連線

這邊我將 Tokio Runtime 的產生放在 NEON 的 main 中，讓 Library 載入時就去生成 Runtime，並初始化用 once_cell 建立的 static

```rust
static RUNTIME: OnceCell<Runtime> = OnceCell::new();
static CONNECTION: OnceCell<SqliteConnection> = OnceCell::new();
```

```rust
#[neon::main]
fn main(mut cx: ModuleContext) -> NeonResult<()> {
    /* 其他邏輯 */
    if let Err(err) =

        // 初始化 Tokio Runtime
        RUNTIME.get_or_try_init(|| Runtime::new().or_else(|err| cx.throw_error(err.to_string())))
    {
        Err(err)
    } else {
        Ok(())
    }
}
```

接著在建立連線時，將 SQLite3 連線初始化

```rust
fn connect(mut context: FunctionContext) -> JsResult<JsPromise> {

    // 確認連線沒有重複建立
    if CONNECTION.get().is_some() {
        return context.throw_error("Connection to database has already been initalized, could not initalize connection twice.");
    };

    // 確認 Tokio Runtime 存在
    if RUNTIME.get().is_none() {
        return context.throw_error(
            "Could not get the Tokio runtime, which is required for connecting to database.",
        );
    };

    // 從 context 中取得用戶從 JS 傳入的連線 URL
    // JS 的 Type 與 Rust 不同，所以需要進行轉換
    let Ok(connection_link_js) = context.argument::<JsString>(0) else {
        return context.throw_error("No connection link found.")
    };
    let connection_link = connection_link_js.value(&mut context);

    // 由於 SQLx 開啓資料庫連線是非同步的，為了避免阻塞線程，需要建立 JsPromise
    let channel = context.channel();
    let (deferred, promise) = context.promise();

    // 取得 Tokio Runtime
    let rt = RUNTIME.get().unwrap();

    // 切換到 Tokio 的 async 環境中
    rt.spawn(async move {

        // 連線到 SQLite
        let connection_result = SqliteConnection::connect(&connection_link).await;

        // 這邊開始設定 JsPromise 的行為
        deferred.settle_with(&channel, move |mut context| {

            // 如果成功，初始化 static 並回傳 true
            if let Ok(connection) = connection_result {
                CONNECTION.set(connection).unwrap();
                Ok(context.boolean(true))
            } else {
                context.throw_error("Unable to connect with provided info.")
            }
        });
    });

    Ok(promise)
}
```

這樣就完成了 SQLx 的設定，可以嘗試寫 API 進行資料庫的 CRUD 了，這邊我又寫了一支 `execute()` 用來進行資料庫操作，然後就......

> 程式碼在[這](https://github.com/ming900518/rusty-sqlite3/blob/f08a3107489bfbb6119557eb6a75c85d3536a68d/src/lib.rs#L44)

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">NodeJS SQLite3 Connection with Rust SQLx! <a href="https://t.co/sJQCZd3fk0">pic.twitter.com/sJQCZd3fk0</a></p>&mdash; Ming Chang (@mingchang137) <a href="https://twitter.com/mingchang137/status/1653973158935031808?ref_src=twsrc%5Etfw">May 4, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

成功了！

## 型別轉換

嘿，就如本段標題所述，我寫的太開心，完全忘了型別轉換這回事，如果仔細看上一段最後一篇 Tweet 的附圖，你會發現 column `user_id` 的 field 居然被 parse 成 string 了

這樣可不對，必須要想辦法修正它，然而我發現並沒有這麼簡單

下面是我整理的 SQLite3、Rust 與 JavaScript 型別對應表（僅列出不需要另外裝套件的型別，[參考文獻](https://docs.rs/sqlx/latest/sqlx/sqlite/types/index.html)）

| SQLite3         | Rust                              | JavaScript              |
| --------------- | --------------------------------- | ----------------------- |
| `BOOLEAN`       | `bool`                            | `boolean`               |
| `INTEGER`       | `i8` `i16` `i32` `u8` `u16` `u32` | `number`                |
| `BIGINT` `INT8` | `i64`                             | `number`                |
| `REAL`          | `f32` `f64`                       | `number`                |
| `TEXT`          | `&str` `String`                   | `string`                |
| `BLOB`          | `&[u8]` `Vec<u8>`                 | `Buffer` (Node.js 才有) |

啊這

我決定先不去處理 `BLOB` 與 `BOOLEAN` 型別， `BLOB` 我目前沒有用到 ，之後會再補，而 `BOOLEAN` 可以透過數字 0 和 1 表示，JS 的 type coercion 可以自動處理這塊

而數字型別與文字型別，我目前是用下面這串程式處理的

```rust

// 建立 JsArray
let array = context.empty_array();

// 將查詢出來的結果 for_each 出來處理
results.into_iter().for_each(|row| {

    // 取得 column 長度
    let column_len = row.columns().len();

    // 由於資料結構是 object[] 所以需要建立 JsObject
    let object = context.empty_object();

    // 將每個 column 下的 field for_each 出來
    (0..column_len).for_each(|index| {

        // 由於 column 都是 string ，所以可以直接轉換成 JsString 不需要做型別判定
        let column = context.string(row.column(index).name());

        // 嘗試 parse 目標欄位的型別
        // 這邊需要由嚴格往寬鬆進行，如果一剛開始就用 string 進行 parsing ，就會變會全部都是 string 的情況了

        // 第一關：是否為整數，且是否為空（Rust 的 integer 和 float 是分開的，不能合在一起稱作 number，如果不用 Option<T> 判斷是否為空則遇到空值時會被 parse 成 string 的 'null'）
        let field = if let Ok(option_i32_value) =
            row.try_get::<Option<i32>, usize>(index)
        {
            if let Some(i32_value) = option_i32_value {
                // 不為空，填入 JS 的 number
                context.number(i32_value).as_value(&mut context)
            } else {
                // 為空，填入 JS 的 null
                context.null().as_value(&mut context)
            }
        }

        // 第二關：是否為浮點數
        else if let Ok(f64_value) = row.try_get::<f64, usize>(index) {
            // 是浮點數，填入 JS 的 number
            context.number(f64_value).as_value(&mut context)
        }

        // 第三關：是否為字串
        else if let Ok(string_value) = row.try_get::<&str, usize>(index) {
            // 是字串，填入 JS 的 string
            context.string(string_value).as_value(&mut context)
        }

        // 都無法 parse 過的情況目前用 undefined 處理
        else {
            context.undefined().as_value(&mut context)
        };

        // 將資料塞回 JS 的 object 中
        object.set(&mut context, column, field).unwrap();
    });

    // 找到 array 的長度，並將處理好的物件塞入 array 的最後
    let array_len = array.len(&mut context);
    array.set(&mut context, array_len, object).unwrap();
});
```

這樣就搞定部分的型別問題了！

> 其他型別？ [eta son](https://www.urbandictionary.com/define.php?term=eta%20son)

<blockquote class="twitter-tweet"><p lang="zh" dir="ltr">rusty-sqlite3 更新至 0.4.2 版<br><br>1. 修復了已知的型別錯誤問題，現在 null 不會再被 parse 成 string 或 0 了<br>2. 支援 SQL 參數<br>3. 增加 TypeScript 支援<br>4. 優化 README ，並附上使用範例<br>5. MIT 開源<a href="https://t.co/kPffxogBTQ">https://t.co/kPffxogBTQ</a> <a href="https://t.co/anMBZQTuvE">pic.twitter.com/anMBZQTuvE</a></p>&mdash; Ming Chang (@mingchang137) <a href="https://twitter.com/mingchang137/status/1654379292409872386?ref_src=twsrc%5Etfw">May 5, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## 結語

在這次的開發過程中，我學會了以下幾件事情

1. 如何用 Rust 寫 Node.js 套件
2. 如何在 Library 中用 `static`
3. WebAssembly 目前還沒有辦法提供完整的非同步（希望未來 WASI 可以補上這一塊）

當然，在 Rust 相關的文章中，絕對不能少了性能測試啊

所以

<blockquote class="twitter-tweet"><p lang="zh" dir="ltr">稍微測了下 node-sqlite3 跟我前幾天花幾個小時寫的 rusty-sqlite3 性能<a href="https://t.co/BzcnYQpgdy">https://t.co/BzcnYQpgdy</a><br>結論：Tokio 萬歲</p>&mdash; Ming Chang (@mingchang137) <a href="https://twitter.com/mingchang137/status/1655398029657260037?ref_src=twsrc%5Etfw">May 8, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

RPS node-sqlite3 1813.5947 / rusty-sqlite3 1963.5887 ，我甚至還沒優化過呢 😂

未來這個套件還會繼續更新，如果有任何問題或想要的功能，歡迎在 [GitHub Repository](https://github.com/ming900518/rusty-sqlite3) 中提 Issue 或 PR

感謝你能看到這裡，我們下次再見～
