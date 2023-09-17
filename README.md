# self_signed_certificates

# создание центра сертификации, где crt - сертификат, а key - приватный ключ

```
openssl req -x509 -sha256 -days 3653 -newkey rsa:2048 -keyout root_ca.key -out root_ca.crt 


openssl req `# создать новый сертификат/ключ 
  -subj 'CN=ROOT CA' `# FQDN субъекта
  -x509 -sha256 `# создать сертификат, а не запрос на подпись сертификата` 
  -days 3653 `# на 10 лет` 
  -newkey rsa:2048 `# сгенерировать новый ключ размером 2048 бит с алгоритмом RSA` 
  -keyout root_ca.key `# файл для сохранения ключа` 
  -out root_ca.crt # файл для сохранения сертификата
```

### создаем сертификат для приложения

```
openssl genrsa -out localhost.key 2048

openssl genrsa `# сгенерировать ключ с алгоритмом RSA` 
  -out localhost.key `# файл для сохранения ключа` 
  2048 # размер ключа в битах
```

### подпись сертификата ключа

```
openssl req -key localhost.key -new -out localhost.csr

openssl req `# запрос подписи сертификата` 
  -new `# новый запрос` 
  -subj 'CN=localhost' `# FQDN субъекта` 
  -key localhost.key `# файл ключа` 
  -out localhost.csr `# файл для запроса
```

### создать файл localhost.ext
### подписать

```
openssl x509 -req -CA root_ca.crt -CAkey root_ca.key -in localhost.csr -out localhost.crt -days 365 -CAcreateserial -extfile localhost.ext



openssl x509 -req `# Подпись сертификата` 
  -CA root_ca.crt `# Сертификат CA` 
  -CAkey root_ca.key `# Ключ CA` 
  -in localhost.csr `# Запрос на подпись` 
  -out localhost.crt `# Файл для полученного сертификата` 
  -days 365 `# на 1 год` 
  -CAcreateserial`# сгенерировать идентификатор` 
  -extfile localhost.ext # файл с расширениями
```


### Для удобства создаем цепочку
```
cat localhost.crt root_ca.crt > localhost_full.crt
```

## Чтоб использовать ключи, экспортируем их в кейстор

```
openssl pkcs12 -export -in localhost_full.crt -inkey localhost.key --name localhost -out keystore.p12

openssl pkcs12 
  -export `# экспорт в формате PKCS#12` 
  -in localhost_full.crt `# экспортируемая цепочка сертификатов` 
  -inkey localhost.key `# экспортируемый ключ` 
  -name localhost `# алиас ключа` 
  -out keystore.p12 # целевое хранилище ключей
```

### добавить центр сертификации ROOT_CA в мозиллу
### добавить сертификат в системв

```
sudo cp root_ca.crt /usr/local/share/ca-certificates/

sudo update-ca-certificates
```