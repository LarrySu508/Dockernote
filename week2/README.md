# Docker
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
#docker run -itd --name mywebserver -p 8080:80 -v /mydata:/usr/local/apache2/htdocs lanceray86/test
\\開啟另一個terminal
#cd /mydata
#echo hi > hi.htm 
#curl 127.0.0.1:8080/hi.htm
\\這樣就可以看到上傳的Image可以在另一台機子上運行
```
## 3.兩台Docker對外以不同port，但網頁是一樣，因為共享相同資源
```

```
