# LINE Notify 取得用戶 Token API 串接

[LINE Notify API Documentation（英文）](https://notify-bot.line.me/doc/en/)

## 設定 LINE Notify Registered Service
串接 LINE Notify, 需要先在 LINE Notify 註冊一個 Service

註冊 Service 時，按需填寫表格中的內容

其中 Service URL 與 Callback URL 需要填寫可以正常使用的網域

> Callback URL 可以填寫多個，以換行作為區隔
> 在這邊設定的值才能作為 LINE Notify 回傳資訊的目標

填寫完畢後，經過 Email 認證即可完成 LINE Notify Registered Service 設定

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
            "https://notify-bot.line.me/oauth/authorize?response_type=code&client_id=tQJrXoXNwVParKfUQ0LZzA&redirect_uri=https://share.eztw.in/api/callback&scope=notify&state=12345"
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

這邊也可以直接利用前端連結進行導向

網址中需要提供 Parameter 如下


2. 待用戶於 LINE 頁面完成操作後，LINE 將會把資訊回傳到 GET API Parameter 中 `redirect_uri` 指定的目標

在上方的 Rust Code 中，我建立了一支 API `/api/callback`，接收回傳資訊，Parameter如下


