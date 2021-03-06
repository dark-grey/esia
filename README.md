> Изменения в форке:
> - обновлен `psr/http-client` до версии `1.0`
> - добавлен метод `setRedirectUrl` в `Esia\Config`

# Единая система идентификации и аутентификации (ЕСИА) OpenId 

[![Build Status](https://travis-ci.org/fr05t1k/esia.svg?branch=master)](https://travis-ci.org/fr05t1k/esia)

# Описание
Компонент для авторизации на портале "Госуслуги".

# Внимание!
Получив токен вы можете выполнять любые API запросы. Библиотека не поддерживает все существующие методы в API, а предоставляет только самые базовые. Основная цель библиотеки - получение токена.

# Установка

Добавьте в composer.json:
```
"repositories": [
  {
    "type": "vcs",
    "url": "https://github.com/dark-grey/esia"
  }
],
```
```
"dark-grey/esia" : "^2.0"
```
или
```
composer require dark-grey/esia
```

# Как использовать 

Пример получения ссылки для авторизации
```php
<?php 
$config = new \Esia\Config([
  'clientId' => 'INSP03211',
  'redirectUrl' => 'http://my-site.com/response.php',
  'portalUrl' => 'https://esia-portal1.test.gosuslugi.ru/',
  'scope' => ['fullname', 'birthdate'],
]);
$esia = new \Esia\OpenId($config);
$esia->setSigner(new \Esia\Signer\SignerPKCS7(
    'my-site.com.pem',
    'my-site.com.pem',
    'password',
    '/tmp'
));
?>

<a href="<?=$esia->buildUrl()?>">Войти через портал госуслуги</a>
```

После редиректа на ваш `redirectUrl` вы получите в `$_GET['code']` код для получения токена

Пример получения токена и информации о пользователе

```

$esia = new \Esia\OpenId($config);

// Вы можете использовать токен в дальнейшем вместе с oid 
$token = $esia->getToken($_GET['code']);

$personInfo = $esia->getPersonInfo();
$addressInfo = $esia->getAddressInfo();
$contactInfo = $esia->getContactInfo();
$documentInfo = $esia->getDocInfo();

```
# Конфиг

`clientId` - ID вашего приложения.

`redirectUrl` - URL куда будет перенаправлен ответ с кодом.

`portalUrl` - по умолчанию: `https://esia-portal1.test.gosuslugi.ru/`. Домен портала для авторизация (только домен).

`codeUrlPath` - по умолчанию: `aas/oauth2/ac`. URL для получения кода.

`tokenUrlPath` - по умолчанию: `aas/oauth2/te`. URL для получение токена.

`scope` - по умолчанию: `fullname birthdate gender email mobile id_doc snils inn`. Запрашиваемые права у пользователя.

`privateKeyPath` - путь до приватного ключа.

`privateKeyPassword` - пароль от приватного ключа.

`certPath` - путь до сертификата.

`tmpPath` - путь до дериктории где будет проходить подпись (должна быть доступна для записи).

# Токен и oid

Токен - jwt токен которые вы получаете от ЕСИА для дальнейшего взаимодействия

oid - уникальный идентификатор владельца токена

## Как получить oid?
Если 2 способа:
1. oid содержится в jwt токене, расшифровав его
2. После получения токена oid сохраняется в config и получить можно так 
```php
$esia->getConfig()->getOid();
```

## Переиспользование Токена

Дополнительно укажите токен и идентификатор в конфиге
```php
$config->setToken($jwt);
$config->setOid($oid);
```
