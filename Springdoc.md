# Springdoc OpenAPI UI（Swagger）爬坑記錄（持續更新）

如果純粹只是要Swagger界面，那就只需要把Dependency加進去，Run起來連線到:8080/swagger-ui.html就可以了，但是Swagger有很多東西其實可以額外做設定，讓他變得更實用

下面我就從加入Dependency開始，把在Spring Boot專案的設定流程都做個記錄

## 環境設定

### 安裝Swagger
我們只需要在Maven或Gradle上加入這個Dependency就可以了：[Springdoc OpenAPI UI](https://mvnrepository.com/artifact/org.springdoc/springdoc-openapi-ui)

Build完Run起來，在瀏覽器中連線到 localhost:8080/swagger-ui.html

如果一切順利，你就會看到Swagger的界面，跟一堆你已經寫好的API

![Screen Shot 2021-08-31 at 10 57 39 AM (2)](https://user-images.githubusercontent.com/15919723/131434582-a09cfb7d-41c8-47a8-bed6-d44c69186c5c.png)

### 設定檔
如果要進行下面的設定，我們就會需要設定檔來對Swagger進行設定

在Spring Boot的package底下建立一個Java Class，命名為`OpenApi30Config`，然後把底下這些貼進去

```
import org.springframework.context.annotation.Configuration;

@Configuration

public class OpenApi30Config {

}

```

## 爬坑

### 問題一：OpenAPI definition是啥鬼，可以改掉嗎
當然可以
在剛剛建立的設定檔中，加入下面這個Spring Boot Annotation
```
@OpenAPIDefinition(info = @Info(title = "取一個看得懂的名字", version = "v1"))
```
存檔，Run，完工

![Screen Shot 2021-08-31 at 11 08 04 AM](https://user-images.githubusercontent.com/15919723/131435291-5a821b4e-4b75-4a70-8010-bf4d7ee2d104.png)

### 問題二：JWT怎麼加
哈哈，這個坑我搞超久
在剛剛建立的設定檔中，加入下面這個Annotation
```
@SecurityScheme(
        name = "bearerAuth",
        type = SecuritySchemeType.HTTP,
        bearerFormat = "JWT",
        scheme = "bearer"
)
```
然後到Controller那邊，找出需要JWT的API並加入這個Annotation
```
@Operation(security = @SecurityRequirement(name = "bearerAuth"))
```
存檔，Run，會發現Swagger出現了這個按鈕

![Screen Shot 2021-08-31 at 11 13 37 AM](https://user-images.githubusercontent.com/15919723/131435785-ddb7b0c6-7ab3-43a7-aa29-250c2beec3da.png)

點進去會彈出一個視窗，把JWT填進去再按下Authorize，Swagger就會自動在Header那邊加入JWT了

![Screen Shot 2021-08-31 at 11 17 01 AM](https://user-images.githubusercontent.com/15919723/131436082-951ff7cb-8a4e-4fe7-b1da-97977d3a20d8.png)

### 問題三：我腦子不行，記不得API Name，可以加註釋嗎
當然，Swagger的存在就是為了這狀況而生的

在API的Controller那邊，加入這個Annotation
```
@Operation(summary = "我是註釋")
```
存檔，Run，完工

![Screen Shot 2021-08-31 at 11 22 25 AM](https://user-images.githubusercontent.com/15919723/131436536-5c3ac78d-7958-4414-817b-6bcb17e10c30.png)

### 問題四：為啥我的API跟彩虹一樣，明明我只用POST啊
忘記設定method囉，在`@RequestMapping`裡加入`method = RequestMethod.POST`即可
