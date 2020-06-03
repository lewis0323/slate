# Modscan

Modscan 是一套用來測試設備通訊是否正常的程式，即檢驗通訊一來一往的封包是否正常、無遺失

若有不穩定的情況，可以將 <font color=blue>response timeout</font> 時間拉長，提高設備通訊穩定性 **(建議 timeout 最大設五秒)**

以下說明標準 Modbus Protocol 跟太陽能逆變器相關 Protocol 的 modscan 使用方式

## Modbus Protocol

此方式僅用在標準 Modbus protocol 設備上

If you have any interests in Modbus protocol, take a look <a href="https://www.modbustools.com/modbus.html" target="_blank">Modbus Protocol</a>.

```shell
	cd /opt/ienbox/core/bin
	java -Djava.library.path=../lib -jar modtool.jar -m rtu -b 9600 -d 8 -s 1 -p none -a 1 -r 100 -c 1 -t 4 -i /dev/ttyS1
```

Parameter | Description
--------- | -----------
-m rtu/tcp | Modbus transport protocol
-a # | Slave address
-r # | Start reference
-c # | Number of values o read
-t # | 0: coil, 1: discrete input, 3:input register, 4:holding register
-b # | Baudrate (e.g. 9600, 19200)
-d # | Databits
-s # | Stopbits
-p # | Parity
-i target | SERIALPORT/HOST
-P # | TCP/UDP port number
-l # |  Poll rate in ms, (1000 is default)
--timeout # | Response timeout in ms, (1000 is default)

執行結果如下圖：

![Image](Modscan/modtool.png)

<aside class="notice">
You can use <code>modtool.jar -h</code> for help.
</aside>

## Solar Inverter Protocol

此方式使用於太陽能逆變器，支援以下廠牌太陽能逆變器： 茂迪、固德威、Kaco、Kstar

* 透過 FTP/WinSCP 方式將檔案放至 iENBox 中，預設位置： /media/sd/home/cht

* cp /media/sd/home/cht/modInverter.jar /opt/ienbox/core/bin

參考： <a href="https://drive.google.com/file/d/1qvH0yF-jqr684uNDh5cukGoRWf2ftSXg/view?usp=sharing" target="_blank">modInverter使用說明</a>

```shell
	### Kaco Inverter Sample (SlaveId = 1)
	java -Djava.library.path=../lib -jar modInverter.jar -m kaco -a 1 -i /dev/ttyS1
	### Motech Inverter Sample (SlaveId = 1 ... 10)
	java -Djava.library.path=../lib -jar modInverter.jar -m motech --ms 1 --me 10 -i /dev/ttyS1
```

Parameter | Description
--------- | -----------
-m motech/goodwe/kaco/kstar | CHT supported Inverter Protocol
-a # | Slave address
-r # | Slave reference
-c # | Number of values o read
-t # | 0: coil, 1: discrete input, 3:input register, 4:holding register
-b # | Baudrate (e.g. 9600, 19200)
-d # | Databits
-s # | Stopbits
-f # | The path of ID/SN for Goodwe
-p # | TCP/UDP port number (502 is default)
-i target | SERIALPORT/HOST
--ms # | The start of slave reference is queried in Motech (1 is default)
--me # | The end of slave reference is queried in Motech (250 is default)
--timeout # | Response timeout in ms, (1000 is default) 

<aside class="notice">
You can use <code>modInverter.jar -h</code> for help.
</aside>
