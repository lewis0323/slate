# iENCentre

說明 iENCentre 操作設定，以及相關障礙排除

## iEN 報表無資料 (No Raw Data)

下列方式可以先自行排除，若都不能解決，建議請 TL 協助幫忙，<a href="https://drive.google.com/open?id=1pomWgNIVvLxeuZpJsh7u5iLX20bvQtfbCQxklhX-1Lg" target="_blank">iEN報表無資料問題(Google Doc)</a> 供參考

### iENBox Buffer

可以參考前面 [iENBox 環境](#ienbox-2) 或 <a href="https://drive.google.com/file/d/1WFLOls5F_xNIoI2yBZLPrTm8UrASIXl9/view?usp=sharing" target="_blank">iEN-Box 如何緩存上傳 rawdata files</a> 所述，查看 `upload/buffer` 內是否有 raw data

1. 若沒有，表示 iENBox 有正常上拋回平台，**平台 DataBase** 問題可能性較大

2. 若有 raw data，建議重啟服務，若重啟服務仍不行，請往下逐一確認問題點

若案場是<font color=blue>固定IP</font>，可以透過平台的 <font color=blue>**控制伺服器維運**</font> 進入 iENBox Webservice 作重啟 iENBox 動作，如下圖

![Image](iENCentre/restart.png)

### Firewall Rules

確認 iENBox 對 iEN 平台的 8080 port 是有連通的

* telnet 211.72.253.88 8080，確認網路有無連通

* wget http://211.72.253.88:8080/ienc/MiddlewareMSH，確認是否能下載

若網路有連通，但不能下載 MiddlewareMSH，表示平台有問題，可以確認平台 Middleware 狀態

![Image](iENCentre/middleware.png)

### DNS Server

嘗試上述方式都不行的話，建議修改 iEN 平台 <font color=blue>**主要平台訊息接收URL**</font> 內容

原因有可能是客戶端防火牆，將 DNS Server 給擋住了

原本: http://**ien.com.tw**:8080/ienc/MiddlewareMSH

修改: http://**211.72.253.88**:8080/ienc/MiddlewareMSH

即 <font color=blue>ien.com.tw</font> DNS 為 <font color=blue>211.72.253.88</font> 意思

<font color=red>上述方式都不能解決，建議抓 iENCentre Log 給 TL 幫忙查測</font>

## 開啟 iENCentre LOG 紀錄

若 iENCentre 沒有產生 ien.log 檔案，修改 `log4j.properties`

位置: webapps/portal/WEB-INF/log4j.properties

將前10行換成下面這些之後儲存重啟服務，應能產生 ien.log

```shell
log4j.rootLogger=INFO, ien

log4j.logger.ien=INFO, ien
log4j.appender.ien=org.apache.log4j.RollingFileAppender
log4j.appender.ien.File=${catalina.home}/logs/ien.log
log4j.appender.ien.Encoding=UTF-8
log4j.appender.ien.MaxFileSize=10MB
log4j.appender.ien.MaxBackupIndex=5
log4j.appender.ien.layout=org.apache.log4j.PatternLayout
log4j.appender.ien.layout.ConversionPattern=[%d{MM/dd HH:mm:ss}] %5p (%C{1}:%L) - %m%n
```

沒產生 ien.log 前的設定畫面，可以發現 rootLogger 設為 OFF

![Image](iENCentre/log.png)

## 更換 iENCentre 資料庫使用者帳密

修改下列兩個檔案，將 `database.user` 、 `database.password` 替換欲修改的帳密

1. ienc.properties:		\webapps\ienc\WEB-INF\ienc.properties

2. portal.properties:	\webapps\portal\WEB-INF\portal.properties

![Image](iENCentre/db.png)

<aside class="warning">
設定完須將 MSSQL 服務重啟，再將 iENCentre 服務重啟
</aside>  

## iENCentre 語音告警設定 (Voice Notify)

需將告警事件單的 <font color=blue>**事件等級事件嚴重程度**</font> 設定為重要以上階級，才能收到語音告警

將等級程度設為 <font color=red>緊急</font> 、 <font color=red>警戒</font> 、 <font color=red>重要</font> 以上

![Image](iENCentre/voice.png)

## Edge Computing (邊緣計算) 設定

需要修改 iENCentre 與 iENBox 參數設定，<a href="https://docs.google.com/document/d/1yk3dYvlHknvMeqRwsHxMFqblRAKBpOALbgp690yXjTA" target="_blank">Edge Computing設定(Google Doc)</a>，供參考

1. 勾選控制伺服器的 `Edge Computing` ，使平台不再做 ETL 計算

![Image](iENCentre/edge_ien.png)

2. 設定 iENBox `FOG_COMPUTING_ENABLE` 為 <font color=blue>TRUE</font>

![Image](iENCentre/edge_box.png)

<aside class="notice">
iENBox 版本須為 1.27.1 才能使用 Edge Computing 功能
</aside>  

