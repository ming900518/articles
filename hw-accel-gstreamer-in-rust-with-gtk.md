# 用 Rust 搭配 GTK 與 GStreamer 實作簡易硬體加速全螢幕影片播放器

最近因為工作上有在 Raspberry Pi 上流暢播放 1080p VP9 和 4K HEVC 影片，並提供其他程式串接的需求，於是花了數個禮拜去研究 GStreamer 。雖然最後有成功做出符合要求的播放器軟體，但因為 Rust 資源實在是太少了，開發過程幾乎都只能看著 Python 範例或 `gst-launch-1.0` 指令 pipeline 搭配 C 原始碼，再透過人腦翻譯成 Rust ，十分痛苦。

這篇文章會從全新的 Rust 專案開始，一步步教學如何用 GTK 與 GStreamer 實作可以正常調用常見硬體加速的全螢幕影片播放器。

> 照著本文實作出的程式，理論上可以在任何能安裝 GTK 和 GStreamer Runtime 的環境中執行，我只有在 macOS Sequoia 及 Debian trixie 環境下測試過

> 由於 docs.rs 上的 GTK/GStreamer 文件不太完整，我個人比較推薦閱讀他們官網提供的 Rustdoc
> - GTK4：[https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/)
> - GStreamer：[https://gstreamer.freedesktop.org/documentation/rust/stable/latest/docs/index.html?gi-language=rust](https://gstreamer.freedesktop.org/documentation/rust/stable/latest/docs/index.html?gi-language=rust)

<h2 id="content-table">傳送門</h2>
<ul>
    <li><a href="#step1">專案設定與依賴安裝</a></li>
    <li><a href="#step2">利用 GTK 建立程式主畫面</a></li>
    <li><a href="#step3">GStreamer 設定</a></li>
    <li><a href="#step4">取得影片路徑</a></li>
    <li><a href="#step5">用 GStreamer Play 播放影片</a></li>
    <li><a href="#step6">讓影片在既有的視窗中渲染</a></li>
    <li><a href="#step7">優化使用體驗：讓用戶在播本地影片時可以不用輸入帶 <code>file://</code> 開頭的絕對路徑</a></li>
</ul>

如果 TL;DR 也可以直接看<a href="#source">完整範例程式碼</a>

<h2 id="step1">專案設定與依賴安裝</h2>

首先當然是用萬能的 Cargo 產生專案

```shell
cargo new *改成你的專案名稱*
```

接著引入 GTK 與 GStreamer 依賴

```shell
cargo add gtk4
```
```shell
cargo add gstreamer
```

GStreamer 有提供好用的 `gstreamer-play` ，它可以簡化 GStreamer 程式中處理不同格式及來源的影片與軟硬體相容的相關問題

```shell
cargo add gstreamer-play
```

> 1. 如果想要使用 GTK3 ，除了 `gtk4` 要改成安裝 `gtk` crate 外， `gstreamer` 的版本也需要降級至 0.21 版，否則會出現因為依賴樹不相同導致無法正常引用 trait method 的問題
> 2. 根據我的測試，連 Raspberry Pi 的奇葩硬體都能正常被 `gstreamer-play` 中的 `playbin` 認出並調用可用的硬體加速，所以相容性問題應該還好，如果你已經知道你的硬體不支援 `playbin` ，那本文可能就不適用於你的硬體了

安裝完依賴，接下來需要在電腦中安裝 GTK 和 GStreamer 編譯所需的 library ，下面以 Debian 與 macOS 為例：

1. Debian
    ```shell
    sudo apt install libgtk-4-dev libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev libgstreamer-plugins-bad1.0-dev gstreamer1.0-plugins-base gstreamer1.0-plugins-good gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly gstreamer1.0-libav gstreamer1.0-tools gstreamer1.0-x gstreamer1.0-alsa gstreamer1.0-gl gstreamer1.0-gtk3 gstreamer1.0-gtk4 gstreamer1.0-qt5 gstreamer1.0-pulseaudio
    ```

2. macOS
    ```shell
    brew install gtk4 gstreamer 
    ```

做完以上操作，應該就能正常編譯專案了

<h2 id="step2">利用 GTK 建立程式主畫面</h2>

首先，我們需要先初始化 GTK 並建立一個視窗給 GStreamer 渲染

```rust
use gtk4::prelude::*;
use gtk4::{Application, ApplicationWindow, glib};

fn main() -> glib::ExitCode {
    let app = Application::builder() // GTK 程式的 Builder
        .application_id("tw.mingchang.example-player") // 給你的程式一個獨一無二的 ID
        .build(); // 產生程式！

    app.connect_activate(|app| { // 當 GTK 成功啓用剛剛產生的程式後，執行這個 closure
        init_gstreamer().expect("Unable to init GStreamer."); // 初始化 GStreamer
        let window = init_window(app); // 建立 GTK 視窗
    });

    app.run() // 執行我們已經建立好的 GTK 程式
}

fn init_window(app: &Application) -> ApplicationWindow {
    let window = ApplicationWindow::builder() // 建立主視窗
        .application(app) // 指定這個視窗所屬的程式
        .title("Player") // 給這個視窗一個標題
        .build(); // 產生視窗！

    window.present(); // 讓視窗顯示出來
    window // 回傳建立好的視窗
}
```

透過 `cargo run` 執行後，會看到一個什麼都沒有的空白視窗

<div style="display: flex; flex-flow: row">
    <div style="margin: auto; width: 80vw; padding: 10px; display: flex; flex-flow: column; align-items: center">
        <img
            srcset="
                https://mingchang.tw/images/4c2c1089-368a-4027-ef37-16e838554d00/sm  360w,
                https://mingchang.tw/images/4c2c1089-368a-4027-ef37-16e838554d00/md  432w,
                https://mingchang.tw/images/4c2c1089-368a-4027-ef37-16e838554d00/lg  576w,
                https://mingchang.tw/images/4c2c1089-368a-4027-ef37-16e838554d00/xl  720w
                https://mingchang.tw/images/4c2c1089-368a-4027-ef37-16e838554d00/2xl 864w
                https://mingchang.tw/images/4c2c1089-368a-4027-ef37-16e838554d00/max 1080w
            "
            class="rounded-lg"
            style="max-height: 50vh"
            src="https://mingchang.tw/images/4c2c1089-368a-4027-ef37-16e838554d00/lg"
        />
    </div>
</div>
<br>

接著，我們利用 GTK 提供的設定，將視窗設定為全螢幕

```rust
fn init_window(app: &Application) -> ApplicationWindow {
    let window = ApplicationWindow::builder()
        .application(app)
        .fullscreened(true) // 加入這行
        .title("Player")
        .build();

    window.present();
    window
}
```

這下就成功建立一個全螢幕視窗，可以給 GStreamer 渲染了

<h2 id="step3">GStreamer 設定</h2>

在開始對 GStreamer 做操作前，我們需要先初始化 GStreamer 

```rust
fn init_gstreamer() -> Result<(), Box<dyn std::error::Error>> {
    gstreamer::init()?; // 利用 GStreamer 提供的 function 初始化 GStreamer
    Ok(())
}
```

接著載入所有 plugin ，讓程式可以支援盡可能多的影音格式與編解碼器

> 之所以這麼做是為了提高範例用程式的可用性，在正常情況下不太建議這麼做，會浪費大量資源
> 如果事先就知道要在哪種硬體播放哪種格式的影片，可以只引入需要的那幾個 plugin 就好

```rust
use gstreamer::Registry; // 引入 Registry

fn init_gstreamer() -> Result<(), Box<dyn std::error::Error>> {
    gstreamer::init()?;

    let registry = Registry::get(); // 取得 GStreamer 的 plugin 列表，詳細資訊可以在 https://gstreamer.freedesktop.org/documentation/rust/stable/latest/docs/gstreamer/struct.Registry.html 查看官方文件

    for plugin in registry.plugins() {
        if plugin.is_loaded() { // 如果 plugin 已經被 GStreamer 引入了，不要重複引入
            println!(
                "Preloaded GStreamer plugin {} detected. {}",
                plugin.plugin_name(),
                plugin.description()
            );
        } else if let Err(error) = plugin.load() { // 引入尚未引入的 plugin ，並利用 if let 語法對引入 plugin 的結果做 pattern matching
            eprintln!(
                "Failed to load GStreamer {} plugin: {error:?}.",
                plugin.plugin_name()
            ); // 無法載入 plugin 不會影響 GStreamer 的運作，但該 plugin 所提供的功能將會無法正常使用，這邊我選擇將錯誤訊息輸出，可以根據需求改寫成需要的行為
        } else {
            println!(
                "GStreamer plugin {} loaded. {}",
                plugin.plugin_name(),
                plugin.description()
            ); // 成功載入 plugin 時也可以在這裡做點事情
        }
    }

    Ok(())
}
```

```rust
fn main() -> glib::ExitCode {
    // 省略......

    app.connect_activate(|app| {
        init_gstreamer().expect("Unable to init GStreamer."); // 加入這行初始化 GStreamer ，當初始化失敗時直接 panic
        let window = init_window(app);
    });

    // 省略......
}
```

<h2 id="step4">取得影片路徑</h2>

這個部分就要看業務需求而定了，我自己是比較喜歡用 `serde_json` 做 JSON 設定檔或用 `clap` 做完整的 CLI ，但這邊為了簡單展示，直接從指令行讀取參數

由於在 GTK 中， 指令行處理需要額外指定 flag ，我們需要將 `main()` 改寫成下面這樣

```rust
fn main() -> glib::ExitCode {
    let app = Application::builder()
        .application_id("tw.mingchang.example-player")
        .flags(ApplicationFlags::HANDLES_COMMAND_LINE) // 新增這一行，將「處理指令行」的 flag 加到 GTK 程式中
        .build();

    app.connect_command_line(move |app, command_line| { // 將原有的 `connect_activate` 改為 `connect_command_line` ，closure 會多一個型別為 `&ApplicationCommandLine` 的匿名函數，我將其命名為 `command_line`
        let args = command_line.arguments(); // 從指令行中取出所有參數

        let Some(video_path) = args
            .get(1) // 「所有參數」包含程式本身的路徑，以 0-based 來看的話，要取得第二個參數要用 `get(1)` 才行
            .and_then(|os_string_arg| os_string_arg.to_str()) // 將取出的參數從 `OsString` 型別轉換成 `&str` 
            .map(ToOwned::to_owned) // 再將 `&str` 轉換為 `String` 避免後面為 GStreamer 開啓新線程時存取這個值會發生的 lifetime issue
        else {
            panic!("Video path needed."); // 找不到參數時，直接 panic 退出程式
        };

        let window = init_window(app);
        init_gstreamer().expect("Unable to init GStreamer.");

        0
    });

    app.run()
}
```

<h2 id="step5">用 GStreamer Play 播放影片</h2>

好了，接下來就是重頭戲了：讓 GStreamer 開始播影片

首先先透過 `gstreamer-play` 中的 `Play` struct 嘗試播放影片

```rust
use gstreamer_play::{Play, PlayMessage, PlayVideoRenderer};

fn play_video(path: &str) -> Result<(), Box<dyn std::error::Error>> {
    let play = Play::new(None::<PlayVideoRenderer>); // 建立一個新的播放器，不指定渲染器讓 `gstreamer-play` 自己決定如何渲染
    play.set_uri(Some(path)); // 將從指令行參數讀進來的路徑設定進播放器中
    play.play(); // 開始播放！

    for msg in play.message_bus().iter_timed(ClockTime::NONE) { // 從 GStreamer Message Bus 取得播放時的訊息
        match PlayMessage::parse(&msg) { // 用 `PlayMessage` 解析接收回來的訊息
            Ok(PlayMessage::EndOfStream) => { // 當解析成功，訊息內容是「串流結束」時
                println!("Video ended."); // 輸出「影片結束」訊息
                play.stop(); // 停止播放
                break; // 中斷讀取 Message Bus
            }
            Ok(PlayMessage::Error { error, details }) => { // 當解析成功，訊息內容是「錯誤」時
                eprintln!("GStreamer error: {error} ({details:?})"); // 將錯誤訊息輸出
                play.stop(); // 停止播放
                break; // 中斷讀取 Message Bus
            }
            Ok(PlayMessage::Warning { error, details }) => { // 當解析成功，訊息內容是「警告」時
                eprintln!("GStreamer warning: {error} ({details:?})"); // 將警告訊息輸出後，繼續播放
            }
            Ok(_) => (), // 暫時不處理其他訊息
            Err(_) => unreachable!(), // 理論上不會發生解析錯誤的場景
        }
    }

    play.message_bus().set_flushing(true); // 確保所有訊息均已送完

    Ok(())
}
```

```rust
fn main() -> glib::ExitCode {
    // 省略......

    app.connect_command_line(move |app, command_line| {
        // 省略......
        init_gstreamer().expect("Unable to init GStreamer.");

        std::thread::spawn(move || { // 建立一個新的線程給播放器使用
            play_video(&video_path)
                .unwrap_or_else(|error| panic!("Unable to play video from {}: {error:?}", &video_path)); // 當無法正常播放時，輸出不能播放該路徑
        });

        0
    });

    // 省略......
}
```

測試時，先利用 `yt-dlp` 取得 YouTube 影片的實際播放網址，並塞到參數中執行！

```shell
cargo run -- $(yt-dlp -f 315 --get-url https://www.youtube.com/watch\?v\=nHnFfHaZZSw)
```

> 這邊的影片是我[去年去日本搭 Hello Kitty Shinkansen](/blog?filename=japan-travel-log-2024-d6.md#kodama849) 時，自己用 iPhone 在博多站錄的影片，最高畫質是 4K 60FPS HDR ，編碼是 VP9 ，影片連結有鎖 IP 跟時間，所以沒辦法共用  
> 可以透過以下指令查詢到 YouTube 轉換的各種格式，查到 ID 後，替換掉 `yt-dlp` 指令 `-f` 參數後的數字即可更換畫質
>
> ```shell
> ytdlp --list-formats https://www.youtube.com/watch\?v\=nHnFfHaZZSw
> ```
> 另外， YouTube 在近幾年都將影片的視訊部分與音訊部分分開了，所以沒有聲音是正常的

<div style="display: flex; flex-flow: row">
    <div style="margin: auto; width: 80vw; padding: 10px; display: flex; flex-flow: column; align-items: center">
        <img
            srcset="
                https://mingchang.tw/images/56af42ea-9c9a-46c9-0d9c-f3052f637100/sm  360w,
                https://mingchang.tw/images/56af42ea-9c9a-46c9-0d9c-f3052f637100/md  432w,
                https://mingchang.tw/images/56af42ea-9c9a-46c9-0d9c-f3052f637100/lg  576w,
                https://mingchang.tw/images/56af42ea-9c9a-46c9-0d9c-f3052f637100/xl  720w
                https://mingchang.tw/images/56af42ea-9c9a-46c9-0d9c-f3052f637100/2xl 864w
                https://mingchang.tw/images/56af42ea-9c9a-46c9-0d9c-f3052f637100/max 1080w
            "
            class="rounded-lg"
            style="max-height: 50vh"
            src="https://mingchang.tw/images/56af42ea-9c9a-46c9-0d9c-f3052f637100/lg"
        />
    </div>
</div>
<br>

嗯......為什麼 GStreamer 開了另一個叫做「OpenGL renderer」的視窗來播影片，而不是直接用我們剛剛建立好的視窗來播放呢？

<h2 id="step6">讓影片在既有的視窗中渲染</h2>

`gstreamer-play` 其實有點像是官方提供的懶人包，它調用的 [`playbin`](https://gstreamer.freedesktop.org/documentation/playback/playbin.html?gi-language=c) 會自動將傳入的路徑解析後對內容進行分析，分析後再用它認為最適合的播放方式進行播放

在我的 MacBook Air 及 Debian 桌機上， `playbin` 都選擇了 [`glimagesink`](https://gstreamer.freedesktop.org/documentation/opengl/glimagesink.html?gi-language=c) 來播放影片，雖然 `glimagesink` 確實能正常調用硬體加速播放用戶指定的影片，但渲染在另一個視窗會讓我們沒辦法很好的掌控播放畫面。

> 我的 MacBook Air M1 只有 H.264 (AVC1) 跟 H.265 (HEVC) 硬體解碼（Apple Silicon 從 M3 開始才有辦法硬解 AV1 ，其他編碼一樣沒有），所以「調用硬體加速」我是用 Debian 桌機上的 Intel UHD Graphics 770 測試的，不同硬體可能會有不同結果

> 其實 `glimagesink` 也能透過設定 window handle 跟 display handle (Wayland) 的方式指定輸出到某個視窗/螢幕上，但根據我自己~~被 Wayland 折磨了三天~~的經驗，我覺得這個部分改天等我找到更好的做法後再出一篇文章好了

經過~~我爬坑時連續三天爬文測試~~仔細的研究，我發現 GStreamer 有提供一個特殊的 sink 可以將影片作為 GTK Paintable ，讓我們可以在 GTK Picture widget 中運用：[`gtk4paintablesink`](https://gstreamer.freedesktop.org/documentation/gtk4/index.html?gi-language=c) ，這個 sink 提供了 `paintable` property ，這個就是我們的 GTK Paintable 了

讓我們用 `gtk4paintablesink` 搭配 `PlayVideoOverlayRenderer` 嘗試在 GTK 中直接播放影片

> 這個 sink 是提供給 GTK 4 用的，GTK 3 請改用 [`gtkglsink`](https://gstreamer.freedesktop.org/documentation/gtk/gtkglsink.html?gi-language=c#gtkglsink) ，該 sink 提供了 `widget` property 可以直接取得 Widget ，兩個 sink 設定方式很像，後面有遇到 GTK 4 與 GTK 3 設定不一樣的地方，會在範例程式碼中另行標註

```rust
use gstreamer_play::{Play, PlayMessage, PlayVideoOverlayVideoRenderer};

fn play_video(path: &str) -> Result<Paintable, Box<dyn std::error::Error>> {
    let gtk4paintablesink = ElementFactory::make_with_name("gtk4paintablesink", None)?; // 引入 `gtk4paintablesink`， GTK 3 請改成 `gtkglsink`
    let glsinkbin = ElementFactory::make_with_name("glsinkbin", None)?; // 由於 `gtk4paintablesink` 接受的格式為 OpenGL 材質，我們需要利用 `glsinkbin` 將影片轉換過去才能正常播放
    glsinkbin.set_property("sink", &gtk4paintablesink); // 在 `glsinkbin` 中設定 `sink` property 為我們剛剛引入的 `gtk4paintablesink`

    let play = Play::new(Some(PlayVideoOverlayVideoRenderer::with_sink(&glsinkbin))); // 在播放器指定 Renderer 為帶 sink 的 `PlayVideoOverlayVideoRenderer`

    let paintable: Paintable = gtk4paintablesink.property("paintable"); // 從 sink 中取得 `Paintable`， GTK 3 請將目標改成 `widget`，型別改成 `Widget`

    play.set_uri(Some(path));
    play.play();

    std::thread::spawn(move || {  // 由於我們需要回傳從 sink 取得的 `Paintable` 回主線程，所以將原本於 `connect_command_line` closure 開啓的線程改到這邊
        for msg in play.message_bus().iter_timed(ClockTime::NONE) {
            // 省略......
        }
        play.message_bus().set_flushing(true);
    });

    Ok(paintable) // 回傳建立好的 `Paintable`
}
```

```rust
fn main() -> glib::ExitCode {
    // 省略......
    app.connect_command_line(move |app, command_line| {
        // 省略......
        init_gstreamer().expect("Unable to init GStreamer.");

        let widget = Picture::new(); // 建立 `Picture` 供等等從 GStreamer 回傳回來的 `Paintable` 渲染（GTK 3 不需要做這步）

        let paintable = play_video(&video_path)
            .unwrap_or_else(|error| panic!("Unable to play video from {}: {error:?}", &video_path)); // 由於已經在 `play_video` 中建立新線程，這邊的 `spawn` 就不再需要了

        widget.set_paintable(Some(&paintable)); // 將取回的 `Paintable` 塞進 `Picture` 中（GTK 3 取回的是 `Widget`，不需要做這步）

        window.set_child(Some(&widget)); // 將 `Picture` widget 設定為 GTK 視窗的子元素（GTK 3 直接將 `play_video` 回傳的 widget 在這塞進來就可以了）

        0
    });

    app.run()
}
```

再用 `cargo run -- $(yt-dlp -f 315 --get-url https://www.youtube.com/watch\?v\=nHnFfHaZZSw)` 執行

<div style="display: flex; flex-flow: row">
    <div style="margin: auto; width: 80vw; padding: 10px; display: flex; flex-flow: column; align-items: center">
        <img
            srcset="
                https://mingchang.tw/images/6f54d36e-44a9-4ebf-38ec-8abee2c88500/sm  360w,
                https://mingchang.tw/images/6f54d36e-44a9-4ebf-38ec-8abee2c88500/md  432w,
                https://mingchang.tw/images/6f54d36e-44a9-4ebf-38ec-8abee2c88500/lg  576w,
                https://mingchang.tw/images/6f54d36e-44a9-4ebf-38ec-8abee2c88500/xl  720w
                https://mingchang.tw/images/6f54d36e-44a9-4ebf-38ec-8abee2c88500/2xl 864w
                https://mingchang.tw/images/6f54d36e-44a9-4ebf-38ec-8abee2c88500/max 1080w
            "
            class="rounded-lg"
            style="max-height: 50vh"
            src="https://mingchang.tw/images/6f54d36e-44a9-4ebf-38ec-8abee2c88500/lg"
        />
    </div>
</div>
<br>

成功了！而且硬體加速也能正常的被調用！

> macOS 似乎在某些情況下會有無法正常渲染畫面的問題，重新進入全螢幕模式或改變一下視窗大小即可。

至此，我們成功的用 120 行 Rust 程式碼建立了一個非常簡單的硬體加速全螢幕影片播放器。

<h2 id="step7">優化使用體驗：讓用戶在播本地影片時可以不用輸入帶 <code>file://</code> 開頭的絕對路徑</h2>

如果你所在的地區或網路環境不適合連線到 YouTube ，GStreamer 其實也支援播放本地影片，由於我們直接從參數中取得影片的路徑，所以理論上也可以直接播放本地影片檔？

執行 `cargo run -- ./test.mp4` 咦，影片怎麼播不出來？

遇到影片播不出來的時候，我們可以透過環境變數，將 GStreamer 的 log 顯示出來：`GST_DEBUG=3`

> 數字越大，log 量越多，個人推薦最多看到 3 即可，如果 3 還是看不出問題，那在改成大於 3 的同時建議將 log pipe 到文件去，會比較好閱讀

```shell
0:00:01.531320000 96705 0x600002b10c30 WARN            urisourcebin gsturisourcebin.c:1749:gen_source_element:<urisourcebin0> error: Invalid URI "./test.mp4".
0:00:01.531353542 96705 0x600002b10c30 ERROR               gst-play gstplay.c:985:on_error:<play0> Error: Failed to play (gst-play-error-quark, 0)
GStreamer error: Failed to play (Some(Structure(error-details)))
```

出現這個問題的原因是，由於 `gstreamer-play` 所調用的 `playbin` 背後其實是利用 `uridecodebin` 進行路徑解析，而其並[不支援相對路徑，且要求在本地文件路徑前加上 `file://` 前綴](https://gstreamer.freedesktop.org/documentation/playback/playbin.html#usage) ，這對於一般用戶而言非常不直覺

如果想要優化這個問題，當用戶直接播放本地檔案時，我們需要先處理過路徑才行：

```rust
fn main() -> glib::ExitCode {
    // 省略......
    app.connect_command_line(move |app, command_line| {
        // 省略......

        let pathbuf = PathBuf::from(&video_path); // 將 `String` 型別的路徑轉換成 `PathBuf` 方便處理

        if pathbuf.is_file() { // 檢查是不是本地文件
            if !pathbuf.is_absolute() { // 如果路徑不是絕對路徑
                std::path::absolute(&video_path) // 嘗試取得絕對路徑
                    .unwrap_or_else(|error| {
                        panic!("Unable to get absolute path of {video_path}: {error:?}") // 無法取得絕對路徑時，顯示錯誤訊息
                    })
                    .to_str() // 將 `PathBuf` 轉換為 `&str`
                    .expect("Non UTF-8 path is not supported.") // 路徑中含有非 UTF-8 字元時，轉換有可能會失敗，顯示錯誤訊息
                    .to_owned() // 將 `&str` 轉換為 `String`
                    .clone_into(&mut video_path); // 將轉換過來的 `String` 塞回 `video_path`
            }

            video_path = format!("file://{video_path}"); // 在路徑前加上 `file://`
        }

        // 省略......
    });

    app.run()
}
```

> 請注意：這邊的程式碼並不是很嚴謹，有很多情況沒有處理到，**可能會有安全問題，請不要直接用在正式用途**

這樣就能直接用相對路徑播放本地影片了。

<h2 id="source">完整範例程式碼</h2>

- `Cargo.toml`
    ```toml
    [package]
    name = "gtk-gstreamer-player"
    version = "0.1.0"
    edition = "2024"
    authors = ["Ming Chang <mail@mingchang.tw>"]
    
    [dependencies]
    gstreamer = "0.23.5"
    gstreamer-play = "0.23.5"
    gtk4 = "0.9.6"
    
    [profile.release]
    strip = true
    lto = "fat"
    opt-level = 3
    codegen-units = 1
    panic = "abort"
    
    [profile.dev]
    strip = false
    panic = "unwind"
    
    [lints.clippy]
    all = { level = "warn", priority = -1 }
    pedantic = { level = "warn", priority = -1 }
    nursery = { level = "warn", priority = -1 }
    ```

- `main.rs`
    ```rust 
    use std::path::PathBuf;
    
    use gstreamer::{ClockTime, ElementFactory, Registry};
    use gstreamer_play::{Play, PlayMessage, PlayVideoOverlayVideoRenderer};
    use gtk4::gdk::Paintable;
    use gtk4::gio::ApplicationFlags;
    use gtk4::{Application, ApplicationWindow, glib};
    use gtk4::{Picture, prelude::*};
    
    fn main() -> glib::ExitCode {
        let app = Application::builder()
            .application_id("tw.mingchang.example-player")
            .flags(ApplicationFlags::HANDLES_COMMAND_LINE)
            .build();
    
        app.connect_command_line(move |app, command_line| {
            let args = command_line.arguments();
    
            let Some(mut video_path) = args
                .get(1)
                .and_then(|os_string_arg| os_string_arg.to_str())
                .map(ToOwned::to_owned)
            else {
                panic!("Video path needed.");
            };
    
            let pathbuf = PathBuf::from(&video_path);
    
            if !pathbuf.is_file() {
                if !pathbuf.is_absolute() {
                    std::path::absolute(&video_path)
                        .unwrap_or_else(|error| {
                            panic!("Unable to get absolute path of {video_path}: {error:?}")
                        })
                        .to_str()
                        .expect("Non UTF-8 path is not supported.")
                        .to_owned()
                        .clone_into(&mut video_path);
                }
    
                video_path = format!("file://{video_path}");
            }
    
            let window = init_window(app);
            init_gstreamer().expect("Unable to init GStreamer.");
    
            let widget = Picture::new();
            window.set_child(Some(&widget));
    
            let paintable = play_video(&video_path)
                .unwrap_or_else(|error| panic!("Unable to play video from {}: {error:?}", &video_path));
            widget.set_paintable(Some(&paintable));
    
            0
        });
    
        app.run()
    }
    
    fn init_window(app: &Application) -> ApplicationWindow {
        let window = ApplicationWindow::builder()
            .application(app)
            .title("Player")
            .fullscreened(true)
            .build();
    
        window.present();
        window
    }
    
    fn init_gstreamer() -> Result<(), Box<dyn std::error::Error>> {
        gstreamer::init()?;
    
        let registry = Registry::get();
    
        for plugin in registry.plugins() {
            if plugin.is_loaded() {
                println!(
                    "Preloaded GStreamer plugin {} detected. {}",
                    plugin.plugin_name(),
                    plugin.description()
                );
            } else if let Err(error) = plugin.load() {
                eprintln!(
                    "Failed to load GStreamer {} plugin: {error:?}.",
                    plugin.plugin_name()
                );
            } else {
                println!(
                    "GStreamer plugin {} loaded. {}",
                    plugin.plugin_name(),
                    plugin.description()
                );
            }
        }
    
        Ok(())
    }
    
    fn play_video(path: &str) -> Result<Paintable, Box<dyn std::error::Error>> {
        let gtk4paintablesink = ElementFactory::make_with_name("gtk4paintablesink", None)?;
        let glsinkbin = ElementFactory::make_with_name("glsinkbin", None)?;
        glsinkbin.set_property("sink", &gtk4paintablesink);
    
        let play = Play::new(Some(PlayVideoOverlayVideoRenderer::with_sink(&glsinkbin)));
    
        let paintable: Paintable = gtk4paintablesink.property("paintable");
    
        play.set_uri(Some(path));
        play.play();
    
        std::thread::spawn(move || {
            for msg in play.message_bus().iter_timed(ClockTime::NONE) {
                match PlayMessage::parse(&msg) {
                    Ok(PlayMessage::EndOfStream) => {
                        println!("Video ended.");
                        play.stop();
                        break;
                    }
                    Ok(PlayMessage::Error { error, details }) => {
                        eprintln!("GStreamer error: {error} ({details:?})");
                        play.stop();
                        break;
                    }
                    Ok(PlayMessage::Warning { error, details }) => {
                        eprintln!("GStreamer warning: {error} ({details:?})");
                    }
                    Ok(_) => (),
                    Err(_) => unreachable!(),
                }
            }
            play.message_bus().set_flushing(true);
        });
    
        Ok(paintable)
    }
    ```
