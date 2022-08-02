# 從PostgreSQL中查詢資料表資訊

```
select c.table_name  as 資料表名稱,
       c.column_name as 欄位名稱,
       c.is_nullable as 是否可為空,
       c.data_type   as 型別（資料庫）,
       c.udt_name    as 型別（C語言）,
       pgd.description  註釋
from pg_catalog.pg_statio_all_tables as st
         inner join pg_catalog.pg_description pgd on pgd.objoid = st.relid
         inner join information_schema.columns c on (
            pgd.objsubid = c.ordinal_position and
            c.table_schema = st.schemaname and
            c.table_name = st.relname
         )
where c.table_schema = '*填入要查詢的Schema名稱*'
  and c.table_name = '*填入要查詢的Table名稱*';
```

執行上面的SQL之後即可匯出資料表的資訊了  
可以搭配IntelliJ IDEA Ultimate/DataGrip的查詢結果匯出功能，匯出成Excel檔
