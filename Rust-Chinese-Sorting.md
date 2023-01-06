# 在Rust正確的進行中文排序

## TL;DR

用這個我寫的crate進行排序就可以了，方便省事：[https://crates.io/crates/sort_zh](https://crates.io/crates/sort_zh)

## 問題

在幾乎所有的程式語言，對 Array 或 Vector 進行 `sort()` 的做法都是直接拿 Unicode 的 Hex Code 進行排序

乍看之下沒什麼問題，但當你想要對中文進行排序的時候，神奇的事情發生了

```rust
fn main() {
    let mut test = vec!["a", "2", "三", "1", "一", "3", "二", "b"];
    test.sort();
    println!("{:?}", test);
}
```

```
["1", "2", "3", "a", "b", "一", "三", "二"]
```

看出問題點了嗎？為什麼「三」排序在「二」之前呢？

這是因為「三」這個中文字的 Hex Code (`U+4E09`) 比「二」的 Hex Code (`U+4E8C`) 還前面

如果我們直接對中文字一到十進行排序，則會出現：

```
["一", "七", "三", "九", "二", "五", "八", "六", "十", "四"]
```

這顯然不是我們想看到的結果

## 踩在 ICU 的肩膀上

在 Unicode 的世界，其實有一個東西叫 [ICU（International Components for Unicode）](https://en.wikipedia.org/wiki/International_Components_for_Unicode)，為的就是要解決 Unicode 運用在其他語言中所遇到的問題

首先先到 [crates.io](https://crates.io/) 查一下有沒有 ICU 相關的 crate，我最後選擇了[rust_icu_ucol](https://crates.io/crates/rust_icu_ucol/3.0.0)，看起來像是 Google 為了自家產品而撰寫的 crate

接著就開始寫 Code 吧

#### `Cargo.toml` 設定
```toml
[package]
name = "test_zh_sort"
version = "0.1.0"
edition = "2021"

[dependencies]
rust_icu_ucol = "3.0.0"
```
#### 主程式
```rust
use rust_icu_ucol::UCollator;

fn main() {
    // 先取得collator
    // 這邊的範例是zh-TW（台灣用的繁體字），也可以改成zh-HK、zh-CN等等
    let collator = UCollator::try_from("zh-TW").unwrap();
    // 設定用來測試的string slice，放在Vec中
    let mut test_value = vec!["肆", "1", "一", "2", "二", "參", "正", "十二測試", "拾貳測試", "貳拾測試", "拾測試二", "十測試二"];
    // 把測試array，透過較為快速的unstable sort進行排序
    // 由於我們要利用collator進行排序，所以用sort_unstable_by()這個function
    test_value.sort_unstable_by(|a_value, b_value| {
        // 利用rust_icu_ucol提供的function進行比較，成功的話會回傳Ordering
        collator
            .strcoll_utf8(a_value, b_value)
            .expect("Failed to collate with collator.")
    });
    // 排完後把結果print出來
    println!("{:?}", test_value);
}
```
#### 執行結果
```
cargo run --color=always --package test_zh_sort --bin test_zh_sort
    Finished dev [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/test_zh_sort`
["1", "2", "一", "二", "十二測試", "十測試二", "正", "拾測試二", "拾貳測試", "參", "貳拾測試", "肆"]
```

乍看之下，排序似乎是成功了，但仔細一點看的話可以發現數字的排序是錯的

這是因為 ICU 提供的 Collator 預設是利用中文的筆畫數進行排序

但很多時候（如製作檔案管理器）可能也會需要進行中文數字的排序

那該怎麼辦呢？

## 處理中文數字的排序

中文數字比較麻煩，有分大小寫，在進行轉換的時候如果不做分別的話，會有排序混亂的問題

我想到的做法是：
1. 先從輸入的 array for each 出每一個 string slice，並利用 `enumerate()` 查出這些 string slice 在原本的 array 中的 index，將其利用 Tuple 與 string slice 一併儲存
2. 將每一個 string slice 用 `chars()` 轉成 `Chars` （這個 Struct 有 implement Iterator trait，可以直接用 Iterator trait 底下的 function）
3. 將所有 string slice 分成四個 Vec：

    1. ASCII 相容字，如英文、阿拉伯數字。這些字僅需要利用一般的 `sort()` 就能正常排序了。
    2. 中文大寫數字
    3. 中文小寫數字
    4. 中文文字。這些字僅需要利用 `rust_icu_ucol` 提供的 `strcoll_utf8()` 就能正常排序了。

4. 把中文數字的部分轉換成阿拉伯數字，這一步會用到另一個crate：[chinese-number](https://crates.io/crates/chinese-number)

6. 分別對四個 Vec 進行排序

5. 將排序好的四個 Vec，根據需求組合起來

> 如果您有更好的做法，歡迎在 [GitHub Repo](https://github.com/ming900518/sort_zh) 開 Issue 或 Pull Request

那麼就直接上 Code 吧，在 `Cargo.toml` 中加入 `chinese-number = "0.6.5"`
> 如果有更新的版本也可以用，但下面展示的 example code 可能就不適用了

#### 一、中文大小寫數字字典
```rust
static LOWERCASE_NUM: [char; 25] = [
    '零', '一', '二', '三', '四', '五', '六', '七', '八', '九', '十', '百', '千', '萬', '億', '兆',
    '京', '垓', '秭', '穰', '溝', '澗', '正', '載', '極',
];

static UPPERCASE_NUM: [char; 25] = [
    '零', '壹', '貳', '參', '肆', '伍', '陸', '柒', '捌', '玖', '拾', '佰', '仟', '萬', '億', '兆',
    '京', '垓', '秭', '穰', '溝', '澗', '正', '載', '極',
];
```
放在最外層以便存取

#### 二、解析中文數字
```rust
fn parse_zh_number(chars: Chars) -> (bool, Result<i64, ChineseNumberParseError>) {
    // 用於判斷中文數字是否為大寫的變數
    let mut upper_case = false;
    // 用於判斷中文數字字元長度的變數
    let mut zh_number_size = 1_usize;
    // 將每個字元撈出來，拿去與上一步製作的字典做比對
    chars.clone().enumerate().for_each(|(i, char)| {
        // 第一個字元如果是大寫數字，則利用upper_case變數進行記錄
        if i == 0_usize && UPPERCASE_NUM.contains(&char) {
            upper_case = true
        }
        // 當字元不在字典檔中，計算數字長度並記錄到zh_number_size變數中
        if !UPPERCASE_NUM.contains(&char) && !LOWERCASE_NUM.contains(&char) {
            // 由於Rust的index使用usize型別，需要另行轉換才能運用
            zh_number_size = (i as u32 - 1) as usize;
        }
    });
    // 回傳tuple
    (
        upper_case,
        // 利用chinese-number提供的function，將中文數字轉換為阿拉伯數字並回傳
        parse_chinese_number_to_i64(
            ChineseNumberCountMethod::TenThousand,
            String::from_iter(chars.collect::<Vec<char>>()[0..zh_number_size].iter()),
        ),
    )
}
```

#### 三、排序 ASCII 與中文數字轉換過來的阿拉伯數字
```rust
// 這邊我們會拿到Vec<(usize, T)>，其中usize是index，而指定 Ord 特性限制 （trait bound）的泛型部分則是需要排序的元素
// 由於最後進行合併時，會從參數指定的Vec中撈回中文，所以僅需回傳usize的部份即可
fn sort_number<T: Ord>(mut vec: Vec<(usize, T)>) -> Vec<usize> {
    // 採用較為快速的unstable sort，因為我們需要從tuple中存取泛型的部分進行排序，所以一樣要用sort_unstable_by()自訂比較
    vec.sort_unstable_by(|(_, a), (_, b)| a.cmp(b));
    // 將處理好的Vec unzip，將Vec<(usize, T)>分離成(Vec<usize>, Vec<T>)
    // 由於我們只需要usize的部分，所以T可以直接忽略
    let (processed_vec, _): (Vec<usize>, Vec<_>) = vec.into_iter().unzip();
    // 回傳排序後的index
    processed_vec
}
```

#### 四、排序中文文字
```rust
// 需要同時傳入collator，我們會利用它進行排序
// 一樣回傳usize即可
fn sort_zh_word(mut vec: Vec<(usize, &str)>, collator: UCollator) -> Vec<usize> {
    // 一樣使用unstable sort，針對&str的部分排序
    vec.sort_unstable_by(|(_, a_value), (_, b_value)| {
        // 使用傳入的collator，利用rust_icu_ucol提供的function進行比較
        collator
            .strcoll_utf8(a_value, b_value)
            .expect("Failed to collate with collator.")
    });
    let (processed_vec, _): (Vec<usize>, Vec<_>) = vec.into_iter().unzip();
    // 回傳排序後的index
    processed_vec
}
```

#### 五、主程式
```rust
fn main() {
    // 先取得collator
    // 這邊的範例是zh-TW（台灣用的繁體字），也可以改成zh-HK、zh-CN等等
    let collator = UCollator::try_from("zh-TW").unwrap();
    // 設定用來測試的string slice，放在Vec中
    let mut test_value = vec!["肆", "1", "一", "2", "二", "參", "正", "十二測試", "拾貳測試", "貳拾測試", "拾測試二", "十測試二"];

    // 將等等會用到的四個小Vec都先開好
    let mut ascii_word_vec: Vec<(usize, &str)> = Vec::new();
    let mut zh_upper_number_vec: Vec<(usize, i64)> = Vec::new();
    let mut zh_lower_number_vec: Vec<(usize, i64)> = Vec::new();
    let mut zh_word_vec: Vec<(usize, &str)> = Vec::new();

    // 對測試值進行iterate，並將index透過enumerate()一併foreach出來
    test_value.iter().enumerate().for_each(|(i, element)| {
        // 將文字轉換為Chars
        let chars = element.chars();
        // 透過peek取得下一筆資料，並確認是否為ASCII字元
        if chars.clone().peekable().peek().unwrap().is_ascii() {
            // 是則推入ascii_word_vec中
            ascii_word_vec.push((i, element))
        } else {
            // 透過第二步製作的parse_zh_number()解析中文數字
            match parse_zh_number(chars.clone()) {
                // 這邊僅對轉換結果做判斷
                (upper_case, Ok(parsed)) => {
                    if !upper_case {
                        // 如果不是大寫數字，推入zh_lower_number_vec中
                        zh_lower_number_vec.push((i, parsed))
                        
                    } else if upper_case {
                        // 如果是大寫數字，推入zh_upper_number_vec中
                        zh_upper_number_vec.push((i, parsed))
                    } else {
                        // 都不是，視為文字，推入zh_word_vec中
                        zh_word_vec.push((i, element))
                    }
                }
                // 轉換失敗，視為文字，推入zh_word_vec中
                (_, Err(_)) => zh_word_vec.push((i, element)),
            }
        }
    });

    // 將ASCII排序
    let mut final_vec = sort_number(ascii_word_vec);
    // 將中文小寫數字排序
    final_vec.append(&mut sort_number(zh_lower_number_vec));
    // 將中文大寫數字排序
    final_vec.append(&mut sort_number(zh_upper_number_vec));
    // 將中文文字排序
    final_vec.append(&mut sort_zh_word(zh_word_vec, collator));

    // 將組合好的index從test_value取回原值，並組合成新的test_value
    test_value = final_vec
        .into_iter()
        .map(|i| test_value[i])
        .collect::<Vec<&str>>();

    // 組合後把結果print出來
    println!("{:?}", test_value);
}
```
#### 執行結果
```
cargo run --color=always --package test_zh_srt --bin test_zh_srt
    Finished dev [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/test_zh_srt`
["1", "2", "一", "二", "十測試二", "十二測試", "參", "肆", "拾測試二", "拾貳測試", "貳拾測試", "正"]
```

這樣就搞定了！

## 關於 [`sort_zh`](https://crates.io/crates/sort_zh)
有鑑於我個人就碰過多次需要對中文進行排序的狀況，因此我把這些 Code 寫成一個 Library 叫 sort_zh

只需要在 `Cargo.toml` 中將本 crate 加入到依賴中，即可在 `Vec<&str>` 類型的物件上取用 `sort_zh()` function，預設是僅透過繁體中文（台灣）進行 ICU 預設排序，也可以透過 `SortZhOptions` 進行排序的設定。

詳細資訊，可參閱 [Rust Doc](https://mingchang.tw/sort_zh/sort_zh/index.html) 與 [GitHub Repository](https://github.com/ming900518/sort_zh)，也歡迎提供優化與功能建議。

## 後記
在寫這篇文章的時候，我原本以為 PostgreSQL 應該有辦法正常排序，所以就沒有特地為我工作比較常用的  Java 想解決方案

結果......

```sql
postgres=# select * from testdb.test;
 data 
------
 七
 四
 一
 六
 五
 二
 三
(7 rows)

postgres=# select * from testdb.test order by data;
 data 
------
 一
 七
 三
 二
 五
 六
 四
(7 rows)

postgres=# select * from testdb.test order by data collate "zh-Hant-TW-x-icu";
 data 
------
 一
 七
 二
 三
 五
 六
 四
(7 rows)
```
看來下一篇文章的標題很有可能就是「在Java正確的進行中文排序」了（苦笑）
