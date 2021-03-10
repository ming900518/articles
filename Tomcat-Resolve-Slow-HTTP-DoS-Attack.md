# Tomcat的Slow HTTP Denial of Service Attack弱點處理方式

先來科普一下，Slow HTTP Denial of Service Attack其實就是DoS攻擊（阻斷服務攻擊）的一種，只是這邊特別強調HTTP的部分，這種攻擊利用各種方式榨乾伺服器的資源，讓伺服器上的服務無法正常運作甚至崩潰。所以不用想都知道資安檢測是絕對不會錯過這個大漏洞的（笑
下面來說說怎麼處理

1. 找到Tomcat的安裝目錄並切換到那個目錄

  > 可以找找/opt，我自己都習慣裝在那裡面

2. 輸入指令開啓Tomcat設定檔（nano可以替換成自己喜歡的編輯器）：`nano conf/server.xml`

3. 找到`Connector`段落，並在`port="80"`（或你自己指定給HTTP用的Port號）那邊替換以下內容：

  * `protocol="HTTP/1.1`換成`protocol="org.apache.coyote.http11.Http11NioProtocol"`
  * `connectionTimeout="20000"`換成`connectionTimeout="8000"`

  > *BEFORE:*  
  > ![1615357109941](https://user-images.githubusercontent.com/15919723/110585400-adc9b400-81ab-11eb-875e-0ce7e8df900e.jpg)

  > *AFTER:*  
  >![1615357145216](https://user-images.githubusercontent.com/15919723/110585479-d05bcd00-81ab-11eb-92a2-7d490d9a54e4.jpg)

4. 存檔並重啓Server
