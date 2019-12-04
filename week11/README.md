# 第十一周
## Docker swarm service replicas模式 與 global模式(全在ROOT模式下操作)      
 * replicas模式：產生副本平均分配到可開啟服務的機子上，當此時有台機子關機，replicas副本會移到其他台上，但關掉的機子開啟時不會重新分配給此機。
 * global模式：固定給所有可開啟服務的機子上產生一個副本，當此時有台機子關機，他身上global副本也跟著關掉，replicas副本移到其他台上，重開啟時global副本也跟著開啟，之前replicas副本不會移回來。
```
//vm1，開啟瀏覽器上的圖形化服務監控觀察
#docker service create --name s1 --replicas 3 httpd
#docker service create --name s2 --mode global httpd
#docker service create --name s3 --replicas 5 httpd
//vm3
#poweroff   //關閉vm3
//觀看瀏覽器發現vm3的replicas副本全移到vm1,vm2上，vm3的global副本也跟著關掉
//vm3 開機
//觀看瀏覽器發現vm3原本的replicas副本還在vm1,vm2上，而vm3的global副本跟著開啟在vm3上
//要把replicas副本配回vm3上用以下操作
//vm1   
#docker service scale s1=1
#docker service scale s1=3  //重新平均分配s1服務到三台機子上
#docker service scale s3=1 
#docker service scale s3=5  //重新平均分配s3服務到三台機子上
```
> ## global副本就有點像K8S的DaemonSet。
## Docker swarm service update 與 rollback
```
//vm1先把剛剛的服務清空
#docker service rm s1
#docker service rm s2
#docker service rm s3
#docker service create --name web --replicas 3 httpd:latest
#docker service ps web  //查看開啟web服務的node，及其所用的Image皆為httpd:latest
#docker service update --image httpd:alpine web
#docker service ps web  //查看開啟web服務的node，原本的Image httpd:latest皆shutdown，更新後的Image httpd:alpine皆為running
#docker service update --rollback web   //還原回原本版本httpd:latest
#docker service ps web  //查看開啟web服務的node，剛剛更新的Image httpd:alpine皆shutdown，還原成Image httpd:latest的版本(狀態為running)
```
## Docker swarm label(標籤)作用
```
//vm1
#docker node update --label-add env=test vm2    //把vm2貼個測試環境的標籤(env=test)
#docker node inspect --pretty vm2   //可以觀察到剛剛貼的標籤 env=test，此指令還可觀察到vm2的狀態資訊
#docker node update --label-add env=product vm3    //把vm3貼個測試環境的標籤(env=product)
#docker node update --label-add cpu=best vm3    //把vm3貼個測試環境的標籤(cpu=best)
#docker node inspect --pretty vm3   //可以觀察到剛剛貼的標籤 env=product,cpu=best
#docker service create --constraint node.labels.env==test --replicas 3 --name web httpd
//觀看瀏覽器的Docker swarm，會發現三個web服務副本都開在有env=test標籤的vm2上
#docker service create --constraint node.labels.env==product --replicas 2 --name web2 httpd
//觀看瀏覽器的Docker swarm，會發現兩個web2服務副本都開在有env=product標籤的vm3上
#docker service create --constraint node.labels.cpu==best --replicas 1 --name web3 httpd
//觀看瀏覽器的Docker swarm，會發現一個web3服務副本開在有cpu=best標籤的vm3上
#docker service create --constraint node.labels.env!=test --replicas 2 --name web4 httpd
//觀看瀏覽器的Docker swarm，會發現兩個web4服務副本個別開在env!=test標籤的vm1,vm3上
```
> ## 雖然可以用--constraint node.labels.env!=test開服務，但參數不能這樣下--constraint node.labels.env!=test&&cpu==best，並無且(&&)的用法喔
## Docker swarm 服務變換label(標籤)操作
```
//vm1先把剛剛的服務清空
#docker service rm web
#docker service rm web2
#docker service rm web3
#docker service rm web4
#docker service create --constraint node.labels.env==test --replicas 3 --name web httpd
//觀看瀏覽器的Docker swarm，會發現三個web服務副本都開在有env=test標籤的vm2上
#docker service update --constraint-rm node.labels.env==test web
//觀看瀏覽器的Docker swarm，會發現三個web服務副本沒標籤條件，所以就平均分配到每台機子上
#docker service update --constraint-add node.labels.env==product web
//觀看瀏覽器的Docker swarm，會發現三個web服務副本因為新增了標籤條件env==product，所以都開在有env=product標籤的vm3上
```