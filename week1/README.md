# Docker
Docker是一個容器管理軟體，他是一個推疊概念的容器，比如說你在CentOS下要裝Apache,MySQL,PHP，他會先有一層基底Docker，再來是CentOS疊在上面，再來是Apache，再來是MySQL，再來是PHP，這樣就疊出四個容器了。而他封裝後會變成Image，執行後才會變成容器。   
## 1.Docker建置
### 1.先更新你的機子
```
yum update
```
### 2.再來下載最新Docker套件
```
yum -y install docker
```
### 3.啟動與開機自動啟動Docker
```
systemctl start docker
systemctl enable docker
```
### 4.下載Image，並啟動
```
docker pull centos:latest
docker run -it docker.io/centos:latest
```
成功啟動的會@後面會出現亂數，如下圖：
>![image](https://github.com/LarrySu508/Linux_note/blob/master/Week3/rundocker.png)
## 2.Docker的基本操作
### 1.其他指令
```
docker images   //你本機中有的Image列出
docker ps       //你目前所有容器清單
```
### 2.設定Docker名稱。
![image](https://github.com/LarrySu508/Linux_note/blob/master/Week4/2-1.png)
### 3.用Docker名稱或ID開啟。
![image](https://github.com/LarrySu508/Linux_note/blob/master/Week4/2-2.png)
### 4.關閉開啟的Docker。
![image](https://github.com/LarrySu508/Linux_note/blob/master/Week4/2-3.png)
### 5.於Docker中顯示IP。
#### 若有兩台Docker，請先在兩台上下載net-tools。
```
yum -y net-tools
```
![image](https://github.com/LarrySu508/Linux_note/blob/master/Week4/2-4.png)
### 6.Docker刪除容器指令。
```
docker rm -f id(name)     //-f為強制執行
docker rm -f $(docker ps -a -q)   //刪除ps中能看到的所有容器
docker rmi id(name)
```
### 7.Docker image上傳
#### 1.先到Docker官網申請帳號。
> [Docker.com](https://www.docker.com/)
#### 2.容器匯出image。
![image](https://github.com/LarrySu508/Linux_note/blob/master/Week4/2-5.png)
#### 3.更改標籤名稱(上傳才不會撞名稱)。
![image](https://github.com/LarrySu508/Linux_note/blob/master/Week4/2-6.png)
#### 4.登入Docker，並push上去。
![image](https://github.com/LarrySu508/Linux_note/blob/master/Week4/2-7.png)
#### 5.最後可在網站上看到你傳的image。
![image](https://github.com/LarrySu508/Linux_note/blob/master/Week4/2-8.png)
## 3.Docker 開啟 httpd
### 1.先把容器清空，接著載httpd:latest，再把httpd:latest映像開啟成容器。
![image](https://github.com/LarrySu508/Linux_note/blob/master/Week5/m.png)
### 2.可直接開啟瀏覽器本地端查看，也可去根目錄mydata裡的aa.htm，再用瀏覽器去觀看。
![image](https://github.com/LarrySu508/Linux_note/blob/master/Week5/n.png)
![image](https://github.com/LarrySu508/Linux_note/blob/master/Week5/o.png)
### 3.有個指令很重要，你如果要看你開啟的Docker容器IP，請下下面指令：
```
docker inspect 你Docker的ID或是你給容器的名稱
```
## 4.其他另類操作
### 1.啟動可以附加指令
```
[root@lacalhost user]#docker run -it -d --name test1 chusiang/takaojs1607 /bin/sh -c "while true; do echo "hi"; sleep 1; done"
[root@lacalhost user]#docker logs 338  //338為開啟機子代號。
hi
hi
hi
hi   //因為第一個指令有印hi的規則，所以會看到這些hi。
```
### 2.機子開啟後，如果做離開機子的動作，這機子就一起刪掉
```
[root@lacalhost user]#docker run -it --name test1 --rm chusiang/takaojs1607 /bin/sh
```
### 3.Docker機子與虛擬機共用資料夾
#### 1.建一個檔案和資料夾在根目錄上
```
[root@lacalhost /]#mkdir /mydata
[root@lacalhost /]#echo "hello world" > hello.txt
[root@lacalhost /]#docker run -it -v /mydata:/data chusiang/takaojs1607 /bin/sh
root@f63658996341:/# cd data
root@f63658996341:/data# ls
hi.txt
root@f63658996341:/data# cat hi.txt 
hello
root@f63658996341:/data# touch {a..d}
root@f63658996341:/data# ls
a b c d hi.txt
```
```
//另開一個terminal
[root@lacalhost user]#cd /mydata
[root@lacalhost mydata]#echo "hello" > hi.txt
[root@lacalhost mydata]#ls
hi.txt
[root@lacalhost mydata]#ls
a b c d hi.txt
```
### 4.當docker的機子在運行指令時，要在另一個終端機執行指令
```
//terminal1
root@f63658996341:/data# /bin/bash -c "while true; do echo "hi"; sleep 3; done"
hi
hi
hi
hi
```
```
//terminal2
[root@lacalhost mydata]#docker exec -it f63 bash
root@f63658996341:/#
```
### 5.快速離開容器
按ctrl+p+q，就可快速離開容器。
### 6.當下載的image很小時，執行容器時發生指令查不到
可以直接查說像ping command not found，我用ubuntu的64.2M容器。    
```
[root@lacalhost /]#docker run -it ubuntu bash
root@ccea9d4e9655:/# ping 8.8.8.8
bash: ping: command not found
root@ccea9d4e9655:/# apt-get updata
root@ccea9d4e9655:/# apt-get install iputils-ping
//這樣ping就可執行
```
如果是ifconfig找不到，可以下apt install net-tools。
### 7.
