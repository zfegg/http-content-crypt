Http Content Signature/Crypt
============================

HTTP content crypt/signature for PSR7 middleware

Installation
------------

Install via composer:

~~~bash
# composer require zfegg/http-content-crypt
~~~

Usage
-----

### ContentCryptMiddleware

Content crypt using RSA+AES.

#### HTTP stream:

~~~
POST /action HTTP/1.1
Host: localhost
Content-Type: application/json
Accept: application/json
X-Content-Encoding: rsaaes, base64
X-Crypto-Key: keyid=1; data=`Urlencode(BASE64.encode(RSA.encode(AesKey)))`

`BASE64.encode(AES.encode('{"test":"test content"}'));`

HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: n
X-Content-Encoding: rsaaes, base64

`BASE64.decode(AES.decode('{"test":"test response content"}'));`
~~~

#### Slim example:

~~~php

use Psr\Http\Message\ServerRequestInterface;

$app = new \Slim\App($settings);

$container = $app->getContainer();
$container[ContentCryptMiddleware::class] = function () {
    $middleware = new ContentCryptMiddleware();

    $rsa = Rsa::factory([
        'public_key' => '',
        'private_key' => '',
        'binary_output' => false,
    ]);

    $middleware->setFetchRsaCallback(function ($keyId, ServerRequestInterface $request) use ($rsa) {
        return $rsa;
    });
    return $middleware;
};

$app->post('/test', function (\Psr\Http\Message\ServerRequestInterface $request, \Slim\Http\Response $response) {
    $rawBody = $request->getBody();
    return $request->write($rawBody);
})->add(ContentCryptMiddleware::class);

$app->run();
~~~


### ContentSignatureMiddleware

Content signature verification using hash HMAC.


在`POST`, `PUT`, `PATCH` 请求中, 对HTTP内容进行 HMAC-HASH 方式签名计算.

内容签名主要是用于校验传输内容的合法性, 避免接口泄漏, 被恶意使用.

#### HTTP stream:

~~~
POST /action HTTP/1.1
Host: localhost
Content-Type: application/json
Accept: application/json
Content-Signature: keyid=1; value=(hash_hex); alg=(md5|sha1|...);

payload

HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: n
Content-Signature: keyid=1; value=(hash_hex); alg=(md5|sha1|...);

payload
~~~

#### Slim example:

~~~php
use Psr\Http\Message\ServerRequestInterface;

$app = new \Slim\App($settings);

$container = $app->getContainer();
$container[ContentSignatureMiddleware::class] = function () {
    $middleware = new ContentSignatureMiddleware();
    $middleware->setFetchRsaCallback(function ($keyId, ServerRequestInterface $request) {
        return "123456";
    });
    return $middleware;
};

$app->post('/test', function (ServerRequestInterface $request, $response) {
    $rawBody = $request->getBody();
    return $request->write($rawBody);
})->add(ContentSignatureMiddleware::class);

$app->run();
~~~
