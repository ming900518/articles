# 安全的SSH連線
其實我認為有設IP白名單就差不多了啦，但是我們親愛的臺南市政府智慧發展中心似乎非常重視這塊，那就順便記錄下修改SSH設定檔的方法囉

先用自己習慣的文字編輯器打開SSH Daemon的設定檔：`sudo nano /etc/ssh/sshd_config`

### KexAlgorithms
加密演算法diffie-hellman-group1-sha1由於長度只有1024位元，被認為是弱加密，所以我們必須把這個演算法禁用掉

好笑的是，我第一次在看SSH的設定檔時找都找不到設定的地方，後來才發現sshd的設定檔如果沒有設定預設是全開放的

設定下面這行即可
```
kexalgorithms curve25519-sha256,curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group-exchange-sha256,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512,diffie-hellman-group14-sha256,diffie-hellman-group14-sha1
```

### Cipher
傳統的分區加密法（Legacy block ciphers）會有被生日攻擊（Birthday Attack）的可能

  > 不要問我生日攻擊是啥，反正就是某種運用數學的生日問題去破解密碼的方式

一樣，沒有設定在設定檔中預設是全開的，我的天（（  

設定下面這行即可
```
ciphers aes128-ctr,aes192-ctr,aes256-ctr,aes128-cbc
```

### HMAC algorithms
雜湊訊息鑑別碼（Hash-based message authentication code）的演算法如果是弱加密也是會被掃出來的，所以也得禁用掉才行  

設定下面這行即可
```
macs umac-64@openssh.com,hmac-sha2-256,hmac-sha2-512
```

上面那些設定完後，存檔離開

先用`sshd -T`確認，再用`systemctl restart sshd`重新啓動SSH Daemon就可以了。
