# iENBox

說明 iENBox 環境，以及相關障礙排除經驗

大略分為 [iENBox同步異常](#restore-ienbox) 、 [停用iENBox服務](#ienbox-service) 、 [iENBox更新的方式](#ienbox-3)

相關資料： <a href="https://drive.google.com/open?id=1Z-syFzXTG4-eEB7VUSGyWt_ZOC8phfG8" target="_blank">iENBox教育訓練</a> 、 <a href="https://drive.google.com/open?id=1-6OyF8CFe69U9L563_LCbSUEJrqA6CUK" target="_blank">障礙排除SOP</a>

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

	若有需要看詳細的 iENBox Log 資訊，可以將 `LOG.PROPERTIES` 作下列修改<font color=blue> (擇一方式)</font>：

	1. log4j.logger.IENBOX.MSG=<font color=red>INFO</font>, IENBOXFileAppender

	2. log4j.logger.IENBOX.NOTIFY=<font color=red>DEBUG</font>, IENBOXFileAppender
	
* Lib： 這邊是相關 library，只有在尚未釋出 iENBox 新版本，Release Candidate 階段先將 lib 作蓋檔，大多蓋在 jars/cht 裡
	
	有做蓋檔動作，得用 `chmod a+x`，確認 jars 有權限可以執行

* Raw Data： 確認 iENBox 是否存留 raw data，通常當<font color=blue>報表沒數值</font>來這邊作確認，參考資料： <a href="https://drive.google.com/file/d/1WFLOls5F_xNIoI2yBZLPrTm8UrASIXl9/view?usp=sharing" target="_blank">iEN-Box 如何緩存上傳 rawdata files</a>

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

## iENBox 納管其他 iENBox (iENBox Protocol)

參考： <a href="https://drive.google.com/open?id=13BCjy7DBRGroE2oNK_OkXACVogV9Tpq_" target="_blank">iEN-Box 納管其他 iEN-Box</a>

1. 在新增控制器 -> 通訊協定選擇「IENBox」

2. 填入欲納管 iENBox 的 IP 及 埠號: 4511

3. 暫存器位置： 監控點ID <font color=red>(註: 不需要帶 **$** 符號)</font>

![Image](iENBox/addIENBox.png)

## iENBox 升級包及更新

講述 iENBox 升級包如何產生，以及如何將案場的 iENBox 做更新

### 升級包製作

此升級包需要 iENBox 版號對應才能更新，例如： iENBox_1.24.15-1.26.3

那麼需要 iENBox 1.24.15 packages 與 iENBox 1.26.3 packages

執行 MakeBoxPatch-NoSQL.bat，將對應的 packages 填入

`MakeBoxPatch-NoSQL.bat <舊版位置> <新版位置> <升級包名稱>`

若要製作 iENBox all-1.26.3，則舊版位置為 null，即建立空的資料夾帶入

參考： <a href="https://drive.google.com/open?id=1F91nYGgeTILX_D_N6xW8SJiW3O8iZexl" target="_blank">iENBox安裝檔與升級包製作</a>

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

將 IP 以及檔名改成相對應的名稱，如紅框所示，執行 `upgrade.bat` 進行更新

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
