# LINE Notify 取得用戶 Token API 串接

[LINE Notify API Documentation（英文）](https://notify-bot.line.me/doc/en/)

## 設定 LINE Notify Registered Service
串接 LINE Notify, 需要先在 LINE Notify 註冊一個 Service
![swappy-20230117_090618](https://user-images.githubusercontent.com/15919723/212791581-7d7622b8-e3e2-44d6-8325-f768d3bdd5e4.png)

註冊 Service 時，按需填寫表格中的內容
![swappy-20230117_090856](https://user-images.githubusercontent.com/15919723/212791616-afcbe7ef-1352-4eea-9112-4a995a509c59.png)

其中 Service URL 與 Callback URL 需要填寫可以正常使用的網域

> Callback URL 可以填寫多個，以換行作為區隔  
> 在這邊設定的值才能作為 LINE Notify 回傳資訊的目標

填寫完畢後，經過 Email 認證即可完成 LINE Notify Registered Service 設定
![swappy-20230117_091148](https://user-images.githubusercontent.com/15919723/212791662-17fa79ea-6eab-40fa-ac9a-6e91db570242.png)


## 建立API

用戶需要透過 LINE Notify 的 API 建立 Token，我們才能進行通知的推播

這邊提供利用 Rust Axum Framework 建立的 API Service 範例以供參考

```rust
#![forbid(unsafe_code)]

use axum::Router;
use reqwest::StatusCode;
use std::net::SocketAddr;
use axum::routing::get;
use axum::response::IntoResponse;
use axum::response::Html;
use axum::response::Redirect;
use serde::Deserialize;
use axum::extract::Query;

#[derive(Deserialize)]
struct CallbackValue {
    code: Option<String>,
    state: String,
    error: Option<String>,
    error_description: Option<String>
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(|| async { Redirect::permanent(
            "https://notify-bot.line.me/oauth/authorize?response_type=code&client_id=tQJrXoXNwVParKfUQ0LZzA&redirect_uri=https://api.url/api/callback&scope=notify&state=12345"
        ) }))
        .route("/api/callback", get(callback));

    let addr = SocketAddr::from(([0, 0, 0, 0], 3000));

    println!("Listening on {}", addr);

    axum::Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .expect("Server startup failed.");
}

async fn callback(param: Query<CallbackValue>) -> impl IntoResponse {
    match &param.0.code {
        Some(code) => (StatusCode::OK, Html::from(format!("State: {}. Token: {}",param.0.state, code))),
        _ => (StatusCode::INTERNAL_SERVER_ERROR, Html::from(format!("Error occured: {}, description from LINE: {}", param.0.error.unwrap_or("No error code".to_string()), param.0.error_description.unwrap_or("No error description.".to_string())))),
    }
}

```

1. 將用戶導向 LINE Notify 提供的 GET API，供用戶進行驗證與確定推播群組  
    上方的 Rust Code 透過 Router 產生 HTTP Status 302，將所有連線到 Root 的用戶導向  
    網址中需要提供 Parameter 如下

    | Parameter       | Description                                                                                                                    |
    |-----------------|--------------------------------------------------------------------------------------------------------------------------------|
    | `response_type` | 固定為`code`                                                                                                                   |
    | `client_id`     | 填入設定 Registered Service 時，LINE 建立的 Client ID                                                                          |
    | `redirect_uri`  | 填入用戶完成操作後，LINE 需要導向的頁面 僅能填入設定 Registered Service 時，於 Callback URL 欄位指定的 URL                     |
    | `scope`         | 固定提供`notify`                                                                                                               |
    | `state`         | 指定需要一併回傳回 Callback URL 的值，作為避免 CSRF 攻擊的手段                                                                 |
    | `response_mode` | 非必填，這邊可以設定是否採用POST導向 如需設定可參考 [Spec](https://openid.net/specs/oauth-v2-form-post-response-mode-1_0.html) |

    > 這邊也可以直接利用前端進行導向，不一定需要透過後端處理

2. 待用戶於 LINE 頁面完成操作後，LINE 將會把資訊回傳到 GET API Parameter 中 `redirect_uri` 指定的目標  
    在上方的 Rust Code 中，我建立了一支 API `/api/callback`，接收回傳資訊
    成功時的Parameter如下

    | Parameter | Description              |
    |-----------|--------------------------|
    | `code`    | 用戶產生的Token          |
    | `state`   | 回傳回 Callback URL 的值 |

    失敗時的Parameter如下
    
    | Parameter           | Description              |
    |---------------------|--------------------------|
    | `error`             | OAuth2 錯誤碼            |
    | `error_description` | 錯誤註釋                 |
    | `state`             | 回傳回 Callback URL 的值 |
