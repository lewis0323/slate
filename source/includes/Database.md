# Database

## 新增 Postgresql 欄位

以 1.24.14-1.26.7 升級為例，如下圖來說明新增欄位流程：

![Image](Database/Postgresql/TL.png)

Step 1. 登入 Postgresql，輸入使用者密碼

![Image](Database/Postgresql/login.png)

Step 2. 尋找 Raw Data/ETL Data 存放位置，會長在 customer 底下

位置為 ienc_system -> Schemas -> ienc_customer_1 

<font color=red>(註: 因為是企業版，故客戶ID 大多為1，理論上要對應到實際客戶ID)</font>

![Image](Database/Postgresql/schema.png)

Step 3. 找到相對應的 Table ，在 Columns 點選右鍵 -> Create -> Columns

填入相關資訊，以建立新欄位，這個例子就是取 <font color=blue>_decorated</font> ，型態設為 <font color=blue>boolean</font>

![Image](Database/Postgresql/create.png)
