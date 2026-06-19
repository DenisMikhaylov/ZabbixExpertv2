Материалы
URL Service and SLA create
https://www.youtube.com/watch?v=5N5uK4vJsAY


Добавляем теги к хостам

```
Host: Zabbix server
  Tags
  Name: Monitoring
  Value: Zabbix

Host: Red OS
  Tags
  Name: Monitoring
  Value: RedOS

```
Создание Service 

```
Services-> services
в верхнем правом углу нажать Edit
Create service
  Name: Monitoring
  Tags:
    Name: Zabbix
    Value: monitoring
  Advanced configuration: check
  Additional rules:
    add...
      Set status to High
      Contion: if at least n% of child servicesa have Status status or above
      N 50%
      Status: Warning
      Add
  Add
```

Зайти в Monitoring

```
Create service
Name Server host
  Tags:
    Name: zabbix
    Value: host
  Additional rules:
    add...
      Set status to Disaster
      Contion: if at least n child servicesa have Status status or above
      N 2
      Status: Warning
      Add
Add
```
Зайти в Server host

```
Create service
Name: Zabbix
  Tags:
    Name: zabbix host
    Value: zabbix
Problem Tags
  Name: Monitoring
  Operation: equals
  Value: Zabbix
Add
```
```
Create service
Name: RedOS
  Tags:
    Name: RedOS host
    Value: RedOS
Problem Tags
  Name: Monitoring
  Operation: equals
  Value: RedOS
Add
```

Создание SLA

```
Services-> SLA

Create SLA
  Name: Monitoring
  SLO: 90%
  Reporting period: Daily
  Schedule: Custom
    9:00-19:00
  Service tags
    name: zabbix
    Value: monitoring

    name: zabbix
    Value: host


```

```
Services-> SLA

Create SLA
  Name: Monitoring
  SLO: 95%
  Reporting period: weeks
  Schedule: Custom
    9:00-19:00
  Service tags
    name: zabbix
    Value: monitoring

    name: zabbix
    Value: host


```

Просмотр отчета
```
Services-> SLA report
Выбрать SLA 
```


