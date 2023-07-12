# 我選擇寫 Rust ，因為我知道我自己並不聰明

一年前，我的 GitHub 上還看不到 Rust Code ，目前 Rust Code 則佔了我自己寫的程式的 69.1 % ，現在的我已經可以很自豪的說我可以用 Rust 寫幾乎所有程式。

分享自己寫程式的經驗，以及為什麼我選擇了 Rust ，可以給別人在選擇該不該學習某種語言時，一點不同於他人的觀點。

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

-   `const` / `let` / `var` 跟即將到來的 `using` ：特別點名 `const` ，它並不等價於其他語言的 `const` 、 `let mut` ，它只確保變數不會被重新宣告或重新指定，並不確保裡面的 field 也不會被重新宣告或重新指定，遇到這坑的次數已經多到數不清了

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

**以上的一切，其實都可以歸咎於「skill issue」，但我只是想寫個程式，真的有需要自己考慮這麼多？**

## 我的 Rust 初體驗
