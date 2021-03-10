# 在Tomcat中關閉不安全的TLS 1.0與1.1
反正開了就是不會過弱點掃描，有用SSL的Tomcat Server最好是都關掉😂

1. 找到Tomcat的安裝目錄並切換到那個目錄

  > 可以找找/opt，我自己都習慣裝在那裡面

2. 輸入指令開啓Tomcat設定檔（nano可以替換成自己喜歡的編輯器）：`nano conf/server.xml`

3. 翻到Connect段落，並在SSL的設定（通常是`port="443"`或`port="8443"`）最後面加上`sslEnabledProtocols="TLSv1.2"`

*BEFORE:*
![1615350426488](https://user-images.githubusercontent.com/15919723/110576673-1c9f1100-819c-11eb-84ef-a92456daaafc.jpg)

*AFTER:*
![1615350447795](https://user-images.githubusercontent.com/15919723/110576705-27f23c80-819c-11eb-8061-d12cfcd79f47.jpg)

4. 存檔並重啓Server應該就搞定了
