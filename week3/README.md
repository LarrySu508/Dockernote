# 第三周
## 1.Docker Image做匯出匯入(不經由雲端傳送)
先有兩台虛擬機(L1,L2)，本筆記用的是Linux Centos 7，網路記得設兩張一個是NAT，另一個為HostOnly，兩台都一樣。
### 1.匯出動作
```
\\L1 Root模式
#docker pull busybox     \\如果沒Images可拉這個，檔案小又有網路工具。
#docker tag 192.168.42.56:5000/busybox:latest busybox:latest    \\如果你要用自己的檔案但名字太長，可以像這樣把192.168.42.56:5000/busybox:latest改成busybox:latest。
#docker run -it busybox sh
#mkdir -p /mydata
#cd /mydata
#touch {a..d}
#ls     \\顯示{a..d}
\\ctrl+p+q跳出container
#docker ps
#docker save busybox:latest > busybox_save.tar
#docker export quizzical_almeida > busybox_export.tar      \\這兩行指令為備份用，quizzical_almeida為docker ps中運行busybox的names。
#ls -al *.tar   \\列出剛剛包裝好的tar檔。
```
### 2.匯入動作
```
\\L2 Root模式
#ifconfig       \\紀錄Localhost IP。
#systemctl start sshd
\\L1 Root模式
#ping 192.168.56.106    \\ping L2看可否連線。
#scp *.tar user@192.168.56.106:/home/user
\\L2 Root模式
#cd /home/user
#ls     \\會看到busybox_save.tar,busybox_export.tar。
#docker rmi busybox:latest     \\如果L2上有busybox:latest記得移除，才能匯入。
#cat busybox_export.tar | docker import - busybox1     \\把busybox_export.tar輸出到busybox1。
#docker images      \\確認是否有輸出busybox1。
#docker load < busybox_save.tar
#docker images      \\會輸出busybox:latest。
#docker run -it busybox:latest sh      \\產生的container為原本busybox:latest的Image檔案。
#ls     \\所以沒看到mydata。
\\開另一個terminal進到Root模式。
#docker run -it busybox1:latest sh      \\產生的container為剛剛匯出有建資料夾的Image檔案。
#ls /mydata     \\所以會顯示{a..d}。
```
> ### 如果網卡IP跑掉沒顯示，可以下  dhclient enp0s8 (enp0s8為網卡名稱)。
## 2.兩個Container(C1,C2)互相連線
首先開兩個terminal(t1,t2)。
```
\\t1 Root模式
#docker run -it --name c1 --rm busybox sh
/ #ifconfig         \\查C1的IP 172.17.0.2
\\t2 Root模式
#docker run -it --name c2 --rm busybox sh
/ #ifconfig         \\查C2的IP 172.17.0.3
\\t1 Container模式
/ #ping 172.17.0.3  \\C1可以與C2連線
\\t2 Container模式
/ #ping 172.17.0.2  \\C2可以與C1連線
\\但如果要讓C2直接下ping c1要再加東西
\\t2 Container模式
/ #exit
#docker run -it --name c2 --rm --link c1 busybox sh    \\下這個指令前C1一定要開啟
\\此指令是在Container的/etc/hosts裡加上172.17.0.2    c1  2cba578a90a，記錄C1的IP與代號
/ #ping c1      \\現在測試就可以是成功的
```
## 3.Docker建立Apache php及Mariadb
參考資料：[docker安裝apache、mariadb、php](https://blog.yslifes.com/archives/2523)
```
#docker pull mariadb
#docker run -d --name mariadb -e MYSQL_ROOT_PASSWORD=123456 --restart unless-stopped mariadb
#docker ps      \\檢查Mariadb Container是否有開啟
#docker exec -it mariadb bash
/#mysql -u root -p      \\登入database，輸入剛剛設的密碼123456
>create database test;  \\建資料庫 test 
>show databases;    \\確認是否建成功
\\開另一個terminal
#mkdir -p /opt/www-data
#docker pull php:7.2-apache \\拉Apache php Image
#docker run -d --name apache --restart unless-stopped -p 80:80 -v /opt/www-data:/var/www/html --link mariadb:mariadb php:7.2-apache  \\啟動
#docker exec -it apache sh      \\連接
#docker-php-ext-install mysqli  \\導入套件
#docker-php-ext-enable mysqli
#apachectl restart
\\再開另一個terminal
#cd /opt/www-data
#gedit index.php       \\寫入以下內容，記得'mariadb', 'root', 'youpass', 'test'，分別要寫入'容器名稱','帳戶','密碼','資料庫名稱'
<?php
$db = new mysqli('mariadb', 'root', '123456', 'test');
if (mysqli_connect_errno())
{
echo '<p>' . 'Connect DB error';
exit;
}
else
{
echo "Connection success";
}
?>
\\開瀏覽器輸入虛擬機IP就可以看到預設網頁。
\\再開另一個terminal
#cd /opt/www-data/
#echo "hello world" > 123.htm 
\\開瀏覽器輸入[虛擬機IP/123.htm]就可以看到hello world。
#docker ps      \\查詢容器代號
\\如果要在別台用現在設的狀態，要儲存Apache及database，把Apache及database存成Images。
#docker commit 914 myphp:7.2-apache
#docker commit fa3 mymariadb:latest     \\914,fa3為容器代號
```

