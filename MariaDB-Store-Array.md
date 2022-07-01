# 在MariaDB中存放&讀取Array

客戶需求是要把原本只能上傳一個檔案的地方，改成可以上傳無限多個

我們系統的存法是把檔案上傳到伺服器後，然後在MariaDB中存網址  
所以當初一個文件就只有開一個`VARCHAR(128)`的格子儲存連結

正常的做法會是開一個新的Table存文件  
但是因為要改的地方太多了，我自己是覺得太麻煩  
而且我最近都改用PostgreSQL了  

想試試看有沒有辦法像PostgreSQL一樣，存Array進去，撈出來的時候也用Array或用List接

## 聲明

1. **這樣做正規化會有問題，我不推薦這樣用，寫這篇只是做個記錄跟分享一下做法**   
~~（正規化狂熱者 e.g.我的資料庫概論教授，麻煩不要追殺我）~~
2. 我看文件上寫MariaDB 10.6以上可以用，但以下操作的環境都是MariaDB 10.8.3
3. MariaDB的JSON存法跟MySQL不一樣，function也都不一樣  
我沒有特別測過MySQL，但我猜應該不能用

## 思路

第一件事當然是拜見Google大神  

在查詢的過程中，有查到有人用JSON Array來存多筆資料 ~~太好了不止我一個偷懶，不做正規化~~  
也查到MariaDB其實內部也是用 `LONGTEXT` 來處理JSON的，本質上就是可變長度的字串

接著跑去MariaDB的Knowledge Base，翻了大概10分鐘找到[JSON的章節](https://mariadb.com/kb/en/json-functions/)

然後我看到了 [`JSON_TABLE()`](https://mariadb.com/kb/en/json_table/)  
這個function可以把JSON做成Table 
這樣就可以直接將Query Result用 `List` 接收了

這樣就能達到我想要實現的功能了

## 測試

開啓IntelliJ IDEA Ultimate，並建立一個測試用的Table  
（個人不用DataGrip，啊反正我會用到的功能IDEA都有，多灌個IDE浪費空間幹嘛）  

```
create table test
(
    content varchar(128) null
);
```

接著塞入JSON資料（由於是要存放多個網址，所以直接擺JSON Array）：

```
insert into schema.test (content) values ('[{"link": "https://mingchang.tw"}, {"link": "https://apple.com"}, {"link": "https://www.google.com"}]')
```

SELECT出來，看起來是這樣的

|   | content                                                                                               |
|---|-------------------------------------------------------------------------------------------------------|
| 1 | [{"link": "https://mingchang.tw"}, {"link": "https://apple.com"}, {"link": "https://www.google.com"}] |

由於我們這邊用 `VARCHAR(128)` 來存資料  
所以後端（我用Spring）那邊直接用Jackson，將Array轉成 `String` 物件的JSON就可以存了

接下來要來想辦法撈出這些資料  
經過大概幾小時的反覆測試，總算寫出下面這串SQL：

```
select temp_table.*
from json_table((select content from test), '$[*]' columns(link varchar(128) path '$.link')) as temp_table;
```

這邊用到上面提到的 `JSON_TABLE()` function，將JSON Array做成像下面這樣的Table：

|   | link                   |
|---|------------------------|
| 1 | https://mingchang.tw   |
| 2 | https://apple.com      |
| 3 | https://www.google.com |

這樣後端只需要用 `List<String>` ，就可以拿到三個網址了  
