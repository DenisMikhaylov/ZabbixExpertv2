Использование PKI
Сценарий:

развертывание корпоративного CA 

Создание центра сертификации

Настройка атрибутов базы CA в конфигурации ssl
```
# nano /etc/ssl/openssl.cnf
```
```
...
[ CA_default ]
...
dir           = /root/CA

certificate   = /var/CA/ca.crt

crl           = /var/CA/ca.crl

private_key   = $dir/ca.key
...
```
Созданеи структуры PKI
```
cd
mkdir CA
mkdir CA/certs
mkdir CA/newcerts
mkdir CA/crl
touch CA/index.txt
echo "01" > CA/serial
echo "01" > CA/crlnumber
mkdir /var/CA
```
Создание зашифрованного приватного ключа
```
# openssl genrsa -des3 -out CA/ca.key 2048

Generating RSA key, 2048 bits
Enter PEM pass phrase:Pa$$w0rd
Verifying - Enter PEM pass phrase:Pa$$w0rd
```
Настройка атрибутов организации в конфигурации ssl
```bash
nano /etc/ssl/openssl.cnf
```
```
...
[ req_distinguished_name ]
...
countryName_default           = RU
stateOrProvinceName_default   = Moscow region
localityName_default          = Moscow
0.organizationName_default    = cko
organizationalUnitName_default = noc
emailAddress_default          = noc@corp.ru

[ req_attributes ]
...
```
Создание самоподписанного корневого сертификата
```
openssl req -new -x509 -days 3650 -key CA/ca.key -out /var/CA/ca.crt
```
```
Enter pass phrase for ca.key:Pa$$w0rd
...
Common Name (eg, YOUR name) []:corp.local
```

Инициализация списка отозванных сертификатов

```
# openssl ca -gencrl -out /var/CA/ca.crl
```
```
Enter pass phrase for ./CA/ca.key:Pa$$w0rd
```

Создание сертификата сервиса, подписанного CA

Создание приватного ключа сервиса
```
# openssl genrsa -out www.key 2048
# chmod 400 www.key
```
Создание запроса на сертификат

```
# openssl req -new -key www.key -out www.req   #-sha256
```
```
...
Common Name (eg, YOUR name) []:server.corp.local
...
```
```
# openssl req -text -noout -in www.req
```
Подпись запроса на сертификат центром сертификации
```
# openssl ca -days 365 -in www.req -out www.crt
```

