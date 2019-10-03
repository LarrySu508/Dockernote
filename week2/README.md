# 第二周
## 1.登入Docker
```
#docker login
```
## 2.Docker image上傳並於另一機拿來用
### 1.準備兩台有Docker的虛擬機(可用再製再開另一台)
```
\\虛擬機1
#docker commit 345 test:01   \\容器轉Image，345容器代號，test:1 Image名稱
#docker tag test:01 lanceray86/test:01   \\更改tag名稱
#docker login   \\登入帳號
#docker push lanceray86/test:01   \\上傳到你的帳號(帳號/Image名稱)
```
### 2.執行下方指令前確認再製的虛擬機Docker有開啟
```
\\虛擬機2
#docker pull lanceray86/test:01   \\拉Image
#docker run -itd --name mywebserver -p 8080:80 -v /mydata:/usr/local/apache2/htdocs lanceray86/test    \\建mywebserver容器開port 8080:80
\\開啟另一個terminal
#cd /mydata
#echo hi > hi.htm 
#curl 127.0.0.1:8080/hi.htm
\\這樣就可以看到上傳的Image可以在另一台機子上運行
```
## 3.兩台Docker對外以不同port，但網頁是一樣，因為共享相同資源
延續用上述的虛擬機2。
```
\\terminal1
#docker run -itd --name mywebserver2 -p 8081:80 --volumes-from 458 lanceray86/test   \\再開一個一樣的容器，但這次開的port稍微不同
\\   再開另一個terminal2
#curl 127.0.0.1:8081/hi.htm    \\可以看到和curl 127.0.0.1:8080/hi.htm的輸出一樣，這是因為共用同一個同步資料夾/mydata(--volumes-from就是這個連結的主要指令)
\\terminal1
#docker exec -it mywebserver2 bash  \\進入mywebserver2容器
#cd htdocs/      \\到http server資料夾
#ls          \\查看有哪些網頁，應該只會看到在2-2建的hi.htm
#echo hello > hello.htm   \\再建一個hello.htm，這是在mywebserver2容器建的
\\terminal2
#curl 127.0.0.1:8080/hello.htm      \\在另一個mywebserver容器也看得到mywebserver2容器建的hello.htm 
#curl 127.0.0.1:8081/hello.htm      \\出現一樣的結果
```
## 4.特定網卡IP顯示網頁
```
\\先把剛剛虛擬機2的webserver容器移除掉
#docker run -itd --name mywebserver -p 127.0.0.1:8080:80 -v /mydata:/usr/local/apache2/htdocs lanceray86/test     \\限定只有本地網卡IP127.0.0.1才能開啟網頁，如果用其他網卡IP則無法顯示
\\另外開一個terminal
#curl 127.0.0.1:8080/hi.htm    \\會顯示之前寫入的hi
#curl 127.17.0.1:8080/hi.htm   \\如果你有另一張網卡IP為127.17.0.1，你用這IP連則無法顯示
```
## 5.Docker某些參數使用可以重複數次
像上一節第四節的mywebserver掛載目錄為/mydata(虛擬機本身目錄):/usr/local/apache2/htdocs(容器目錄)，這裡在增加一個掛載目錄/mydata2(虛擬機本身目錄):/data(容器目錄)
```
\\先把上一節的webserver容器移除掉
#docker run -itd --name mywebserver -p 8080:80 -v /mydata:/usr/local/apache2/htdocs -v /mydata2:/data lanceray86/test
#docker exec -it mywebserver bash
\\開另一個terminal
#cd /mydata2     \\虛擬機本身掛載目錄
#touch {a..d}
#ls      \\會列出a b c d這些檔案出來
\\回去開啟mywebserver容器的terminal
#cd /data
#ls      \\會列出a b c d這些檔案出來
```
## 6.傳值到容器系統內，用於MySQL建帳號密碼
首先Linux系統上的值可以這樣設定。
```
#myname=mary      \\注意不要空格
#echo $myname     \\輸出為mary
#set | grep myname  \\輸出為myname=mary
#docker run -itd --name mywebserver -p 8080:80 -v /mydata:/usr/local/apache2/htdocs -e myname=tom -e myage=15 lanceray86/test   \\用於docker把myname=tom,myage=15的值傳入
#docker exec -it mywebserver bash    \\進入mywebserver
#set | grep myname      \\輸出為myname=tom
#set | grep myage      \\輸出為myage=15
#exit     \\先離開等一下要導入MySQL
#docker pull mysql 
#docker run --name mydb -e MYSQL_ROOT_PASSWORD=123456 -d mysql    \\建mysql，並設root密碼為123456
#docker exec -it mydb bash      \\進入mydb
#mysql -uroot -p      \\進入mysql編寫前，會需要輸入密碼，就是剛剛設的123456
```
## 7.Registry做Push和Pull
當網路不穩定且映像檔很大時，或者這資料太重要不想放在網上時，可以用這個方法。
```
\\先建好兩台虛擬機(L1,L2)
\\L1
#docker pull registry
#docker run -d -p 5000:5000 --restart always --name registry registry:2    \\-d背景執行，--restart always虛擬機開機時此Docker也跟著開啟
#docker tag httpd:latest 127.0.0.1:5000/httpd:latest        \\127.0.0.1:5000為本機port 5000取代原本上傳到網上的帳戶名稱
#docker push 127.0.0.1:5000/httpd:latest    \\image push到127.0.0.1:5000，也就是registry那台上
\\L1本地端的pull實作
#docker rmi 127.0.0.1:5000/httpd:latest     \\先把127.0.0.1:5000/httpd:latest image刪除
#docker images      \\確認是否刪除127.0.0.1:5000/httpd:latest
#docker pull 127.0.0.1:5000/httpd:latest      \\再做pull的動作
#docker images      \\確認是否pull到127.0.0.1:5000/httpd:latest
\\先確認L1 IP位址 192.168.42.105(hostonly addr)
\\L2先ping 192.168.42.105看是否連得到
#gedit /etc/docker/daemon.json      \\設定/etc/docker/daemon.json
\\輸入{"insecure-registries":["192.168.42.105:5000"]}，儲存
#systemctl restart docker     \\重啟Docker
#docker pull 192.168.42.105:5000/httpd:latest   \\這樣就可以pull下來了
#docker images      \\確認是否pull到192.168.42.105:5000/httpd:latest
\\也可從L2傳到L1
#docker pull busybox
#docker tag busybox:latest 192.168.42.105:5000/busybox:latest
#docker push 192.168.42.105:5000/busybox:latest
\\L1
#gedit /etc/docker/daemon.json      \\設定/etc/docker/daemon.json
\\輸入{"insecure-registries":["192.168.42.105:5000"]}，儲存
#systemctl restart docker     \\重啟Docker
#docker pull 192.168.42.105:5000/busybox:latest
#docker images      \\確認是否pull到192.168.42.105:5000/busybox:latest
#docker run -it 192.168.42.105:5000/busybox:latest sh   \\確認busybox可否運行
\\busybox檔案雖小但有很完整的網路工具
```
Docker容器指令參考資料：[Docker Container 指令：Docker run & Docker exec](https://www.jinnsblog.com/2018/10/docker-container-command.html)
