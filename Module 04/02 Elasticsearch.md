1. Устиановка Elasticsearch
[Инструкция по установке](https://www.elastic.co/guide/en/elasticsearch/reference/current/_installation.html)
2. Настройка
   
```
nano /etc/zabbix/zabbix_server.conf
```
```
### Option: HistoryStorageURL
#       History storage HTTP[S] URL.
#
# Mandatory: no
# Default:
# HistoryStorageURL=
HistoryStorageURL=http://hostnameelastic:9200

### Option: HistoryStorageTypes
#       Comma separated list of value types to be sent to the history storage.
#
# Mandatory: no
# Default:
# HistoryStorageTypes=uint,dbl,str,log,text
HistoryStorageTypes=uint,dbl,str,log,text
```
```
nano /etc/zabbix/web/zabbix.conf.php
```
```
<?php
// Zabbix GUI configuration file.

global $DB, $HISTORY;
$DB['TYPE']     = 'MYSQL';
$DB['SERVER']   = 'zabbixDB';
$DB['PORT']     = '3306';
$DB['DATABASE'] = 'zabbix';
$DB['USER']     = 'zabbix';
$DB['PASSWORD'] = 'zabbix';

// Schema name. Used for IBM DB2 and PostgreSQL.
$DB['SCHEMA'] = '';

$ZBX_SERVER      = 'localhost';
$ZBX_SERVER_PORT = '10051';
$ZBX_SERVER_NAME = '';

$IMAGE_FORMAT_DEFAULT = IMAGE_FORMAT_PNG;

$HISTORY['url']   = [
        'uint' => 'http://hostnameelastic:9200',
        'text' => 'http://hostnameelastic:9200',
        'str' => 'http://hostnameelastic:9200',
        'log' => 'http://hostnameelastic:9200',
        'dbl' => 'http://hostnameelastic:9200'
];
$HISTORY['types'] = ['uint', 'text', 'str', 'log', 'dbl'];

```

Создание индексов
```
curl -X PUT \
 http://localhost:9200/uint \
 -H 'content-type:application/json' \
 -d '{
   "settings" : {
      "index" : {
         "number_of_replicas" : 1,
         "number_of_shards" : 5
      }
   },
   "mappings" : {
      "values" : {
         "properties" : {
            "itemid" : {
               "type" : "long"
            },
            "clock" : {
               "format" : "epoch_second",
               "type" : "date"
            },
            "value" : {
               "type" : "long"
            }
         }
      }
   }
}'
```
```
curl -X PUT \
 http://hostnameelastic:9200/dbl \
 -H 'content-type:application/json' \
 -d '{
   "settings" : {
      "index" : {
         "number_of_replicas" : 1,
         "number_of_shards" : 5
      }
   },
   "mappings" : {
      "values" : {
         "properties" : {
            "itemid" : {
               "type" : "long"
            },
            "clock" : {
               "format" : "epoch_second",
               "type" : "date"
            },
            "value" : {
               "type" : "double"
            }
         }
      }
   }
}'
```
```
curl -X PUT \
 http://your-elasticsearch.here:9200/text \
 -H 'content-type:application/json' \
 -d '{
   "settings": {
      "index": {
         "number_of_replicas": 1,
         "number_of_shards": 5
      }
   },
   "mappings": {
      "properties": {
         "itemid": {
            "type": "long"
         },
         "clock": {
            "format": "epoch_second",
            "type": "date"
         },
         "value": {
            "fields": {
               "analyzed": {
                  "index": true,
                  "type": "text",
                  "analyzer": "standard"
               }
            },
            "index": false,
            "type": "text"
         }
      }
   }
}'
```

Похожий запрос требуется выполнить для создания сопоставлений по значениям истории Символ и Журнал (лог) с соответствующим исправлением типа.

[Импорт метрик ](https://sohabr.net/habr/post/415955/)


