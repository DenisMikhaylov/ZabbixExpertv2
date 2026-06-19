---
Практическая работа:
    Название: 'Пользовательские проверки'
    Действие: 'Модуль второй'
    Задача: ' Кастомные проверки'
---
# **Расширение сбора информации**

В данной практической работе мы добавим расширенный сбор данных.

Этапы создания стенда:

- Добавление расширенных проверок проверок


Имена пользователей и пароли:
```
Для denian
login: sa 
password: 111

login: root 
password: 111
```
```
Для Windows
login: Administrator 
password: Pa$$w0rd
```
### **Практическая работа**

### **Задача 1: Внешние проверки с использованием ExternalScripts**

1. Подключиться по SSH к zabbix Server

2. Получить текущий путь для внешних скриптов

```
zabbix_server --help | grep ExternalScripts
```

3. Настройка внешних скриптов

```
nano /etc/zabbix/zabbix_server.conf
```
4. Изменить следующе строки
```
...
Timeout=30
...
ExternalScripts=/etc/zabbix/externalscripts
...
```
4.1 Перезапустить zabbix server
```
systemctl restart zabbix-server.service
```
5. Создание директории для внешних скриптов

```
mkdir /etc/zabbix/externalscripts
```
6. Создание первого скрипта 
Скрипт для получение среднего значение пинга
```
nano /etc/zabbix/externalscripts/ping_avg.sh
```
```
#!/bin/sh
ping -c"$1" "$2" | tail -n1 | cut -d'/' -f5
```
6.1. Добавление прав на скрипт
```
chmod +x /etc/zabbix/externalscripts/ping_avg.sh
```
6.2. Проверка работы скрипта
```
/etc/zabbix/externalscripts/ping_avg.sh 3 ya.ru
```

6.3. Настройка приминения первого скрипта
```
Data collection->Hosts->ya.ru или google.com
  Items
    Name: Ping AVG
    Type: External Check
    Key: ping_avg.sh[3,"{HOST.CONN}"]
    Type of information: Numeric (float)
    Units: ms
```

7. Создание второго скрипта 
Сервис speedtest

7.1. Установка speedtest на Server
```
apt install speedtest-cli
```
7.2. Проверка работы приложения
```
time speedtest-cli

speedtest-cli --csv-header

speedtest-cli --csv
```

7.3. Настройка внешнего скрипта
```
nano /etc/zabbix/externalscripts/speedtest.sh
```
```
#!/bin/sh

if [ "x$1" = xupload ]
then
        A="--no-download"
        F=8
elif [ "x$1" = xdownload ]
then
        A="--no-upload"
        F=7
else
        exit 1
fi

speedtest-cli --csv $A | cut -d',' -f $F
```
7.4. Выдача прав на скрипт
```
chmod +x /etc/zabbix/externalscripts/speedtest.sh
```
7.5. Проверка работы
```
/etc/zabbix/externalscripts/speedtest.sh upload
```

```
/etc/zabbix/externalscripts/speedtest.sh download
```

7.6. Настройка мониторинга
```
Data collection->Hosts->Zabbix Server
  Items
    Name: speedtest download
    Type: External Check
    Key: speedtest.sh[download]
    Type of information: Numeric (float)
    Update interval: 30m

...
Data collection->Hosts->ZAbbix server
  Items
    Name: speedtest upload
    Type: External Check
    Key: speedtest.sh[upload]
    Type of information: Numeric (float)
    Update interval: 30m
 
...
```
8. Создание третьего скрипта

Использование утилиты nmap

8.1. Установить на сервер
```
apt install nmap
```
8.2. Создание скрипта
```
nano /etc/zabbix/externalscripts/detect_host_nmap.sh
```
```
#!/bin/sh
sudo /usr/bin/nmap -O $1 | grep -v 'Starting Nmap\|Host is up\|Nmap done'
```
8.3. Выдача прав на скрипт
```
chmod +x /etc/zabbix/externalscripts/detect_host_nmap.sh
```
8.4. Настройка сбора данных
```
Data collection->Hosts->Gate
  Items
    Name: Detect host operating system by nmap
    Type: External Check
    Key: detect_host_nmap.sh["{HOST.CONN}"]
    Type of information: Text
```

9. Создание проверок по средствам trapper

9.1 Переключиться в веб интерфейс Zabbix

9.2. Создание элемента
```
Data collection->Hosts->server->Items
  Name: my item
    Type: Zabbix trapper
    Key:  my.item
    Allowed hosts: 127.0.0.1, 192.168.10.0/24
```
9.3. Подключиться к Server по SSH

9.4. Установка Zabbix sender на Server

```
apt install zabbix-sender
```
9.5. Отправка данных на Item
```
zabbix_sender -z 192.168.10.10 -p 10051 -s server -k my.item -o 1
```

9.6. Перенастройка speedtest на Trapper
```
Data collection->Hosts->server->Items
  Name: speedtest download trap
    Type: Zabbix trapper
    Key:  speedtest.download
    Type of information: Numeric (float) или Numeric (unsigned)
    Allowed hosts: 127.0.0.1
...
  Name: speedtest upload trap
    Type: Zabbix trapper
    Key:  speedtest.upload
    Type of information: Numeric (float) или Numeric (unsigned)
    Allowed hosts: 127.0.0.1
...

```
9.7. Создание скрипта для Trapper

```
nano /root/speedtest.sh
```

```
#!/bin/sh

### speedtest-cli ### result bits/s
MY_RES=`speedtest-cli --csv --secure`
MY_DOWNLOAD=`echo $MY_RES | cut -d',' -f7`
MY_UPLOAD=`echo $MY_RES | cut -d',' -f8`

### speedtest ### result Bytes/s (use preprocess Custom multiplier)
#MY_RES=`speedtest -f csv`
#MY_DOWNLOAD=`echo $MY_RES | cut -d',' -f6`
#Y_UPLOAD=`echo $MY_RES | cut -d',' -f7`

zabbix_sender -z 127.0.0.1 -p 10051 -s Server -k speedtest.download -o $MY_DOWNLOAD
zabbix_sender -z 127.0.0.1 -p 10051 -s Server -k speedtest.upload -o $MY_UPLOAD
```
9.8. Выдача прав на скрипт
```
chmod +x /root/speedtest.sh
```

9.9. Добавлание запуска скрипта в расписание
```
crontab -l
...
X * * * * /root/speedtest.sh >/dev/null 2>&1
```

### **10. Создание проверок по средствам UserParameter**

10.1. Переключаемся на gate

10.2. Настройка файла конфигурации для Zabbix Agent

```
nano /etc/zabbix/zabbix_agentd.d/my.linux.disk.discovery.conf
```
10.3. Добавить строку в файл конфигурации
```
UserParameter=my.disks.discovery,/bin/lsblk -dJ | /bin/sed -e 's/blockdevices/data/' -e 's/name/{#NAME}/g' -e 's/type/{#TYPE}/g'
```

10.4. Gереключаемся на Server
10.5. Проверяем работу нового ключа

```
zabbix_get -s 192.168.10.1 -k my.disks.discovery 
```
10.6. Установка дополнително ПО для удобсва чтения

```
apt install jq
```
Повторная проверка 
```
zabbix_get -s 192.168.10.1 -k my.disks.discovery | jq
```

11. Настройка монитринга DHCP на Gate

11.1. Переключиться на gate

11.2. Установка средст мониторинга 
```
apt install dhcpd-pools
```

11.3. Получение информации о работе DHCP 
```
dhcpd-pools -l /var/lib/dhcp/dhcpd.leases -c /etc/dhcp/dhcpd.conf
```

11.4. Создание скрипта для мониторинга DHCP

```
nano /usr/local/bin/dhcp_stat.sh
```
```
#!/bin/sh

CMD='/usr/bin/dhcpd-pools -l /var/lib/dhcp/dhcpd.leases -c /etc/dhcp/dhcpd.conf -f c | grep 192.168.'
MAX=`eval $CMD | cut -d'"' -f8`
CUR=`eval $CMD | cut -d'"' -f10`

eval RES=\$$1

echo $RES
```
11.5. Выдача прав на скрипт
```
chmod +x /usr/local/bin/dhcp_stat.sh
```
11.6. Проверка отрабатывания скрипта
```
/usr/local/bin/dhcp_stat.sh MAX
```
```
/usr/local/bin/dhcp_stat.sh CUR
```
11.7. Настройка Zabbix agent для сборки статистики по DHCP

```
nano /etc/zabbix/zabbix_agentd.d/dhcp_stat.conf
```

```
UserParameter=dhcp.stat[*],/usr/local/bin/dhcp_stat.sh $1
```

11.8. Переключаемся на server и проверяем возврат информации
```
zabbix_get -s 192.168.10.1 -k dhcp.stat[CUR]
```
```
zabbix_get -s 192.168.10.1 -k dhcp.stat[MAX]
```


12. Настройка Вычисляемых элементов

12.1. Открыть веб консоль Zabbix

12.2. Настройка мониторинга DHCP 
```
Data collection->Hosts->gate->Items
  Name: DHCP stat CUR
    Type: Zabbix agent
    Key: dhcp.stat[CUR]

  Name: DHCP stat MAX
    Type: Zabbix agent
    Key: dhcp.stat[MAX]
```
12.3. Добавление вычисляемого элемента
```    
  Name: DHCP stat CUR MAX percent
    Type: Calculated
    Key:  DHCP.stat.CUR.MAX.percent
    Formula: last(//dhcp.stat[CUR])/last(//dhcp.stat[MAX])*100
```


13. Создание зависимых элементов

13.1. Подключиться на debian по SSH

13.2. Установка веб сервера
```
apt install nginx
```
13.3. Настрйка прав для файла лога
```
chmod o+r /var/log/nginx/access.log
```

13.4. Открыть веб консоль Zabbix

13.5. Создать новый Item 
```
Data collection->Host-> debian

  Items
    Name: HTTP Access Log
    Type: agent zabbix active
    Key: log[/var/log/nginx/access.log,"^.*",,,skip,\0,,,]
    Type of Information:	Log
```

13.6. Создание дочерних элементов
```
  Items
    Name: Status Code
    Type: Dependent Item
    Key: HTTPStatusCode
    Type of Information:	Numeric (unsigned)
    Master item: HTTP Access Log
Preprocessing:
Regular Expression:
^(\S+) (\S+) (\S+) \[([\w:\/]+\s[+\-]\d{4})\] \"(\S+)\s?(\S+)?\s?(\S+)?\" (\d{3}|-) (\d+|-)\s?\"?([^\"]*)\"?\s?\"?([^\"]*)\"
Output: \8
Type : Numeric (unsigned)


  Items
    Name: Path
    Type: Dependent Item
    Key: HTTPPath
    Type of Information:	text
    Master item: HTTP Access Log
Preprocessing:
Regular Expression:
^(\S+) (\S+) (\S+) \[([\w:\/]+\s[+\-]\d{4})\] \"(\S+)\s?(\S+)?\s?(\S+)?\" (\d{3}|-) (\d+|-)\s?\"?([^\"]*)\"?\s?\"?([^\"]*)\"
Output: \6
Type : text
```

13.7. Открыть сайт в браузере http://192.168.10.1 или Http://<внешний ip адрес>

13.8. Првоерить полученные данные.

