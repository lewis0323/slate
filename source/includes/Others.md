# Others

## iENBox WebService 服務不正常運作

iENBox 有機率性服務開啟會失敗，以致於卡在網頁都是 404 狀態，通常重開機能解決

原因為亂數效能關係，先前有針對此問題做改善，但仍有一定機率會發生

透過 `wget` 判斷 iENBox WebService 有無開啟成功，若 WebService 抓取失敗為服務沒有正常開啟，將 iENBox 程序砍掉重啟

```shell
	#!/bin/bash
	while [ TRUE ]
	do
		PID=`cat /opt/ienbox/storage/ienbox.pid`
		### 抓取 iENBox WebService 網頁
		wget http://127.0.0.1:4511
		### 若抓取失敗，則重啟服務
		if [ -f index.html ]; then
			rm index.html
		else
			kill $PID
		fi
		sleep 600
	done
```

## Litebox MQTT 不穩定重開機

為偵測 Litebox 是否已撥號上網，若有連線能力後，透過 `netstat` 查看 MQTT 連線狀態

若 MQTT 連線異常，則將 Litebox 重開機，此問題為老問題，與 Litebox 硬體設備有關

連結: <a href="http://bit.ly/2UP04DS" target="_blank">一鍵執行</a>

```shell
	#!/bin/bash
	while [ TRUE ] 
	do
		ip=`ifconfig | grep eth0 | awk 'NR==1{print $1}'`
		### 確認網路有撥接上
		if [ $ip == "eth0" ]; then
			echo $ip 
			sleep 60
			### 確認 MQTT 連線狀態
			MQTT=`netstat -p | grep 1883 | awk '{print $6}'`
			if [ $MQTT != "ESTABLISHED" ]; then
				reboot
			fi 
		fi
	done
```


