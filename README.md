# Desenvolvendo para desenvolvedores

Uma série de recomendações de boas práticas de desenvolvimento, para auxiliar o entendimento e a manutenção de códigos de projetos de software.

## Inversão de dependências (DIP)

> Em vez de classes específicas (implementações concretas), use interfaces ou classes abstratas para definir contratos e interações entre diferentes partes do sistema. 

Ao invés disso:
```php

/** @var ContainerInterface $container */
$container = null;

$client = $container->get('guzzle');
$client = $container->get('GuzzleHttp');
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

// Service.php
<php

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

Ao invés disso:

```php

// .env
API_BASE_URL=https://www.someapi.com/api/v1

// Service.php
<php

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
