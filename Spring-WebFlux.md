# Spring WebFlux 學ㄔˉ習ㄕˇ記錄（持續更新）

## 目錄

[前情提要](#前情提要)

[Day 1：從Spring Initializr開始](#day-1從spring-initializr開始)

## 前情提要

Spring MVC總是有極限的，最近有用到一點WebFlux，但我完全沒有寫非同步的經驗

（我說真的，我看別人寫的code第一次覺得自己好像在看天書）

所以就打算跟MVC一樣從頭吃一次屎，從Rx到R2DBC應該都會碰

由於是學習記錄，並不保證這邊寫的所有範例都能在你的環境中正常使用

如果有前輩路過發現有錯，歡迎pull request

## Day 1：從Spring Initializr開始

設定長下面這樣：

<img width="1291" alt="Screen Shot 2021-12-25 at 11 40 27 AM" src="https://user-images.githubusercontent.com/15919723/147376858-d1292e6a-e02a-49ed-adb1-702533cd1841.png">

這裡有連結：[點我](https://start.spring.io/#!type=gradle-project&language=java&platformVersion=2.6.2&packaging=jar&jvmVersion=17&groupId=su.mingchang&artifactId=webflux-test&name=webflux-test&description=Test%20project%20for%20Spring%20WebFlux&packageName=su.mingchang.webflux-test&dependencies=web,webflux,devtools,mariadb,data-jdbc,data-r2dbc,lombok)

- 使用Gradle，因為我比較愛用Gradle
- Spring MVC跟WebFlux都有，因為我想做直接比較
- MariaDB FTW

按下Generate，把專案下載下來之後匯入IDE中（我個人是用IntelliJ IDEA，Eclipse我覺得很難用，不要噴我）
