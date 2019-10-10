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
