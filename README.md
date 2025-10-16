# Desenvolvendo para desenvolvedores

Uma série de recomendações de boas práticas de desenvolvimento, para auxiliar o entendimento e a manutenção de códigos de projetos de software.

## Inversão de dependências (DIP)

> Em vez de classes específicas (implementações concretas), use interfaces ou classes abstratas para definir contratos e interações entre diferentes partes do sistema. 

Ao invés disso:
```php

/** @var ContainerInterface $container */
$container = null;

$client = $container->get('guzzle');
// ou...
$client = $container->get('GuzzleHttp');
// ou...
$client = $container->get('HttpClient');
```

Faça isto:
```php

/** @var ContainerInterface $container */
$container = null;
$client = $container->get(\Psr\Http\Client\ClientInterface::class);
```

## Semântica

Considerando que o projeto possui configurações definidas pela variável `$config`: 

```php
...
$config["DB"]["host"] = "";
$config["DB"]["username"] = "";
$config["DB"]["password"] = "";
$config["DB"]["name"] = "";

...
```

Ao invés disso:

```php
$config = $container->get('settings'); // nome alterado da referência
```

Faça isso:

```php
$config = $container->get('config'); // nome CORRETO da referência
```

## Transporte de dados

Utilizar classes ao invés de arrays para transportar dados.
Uma classe tem uma estrutura explicita do objeto, enquanto o array requer uma analize mais cuidadosa para conhecer o seu shape e sua dimensão.

...

## Configurações de endpoints para serviços externos

Configure somente a BASE_URL do serviço no environment, ao invés da URL completa do serviço, facilitando o entendimento da leitura do Service quanto ao que ele faz e para onde ele aponta.

Ao invés disso:

```php

// .env
API_BASE_URL=https://www.someapi.com/api/v1/contacts/emails

<php

// Service
class CollectContactEmails
{
  // ...
  public function _construct(private readonly EnvironmentInterface $environment, private readonly HttpClientInterface $httpClient) {}

  public function collect(): ServiceResult
  {
    $httpOptions = [
      //...
    ];

    // ao olhar para URL, dentro do serviço, não dá para saber para onde ela aponta.
    // isso dificulta, por exemplo, uma investicação na documentação da integração,
    // por não conseguirmoos saber qual é o endpoint.
    //
    // Podemos olhar para o .env e pode ser que ele tenha alguma URL configurada num .env.example,
    // mas também pode não haver, o que piora ainda mais a situação, o que deixa o developer sem saber
    // sequer qual é a URL base da API, quem dirá qual recurso está sendo requisitado.
    $contacts = $this->httpClient->get($this->environment->get('API_BASE_URL'));

    return ServiceResult::createFromArray($contacts, $httpOptions);
  }
}

```

Faça isso:

```php

// .env
API_BASE_URL=https://www.someapi.com/api/v1

<php

// Service
class CollectContactEmails
{
  // ...
  public function _construct(private readonly EnvironmentInterface $environment, private readonly HttpClientInterface $httpClient) {}

  public function collect(): ServiceResult
  {
    $httpOptions = [
      //...
    ];

    // Agora, está mais explícitp para o developer que o recurso utilizado é o /contacts/email
    // e que no environment deve ter / ser configurado a API_BASE_URL.
    $url = sprintf('%/contacts/emails', $this->environment->get('API_BASE_URL'));

    $contacts = $this->httpClient->get($url, $httpOptions);

    return ServiceResult::createFromArray($contacts);
  }
}

```

## Feedback claros

Faça com que as implementeções devolvam feedbacks claros, ao invés de feedbacks confusos para os usuários / desenvolvedores.

Considerando um teste de validação de NIF, que pode validar se o NIF está correto e se ele possui 9 dígitos:

```php

/**
 * Validate Portuguese NIF (9 digits)
 * @param string $nif
 * @return bool
 */
protected function isValidNIF(string $nif): bool
{
    // Remove any non-digit characters
    $cleanNif = preg_replace('/\D/', '', $nif);
    
    // Must be exactly 9 digits
    if (strlen($cleanNif) !== 9) {
        return false;
    }

    // Basic NIF validation algorithm
    $checkDigit = 0;
    for ($i = 0; $i < 8; $i++) {
        $checkDigit += (int)$cleanNif[$i] * (9 - $i);
    }
    
    $remainder = $checkDigit % 11;
    $expectedLastDigit = $remainder < 2 ? 0 : 11 - $remainder;
    
    return (int)$cleanNif[8] === $expectedLastDigit;
}
```

Ao invés disso:

```php
if (!isValidNIF($nif)) {
  \ValidationException('NIF deve conter 9 caracteres')
}

```

Faça isso:

```php
if (!isValidNIF($nif)) {
  \ValidationException('NIF inválido')
}

```
