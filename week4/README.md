# 第四周
## Docker network(參考:[[Docker] Bridge Network 簡介](https://godleon.github.io/blog/Docker/docker-network-bridge/))
上課內容
![image](https://github.com/LarrySu508/Dockernote/blob/master/week4/IMG_20191008_191706.jpg)
### 1.Docker支援的網路
Docker支援bridge,host,null，三種網路形式。
```
#docker network ls
//列出Docker支援的網路
```
### 2.Docker,Container與虛擬機之間的網路連接概念圖(預設狀態)
![image](https://github.com/LarrySu508/Dockernote/blob/master/week4/a.png)
每個Container的eth0都有對應veth(virtual ethernet)，而每個Container網卡都可以是eth0，因為運用了Docker隔離的技術。
```
#ifconfig
//Docker安裝並執行後下，可以看到docker0的網卡，與圖對應位置為docker0(bridge)，個人docker0 ip為172.17.0.1。
#docker run -it busybox sh
/#ifconfig
//此時查看的是Container的eth0，個人eth0 ip為172.17.0.4。
```
### 3.Docker bridge設定
先刪空Container再開兩個Container，並做設定。
```
#docker rm -f $(docker ps -a -q) 
#docker run -itd --name c1 busybox sh
#docker run -itd --name c2 busybox sh
#brctl show
/*
brctl show意思為bridge control show，可看到interface開了兩個，如果你的Container沒這個指令，下yum install bridge-utils -y安裝套件。
*/
#brctl -h       //-h為help的意思，可以看到brctl後可以接哪些指令。
#docker network inspect bridge      //可看更完整的訊息，可看到bridge的subnet172.17.0.0/16,gateway172.17.0.1，c1,c2兩個Container的macaddress,ipv4address。
```
此為概念圖。    
![image](https://github.com/LarrySu508/Dockernote/blob/master/week4/b.png)      
查看c1,c2的ip address，和ping看看連線狀態。     
```
#docker exec -it c1 ip addr show    //c1 ip 172.17.0.2
#docker exec -it c2 ip addr show    //c2 ip 172.17.0.3
#docker exec -it c1 ping -c 3 8.8.8.8   //成功，因為可以透過veth連到docker bridge，再從bridge到虛擬機的interface連到網際網路。
#docker exec -it c1 ping -c 3 172.17.0.3     //成功ping到c2。
#docker exec -it c1 ping -c 3 c2        //失敗，因為沒做網路設定，所以Container之間只能用IP傳訊。
```
### 4.Docker創建自製網路
```
#docker network create --driver bridge mynet
#docker network ls      //多了一個名字mynet,DRIVER bridge的網路。
#docker network inspect mynet       //可看到新的mynet網路，subnet為172.18.0.0/16,gateway為172.18.0.1
#docker run -itd --name c3 --network mynet busybox sh
#docker run -itd --name c4 --network mynet busybox sh
#docker exec -it c3 ip addr show    //c3 ip 172.18.0.2
#docker exec -it c4 ip addr show    //c4 ip 172.18.0.3
#docker exec -it c3 ping -c 3 172.18.0.3    //成功
#docker exec -it c3 ping -c 3 c4        //跟預設bridge網路不同，結果是成功的。
```
### 5.建下方的拓撲圖網路
![image](https://github.com/LarrySu508/Dockernote/blob/master/week4/c.png)
```
//先把Container刪空
#docker rm -f $(docker ps -a -q) 
#docker run -itd --name c1 --network mynet busybox sh   //c1用自設mynet網路
#docker run -itd --name c2 --network mynet busybox sh   //c2用自設mynet網路
#docker run -itd --name c3 busybox sh       //c3用預設bridge網路
#docker run -itd --name c4 --network mynet busybox sh   //c4用自設mynet網路
#docker network connect bridge c4   //c4連接預設bridge網路
#brctl show   //會看到兩個網路，一個是預設bridge docker0，另一個是自己設的mynet，docker0開兩個veth去對應c3,c4，mynet開三個veth去對應c1,c2,c4。
#docker exec -it c1 ip addr show    //c1 ip 172.18.0.2
#docker exec -it c2 ip addr show    //c2 ip 172.18.0.3
#docker exec -it c3 ip addr show    //c3 ip 172.17.0.2
#docker exec -it c4 ip addr show    //c4 ip兩個 172.18.0.4 , 172.17.0.3
```
測試Container之間連線。
```
#docker exec -it c1 ping -c 3 c2    //成功
#docker exec -it c1 ping -c 3 c4    //成功
#docker exec -it c1 ping -c 3 c3    //失敗
#docker exec -it c1 ping -c 3 172.17.0.2    //失敗，封包loss:100%
#docker exec -it c1 ping -c 3 8.8.8.8    //成功
#docker exec -it c1 ping -c 3 www.google.com    //成功
```
### Docker network其他指令
```
//先把Container刪空
#docker rm -f $(docker ps -a -q) 
#docker network ?   //可查詢問號處可接的指令
#docker network ls   //看到有哪些網路
#docker network rm mynet    //刪除網路mynet
#docker network connect ?  //可以查到問號要接[OPTIONS] 網路名稱 容器名稱
#docker run -itd --name c5 --network host busybox sh    //設定host網路的Container
#ifconfig | more   //確認自己虛擬機的Interface設定
#docker exec -it c5 ip addr show   //確認Container的Interface設定
//會發現Container的Interface設定和虛擬機的Interface設定是一樣的。
#docker run -itd --name c6 --network none busybox sh    //設定none網路的Container
#docker exec -it c6 ip addr show    //會發現Container的Interface幾乎沒有
```
## Docker image產生
### 1.開啟的容器做匯出image。
```
#docker commit 692 test:01      //692為Container代號，test:01為repository:tag
```
### 2.Dockerfile(參考:[Docker – Dockerfile 指令教學，含範例解說](https://www.jinnsblog.com/2018/12/docker-dockerfile-guide.html))
#### 1.指令介紹
1."#"井字符號代表註解。       
2.最一開頭，基底映像檔一定要有，指定映像檔要以哪一個Image為基底建置，格式為"FROM <image>"或"FROM <image>:<tag>"。     
3.接著寫映像檔維護者，意思就是作者可寫可不寫，格式為"MAINTAINER <name>"。
4.LABEL為設定映像檔的Metadata資訊，作者、EMail、映像檔的說明等，格式為：LABEL <key>=<value> <key>=<value> <key>=<value> …，可不寫。     
5.RUN為執行指定的指令，格式為：     
1."RUN <command>"    
2."RUN ["executable", "param1", "param2"]"     
6.ENV為設定環境變數。    
7.WORKDIR為切換工作目錄。    
8.COPY為複製本地端的檔案/目錄到映像檔的指定位置中。     
9.EXPOSE為宣告在映像檔中預設要使用(對外)的連接埠。      
10.CMD為設定映像檔啟動為Container時預設要執行的指令。     
#### 2.編寫Dockerfile(docker hub上查看image，會發現有Dockerfile格式)
##### 1.簡易練習    
```
#gedit Dockerfile &     //檔名一定要一樣
//進入編輯文字檔
FROM ubuntu:latest
MAINTAINER larry larry4623@gmail.com
RUN mkdir -p /home/demo/docker
RUN apt-get update && apt-get install -y apache2
//儲存
#docker build -t myimage:v0.1 .     //執行建立Docker image
#docker images      //查看是否有自己建的image
#docker run -it --name c1 --rm myimage:v0.1 bash     //可以執行看看
```
##### 2.httpd練習
```
#gedit Dockerfile &
//進入編輯文字檔
FROM centos
MAINTAINER larry larry4623@gmail.com
RUN yum update -y
RUN yum install -y httpd net-tools
EXPOSE 80
//儲存
#docker build -t myimage:v0.2 .      //執行建立Docker image
#docker run -it --name c1 -p 8080:80 myimage:v0.2 bash  //開啟Container(c1)
//c1裡
#/usr/sbin/httpd -f /etc/httpd/conf/httpd.conf      //伺服器啟動指令
//開另一個terminal(t2)
#docker inspect c1      //列出所有此Container的資訊
#docker inspect c1 | grep IP    //列出所有c1有關ip的資訊
//到剛剛有c1的terminal(t1)
#cd /var/www/html
#echo "hi" > hi.htm
//回到t2
#curl 127.0.0.1:8080/hi.htm     //結果會印出剛剛設的hi
//t1
#exit   //離開c1
#echo "hello" > hello.htm
#gedit Dockerfile &
//進入編輯文字檔
FROM centos
MAINTAINER larry larry4623@gmail.com
RUN yum update -y
RUN yum install -y httpd net-tools
COPY hello.htm /var/www/html
EXPOSE 80
//儲存
#docker build -t myimage:v0.3 .      //執行建立Docker image
#docker run -it -p 8080:80 myimage:v0.3 bash  //開啟Container
//Container裡
#cd /var/www/html
#ls     //資料夾裡有剛剛拷貝的hello.htm
#cat hello.htm      //印出hello
#/usr/sbin/httpd -f /etc/httpd/conf/httpd.conf &    //伺服器啟動指令
//到t2
#curl 127.0.0.1:8080/hello.htm     //結果會印出hello
```
> ## Docker只有前景執行的概念，沒後景執行的概念，前景執行完Docker會自動關閉。
```
//沿用剛剛t1,t2(terminal 1,terminal 2)
//先把Container刪空
#docker rm -f $(docker ps -a -q) 
//t1
#gedit Dockerfile &
//進入編輯文字檔
FROM centos
MAINTAINER larry larry4623@gmail.com
RUN yum update -y
RUN yum install -y httpd net-tools
COPY hello.htm /var/www/html
EXPOSE 80
CMD [*/usr/sbin/httpd*,*-DFOREGROUND*]
//儲存
//t2
#docker build -t myimage:v0.4 .
#docker run -itd -p 8080:80 myimage:v0.4
#curl 127.0.0.1:8080/hello.htm     //結果會印出hello
```
