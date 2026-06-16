Настройка активного агента на debian-1
Проверить

```
# zabbix_agentd -p

# zabbix_agentd -p | grep agent.version
```

Настройка авторегистрации систем с агентами, работающими в активном режиме
```
Alert - Actions - Auto registration 
  Name: Add linux clients                                         
  Conditions: Host name contains debian                             
  Action operations: 
    Add to host groups: linux clients                              
    Link to templates: linux by Zabbix agent active                
                     
  Set host inventory mode: Automatic
```

  Настройка агента на активный режим
  ```
#Server=server
ListenIP=0.0.0.0
StartAgents=0
ServerActive=<ip server>
Hostname=CLIENTN
```
