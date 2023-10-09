# 第二代部落格：開發體驗、爬坑記錄與反思

## 緣起

今年初，我利用 Yew 框架和我的個人網頁搭建了我的第一代部落格。當時的我只會寫 Angular 跟一點點的 React ，所以寫 Yew （Rust 版 React）時其實有點越級打怪了，從頭 `clone()` 、 `unwrap()` 到尾之外，寫出來的成品（指部落格的部分）連我自己都不太滿意。

後來經由 [ThePrimeagen 的直播](https://www.youtube.com/watch?v=UrMHPrumJEs)而接觸到了 [Leptos](https://leptos.dev) Rust 全端框架。高性能加上接近 JSX 的寫法，讓我對它的開發體驗有些感興趣，於是最終選擇了它搭配 [Tailwind CSS](https://tailwindcss.com) 來更新部落格。

## 開發

### 一、學習 Leptos

在 [Leptos 的 GitHub](https://github.com/leptos-rs/leptos) 中，可以找到[開發者 Greg Johnston 與 Contributor 們共同撰寫的 Book](https://leptos-rs.github.io/leptos/) ，這本書幾乎涵蓋了所有用 Leptos 搭建一個 SSR 全端專案所需的知識，我透過這本書跟改寫我公司專案的 React PoC，了解 Leptos 的 Signal 系統如何運作，以及它和 React Hooks 的差異。

> Leptos 的許多概念都承襲自 Soild.js ，學習時先去 [Solid.js 的互動式教學](https://www.solidjs.com/tutorial/introduction_basics)練習一下，會對理解 Leptos 有很大的幫助。

學習的過程中，最有感的大概就是開發體驗的提升，這邊以部落格的列表顯示為例：

<div class="tg-wrap"><table><thead><tr><th>JSON</th><th>TypeScript with Zod</th><th>Rust</th></tr></thead><tbody><tr><td>```json<br>[<br><br>    {<br>        "name": "文章標題",<br>        "date": "2023-12-31T23:59:59Z",<br>        "url": "filename.md",<br>        "intro": "這篇文章的介紹",<br>        "commit": "fe4636"<br>    }<br>]<br>```</td><td>```typescript<br>const dataSchema = z.object({<br>    name: z.string(),<br>    date: z.string(),<br>    url: z.string(),<br>    intro: z.string(),<br>    commit: z.string()<br>});<br><br>type Data = z.infer&lt;typeof dataSchema&gt;;<br>```</td><td>```rust<br>#[derive(Deserialize)]<br>struct Data {<br>    name: String,<br>    date: String,<br>    url: String,<br>    intro: String,<br>    commit: String<br>}<br>```</td></tr></tbody></table></div>

在 React 中，由於 JavaScript 並沒有原生的 Equal, Partial Equal ，導致如果需要存放資料，我們需要將所有資訊拆開來存放

```javascript

```

在 Yew 中，由於 Rust 支援物件本身的比較，所以可以不需要像 React 一樣，將資訊拆開來存放，但因為 Hook 本身並沒有實作 `Copy` trait ，導致需要在不同地方使用時都需要 `clone()` 或引用

```rust
#[function_component(Test)]
fn test() -> Html {
    let data = use_state(TestData::default());

    // 在 use_effect 中進行資料的撈取，並以 unit 做為 dependency 避免重複撈取
    use_effect_with_deps(
        move |_| {
            wasm_bindgen_futures::spawn_local(async move {
                let fetched_data: TestData = Request::get("API Route")
                    .send()
                    .await
                    .unwrap()
                    .json()
                    .await
                    .unwrap();
                about_us_data.set(data);
            });
        },
        (),
    );

    html!{

    }
}
```

> Yew 的 `UseStateHandle` struct: https://docs.rs/yew/0.21.0/yew/functional/struct.UseStateHandle.html Leptos 的 `RwSignal` struct: https://docs.rs/leptos/0.5.1/leptos/struct.RwSignal.html
