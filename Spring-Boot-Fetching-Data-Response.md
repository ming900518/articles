# 透過Spring從其他API取得二進位檔案後回傳

## 問題描述
由於客戶資料庫的規格在既有的後端中不相容，我額外用 Rust 寫了 Axum w/ SQLx 框架的後端，以 Sidecar 的方式外掛在單體架構之外。

但這卻衍生了另一個問題：權限管理。

為了避免權限不足的使用者也能夠存取這些 API，勢必不能直接將 Axum API 掛到 NGINX Reverse Proxy 去，既有的權限管理機制也並不適合直接搬到 Axum 來。

我只好在 Spring 後端進行權限管理確認後，再由 Spring 請求 Axum 的 API。

## 系統架構分析
### Axum 端
資料庫為 SQL Server，客戶想要利用姓名或身份證查詢其中的內容，之後製作成 Excel 檔讓他們可以下載查看。

Axum API 會將 SQL Server query 出來的結果製作成 Excel 檔（`.xlsx`，MIME Type：`application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`）之後透過**非同步**的方式回傳檔案（下載）。


### Spring 端
Spring API 端有兩種呼叫 API 的方式：Spring MVC 的 [`RestTemplate`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html) 與 Spring WebFlux 的 [`WebClient`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/reactive/function/client/WebClient.html)。

由於前面有提到，Axum 端的下載是透過**非同步**的方式實現的，而根據 [`RestTemplate`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html) 的註釋說明，這個類是一個**同步**客戶端，而且在下方也有提到：
> NOTE: As of 5.0 this class is in maintenance mode, with only minor requests for changes and bugs to be accepted going forward. Please, consider using the org.springframework.web.reactive.client.WebClient which has a more modern API and supports **sync, async, and streaming** scenarios.

既然都直接說了[`WebClient`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/reactive/function/client/WebClient.html) 才支援非同步，而且已經不更新新功能了，那當然就直接使用 [`WebClient`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/reactive/function/client/WebClient.html) 囉。

## 實作

直接上Code

### Axum（Rust）

```rust
pub async fn xlsx_download(filename: String) -> Result<(HeaderMap, StreamBody<ReaderStream<File>>), (StatusCode, String)> {
    match File::open(&filename).await {
        Ok(file) => {
            let stream = ReaderStream::new(file);
            let body = StreamBody::new(stream);

            let mut headers = HeaderMap::new();
            headers.append(
                header::CONTENT_TYPE,
                "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
                    .parse()
                    .unwrap(),
            );
            headers.append(
                header::CONTENT_DISPOSITION,
                format!("attachment; filename=\"{}\"", filename)
                    .parse()
                    .unwrap(),
            );
            remove_file(filename).unwrap_or(());
            Ok((headers, body))
        }
        Err(err) => Err((StatusCode::NOT_FOUND, format!("File not found: {}", err))),
    }
}
```
這邊可以注意到，程式中有指定 `Content-Type` 與 `Content-Disposition` 兩個 header，這兩個 header 分別指定了檔案的 MIME Type 跟檔名。

兩個 header 都需要在 Spring 端加上，並一起回傳給客戶端，下載的時候檔名才不會跑掉。

### Spring （Java）

```java
public @ResponseBody Mono<ResponseEntity<Resource>> getDataFromApi(@RequestBody OldDataRequest request) {
        return WebClient.builder()
                .codecs(clientCodecConfigurer -> clientCodecConfigurer.defaultCodecs().maxInMemorySize(50 * 1024 * 1024))
                .baseUrl("https://axum-base.url")
                .build()
                .get()
                .uri("/uri")
                .exchangeToMono(response -> response.bodyToMono(Resource.class)
                        .map(file -> ResponseEntity.ok()
                                .contentType(MediaType.valueOf("application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"))
                                .header(HttpHeaders.CONTENT_DISPOSITION, response.headers().asHttpHeaders().getContentDisposition().toString())
                                .header(HttpHeaders.ACCESS_CONTROL_EXPOSE_HEADERS, HttpHeaders.CONTENT_DISPOSITION)
                                .body(file)
                        )
                        .switchIfEmpty(Mono.empty())
                );
    }
```
這邊透過 [`WebClient`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/reactive/function/client/WebClient.html) 從 Axum API 那邊將檔案下載回來後，利用 [`Resource`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/Resource.html) interface 將檔案抽取出來。

由於檔案大小可能超出 [`WebClient`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/reactive/function/client/WebClient.html) 預設的 buffer 大小，所以利用 [`ClientCodecConfigurer`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/codec/ClientCodecConfigurer.html) 設定了較高的 [`maxInMemorySize`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/codec/CodecConfigurer.DefaultCodecConfig.html#maxInMemorySize())。

接著透過 [`ResponseEntity`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/ResponseEntity.html)  製作要回傳給前端的 Response。

> 注意在製作 Response 的時候，我們除了有加上`Content-Type` 與 `Content-Disposition` 兩個 header，另外多加了一個 header `Access-Control-Expose-Headers`，這是為了讓前端可以取得 `Content-Disposition` header 的內容。

## 同場加映 - Angular（TypeScript）

```typescript
downloadData() {
    this.http.post<HttpResponse<any>>(`https://spring-base.url/uri`, param, {
        responseType: 'blob' as 'json',
        observe: 'response'
    }).pipe()
        .subscribe(
            data => {
                const header = data.headers.get('content-disposition')
                saveAs(data.body, header.split('"')[1])
            },
            error => {
                alert('系統錯誤：\n' + error.message)
            }
        )
}
```

這邊用了 [FileSaver.js](https://www.npmjs.com/package/file-saver) 協助處理檔案下載功能。

由於要讀取 `Content-Disposition` header 的內容用於填入 [FileSaver.js](https://www.npmjs.com/package/file-saver) 的檔名參數，所以要將整個 response 保留下來，而非像預設值設定的僅保留 body。
