Статьи


[Настройка Zabbix SNMP traps](http://va0816.blogspot.com/2013/06/zabbix-snmp-traps.html)

[Zabbix Documentation SNMP трапы](https://www.zabbix.com/documentation/3.0/ru/manual/config/items/itemtypes/snmptrap)


Установка компонентов на сервер 

```
# apt install snmptt

# systemctl disable snmptt

# systemctl stop snmptt
```
Настройка файла конфигурации

```
# nano /etc/snmp/snmptt.conf
```
```
EVENT general .* "General event" Normal
FORMAT ZBXTRAP $aA $ar
date_time_format = %H:%M:%S %Y/%m/%d
log_file = /tmp/zabbix_traps.tmp
```
Настройка службы SNMP trap
```
# nano /etc/snmp/snmptrapd.conf
```
```
traphandle default snmptt

authCommunity execute writetrap
```
Настройка службы
```
# nano /lib/systemd/system/snmptrapd.service
```
```
ExecStart=/usr/sbin/snmptrapd -Lsd -f -On
```
Перезапуск служб
```
# systemctl daemon-reload
# service snmptrapd restart
```
Настройка Zabbix server
```
# nano /etc/zabbix/zabbix_server.conf
```
```
StartSNMPTrapper=1
SNMPTrapperFile=/tmp/zabbix_traps.tmp
```
```
systemctl restart zabbix-server
```
Настройка трап Item
```
Host: Zabbix
...
  Items
    Name: SNMP Trap test
    Type: SNMP Trap
    Key: snmptrap.fallback
    Type of information: text
```
Создание тестового трап
```
systemctl start snmptrapd
snmptrap -v 1 -c public 127.0.0.1 '.1.3.6.1.6.3.1.1.5.4' '0.0.0.0' 6 33 '55' .1.3.6.1.6.3.1.1.5.4 s "eth0"
```
```
tail /tmp/zabbix_traps.tmp
```

Проверка создания строки
```
Monitoring > Latest data > SNMP Trap test
```
