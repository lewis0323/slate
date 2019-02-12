# iENPortal 開發

說明 iENPortal 環境建置，以及相關程式 function 用途

這部分會針對派工流程做說明

## 環境建置與編譯

Step 1. 安裝 Java SDK (jdk-7u80-windows-x64)，設定環境變數

* JAVA_HOME: 系統安裝 Java 的 JDK 路徑

* JRE_HOME: 系統安裝 Java 的 JRE 路徑

Step 2. 安裝資料庫 (SQL Server)

Step 3. 安裝 iEN 相關系統

Step 4. 編譯與建置 (iENPortal)

![Image](iENPortal/environment.png)

### 編譯 iENPortal

理論上按 Run on Server 即會開啟 Tomcat 並執行 iENPortal，下列為 Tomcat arguments 的參數設定：

`-Xms212m -Xmx804m -XX:PermSize=250M -XX:MaxPermSize=356m`

若有因為其他 BUG 導致 Tomcat 不能執行，建議手動編譯及執行 iENPortal

Step 1. 在 iENPortal 的資料夾中，執行 ant 指令，開始編譯

![Image](iENPortal/ant.png)
 
Step 2. 於 iENPortal/ant/war 的資料夾，產生 portal.war 檔案
 
![Image](iENPortal/war.png)

### 執行 iENPortal

1. 將 portal.war 檔案，複製在 iENCentre\apache-tomcat-7\webapps\portal 資料夾中

2. 將 portal.war 檔案解壓縮至此資料夾中

3. 執行 start.bat 即可開啟 iENCentre 服務
 
![Image](iENPortal/start.png)
 
P.S. 執行 start.bat 後會跳出一視窗，若看到紅框表示 iENCentre 執行完畢，可輸入 http://127.0.0.1:8080/portal，確認 iEN 是否有成功執行

![Image](iENPortal/run.png)

## iENPortal 程式說明
