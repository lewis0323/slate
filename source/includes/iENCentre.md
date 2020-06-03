# iENCentre

說明 iENCentre 操作設定，以及相關障礙排除

## iEN 報表無資料 (No Raw Data)

下列方式可以先自行排除，若都不能解決，建議請 TL 協助幫忙，<a href="https://drive.google.com/file/d/1Gy2fmL8zUoNImWBlnhODI4bhcBWQ9pqV/view?usp=sharing" target="_blank">iEN報表無資料問題(Google Doc)</a> 供參考

### iENBox Buffer

可以參考前面 [iENBox 環境](#ienbox-2) 或 <a href="https://drive.google.com/file/d/1YL__8LtTtaNyB23LoKsnSmskzB_UMR0M/view?usp=sharing" target="_blank">iEN-Box 如何緩存上傳 rawdata files</a> 所述，查看 `upload/buffer` 內是否有 raw data

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

## 告警事件單無觸發 (Troubleshooting)

請先確認 iENBox 是否有觸發事件，並作上拋事件單的動作

`Remaining Capacity in Queue` 為 iENBox 目前事件單排隊的數量

此值越大，表示產生的事件單需等待前面的事件單消化完，才會被上拋至平台

![Image](iENCentre/eventlog.png)

查看 iENBox Log 均為正常的話，但仍舊無法在平台查詢到事件單

確認 <font color=blue>事件單時間過濾</font> 是否有打勾，若無打勾仍收不到平台告警事件單，建議請 TL 協助幫忙

![Image](iENCentre/filter.png)

<aside class="notice">
查看上拋事件單LOG，iENBox 需設為 <font color=red>Debug </font>模式，可以參考上述 <font color=blue>iENBox環境</font> 說明設定為 Debug 模式
</aside> 

## 太陽能平台免登網址 (SimpleSolar)

免登網址： https://ien.com.tw/portal/solar/solarRealtimeInfoReport.action?simpleSolar=true&buildingId=<font color=blue>buildingId</font>

查詢 `buildingId` 方法： 在電站總覽，點選欲查詢的電站名稱，

從左下角訊息列的 <font color=blue>javascript: querySolarRealtimeInfoReport()</font>，查詢到相對應的電站編號

![Image](iENCentre/buildingId.png)

## 自訂報表設定 (CustomReport)

若欲在 iENCentre 設定自訂報表選單，可以執行連結的 SQL 檔案

執行 SQL 指令後，重新登入即可從 報表選單 看到 自訂報表 ，連結： <a href="http://bit.ly/2NnP1AM" target="_blank">
自訂報表.sql</a>

## 選單管理設定 (functionManage)

若在新版 iENCentre 沒有顯示 選單管理頁面，可以執行連結的 SQL 檔案

執行 SQL 指令後，重新登入即可看到 選單管理，連結： <a href="http://bit.ly/2MuzxLC" target="_blank">
選單管理權限.sql</a>

## 設定 iENBox 斷線重試次數 (Disconnect Retry)

經常性收到 iENBox 斷線事件單告警，其原因為網路不穩定或者 4G 網路較易發生 handoff 狀況

建議將 `Disconnect Retry` 次數調長，降低斷線事件單告警次數

![Image](iENCentre/retry.png)

## iENCentre 語音告警設定 (Voice Notify)

需將告警事件單的 <font color=blue>**事件等級事件嚴重程度**</font> 設定為重要以上階級，才能收到語音告警

將等級程度設為 <font color=red>緊急</font> 、 <font color=red>警戒</font> 、 <font color=red>重要</font> 以上

![Image](iENCentre/voice.png)

## Edge Computing (邊緣計算) 設定

需要修改 iENCentre 與 iENBox 參數設定，<a href="https://docs.google.com/document/d/1yk3dYvlHknvMeqRwsHxMFqblRAKBpOALbgp690yXjTA" target="_blank">Edge Computing設定(Google Doc)</a>，供參考

1.勾選控制伺服器的 `Edge Computing` ，使平台不再做 ETL 計算

![Image](iENCentre/edge_ien.png)

2.設定 iENBox `FOG_COMPUTING_ENABLE` 為 <font color=blue>TRUE</font>

![Image](iENCentre/edge_box.png)

<aside class="notice">
iENBox 版本須為 1.27.1 才能使用 Edge Computing 功能
</aside>  

