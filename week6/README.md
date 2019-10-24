# 第六周         
## Docker架設Wordpress       
要先有Git與Docker Compose。      
```
#yum install git -y
#curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
#chmod +x /usr/local/bin/docker-compose
```        
接著載Wordpress。             
```
#git clone https://github.com/ayubiz/learn_docker_by_examples 
#cd learn_docker_by_examples/
#cd ch09
#gedit docker-compose.yml & //可以觀察此檔設定，建置時的關聯性。
#docker-compose up -d
#docker-compose ps
#docker ps      //此兩個指令都可看到設定檔建的兩台機器。
```
接著在瀏覽器上輸入虛擬機IP:8080，如:192.168.56.103:8080，設定Wordpress語言、帳號密碼、部落格名稱...。               
在輸入帳號密碼要跟設定檔一樣，也可以用自己帳號密碼，但要與設定檔wordpress的environment一樣。            
## Docker Wordpress備份到另一台虛擬機       
先在Wordpress留言建立文章，接著轉移此虛擬機資料到另一台上，此處主虛擬機為T1，備份用虛擬機為T2。           
```
//T1
#docker volume ls
//會看到docker機子儲存所在的位置與代號名稱，所以可以抓剛剛打好文章的DB做備份複製，此教學用的DB為ch09_db_data。
#docker volume inspect ch09_db_data
//有個mountpoint，為ch09_db_data機子在虛擬機上的位置，此教學位置在/var/lib/docker/volumes/ch09_db_data/_data。
//T2
#systemctl start sshd
#ifconfig       //記住T2 IP，等一下要用ssh傳送。
//T1
#scp -r /var/lib/docker/volumes/ch09_db_data/ user@192.168.56.106:/tmp
//T2
#cd /tmp
#mv ch09_db_data/ /var/lib/docker/volumes/
#yum install git -y
#git clone https://github.com/ayubiz/learn_docker_by_examples 
#curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
#chmod +x /usr/local/bin/docker-compose
#cd learn_docker_by_examples/
#cd ch09
#docker-compose up -d
#ifconfig       //查T2 IP去看是否與T1建的文章一樣，瀏覽器輸入虛擬機IP:8080，如:192.168.56.106:8080，會發現備份成功。
```
## 灰度升级(Gray scale upgrade，又稱局部升级)
一種升级時的平滑切换,當有些伺服器的客户端要進行升级,只對其中一個客户端升级並測試, 確保程序無誤後再全局升级，也就是說所有伺服器不同步更新。
## 增加Docker機器做服務
```
//T1
#gedit docker-compose.yml &     //把ports的8080拿掉，隨機安排ports。
#docker-compose scale wordpress=3 db=2 
#docker-compose ps      //觀察到三台wordpress，對應的不同ports，再到瀏覽器連接試試，會發現得到的結果都一樣。
//此處可以做Load Balance的動作，在三台wordpress前架一台haproxy做Load Balance用。
```