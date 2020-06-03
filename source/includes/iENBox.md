# iENBox

說明 iENBox 環境，以及相關障礙排除經驗

大略分為 [iENBox同步異常](#restore-ienbox) 、 [停用iENBox服務](#ienbox-service) 、 [iENBox更新的方式](#ienbox-3)

相關資料： <a href="https://drive.google.com/file/d/1bv0vPYsVt1-6_Cw0q_6U6F5fipo1r48Z/view?usp=sharing" target="_blank">iENBox教育訓練</a> 、 <a href="https://drive.google.com/file/d/1qxGGkrCWZCTl-AfNOJk3_-q_zwo6McYI/view?usp=sharing" target="_blank">障礙排除SOP</a>

## iENBox 環境

iENBox 可以簡單分為下列五個：

環境 | 描述 | 路徑
--------- | ----------- | -----------
Bin | 主程式 (含 modtool.jar 等程式) | opt/ienbox/core/bin
Log | LOG紀錄 (ienbox.log and driver.log) | opt/ienbox/storage/log
Config | 參數設定 (PROPRIETARY.PROPERTIES) | opt/ienbox/core/config
Lib | 程式 library | opt/ienbox/core/lib/OSGi
Raw Data | 設備資料 | opt/ienbox/storage/upload/buffer

* Bin： 通常在此資料夾，是將 [iENBox服務停用](#ienbox-service) 或作 [Restore iENBox](#restore-ienbox) 的動作，以及 [Modscan](#modscan) 測試設備通訊

* Log： iENBox運作紀錄 (e.g., MQTT連線、IEN平台連線等異常紀錄)

* Config： 在 Maintenance 設定的結果，會儲存在 `PROPRIETARY.PROPERTIES` 檔案

	若有需要看詳細的 iENBox Log 資訊，可以將 `LOG.PROPERTIES` 作下列修改<font color=blue> (擇一方式)</font>，可參考<a href="https://docs.google.com/document/d/1SW1iQCChDK9E344xlR5i_GJC6xQIeRSMmIERfSItshk/edit?usp=sharing" target="_blank">開啟 iENBox Log 設定</a>：

	1. log4j.logger.IENBOX.MSG=<font color=red>INFO</font>, IENBOXFileAppender

	2. log4j.logger.IENBOX.NOTIFY=<font color=red>DEBUG</font>, IENBOXFileAppender
	
* Lib： 這邊是相關 library，只有在尚未釋出 iENBox 新版本，Release Candidate 階段先將 lib 作蓋檔，大多蓋在 jars/cht 裡
	
	有做蓋檔動作，得用 `chmod a+x`，確認 jars 有權限可以執行

* Raw Data： 確認 iENBox 是否存留 raw data，通常當<font color=blue>報表沒數值</font>來這邊作確認，參考資料： <a href="https://drive.google.com/file/d/1YL__8LtTtaNyB23LoKsnSmskzB_UMR0M/view?usp=sharing" target="_blank">iEN-Box 如何緩存上傳 rawdata files</a>

	1. 如果有 Raw Data，先去看 Log，確定非網路問題 (e.g., 防火牆等問題) 後，建議將 iENBox 服務重啟，確認是否消失

	2. 沒有 Raw Data，建議看 iENCentre Log，這邊可以確定 iENBox 有將 Raw Data 上拋回去
	
<aside class="notice">
當 Raw Data 檔名從 <code>current</code> 轉換成 <code>fetch</code> ，表示正在上拋回 iEN 平台
</aside>  

### WebService Maintenance

設定頁面： http://localhost:4511，localhost預設如下：

設備型號 | 預設網址
--------- | -----------
ATOP | 10.0.50.100:4511
Advantech | 192.168.0.1:4511
Windows | 127.0.0.1:4511

WebService Overview 如下圖，可以查看 iENBox 基本資訊

![Image](iENBox/overview.png)

參數說明： 

說明常用到的 iENBox 參數，下圖有很明確的表達參數的用途

* CACHE_FEEDER_INTERVAL： 每個控制器之間，間隔多久時間後再詢問，即 A 控制器 到 B 控制器 要間隔的時間，才做詢問動作 

* JAMOD_READ_DELAY_MS： 詢問一個監控點所能花費的時間，即可以表達為 response timeout 意思

* JAMOD_SEND_DELAY_MS： 問完一個監控點後，再詢問下一個監控點所花費的時間

* JAMOD_RETRY_INTERVAL： 控制器詢問監控點失敗後，間隔多久時間後再做詢問的動作

* DATA_LOGGING_TIMEOUT： 每個控制器詢問所有監控點最多可花費的時間，一旦超過，後面監控點直接 timeout

![Image](iENBox/parameter.png)

## iENBox 參數說明

承上述參數說明，這邊僅稍微描述參數用途，可依其他障礙處理去做設定，了解細節設定的差別性

* IENBOX_TIMEZONE： 設定 iENBox 時區 (預設是 +8:00 UTC)

* IENBOX_MQTT_ENABLE： 啟用 iENBox MQTT 同步納管功能 (預設是 False)

* IENBOX_MQTT_URL_FORMAT： MQTT 連線的 IP (預設是 tcp://mqtt.ien.net.tw:1883)

* DRIVER_RETRY_LIMIT： 詢問設備異常時，嘗試詢問的次數

* DRIVER_RETRY_INTERVAL： 控制器詢問監控點失敗後，間隔多久時間後再做詢問的動作

* DRIVER_TAG_ERROR_LIMIT： 連續 X 點詢問失敗後，就不再詢問後面的監控點

* DRIVER_MODBUS_VALIDATE_WRITTEN： 確保 iEN 平台 AO 點的寫入

* JAMOD_READ_DELAY_MS： 詢問一個監控點所能花費的時間

* JAMOD_SEND_DELAY_MS： 問完一個監控點後，再詢問下一個監控點所花費的時間

* CACHE_EXPIRACY_TIME： iEN 平台資料的暫存時間

* CACHE_FEEDER_WORST_CREDIT： iENBox 逞罰時間，當詢問失敗次數越高後，間隔時間會拉長

* CACHE_FEEDER_INTERVAL： 每個控制器之間，間隔多久時間後再作詢問動作

* SEND_NOTIFY_TIMER_INTERVAL： 事件單保存空間

* SEND_NOTIFY_BUFFER_SIZE： 間隔多久發送事件單

* IOT_ENABLE： 啟用上拋 IoT 平台功能

* IOT_API_KEY： IoT 平台的 API KEY

* IOT_HOST： IoT 平台的 IP

* IOT_RESTFUL_PORT： IoT 平台的 PORT

* IOT_SILENCE_ENABLE： 若 iENBox 會從 IoT 平台抓取資料，並再將資料轉拋回 IoT 平台，此參數需設為 True

* IENBOX_REBOOT_ENABLED： 啟用 iENBox 重開機功能

* IENBOX_REBOOT_CRON： 設定 iENBox 重開機時間

* DRIVER_KARAFUTO_TIMEOUT： IoT 平台的資料 timeout，e.g., 設 60 分鐘，表示 IoT 平台資料超過 60 分鐘，視為 N/A

* FOG_COMPUTING_ENABLE： 設定 Edge Computing 功能

## 停用iENBox Service

說明 iENBox 停用的方式，在測試設備通訊時候使用，避免通訊埠打架

### Windows iENBox

* 服務 -> iENBox -> 停止

![Image](iENBox/stop.png)

<aside class="warning">
記得將 iENW 服務停用，停止 watchdog，避免 iENBox 服務被 watchdog 呼叫起來
</aside> 

### Linux iENBox

debug 檔案可以暫停 iENBox watdog 功能，讓服務殺掉不會再被帶起來

在主程式位置，輸入下列指令，以下圖為例，iENBox 的 PID 為 969，kill 969 即可殺掉 iENBox 服務

```shell
	touch debug
	### check the PID of iENBox and kill it
	ps aux | grep java
	kill $PID
```
![Image](iENBox/none.png)

欲恢復 iENBox 服務，將 debug 檔案移除即可，watchdog 會將 iENBox 服務帶起來

```shell
	rm debug
	ps aux | grep java
```

確認其 iENBox 服務是否恢復運作

![Image](iENBox/recover.png)

## 同步異常處理 (Restore iENBox)

以下為 iENBox Restore 清除的方式，基本上同步異常有很多原因

這邊僅能提供單純確認 iENBox 以及 MQTT Server 是否有問題，若清除 iENBox 後，還仍不能同步

建議在平台新增新的 iENBox，嘗試看看能不能同步新的 iENBox

以上方式均不行，建議得看 iENCentre 的 LOG 紀錄，並請 TL 協助幫忙

### Restore Windows iENBox

* 從服務將 iENBox 服務停止

* iEN/iENBox/core/bin，執行`clean.bat`

### Restore iENBox by WebService

* Reset and Restore

* Restore Default Settings

### Restore iENBox by Shell (Linux)

執行 `clean.sh` 會將 iENBox 作清除動作 (若有 raw data 也會一併清除)

```shell
	cd /opt/ienbox/core/bin
	./clean.sh
```

### 確認 MQTT Server 通訊

可以透過 iENBox WebService 頁面的 Overview 中，查看 MQTT Status 是否是 connected 狀態

當 MQTT Status 狀態是 lost 得確認網路端是否正常，可以使用 ping 、 telnet 來做確認

若網路運作正常，但是 MQTT Server 連不上，更換成 211.72.253.90 嘗試看看 

以上均不行，建議請 TL 確認 MQTT Server 狀態

```shell
	### Check network connection
	ping 168.95.1.1
	### Check MQTT connection
	telnet mqtt.ien.net.tw 1883
```

### 相關同步異常 Log

* StatusCode: 400，Required Control Server ID doesn't match my ID.

表示該 iENBox 被其他案場同步過，與原本納管 control server id 不同，清除 iENBox 即可解決

![Image](iENBox/dismatch.png)

* 同步結果卡在同步中

若一直顯示同步中，建議將 iENBox 版本更新至最新後，再同步一次

原因為可能 iENBox 版本過舊，導致 iENCentre 新版本新增的功能有所衝突

## iENBox 時間校正

從 iEN 平台同步，即可將 iENBox 時間做校時，惟須確認 iENBox 重開機後時間是否會跑掉

若重開機後時間會跑掉，建議同步 iENBox 後，確認目前 iENBox 時間是正確的

再利用 `hwclock -w` 將 目前系統時間 寫入 <font color=blue>hardware clock</font> 

```shell
	### Check iENBox current time
	date
	### Set iENBox hardware clock from system clock
	hwclock -w
```

## iENBox / Litebox 定時重開機

若設備有不穩定情況，建議將 iENBox / Litebox 作定時重開機動作，再觀察幾天看看

### WebService Maintenance

根據案場環境，設定不同重開機條件，iENBox 能定時重啟，Litebox 則是 MQTT 斷線

iENBox：

* Maintenance

IENBOX_REBOOT_ENABLED： TRUE

IENBOX_REBOOT_CRON (重啟時間)： * 0 0 * * ? *，表示為 00:00:00 作重開機動作

前三個分別表示： 秒、分、時，`*` 表示任意值，若只要在 01:XX 重啟，可以表示為 * * 1 * * ? *

![Image](iENBox/reboot.png)

Litebox：

* Box Configuration -> Automatic Maintenance

Reboot when Memory Used: 當記憶體上升到多少的使用率後，作重開機動作

Reboot when MQTT Disconnect Timeout: 當 MQTT 通訊異常多少秒數後，作重開機動作 

Reboot Hours: 間隔多少小時後，作重開機動作

![Image](Litebox/reboot.png)

### 平台 Maintenance 設定

* 維運選單 -> 障礙查測 -> 控制伺服器維運 -> MQTT 遠端維運 -> 選擇 Box Configuration

設定相關重開機條件

![Image](iENBox/rebootiENBox.png)

![Image](Litebox/rebootLiteBox.png)

## iENBox 設備點位無法讀取

此問題為設備通訊不穩定，可以加終端電阻，或換通訊線來改善

另外也能透過調整 <font color=blue>timeout</font> 、 <font color=blue>polling time</font>，提升 iENBox 詢問設備的成功率/

### iEN 平台設定

* 維運選單 -> 設備管理 -> 控制器管理 -> 選擇控制器

* 輪詢時間：表示為 polling time，等於 iENBox 的 `JAMOD_SEND_DELAY_MS`

* 連線逾時；表示為 timeout，等於 iENBox 的 `JAMOD_READ_DELAY_MS`

預設輪詢時間是 50ms，連線逾時是 500ms，輪詢時間建議最長到 <font color=blue>100ms ~ 200ms</font>，而連線逾時最長則是 <font color=blue>5s ~ 7s</font>

![Image](iENBox/device.png)

### iENBox 設定

主要是調整 `JAMOD_SEND_DELAY_MS` 、 `JAMOD_READ_DELAY_MS` 這兩個參數

這邊就針對 iENCentre 設定作說明，不再論述 iENBox，細節參考 <a href="https://hoyuisun.github.io/#ienbox-2">iENBox 環境</a> 的常用 iENBox 參數說明

## iENBox 納管其他 iENBox (iENBox Protocol)

參考： <a href="https://drive.google.com/file/d/1xQzHOHTel9E297pMwCQFMbg-8-a8IpYN/view?usp=sharing" target="_blank">iEN-Box 納管其他 iEN-Box</a>

1. 在新增控制器 -> 通訊協定選擇「IENBox」

2. 填入欲納管 iENBox 的 IP 及 埠號: 4511

3. 暫存器位置： 監控點ID <font color=red>(註: 不需要帶 **$** 符號)</font>

![Image](iENBox/addIENBox.png)

## iENBox 上拋 IoT 大平台設定

在 iENBox 將相關參數做設定，即可上拋回 IoT 大平台，參考連結：<a href="https://drive.google.com/file/d/1_sFtWw1dj8ie3wwc9DTf1NdhubgvL2Am/view?usp=sharing" target="_blank">iOT平台上拋設定</a>

* IOT_ENABLE： 是否啟用 IoT 上拋功能

* IOT_API_KEY： 填入專案的 API_KEY

* IOT_HOST： 預設為 CHT IoT 平台，有需要可以更改為其他平台

* IOT_RESTFUL_PORT：預設為 80 ，可視實際狀態做更動

![Image](iENBox/IoT.png)

<aside class="notice">
若是將資料上拋回 中華 IoT 大平台，<code>IOT_HOST</code>、<code>IOT_RESTFUL_PORT</code>基本上不需要做額外更動
</aside> 

## iENBox 事件單消化不良

此為簡訊有延遲告警的情形，原因是 iENBox 產生過多的事件單，以致於事件單需要排隊消化

然而，當事件單成功上傳至平台才會發簡訊告警，但這個事件單先前就已經產生出來，並在 iENBox 排隊

導致從 iEN 平台查看事件單維護，有簡訊延遲告警的問題

### 解決辦法

1. 新增一台 iENBox 控制伺服器，重新設定點位、告警等條件，再觀察看看

2. 調整下列事件單參數，並再觀察看看

* SEND_NOTIFY_BUFFER_SIZE： 事件單保存空間，可以理解該參數值越小，表示排隊隊伍越短

* SEND_NOTIFY_TIMER_INTERVAL： 間隔多久發送事件單，預設是每分鐘送一次 <font color=red>(註: 一次上傳一筆事件單)</font>

![Image](iENBox/event.png)

<aside class="warning">
若是採用 <code>Public IP</code> 納管，切記須將 <code>MQTT Enable</code> 設為 <code>False</code>，
否則無法將事件單上傳至平台
</aside> 

## iENBox 4511 頁面卡住

此問題發生於 <code>iENBox-1.24.15</code> 版本上，為亂數效能的關係，後採購版本均安裝 `iENBox-1.25.7` 已有修正此現象

其解決方法將 iENBox 資料夾的 <font color = blue>env.sh</font> 檔案進行蓋檔即可，下載連結：<a href="https://bit.ly/3bFfIKw" target="_blank">env.sh</a>

1.利用 WinSCP / FileZilla 將 env.sh 上傳至 iENBox (預設為: /media/sd/home/cht)

2.將檔案複製到 iENBox 資料夾內，位置: /opt/ienbox/core/bin

* 執行此指令： <font color = blue >cp /media/sd/home/cht/env.sh /opt/ienbox/core/bin </font>

## MQTT Broker 狀態確認

若發生 iEN 平台斷線、無法同步等情況，需要確定 MQTT 服務是否正常運作，參考： <a href="https://docs.google.com/document/d/1xli3lcqkSH1JfjuzV5Zac5lHzTcbhyuAaS5twoUmYbI" target="_blank">確認MQTT Broker狀態</a>

1.透過 `MQTTlens` 確認 MQTT Broker 狀態：

   在 MQTTlens 輸入 IP、Port、使用者帳密 (帳密: ien/ienet1308)，建立 MQTT 連線

![Image](iENBox/mqttlens.png)

2.利用 MQTTlens 的 <font color=blue>Publish / Subscribe </font> 確認 MQTT Broker 狀態：
   
   * 先做 <font color=red>Subscribe</font> 訂閱動作，再進行 <font color=red>Publish</font> 發布動作 
   
   下圖為對 test topic 做 Subscribe 訂閱，接著 Publish Gino 訊息給 test topic
   
   可以看到回應 Message: Gino，有回應成功，表示 MQTT Broker 是正常運作的

![Image](iENBox/publishsubscribe.png)

## iENBox 升級包及更新

講述 iENBox 升級包如何產生，以及如何將案場的 iENBox 做更新

### 升級包製作

此升級包需要 iENBox 版號對應才能更新，例如： iENBox_1.24.15-1.26.3

那麼需要 iENBox 1.24.15 packages 與 iENBox 1.26.3 packages

執行 MakeBoxPatch-NoSQL.bat，將對應的 packages 填入

`MakeBoxPatch-NoSQL.bat <舊版位置> <新版位置> <升級包名稱>`

若要製作 iENBox all-1.26.3，則舊版位置為 null，即建立空的資料夾帶入

參考： <a href="https://drive.google.com/file/d/1o3KfMOr2GgFL5k8sLoEC8gVDGpCWjUnQ/view?usp=sharing" target="_blank">iENBox安裝檔與升級包製作</a>

![Image](iENBox/patch.png)

### iENBox Update

分成兩種方式，其一可透過 iEN 雲端平台更新，另一種則為一鍵更新

* iEN 雲端更新： 維運選單 -> 軟體版本管理 -> 選擇要更新的 iENBox -> 更新

透過下列網址，確認升級包檔案是否存在平台，將版號作對應修改即可： ienbox_<font color=red>1.24.15</font>-<font color=red>1.26.3</font>.tar
  
`http://ien.net.tw:8080/ienc/update/ienbox_1.24.15-1.26.3.tar`

![Image](iENBox/OTA.png)

<aside class="notice">
<code>iENBox 可升級版號</code>為更新後的 iENBox 版本
</aside>  

* 一鍵更新： 將升級包放置同個資料夾，修改 `upgrade.bat` 跟 `runcmd.bat`

將 IP 以及檔名改成相對應的名稱，如紅框所示，執行 `upgrade.bat` 進行更新，下載: <a href="http://bit.ly/2DxKDsu" target="_blank">一鍵更新</a>

![Image](iENBox/upgrade.png)

![Image](iENBox/runcmd.png)

* 手動更新： 將 升級包 放至 iENBox SD卡 中，其實就等於手動執行 script 的動作，做跟 runcmd 內容一樣的指令

```shell
	### 解壓縮升級包
	tar xvf ienbox_all-1.26.3.tar
	### 執行patch script
	sh patch.sh /opt/ienbox
```

### 雲端閘道器 iENBox 更新 (Cloud Gateway)

參考： <a href="https://docs.google.com/document/d/1Aid6HHghIUjJDLgsgr2E91PjZwuXL0ClRyv6TEKPa_k" target="_blank">手動更新 Cloud Gateway 上面的 iENBox 版本</a>

Step 1. 將 iENBox 安裝 iso 檔案裡面的 `06-ienbox.tar.gz` 複製到 Cloud Gateway 上的 /opt/ienboxes 中，如此未來新增的 iEN-Box 都會採用最新的版本運行

Step 2. tar zxvf /opt/ienboxes/06-ienbox.tar.gz -C /tmp

Step 3. cp -f /tmp/ienbox/core/hsqldb/bin/ienbox.script /opt/ienboxes/CG0000000007/core/hsqldb/bin/

Step 4. cp -f /tmp/ienbox/core/lib/OSGi/jars/cht/*.jar /opt/ienboxes/CG0000000007/core/lib/OSGi/jars/cht/

Step 5. (cd /opt/ienboxes/CG0000000007/core/bin ; ./clean.sh)

Step 6. (cd /opt/ienboxes/CG0000000007/core/bin ; ./run.sh stop)

Step 7. 確認iENBox版本

![Image](iENBox/CloudiENBox.png)

## Firmware Upgrade

進入 ATOP 網路設定頁面： http://10.0.50.100/，點選 System Setup -> Firmware Upgrade

Step 1. 確認 iENBox Firmware version

![Image](Firmware/version.png)

Step 2. 更新 Firmware

![Image](Firmware/firmware.png)

Step 3. 確認 Firmware 檔案是否正確

![Image](Firmware/confirm.png)

Step 4. iENBox 更新 Firmware 中

![Image](Firmware/update.png)

Step 5. 更新完後會重開機，確認 Firmware version

![Image](Firmware/finish.png)
