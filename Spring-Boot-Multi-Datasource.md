# 在Spring Boot中連接多資料庫

搞了半天，試了一整個上午才搞定（笑），這是我找到最簡單的方式，適用於Spring Boot + MyBatis的環境

### 環境配置
先在Maven/Gradle裡加入下面這個套件：

- [Dynamic Datasource Spring Boot Starter](https://mvnrepository.com/artifact/com.baomidou/dynamic-datasource-spring-boot-starter)

然後如果是要連不同種DB，要記得把另一種DB的JDBC Driver也加進去，下面列幾個常用的：

- [MariaDB Java Client](https://mvnrepository.com/artifact/org.mariadb.jdbc/mariadb-java-client)
- [MySQL Connector/J](https://mvnrepository.com/artifact/mysql/mysql-connector-java)（拜託拜託，能改用MariaDB就改用MariaDB）
- [Microsoft JDBC Driver For SQL Server](https://mvnrepository.com/artifact/com.microsoft.sqlserver/mssql-jdbc)（記得選擇符合自己JDK版本的那版，巨硬很愛Java 16不知道為啥）
- [IBM Data Server Driver For JDBC and SQLJ](https://mvnrepository.com/artifact/com.ibm.db2/jcc)

### application.properties設定（也可以用yaml方式寫）
先把舊的DB設定刪掉，把下面這些貼進去
```
spring.datasource.dynamic.primary=（主DB）
spring.datasource.dynamic.strict=true
spring.datasource.dynamic.datasource.（主DB）.url=（JDBC URL）
spring.datasource.dynamic.datasource.（主DB）.username=（DB使用者名稱）
spring.datasource.dynamic.datasource.（主DB）.password=（DB密碼）
spring.datasource.dynamic.datasource.（主DB）.driver-class-name=（JDBC Driver）

spring.datasource.dynamic.datasource.（副DB）.url=（JDBC URL）
spring.datasource.dynamic.datasource.（副DB）.username=（DB使用者名稱）
spring.datasource.dynamic.datasource.（副DB）.password=（DB密碼）
spring.datasource.dynamic.datasource.（副DB）.driver-class-name=（JDBC Driver）
```
把括號的地方全部替換成自己想要連接的DB資訊。如果需要連接多於2台DB，把副DB的部分複製並改成其他DB資訊

下面這邊是MariaDB + SQLServer的範例
```
spring.datasource.dynamic.primary=mariadb
spring.datasource.dynamic.strict=true
spring.datasource.dynamic.datasource.mariadb.url=jdbc:mysql://localhost:3306/db
spring.datasource.dynamic.datasource.mariadb.username=root
spring.datasource.dynamic.datasource.mariadb.password=root
spring.datasource.dynamic.datasource.mariadb.driver-class-name=org.mariadb.jdbc.Driver

spring.datasource.dynamic.datasource.sqlserver.url=jdbc:sqlserver://localhost:1433;database=db
spring.datasource.dynamic.datasource.sqlserver.username=sa
spring.datasource.dynamic.datasource.sqlserver.password=sa
spring.datasource.dynamic.datasource.sqlserver.driver-class-name=com.microsoft.sqlserver.jdbc.SQLServerDriver
```
改好Run一遍看看有沒有噴錯，沒噴錯就設定完成囉

### Service設定
由於預設情況下都會採用上一步設定的主要DB連線，我們只需要指定哪些Service要用副DB連線就可以了

到要連到其他DB的Service實作（如果你的Service沒有分開寫的話，我強烈建議改掉這個壞習慣）那邊，加入Spring Boot Annotation：`@DS("")`

比如以我上面那個例子，Service實作就要加入`@DS("sqlserver")`才能連到SQL Server

這樣就成功連線囉！
