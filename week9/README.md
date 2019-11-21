# 第九周
## Docker Swarm(參考:[Swarm](https://docs.docker.com/engine/swarm/))
![image](189018.jpg)
### 開三台虛擬機，Docker1,Docker2,Docker3，並把SSH打開連putty用。         
```
//Docker1(vm1)
#hostnamectl set-hostname vm1
#bash
#systemctl start sshd
//Docker2(vm2)
#hostnamectl set-hostname vm2
#bash
#systemctl start sshd
//Docker3(vm3)
#hostnamectl set-hostname vm3
#bash
#systemctl start sshd
``` 
```
//vm1
#gedit /etc/hosts
/*在下方加入三台虛擬機IP
192.168.56.103 vm1
192.168.56.106 vm2
192.168.56.111 vm3
*/
#ping vm2   //成功，因為剛剛的設定，所以會幫你轉成IP
//先清空三台的ssh
#cd .ssh
#rm *   //全按y
#cd 
//vm2
#cd 
#rm -rf .ssh
//vm3
#cd 
#rm -rf .ssh
//vm1
#ssh-keygen    //全按Enter
#ssh-copy-id root@vm2   //輸入yes，在輸入vm2密碼
#exit
$su
#ssh root@vm2   //這樣就可無密碼登入
#exit
#ssh-copy-id root@vm3   //輸入yes，在輸入vm3密碼
#ssh root@vm3   //這樣就可無密碼登入
#exit
#scp /etc/hosts vm2:/etc/hosts  //拷貝剛剛設定的檔案，這樣vm2也可下ping vm1,ping vm3，這樣的指令。
#scp /etc/hosts vm3:/etc/hosts  //拷貝剛剛設定的檔案，這樣vm3也可下ping vm1,ping vm2，這樣的指令。
```     
### 開putty連三個虛擬機
```
//vm2
#docker pull httpd:latest
#docker pull httpd:alpine
//vm3
#docker pull httpd:latest
#docker pull httpd:alpine
//vm1
#docker pull httpd:latest
#docker pull httpd:alpine
#docker pull dockersamples/visualizer
#docker swarm init --advertise-addr 192.168.56.103  //vm1的IP addr
//輸入完下方有一條指令，貼到vm2,vm3上
//vm2
#docker swarm join --token SWMTKN-1-3pu6hszjas19xyp7ghgosyx9k8atbfcr8p2is99znpy26u2lkl-1awxwuwd3z9j1z3puu7rcgdbx 192.168.56.103:2377
//vm3
#docker swarm join --token SWMTKN-1-3pu6hszjas19xyp7ghgosyx9k8atbfcr8p2is99znpy26u2lkl-1awxwuwd3z9j1z3puu7rcgdbx 192.168.56.103:2377
//vm1
#docker node ls //此時會看到docker swarm裡會有三台機器，有一個'*'就是master
#docker run -itd -p 8888:8080 -e HOST=192.168.56.103 -e PORT=8080 -v /var/run/docker.sock:/var/run/docker.sock --name visualizer dockersamples/visualizer   //此時在瀏覽器上輸入IP:PORT，例如：192.168.56.103:8888
```
### 想把初始狀態的叢(cluster)取消
```
//vm2
#docker swarm leave     //vm2這個節點離開工作
//vm1
#docker node ls     //會看到vm2 STATUS是Down的狀態，網頁也是一樣的情形
//vm3
#docker swarm leave     //vm3這個節點離開工作
//vm1
#docker node ls     //會看到vm3 STATUS是Down的狀態，網頁也是一樣的情形
//最後把master節點拉掉要用另外指令
#docker swarm leave --force
#docker node ls     //此時沒任何節點顯示，Docker swarm整個destroy
```
```
//重新建回
#docker swarm init --advertise-addr 192.168.56.103  //vm1的IP addr
//輸入完下方有一條指令，貼到vm2,vm3上，如果指令跑掉請下
#docker swarm join-token worker //還有另一個manager的身份
//vm2
#docker swarm join --token SWMTKN-1-3pu6hszjas19xyp7ghgosyx9k8atbfcr8p2is99znpy26u2lkl-1awxwuwd3z9j1z3puu7rcgdbx 192.168.56.103:2377    //worker身份
//vm3
#docker swarm join --token SWMTKN-1-0n2pndpf71a91wwhxbw460qq03cf9atfleeg1328usrkhmg2mh-9zic8m5cg3y1tf1wti1pvzxcx 192.168.56.103:2377    //manager身份
//vm1
#docker node ls     //看到MANGER STATUS狀態，如果都沒寫東西，就是worker
//節點身份可升級、降級
#docker node demote if6p20c7gcarukmmkqx3s4x2c   //if6p20c7gcarukmmkqx3s4x2c為vm3node的ID
#docker node ls     //看到vm3降級到worker
#docker service create --name web httpd     //啟動web服務的httpd
#docker service ls  //可以觀察到你開啟的服務
#docker service ps web  //可以觀察到web服務開在哪台，瀏覽器上也可看到直觀的圖形化介面
#docker service scale web=3    //會發現相同的服務也開在其他兩台節點上，這樣vm1掛了，vm2,vm3還能代替vm1提供web服務
#docker service scale web=5    //總共會開5個副本
#docker service scale web=2    //這樣縮回2個副本
#docker service scale web=3    //平均分佈在vm1,vm2,vm3上
#docker node update --availability drain vm1    //通常服務不開在manager上，所以可以下這行指令，除了vm1，vm2,vm3會有總共三個Web服務
#docker service scale web=2 
```
### 刪除service
```
//vm1
#docker service rm web  //全部service web移除，瀏覽器上可看到服務全清空
#docker service create --name web --replicas 2 httpd:latest  //增加副本的參數replicas(參數前一定要加--)
#docker ps  //看到vm1主機上開的節點(node)
#docker service ps web     //看到vm1上開的是web.2，意思為第二個提供web服務的節點(node)
#docker exec -it web.2.vmje3ueivvkrlupae5a4j9kl8 sh   //進入vm1開的web服務節點(node)，查看資料(web.2.vmje3ueivvkrlupae5a4j9kl8為節點名稱)
#cd /usr/local
#cd apache2
#cd htdocs
#cat index.html   //可以看到瀏覽器開的網頁192.168.56.103:8888，裡面的資訊
#exit
```
### 提供網路服務設定
```
//vm1
#docker service rm web  //全部service web移除，瀏覽器上可看到服務全清空
#docker service create --name web --replicas 2 -p 5555:80 httpd:latest  //此時可以在瀏覽器上發現開vm1,vm2,vm3的網站可以看到所開的服務，例如；192.168.56.103:5555(vm1),192.168.56.106:5555(vm2),192.168.56.111:5555(vm3)
```
### 解決開啟服務時忘記設定PORT
```
//vm1
#docker service create --name web --replicas 2 httpd:latest
#docker service update --publish-add 5555:80 web
```
### 查看每個Docker內的container網路狀態(直接開是看不到的，#docker exec -it b0bcccdb17de sh + ifconfig是看不到的)
```
//vm1
#docker ps
#docker inspect b0bcccdb17de    //這樣可以看到b0bcccdb17de這個container的IP
#docker ps  //等一下要用到NAMES web.2.vmje3ueivvkrlupae5a4j9kl8
#docker run -it --network container:web.2.vmje3ueivvkrlupae5a4j9kl8 busybox sh
/#ifconfig  //這樣也可以看到IP，busybox上的設定基本是照著b0bcccdb17de這個container的設定，Docker swarm基本上有兩張網卡
/#ping 8.8.8.8  //基本上可以做PING的動作(Bridge連外網用)
/#ping 10.255.0.9  //10.255.0.10為vm2的網卡(overlay與其他docker做連接用)
//Docker swarm基本上有兩張網卡，一張是Bridge連外網用，另一張是overlay與其他docker做連接用
```