# JWT

O JWT é padrão aberto([rfc7519](https://tools.ietf.org/html/rfc7519)) que visa garantir a autenticidade de uma determinada informação.

Ele o token gerado e composto por 3 partes:

## HEADER

Contém as informações sobre o token, algoritmo de hash e tipo do token exemplo:

```php
$header = [
   'alg' => 'HS256',
   'typ' => 'JWT'
];
$header = json_encode($header);
$header = base64_encode($header);
```

## PAYLOAD

É o corpo do token é contém todas as informações que o token carrega com sigo, ele deve ser bem sucinto para que seu trafico na rede não se torne um fardo para aplicação.

```php
$payload = [
   'iss' => 'localhost',
   'name' => 'Diogo',
   'email' => 'diogo.fragabemfica@gmail.com'
];
$payload = json_encode($payload);
$payload = base64_encode($payload);
```

## SIGNATURE

Está é a aparte que garante a autenticidade dos dados uma vez que o hash gerado com base nos dados que esse token carrega mais um chave secreta, de modo que a verificação da validade do token e feita na tentativa de regerear o mesmo token com base nos dados transferidos e na chave secreta. caso qualquer dado do token for alterado o novo hash gerado será diferente do anterior.
Observação o JWT não garante a confidencialidade dos dados uma vez que esses dados são convertidos para base64 então qualquer um que tiver acesso ao token terá acesso a essas informações, então não colocar dados como senhas.

```php
$signature = hash_hmac('sha256',"$header.$payload",'minha-senha',true);
$signature = base64_encode($signature);

echo "$header.$payload.$signature";
```


Agradecimentos a [Diogo Benfica](https://diogobemfica.com.br/intendendo-o-jwt/)