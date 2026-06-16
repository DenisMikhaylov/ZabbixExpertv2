Создаём папку 
/etc/zabbix/ssl для хранения ключей, в ней размещаем файлы ключа и приватного сертификата:
```
mkdir /etc/zabbix/ssl
cd /etc/zabbix/ssl

```

Правим конфигурационный файл /etc/zabbix/nginx.conf:
```
nano /etc/zabbix/nginx.conf
```

```
server {
```
Добавляем строки
```
        listen          443 ssl;
        charset utf-8;

        ssl_certificate     /var/CA/www.crt;
        ssl_certificate_key /var/CA/www.key;

        if ($scheme != "https") {
            return 301 https://$host$request_uri;
        }

```
