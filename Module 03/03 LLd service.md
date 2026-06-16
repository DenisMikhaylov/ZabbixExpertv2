НАстройка мониторинга сервисов в Linux


Создание скрипта 
```
nano service_discovery.py
```
```
# Copyright 2020,2021,2022 Sean Bradley https://sbcode.net/zabbix/
import sys
import os
import json

SERVICES = os.popen('systemctl list-unit-files').read()

ILLEGAL_CHARS = ["\\", "'", "\"", "`", "*", "?", "[", "]", "{", "}", "~", "$", "!", "&", ";", "(", ")", "<", ">", "|", "#", "@", "0x0a"]

DISCOVERY_LIST = []

LINES = SERVICES.splitlines()
for l in LINES:
    service_unit = l.split(".service")
    if len(service_unit) == 2:
        if not [ele for ele in ILLEGAL_CHARS if(ele in service_unit[0])]:
            status = service_unit[1].split()
            if len(status) == 1:
                if (status[0] == "enabled" or status[0] == "generated"):
                    DISCOVERY_LIST.append({"{#SERVICE}": service_unit[0]})
            else:
                if (status[0] == "enabled" or status[0] == "generated") and status[1] == "enabled":
                    DISCOVERY_LIST.append({"{#SERVICE}": service_unit[0]})

JSON = json.dumps(DISCOVERY_LIST)

print(JSON)
```
Проверка запуска
```
python3 service_discovery.py
```

Настройка Zabbix Agent
```
nano /etc/zabbix/zabbix_agentd.d/linuxservice.conf
```
```
UserParameter=service.discovery,python3 /etc/zabbix/service_discovery.py
UserParameter=service.isactive[*],systemctl is-active --quiet '$1' && echo 1 || echo 0
UserParameter=service.activatedtime[*],systemctl show '$1' --property=ActiveEnterTimestampMonotonic | cut -d= -f2

```

Перезапуск службы
```
service zabbix-agent restart
service zabbix-agent status
```
Проверка Работы
```
zabbix_agentd -t service.discovery
```
Настройка Zabbix
```
Templates->Create template
  Template name: Linux service
  Groups: Templates/Soft hardware

  Discovery rules->
    Name: Service Discovery
    Key: service.discovery
    Type:	Zabbix Agent 
    Item prototypes->
      Name: Service Active : {#SERVICE}
      Key: service.isactive["{#SERVICE}"]
      type: Zabbix Agent 
      Type of information: Numeric (unsigned)

      Name: Service Activated Time : {#SERVICE}
      Key: service.activatedtime["{#SERVICE}"]
      Type:	Zabbix Agent
      Type of information: Numeric (unsigned)

      Trigger prototypes->
      Name: Service Not Active : {#SERVICE}
      Expression:
      last(/Linux Services by Sean/service.isactive["{#SERVICE}"])=0
      Severity: Disaster (Optional)

      Name: Service Restarted : {#SERVICE}
      Expression:
      change(/Linux Services by Sean/service.activatedtime["{#SERVICE}"])<>0
      Severity: Warning (Optional)
```
Назначить шалон на Host
