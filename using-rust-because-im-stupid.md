# 我選擇寫 Rust ，因為我知道我自己並不聰明

一年前，我 commit 了我的第一行 Rust code ，目前 Rust Code 則佔了我自己寫的程式的 69.1 %

現在的我已經可以很自豪的說我可以用 Rust 寫幾乎所有程式。

這篇文章我會分享自己寫程式的經驗，以及為什麼我選擇了 Rust ，可以給別人在選擇該不該學習某種語言時，一點不同於他人的觀點。

> **聲明：這篇文章只是「分享」而非「推坑文」**<br>  
> 偏好這種東西因人而異，像 DHH 就喜歡沒 Type 的語言， ThePrimeagen 覺得沒 Type 很糟糕（我也是這派的）  
> 我尊重所有人的選擇，各位也不要強迫別人去學習某種語言或某種開發習慣喔～

## 我曾接觸過的語言

除了 Python 之外，我並沒有受過任何正式的程式語言教育，我現在會寫的 Swift, Java, JavaScript / TypeScript 跟 Rust 都是我自學的，其中有真的用在工作上的語言有 Java, JS / TS 和 Rust

> 這邊我要稍微說下，我不清楚現在的台灣教育有沒有改變，但在我從國小開始上資訊課到高中畢業的這幾年，我**從來沒有**在學校學習過任何程式語言！  
> 大一時總算有碰到一點點......App Inventor  
> 我絕對不是說每個人都要學習程式語言，但至少在我的成長過程中，文組生（對我是文組仔）如果想要學習程式語言，是非常困難的  
> 當年要不是我和我爸媽主動要求，要不然連 Python 都碰不到，希望現在已經有所改變

先來說說 Java 吧

### Java

它算是我第一個正式用於養活自己的語言，我應徵我的第一份工作時，原本打算寫前端，但過沒多久我就對後端產生強烈興趣，於是跟上司討論後，成功的在新的案子中加入後端開發的行列。

在我寫 Java 的兩年間，我成功的將一堆比我還老、沒人維護的依賴更換成有人維護的依賴、將 Java 8 換到 Java 11 再換到 Java 17 、將 Spring 換成 Spring Boot ，這一切都是我利用我的個人時間看技術文章、看書、看演講和自己測試出來的。學習使用 Java 的同時，我感受到了這個語言的特性： Java 對於「物件」與「型別」的執著達到了我從未體會過的強度，帶來的是我就算不開 IDE ，純看程式也能了解我正在處理什麼型別的物件，就程式碼本身的解釋力而言，我個人覺得非常優秀。

看文件的習慣也是從 Java 學到的，大一點的框架與依賴都會提供 JavaDoc ，我經由閱讀文件學習到別人是如何設計 Library 的，也期許自己在寫程式時也要留下足夠的文件，讓他人可以更容易的了解程式的邏輯與實現。

但 Java 也有令人討厭的地方

#### 1. 惡名昭彰的 `NullPointerException`

Java 的 `null` 可以出現在任何非 primitive value 上，**包含為了解決這問題的 `Optional<T>` ！**這絕對是所有 Java 工程師的噩夢，我曾為了一支 API 瘋狂 `NullPointerException` 熬夜看了一整晚的程式碼，最後索性在每個有可能 `null` 的物件都換上 `Optional<T>`，但這問題還是沒有解決，因為其中一個 `Optional<T>` 隔天就 NPE 了......

#### 2. 程式語言的「贅字」實在是太多了

嘿 Java 工程師們，你們的 `main()` 怎麼寫啊？

蛤什麼？你說你都直接用 Code Snippet ？那如果突然要你在只有 JDK 的環境下用記事本寫個小程式呢？  
不要覺得這事不會出現，因為我就遇過，體驗十分糟糕（（

#### 3. 使用者年齡老化嚴重，語言更新無法帶動使用者更新

其實我覺得這點不能完全怪在 Java 身上，畢竟老同志了。但問題是很多老前輩很固執啊

Java 8 推出已經是 2014 的事情了，當時我還在讀國中，猜猜看 2022 年，我前僱主跟當時大三的我說新專案要用哪版的 JDK 開？

「Java 8 ！」

不是說舊版 JDK 不能用，但一堆新語言特性沒辦法用，到後期我甚至得專門去找 for JDK 8 的依賴才能實現某些功能，反而無意義的增加了開發的困擾。

> 有人會說更新可能會有不預期的情況發生，怎麼會是「無意義」呢？  
> 我認為有因才有果，要先清楚的認識到「是什麼造成了 Bug 發生？」。程式語言為了支援利用舊版語言的程式能夠繼續編譯 / 執行，會盡量避免 Breaking Change 的發生，如果升級到沒有 Breaking Changes 的新版後出現問題，絕大部分都是引用的套件導致的，這時要檢討的應該是套件，而不是語言  
> 如果真的出現語言層級的問題，那大概就是語言設計的問題了

### JavaScript / TypeScript

我的第一份工作與現在的工作都有用到 JavaScript ，但我其實先用了 TypeScript ，因為 Angular 開專案後似乎就會直接帶 TypeScript 了，這兩個語言（或者說一個語言跟一個語言補強工具？ Linter ？）在前端幾乎就是獨佔市場，不用也不行

> 我知道啦，現在可以寫 WebAssembly ，你現在看到的 Blog 就是我用 WebAssembly 寫出來的。但現在 WebAssembly 還是會需要 JS Glue Code 啊，沒辦法 100% 擺脫 JS 的魔掌

弱型別 + 動態型別的語言在需要快速迭代的場景非常適合，不需要花時間去處理型別問題（哈哈這是個謊言）可以把更多的時間用於思考實際的程式邏輯，但我坦白說，我對這兩種語言的好感甚至低於 Java

主要有以下幾點

#### 1. 語法與行為繁雜混亂

-   非同步操作有 Callback 、 `Promise` 跟 `async await` 三種用法：  
    這個在使用 Library 時尤其噁心，為了在使用 `sqlite3` 時也能統一使用 `Promise` ，整個套件被我重新包裝過，最後更是直接用 Rust 寫了個 `rusty-sqlite3` ，從 Library 層面就改用 `Promise`

-   function 除了一般的 function 宣告之外還有 arrow function ：  
    arrow function 用於 function 中作為流程的一部分使用，我覺得完全沒有問題，但問題是我已經看過不少次整份 .ts 檔案沒有半行 function ，全是 `const` 開頭的程式了，閱讀起來超級噁心

-   永遠無法瞬間得出目前的變數到底會被 deep copy 還是 shallow copy ，每天都在貓抓老鼠跟 trial and error：  
    寫這篇文章前我花了四天改了一個 JS 後端，只因為 32 行的某個變數突然失去了它的 field ，最後在不同的 method 發現有一行 code shallow copy 了原本的物件，改動到了原本的值。好笑的是，出錯的地方還不是我自己查到的，我拿去跟我部門三個 JS 工程師一起查才查出來。

-   `const` / `let` / `var` 跟即將到來的 `using` ：特別點名 `const` ，它並不等價於其他語言的 `const` 跟 Rust 不加 `mut` 的 `let` ，它只確保變數不會被重新宣告或重新指定，並不確保裡面的 field 也不會被重新宣告或重新指定，遇到這坑的次數已經多到數不清了

-   Node 知名 `Buffer instanceof Uint8Array` 但行為和 `Uint8Array` 不一致問題：

    ```
    Welcome to Node.js v20.3.1.
    Type ".help" for more information.
    > const nodeBuffer = Buffer.from([1, 2, 3]);
    undefined
    > const uint8Array = new Uint8Array([1, 2 ,3]);
    undefined
    > const slicedBuffer = nodeBuffer.slice(1);
    undefined
    > const slicedArray = uint8Array.slice(1);
    undefined
    > slicedBuffer[0]++;
    2
    > slicedArray[0]++;
    2
    > console.log(nodeBuffer, nodeBuffer instanceof Uint8Array);
    <Buffer 01 03 03> true
    undefined
    > console.log(uint8Array, uint8Array instanceof Uint8Array);
    Uint8Array(3) [ 1, 2, 3 ] true
    undefined
    >
    ```

#### 2. 錯誤處理困難

寫這篇文章的幾天前，知名 TypeScript Wizard （雖然我不怎麼喜歡 TS ，但還是會去學習下用法的）分享了 TypeScript 拒絕 Typed Error GitHub Issue 的 TL;DR ：https://twitter.com/mattpocockuk/status/1677788449368047617

其中有一點我覺得很有趣："In JS there's no culture of libraries declaring what errors they throw"

在其他強型別語言， Error 通常都會繼承自代表錯誤或例外的 Interface / Trait ，並在文件中或型別上提供足夠的資訊供 Binary 用戶或 Library 使用者進行錯誤處理。 JavaScript 系列就沒這回事了，雖然也有 [Error](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error) Object，但一堆 Library 根本就沒有定義他到底會 throw 啥東西出來，不清楚情況的人就只能繼續亂 catch 一通，想要好好做處理的人也只能 trial and error 或直接去翻 Library 的原始碼，這導致要在 JS 下寫穩定程式的難度比起其他程式語言更難。

#### 3. `undefined`

還有需要解釋嗎？[所有的錯都是 Java 害的，真就萬惡之源啊](https://twitter.com/BrendanEich/status/1271998084642246657?s=20)

## 我的 Rust 初體驗

我在一年前的差不多這個時候開始寫 Rust ，會對它感興趣，主要是因為我實在受夠以上這些 Bullshit 了，做了兩年多的維護，只覺得天天都在吃屎、天天都在重複之前的錯誤。 Rust 的官網清楚的寫道 "Performance, Reliability, Productivity" ，這深深的打動了我。

後來發現我日常使用的 M1 MacBook Air 所搭載的 [Apple Silicon Linux GPU Driver 也是用 Rust 寫的](https://asahilinux.org/2022/11/tales-of-the-m1-gpu/)，想著既然 GPU Driver 都能用這語言寫了，想必是真的有效？抱著懷疑的態度，我開始了我的第一個 Rust 專案：用當時最有名的後端庫 Actix Web 和 Diesel ORM 寫個後端。

當時的我在開發上遇到了一些問題：

### IDE 支援較不友好

由於我從 Java 轉向 Rust ，當時我用的 IDE 是 IntelliJ IDEA ，Rust Plugin 很大程度的幫助了我進行程式的開發，但......實在是太慢了。而且當時的 IDEA 預設不會進行 macro 的分析，導致一堆 macro 使用起來體驗非常差

後來被 ThePrimeagen 推坑 Neovim ，用原生的 rust-analyzer 後體驗就好不少了

### 我的 `null` 呢？ `try catch` 呢？？

Java 常用的萬惡之源 `null` ，到了 Rust 是 unsafe 的存在。這個觀念讓我痛苦了很久，因為寫了兩年 Java 的我已經變成 `null` 的形狀了啊（大霧），當然在體會到 `Option<T>` 的好之後，反而讓我後來在 Java 也改用 `Optional<T>` 了

至於 `try catch` 轉向 `Result<T, E>` ，對於我而言反而好改，因為我在前公司根本就沒養成要做錯誤處理的習慣......，用上 `Result<T, E>` 後，才知道原先的做法有可能會出錯

但除了這幾個問題之外，就沒啥感覺了

為什麼呢？

## 我笨，但編譯器可不笨啊

我們來做個小實驗，先在 JavaScript 寫個 function 做 Hello world ，然後貼到 [Rust Playground](https://play.rust-lang.org/) ，直接按下 Build！

```rust
function main() {
    console.log("Hello world!");
}
```

```
   Compiling playground v0.0.1 (/playground)
error: expected one of `!` or `::`, found `main`
 --> src/lib.rs:1:10
  |
1 | function main() {
  | -------- ^^^^ expected one of `!` or `::`
  | |
  | help: write `fn` instead of `function` to declare a function

error: could not compile `playground` (lib) due to previous error
```

我們可以很清楚的看到，Rust 編譯器提供了非常有用的資訊！我們應該要使用 `fn` 而不是 `function` 去宣告一個函數。

類似於這種的有效錯誤訊息，在其他語言真的非常少見，比如我們在 JS 中寫 Rust 的 Hello world：

```javascript
fn main() {
    println!("Hello world!");
}

main() // JS 沒有 main function ，需要主動呼叫
```

```
fn main() {
   ^^^^

SyntaxError: Unexpected identifier 'main'
    at internalCompileFunction (node:internal/vm:73:18)
    at wrapSafe (node:internal/modules/cjs/loader:1177:20)
    at Module._compile (node:internal/modules/cjs/loader:1221:27)
    at Module._extensions..js (node:internal/modules/cjs/loader:1311:10)
    at Module.load (node:internal/modules/cjs/loader:1115:32)
    at Module._load (node:internal/modules/cjs/loader:962:12)
    at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:83:12)
    at node:internal/main/run_main_module:23:47

Node.js v20.3.1
```

看了跟沒看一樣，我還是不知道要改什麼啊

Rust 的編譯器跟 Clippy 提供了非常多有效的建議跟幫助，只要習慣看錯誤提示，甚至可以單靠錯誤提示進行語言的學習，這是跟其他語言不同的，比如第一次寫 Rust 一定會碰到的 borrow checker：

```rust
fn main() {
    let val1 = vec![1, 2];
    let _val2 = val1; // 此處加底線，是為了避免 rustc 噴出未使用變數的警告
    println!("{val1:?}");
}
```

```
   Compiling playground v0.0.1 (/playground)
error[E0382]: borrow of moved value: `val1`
 --> src/main.rs:4:15
  |
2 |     let val1 = vec![1, 2];
  |         ---- move occurs because `val1` has type `Vec<i32>`, which does not implement the `Copy` trait
3 |     let _val2 = val1;
  |                 ---- value moved here
4 |     println!("{val1:?}");
  |               ^^^^^^^^ value borrowed here after move
  |
  = note: this error originates in the macro `$crate::format_args_nl` which comes from the expansion of the macro `println` (in Nightly builds, run with -Z macro-backtrace for more info)
help: consider cloning the value if the performance cost is acceptable
  |
3 |     let _val2 = val1.clone();
  |                     ++++++++

For more information about this error, try `rustc --explain E0382`.
error: could not compile `playground` (bin "playground") due to previous error
```

喔～原來物件已經被移動到 `val2` 了，而且編譯器推薦我如果性能影響可接受，可以考慮使用 `clone()`

但這又是怎麼運作的呢，在 JavaScript 明明是可以用的啊

```javascript
const val1 = [1, 2];
const val2 = val1;

console.log(val1);
```

```
$ node index.js
[ 1, 2 ]
```

讓我們執行看看編譯器提示的 `rustc --explain E0382`

> Since MyStruct is a type that is not marked Copy, the data gets moved out of x when we set y. This is fundamental to Rust's ownership system: outside of workarounds like Rc, a value cannot be owned by more than one variable.<br><br>Sometimes we don't need to move the value. Using a reference, we can let another function borrow the value without changing its ownership. In the example below, we don't actually have to move our string to calculate_length, we can give it a reference to it with & instead.

> 這邊附上[網頁版連結](https://doc.rust-lang.org/stable/error_codes/E0382.html)，其實在 Playground 裡也可以直接點錯誤代碼看到同一個頁面喔！

非常清楚明瞭的說明，這點在我學習 Rust 的時候，真的給了我非常大的幫助。

另外，這個編譯器還能避免開發者出錯，比如上面的 JS 例子，稍微修改下就會開始崩壞了

```javascript
const val1 = [1, 2];
const val2 = val1;

/* 假設這邊有 50 行塞在中間 */

val2.pop();

/* 再假設這邊有 70 行塞在中間 */

console.log(val1);
```

```
$ node index.js
[ 1 ]
```

「奇怪了，我的值到底為什麼被改了？？」接著就只能開始逐行檢查，這段時間根據邏輯的複雜性與寫程式的印象，會不斷的加長

如果換成 Rust ，就是另一個故事了

```rust
fn main() {
    let val1 = vec![1, 2];
    let val2 = val1;

    /* 假設這邊有 50 行塞在中間 */

    val2.pop();

    /* 再假設這邊有 70 行塞在中間 */

    println!("{val1:?}");
}
```

除了上面提過的 `move` 錯誤提示外，你還會收到這個東西：

```
   Compiling playground v0.0.1 (/playground)
error[E0596]: cannot borrow `val2` as mutable, as it is not declared as mutable
 --> src/main.rs:7:5
  |
7 |     val2.pop();
  |     ^^^^^^^^^^ cannot borrow as mutable
  |
help: consider changing this to be mutable
  |
3 |     let mut val2 = val1;
  |         +++
```

編譯器提示，需要我們將變數宣告為可變的變數，才能使用 `pop()` ，可變不可變，清楚明瞭，也不會突然就被改了導致需要重新 Review 自己的程式碼，因為**編譯器提供的保證，除非你主動退出（unsafe），否則一定有效，只要遵守編譯器的規則，就能避免很多的問題**

如果想要做到和 JavaScript 相同的行為，我們需要這樣寫才行

```rust
fn main() {
    let mut val1 = vec![1, 2];
    let val2 = &mut val1;

    /* 假設這邊有 50 行塞在中間 */

    val2.pop();

    /* 再假設這邊有 70 行塞在中間 */

    println!("{val1:?}");
}
```

```
   Compiling playground v0.0.1 (/playground)
    Finished dev [unoptimized + debuginfo] target(s) in 0.60s
     Running `target/debug/playground`

[1]
```

`val2` 需要被宣告為 `val1` 的可變引用（mutable reference），並且 `val1` 也必須利用 `mut` 宣告為可變才能這麼做，這使得開發者能清楚的明白自己所操作的是什麼，維護者維護時所需花費的腦力與時間也能大幅降低

學習 Rust 甚至能學到一些新知識，比如由於 Rust 不像 JavaScript 幾乎所有使用場景都是單線程（沒人愛用 Workers 吧？？？），所以還會有很多特殊的規則是使用 JS 這種主要為單線程的語言所學不到的，比如下面這個例子：

```javascript
const globalVal = [];

async function receiveFromExternalSource() {
    const dataList = await externalSource();
    dataList.forEach((data) => globalVal.push(data));
}

receiveFromExternalSource().then(() => console.log(globalVal));
```

這在 JavaScript 是完全合法的，你可能也不會發現任何問題，那如果換到 Rust 呢？

```rust
use tokio;

static mut GLOBAL_VAL: Vec<String> = vec![];

#[tokio::main]
async fn main() {
    receive_data_from_external_source().await;
}

async fn receive_data_from_external_source() {
    let data_list = external_source().await;
    data_list.into_iter().for_each(|data| GLOBAL_VAL.push(data));
}
```

```
   Compiling playground v0.0.1 (/playground)
error[E0133]: use of mutable static is unsafe and requires unsafe function or block
  --> src/main.rs:12:43
   |
12 |     data_list.into_iter().for_each(|data| GLOBAL_VAL.push(data));
   |                                           ^^^^^^^^^^^^^^^^^^^^^ use of mutable static
   |
   = note: mutable statics can be mutated by multiple threads: aliasing violations or data races will cause undefined behavior

For more information about this error, try `rustc --explain E0133`.
error: could not compile `playground` (bin "playground") due to previous error
```

當時看到這個錯誤後，我才恍然大悟，原來做可變的 global variable 竟有可能造成問題！

> 題外話，遇到這種問題，有兩種解法：<br> 第一種：利用「內部可變」物件將值包起來，就可以正常利用了，比如 `Mutex` 互斥鎖、 `RwLock` 讀寫鎖跟 Atomic
>
> ```rust
> use tokio;
> use std::sync::Mutex;
>
> static GLOBAL_VAL: Mutex<Vec<String>> = Mutex::new(vec![]);
>
> #[tokio::main]
> async fn main() {
>     receive_data_from_external_source().await;
> }
>
> async fn receive_data_from_external_source() {
>     let data_list = external_source().await;
>     let mut locked_global_val = GLOBAL_VAL.lock().unwrap();
>     data_list.into_iter().for_each(|data| locked_global_val.push(data));
> }
> ```
>
> <br>
> 第二種：使用 unsafe ，這種做法相當於跟編譯器說「放心吧我知道這邊有問題，交給我自己處理」
>
> ```rust
> use tokio;
>
> static mut GLOBAL_VAL: Vec<String> = vec![];
>
> #[tokio::main]
> async fn main() {
>     unsafe {
>         receive_data_from_external_source().await;
>     }
> }
>
> async unsafe fn receive_data_from_external_source() {
>     let data_list = external_source().await;
>     data_list.into_iter().for_each(|data| GLOBAL_VAL.push(data));
> }
> ```
