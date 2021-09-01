# Spring Boot Admin & Spring Security爬坑記錄

之前當程式出Bug的時候，常常都會需要用SSH連上Server，再自己翻log，這段操作對懶人（我）而言極度不友善

這次從單純的Spring MVC整合到Spring Boot的時候發現了不少套件，其中Spring Boot Admin剛好可以解決這個問題

所以爬坑的時候又到了......

## 環境建構

要先裝以下幾個套件

- [Spring Boot Admin Server Starter](https://mvnrepository.com/artifact/de.codecentric/spring-boot-admin-starter-server)
- [Spring Boot Admin Client Starter](https://mvnrepository.com/artifact/de.codecentric/spring-boot-admin-starter-client)
- [Spring Boot Admin Server UI](https://mvnrepository.com/artifact/de.codecentric/spring-boot-admin-server-ui)
- [Spring Boot Starter Actuator](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-actuator)

當然，裸奔是不好的，所以要一起整合Spring Security才行

- [Spring Boot Starter Security](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-security)

不要興奮的按下Run，因為這時還沒對Spring Security做設定，所有的API在默認狀態下都是需要驗證的，接著看下一步吧

## Spring Security get out of my way!

其實這麼說有點怪怪的，整合這套件的目的其實是要提供Spring Boot Admin一個身份驗證的功能😂

在`application.yaml`（對，我現在改用yaml了，彈性比較大）中填入以下資訊

```
spring:
  security:
    user:
      name: "root"
      password: "root"
```
這邊設定的`name`跟`password`就是Spring Security的帳號密碼，可以自己改成自己想要的帳密

聽說有辦法可以套資料庫，但我好懶，改天再補

然後開一個Java Class叫`SecurityConfig`，把下面的這串複製進去

```
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // Spring Boot Admin（需要登入）
        http
                .authorizeRequests()
                .antMatchers(HttpMethod.GET,"/").authenticated()
                .antMatchers(HttpMethod.GET,"/applications/**").authenticated()
                .antMatchers(HttpMethod.GET,"/wallboard/**").authenticated()
                .antMatchers(HttpMethod.GET,"/journal/**").authenticated()
                .antMatchers(HttpMethod.GET,"/about/**").authenticated()
                .antMatchers(HttpMethod.GET,"/instances/**").authenticated()
                .and()
                .csrf().disable()
                .formLogin().loginPage("/login");
        // Spring Boot Actuator（不允許非本機取用）
        http
                .authorizeRequests()
                .antMatchers("/actuator/**").hasIpAddress("127.0.0.1");
    }
}
```

Spring Boot Admin的網頁都是用GET請求的，所以只需要把所有相關頁面都設為需要驗證即可

Spring Boot Actuator是Spring Boot的監控工具，用來提供Spring Boot Admin資料，把他設定為只有本機可以存取會比較安全

這邊的設定應該還有優化的空間，如果有更好的寫法我再補充上來

接下來就可以開始設定Spring Boot Admin了

## Spring Boot Admin 跟 Spring Boot Actuator設定

在`application.yaml`中填入下面這些設定

```
spring:
  boot:
    admin:
      server:
        enabled: true
      ui:
        title: "Service"
        brand: "Service"
      client:
        instance:
          management-url: "http://localhost:8080/actuator"
          health-url: "http://localhost:8080/actuator/health"
          service-url: "http://localhost:8080/"
        url: "http://localhost:8080"
```

`ui`裡面的`title`跟`brand`就是Spring Boot Admin網頁的Title跟裡面所有提到Spring Boot Admin的地方都會被替換成你設定的值，其他設定應該都不需要變動

接著填入下面這串

```
management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    health:
      show-details: always
    logfile:
      enabled: true
      external-file: "~/logs/console.log"
```

`external-file`後面指定的路徑是Spring Boot Console Log存放的位置，我這邊設定的地方是macOS或Linux的使用者目錄底下的log資料夾，這個位置要記起來，等等還會用到

最後，如果你沒有設定application name，把下面這串的`name`換掉貼上去就可以了

```
spring:
  application:
    name: "Service"
```

## Logback設定

在`resources`裡面建立一個叫做`logback-spring.xml`的檔案，然後把下面這串貼上去

```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <property name="APP_Name" value="logback" />
    　　　<contextName>${APP_Name}</contextName>
    <property name="LOG_HOME" value="~/logs/" />

    <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter" />
    <conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter" />
    <conversionRule conversionWord="wEx" converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter" />
    <property name="CONSOLE_LOG_PATTERN"
              value="${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}" />

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset>utf8</charset>
        </encoder>
    </appender>

    <appender name="FILE"  class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_HOME}/console.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${LOG_HOME}/%d{yyyy-MM-dd}.log</FileNamePattern>
            <MaxHistory>10</MaxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50}:%L - %msg%n</pattern>
        </encoder>
    </appender>

    <logger name="org.springframework.boot.actuate.endpoint.web.servlet" level="INFO"/>

    <root level="INFO">
        <appender-ref ref="STDOUT" />
        <appender-ref ref="FILE" />
    </root>
</configuration>
```

如果你在上一步有改`external-file`的路徑，要把這邊的`LOG_HOME`換成你自己設定的路徑

## 大功告成

改好之後，就存檔Run吧

沒問題的話，連線 http://localhost:8080/ 就會看到Spring Boot Admin的登入畫面了，把帳密填入就能登入進去了

![Screen Shot 2021-09-01 at 10 54 44 AM](https://user-images.githubusercontent.com/15919723/131604215-48b6c123-47df-4ac2-b018-09314ee1b339.png)

## 加分題：整合Swagger

要懶，當然就要一次懶到底啦

Swagger的爬坑記錄在[這裡](Springdoc.md)，如果有需要可以看看

在`application.yaml`中的`ui`裡面加入下面這串

```
          external-views:
            - label: "Swagger UI"
              url: "/swagger-ui.html"
              order: "2000"
```

然後在`SecurityConfig`裡加入下面這串

```
        // Swagger（需要登入）
        http
                .authorizeRequests()
                .antMatchers(HttpMethod.GET, "/swagger-ui.html").authenticated()
                .antMatchers(HttpMethod.GET,"/swagger-ui/**").authenticated()
                .and()
                .csrf().disable()
                .formLogin();
```

Run起來，登入Spring Boot Admin就可以在頂部選單看到Swagger UI的連結了
