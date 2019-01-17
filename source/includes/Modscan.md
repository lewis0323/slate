# Modscan

Modscan用來測試設備通訊是否正常，即檢驗通訊一來一往的封包是否正常、無遺失

若有不穩定的情況，可以將**response timeout**時間拉長，確認穩定性是否提高

以下說明標準Modbus Protocol跟太陽能逆變器相關Protocol的modscan使用方式

## Modbus Protocol

此方式僅用在標準Modbus protocol設備上

If you have any interests in Modbus protocol, maybe you can take a look. [Modbus Protocol](https://www.modbustools.com/modbus.html)

`cd /opt/ienbox/core/bin`

`java -Djava.library.path=../lib -jar modtool.jar -m rtu -b 9600 -d 8 -s 1 -p none -a 1 -r 100 -c 1 -t 4 -i /dev/ttyS1`

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

<aside class="notice">
You can use <code>modtool.jar -h</code> for help.
</aside>

## Solar Inverter Protocol

此方式用在太陽能逆變器，支援以下廠牌: 茂迪、固德威、KACO、Kstar

* 透過FTP/WinSCP方式將檔案放至iENBox中，預設位置為/media/sd/home/cht

* cp /media/sd/home/cht/modInverter.jar /opt/ienbox/core/bin

參考: [modInverter使用說明](http://bit.ly/2QN8DAq)

`java -Djava.library.path=../lib -jar modInverter.jar -m kaco -a 1 -i /dev/ttyS1`

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