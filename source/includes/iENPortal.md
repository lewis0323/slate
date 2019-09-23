# iENPortal

說明 iENPortal 環境建置，以及相關程式 function 用途

這部分會針對派工流程做說明

## 環境建置與編譯

1.安裝 Java SDK (jdk-7u80-windows-x64)，設定環境變數

* JAVA_HOME: 系統安裝 Java 的 JDK 路徑

* JRE_HOME: 系統安裝 Java 的 JRE 路徑

2.安裝資料庫 (SQL Server)

3.安裝 iEN 相關系統

4.編譯與建置 (iENPortal)

![Image](iENPortal/environment.png)

### iENPortal Database

安裝完 SQL Server ，得先對 iENPortal Database 環境做設定，才能將 iENPortal 資料寫入 SQL DB

位置: iENPortal\WebContent\WEB-INF\portal.properties

更改為對應的 SQL DB 設定，例如: `database.user` 、 `database.password`

### 編譯 iENPortal

理論上按 Run on Server 即會開啟 Tomcat 並執行 iENPortal，下列為 Tomcat arguments 的參數設定：

`-Xms212m -Xmx804m -XX:PermSize=250M -XX:MaxPermSize=356m`

若有因為其他 BUG 導致 Tomcat 不能執行，建議手動編譯及執行 iENPortal

1.在 iENPortal 的資料夾中，執行 ant 指令，開始編譯

![Image](iENPortal/ant.png)
 
2.於 iENPortal/ant/war 的資料夾，產生 portal.war 檔案
 
![Image](iENPortal/war.png)

### 執行 iENPortal

1. 將 portal.war 檔案，複製在 iENCentre\apache-tomcat-7\webapps\portal 資料夾中

2. 將 portal.war 檔案解壓縮至此資料夾中

3. 執行 start.bat 即可開啟 iENCentre 服務
 
![Image](iENPortal/start.png)
 
P.S. 執行 start.bat 後會跳出一視窗，若看到紅框表示 iENCentre 執行完畢，可輸入 http://127.0.0.1:8080/portal，確認 iEN 是否有成功執行

![Image](iENPortal/run.png)

## iENPortal 基本說明

說明較為常用修改的地方，基本上 iENPortal 很廣泛，有興趣可一個個翻來看

這邊僅針對我了解部分作說明，主要描述共用環境設定

### i18n (文字設定)

此為存放 iENPortal 頁面的文字資訊，在對應的 properties 可以做名稱定義，中文部分皆須用 <font color=blue>UTF-8</font> 來顯示

位置: iENPortal\WebContent\WEB-INF\i18n

以太陽能平台來說，相關名稱定義均在 `solar_zh_TW.properties` 檔案中

![Image](iENPortal/i18n.png) 

透過此方式，日後若要做文字上的變更，就不用每個頁面都做修改，僅修改 i18n 設定檔，均可套用在各頁面上

下圖紅框，表示為套用 i18n 設定，舉例: <font color=blue>I18N.siteInfoSetting.CHTOSS</font>，網頁呈現顯示 <font color=blue>CHT 定期派工</font>

![Image](iENPortal/name.png) 

### delivery (信件範本)

此為存放 iENPortal Email 報表寄送功能範本，以路燈派工來說，透過 iENPortal 寄送的範本為此資料夾的 LedErrorMail_zh_TW 檔案

修改此檔案內容，即可修改派工 Email 通知信的內容，細節可以 Google `VelocityEngine` 查看相關用法

位置: iENPortal\WebContent\WEB-INF\delivery

![Image](iENPortal/delivery.png) 

### Struts (MVC)

這邊我理解是 MVC 架構，即一個 Java class 會對應到 Java Server Page

Java 會與 JSP 作為相對應，例如空品派工，有個 AirdeviceErrorSubmit.java，那麼會有 AirdeviceErrorSubmit.jsp

JSP 位置: iENPortal\WebContent，如果跟太陽能平台有關的話，則放在這路徑下的 solar 資料夾中

Struts 位置: iENPortal\struts-config，會在此檔案中做顯示、動作間的對應關係

以空品派工為例，actionName 為動作名稱，即這個 action 是做什麼事情

若這個 action 要執行 class 中某個 method 的話，則要帶 method 參數來表示執行哪個 method

最後執行成功，有個 result name，表示會轉到哪個 JSP 來作呈現畫面

可以在 JSP 呼叫 actionName 來執行 Java 中的程式 (即前端跟後端互相配合的概念)

![Image](iENPortal/air.png)

### 查看 iENPortal 原始碼位置

在 iEN 平台上點選相關功能管理，例如控制伺服器管理，那麼可以看到IP: `https://ien.com.tw/portal/system/controllerServerManage.action`

可以知道控制伺服器程式寫在 system\controllerServerManage 這個 action，所有 action 程式碼都放在 portal 底下的 action 中

因此，控制伺服器管理程式位置: iENPortal\src\com\chttl\ienc\portal\action\system\ControllerServerManageAction.java

![Image](iENPortal/source.png)

## iENPortal 派工

iEN 派工均都用 `Activiti Framework` ，可以參考 <a href="https://www.activiti.org/userguide/" target="_blank">Activiti Guide</a>

以下都以我理解的部分作說明，描述大概方向，有個初步認識

因此，實際上可能要自行 trace code 、 Google 相關資料，才會比較完整了解

### Workflow (Activiti)

以 Activiti 架構來包裝派工流程，從 iENPortal\src\com\chttl\ienc\portal\workflow 可以看到 `CHTDispatchingProcess.zip` 這個檔案

這個檔案是以 BPMN 來定義，即使用 XML 格式表達派工流程

從下圖可以看到相關派工定義，有一級維護、二級維護，可利用圖形化介面來生成，參考資料: <a href="https://www.activiti.org/userguide/#activitiDesigner" target="_blank">Activiti Designer editor</a>

而 Workflow 相關資料會存放在 Database 中的 ienc_workflow 資料庫中

![Image](iENPortal/bpmn.png)

### CHTDispatchingService (太陽能派工)

從 workflow\service 可看到 `CHTDispatchingService` 及 `CHTDispatchingServiceImpl` 檔案，分別表示為 Interface 跟 Implementation 

* 產生派工單

流程: createTicket -> startProcess

當從平台派工的時候，會將相關參數帶進來，例如: 客戶ID、事件名稱、回報人等資訊，並設定派工維護群組，這些資料都會被存放在 `properties`

可以從 `propertiesSetToTicket` function 看 ticket 有哪些 properties 欄位

![Image](iENPortal/create.png)

接著流程會跑到 startProcess，這個步驟會產生 taskId 後，發出派工單

簡單說明，會產生 processInstance 後，建立一個 task，將產生的 taskId 存放回去 properties，後續就可以透過 `TaskService` 來查詢派工單

```java
	RuntimeService runtimeService = ServiceConstants.getBean("runtimeService", RuntimeService.class);
	ProcessInstance p = runtimeService.startProcessInstanceByKey(CHTDispatchingDef.NAME, properties);	
			
	// add necessary properties
	HistoryService historyService = ServiceConstants.getBean("historyService", HistoryService.class);
	HistoricProcessInstance hpInst = historyService.createHistoricProcessInstanceQuery().processInstanceId(p.getId()).singleResult();
	TaskService taskService = ServiceConstants.getBean("taskService", TaskService.class);	
	Task task = taskService.createTaskQuery().processInstanceId(p.getId()).singleResult();
	
	runtimeService.setVariables(task.getProcessInstanceId(), properties);
```

如圖紅框所示，產生一個 process，並產生 taskId，作為 Database 查詢所用，後續可以拿這個 taskId，撈相關派工單資訊出來

![Image](iENPortal/startProcess.png)

