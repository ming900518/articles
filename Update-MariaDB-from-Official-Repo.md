# 把舊版Ubuntu的MariaDB更新到最新版

[如果只想要看TL;DR，請點我直接跳到安裝流程](#從mariadb-package-repository安裝最新的mariadb-server)

## 前情提要

我的MariaDB版本是10.6.5，官方Docker版

而公司專案測試機上面裝的是10.3.32，Ubuntu 20.04提供的版本

然後我寫的程式中有一支SQL用到了intersect（交集）

在我本機完全沒問題，但一搬到測試機或正式機上面時就無法正常使用了

![messageImage_1640246833935](https://user-images.githubusercontent.com/15919723/147212658-7198ffc1-2dca-426e-b9c4-0234585c93b9.jpeg)

我原本以為是值沒帶進去，但Debug的時候log是有值的

接著我仔細的看了一下錯誤訊息，發現問題是出在intersect

看了一下[MariaDB的官方文件](https://mariadb.com/kb/en/intersect/)，上面寫的清清楚楚

<img width="777" alt="Screen Shot 2021-12-23 at 4 42 01 PM" src="https://user-images.githubusercontent.com/15919723/147213111-803a4008-e784-4dbb-ad96-d44f18894000.png">

當我正在思考10.3.32為什麼會小於10.3的時候

往下一翻才發現

<img width="781" alt="Screen Shot 2021-12-23 at 4 45 12 PM" src="https://user-images.githubusercontent.com/15919723/147213628-8d96dfb7-d0cb-4e04-b404-3e399db685b0.png">

哪有人把版本需求分開寫的啦😂

好這下找到問題了

然而正當我開心的要更新MariaDB的時候...

```
$ sudo apt upgrade mariadb-server

Reading package lists... Done
Building dependency tree       
Reading state information... Done
mariadb-server is already the newest version (1:10.3.32-0ubuntu0.20.04.1).
```

去年的LTS版居然沒有提供最新的MariaDB！！！！

於是，又到了爬坑的日常

## 從MariaDB Package Repository安裝最新的MariaDB Server

1. 首先，開啓終端機（或者用SSH連進去）
2. 輸入以下指令把MariaDB Package Repository加入apt sources list中 

```
curl -LsS https://r.mariadb.com/downloads/mariadb_repo_setup | sudo bash
```

結果應該會長這樣

```
$ curl -LsS https://r.mariadb.com/downloads/mariadb_repo_setup | sudo bash
[info] Checking for script prerequisites.
[info] Repository file successfully written to /etc/apt/sources.list.d/mariadb.list
[info] Adding trusted package signing keys...
[info] Running apt-get update...
[info] Done adding trusted package signing keys
```

3. 輸入以下指令安裝新版的MariaDB（這邊注意，是Install不是Update，包名不一樣喔，新的MariaDB會自動把舊的移除掉）

```
sudo apt install mariadb-server -y
```

4. 輸入以下指令更新資料庫

```
sudo mysql_upgrade -u [資料庫帳號] -p[資料庫密碼，注意參數跟密碼中間不能輸入空白]
```

結果應該會長這樣

```
$ sudo mysql_upgrade -u root -proot

Phase 1/7: Checking and upgrading mysql database
Processing databases
(略)
Phase 2/7: Installing used storage engines... Skipped
Phase 3/7: Fixing views
(略)
Phase 4/7: Running 'mysql_fix_privilege_tables'
Phase 5/7: Fixing table and database names
Phase 6/7: Checking and upgrading tables
Processing databases
(略)
Phase 7/7: Running 'FLUSH PRIVILEGES'
OK
```

5. 輸入以下指令重新啓用並開啓系統服務

```
sudo systemctl enable mariadb --now
```

操作完，MariaDB就更新完成了

重新執行原本有錯的SQL，也能正常執行了

![Screen Shot 2021-12-23 at 5 07 33 PM](https://user-images.githubusercontent.com/15919723/147216562-da82e34e-f40b-4d59-9036-d30bec740417.png)

## 如果不能連到外網去......

有很多地方的Server不讓連外網，這樣就有點麻煩了

1. 先在一台有網路、系統版本相同的電腦/虛擬機裡開個資料夾

2. 輸入以下指令（範例是MariaDB 10.6.5的，Dependency應該是不會變但DB版本可能要更新一下）

```
sudo apt-get download mariadb-server mariadb-server-10.6 debconf perl-base dpkg tar libacl1 libc6 libcrypt1 libgcc-s1 gcc-10-base libselinux1 libpcre2-8-0 libbz2-1.0 liblzma5 libzstd1 zlib1g galera-4 libssl1.1 libstdc++6 gawk libgmp10 libmpfr6 libreadline8 libtinfo6 readline-common install-info libsigsegv2 iproute2 libbsd0 libcap2 libcap2-bin libdb5.3 libelf1 libmnl0 libxtables12 libdbi-perl perl libperl5.30 libgdbm-compat4 libgdbm6 perl-modules-5.30 perl-base libpam0g libaudit1 libaudit-common libcap-ng0 lsb-base lsof mariadb-client-10.6 debianutils mariadb-client-core-10.6 libmariadb3 mariadb-common mysql-common libncurses6 libreadline5 perl:any mariadb-server-core-10.6 libaio1 liblz4-1 libpmem1 libsystemd0 libgcrypt20 libgpg-error0 passwd libpam-modules libpam-modules-bin libsemanage1 libsemanage-common libsepol1 procps init-system-helpers libncursesw6 libprocps8 psmisc rsync libpopt0 socat libwrap0 adduser
```

3. 把下載好的deb包想辦法丟到要安裝DB的伺服器，並切換到deb包的目錄

4. 在要安裝DB的伺服器上輸入以下指令進行安裝

```
sudo apt remove mariadb-server && sudo dpkg -i *.deb
```

5. 沒有出錯的話，就可以繼續根據上一段的第四步繼續操作了

如果有錯，輸入以下指令並按照畫面中出現的錯誤提示解決衝突

```
sudo apt --fix-broken install
```
