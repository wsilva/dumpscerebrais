+++
title = "Consumindo Google Plus API"
categorias = ["Tecnologia"]
tags = ["Google Plus","PHP"]
date = "2012-07-25T09:59:00-03:00"
+++

Os passos que seguem vão mostrar como consumir de maneira simples a API
do Google Plus usando PHP. A API está publicada no endereço
[https://developers.google.com/+/api/](https://developers.google.com/+/api/)
e permite 10.000 requisições
por dia (no dia que escrevi este artigo).

<!--continua-->

## Hands ON

Primeiro devemos acessar o console no endereço
[https://code.google.com/apis/console#access](https://code.google.com/apis/console#access)

Se não estiver autenticado no google será necessário informar usuário e
senha registrado.

Em seguida devemos clicar em create project.

<img class="img-responsive img-thumbnail" title="create project" alt="create project" src='/assets/images/tela1.png' />

Na tela seguinte em Overview clicamos em Register ao lado de Project ID.

<img class="img-responsive img-thumbnail" title="Register" alt="Register" src='/assets/images/tela2.png' />

Demos um nome para o projeto, antes de avançar verifique se o nome informado
está disponível usando o botão.

<img class="img-responsive img-thumbnail" title="Nome projeto" alt="Nome projeto" src='/assets/images/tela3.png' />

Com o projeto criado vamos em API Access e clique em Create an
Oauth 2.0 client ID

<img class="img-responsive img-thumbnail" title="configurar api" alt="configurar api" src='/assets/images/tela4.png' />

Devemos definir um nome e se desejarmos também podemos informar uma url
de uma imagem para um logo.

<img class="img-responsive img-thumbnail" title="configurar api 2" alt="configurar api 2" src='/assets/images/tela5.png' />

Em seguida devemos selecionar web application, já que vamos acessar a API
por uma outra página e definir também a url de callback, página para onde
serão direcionadas as pessoas após a autorização. Se a url informada não
for localhost o sitema testa se a url informada está disponível.

<img class="img-responsive img-thumbnail" title="dados de callback" alt="dados de callback" src='/assets/images/tela6.png' />

Em seguida baixamos os fontes em PHP da API em
[https://developers.google.com/+/downloads/](https://developers.google.com/+/downloads/) 
 - arquivo com a api: [https://code.google.com/p/google-api-php-client/](https://code.google.com/p/google-api-php-client/) 
 - arquivo com um exemplo inicial de uso: [https://code.google.com/p/google-plus-php-starter/](https://code.google.com/p/google-plus-php-starter/) 

Descompactamos ambos na raiz do site que vai receber a chamada.
No caso testamos com o domínio
*http://localhost* cuja raiz estava na pasta */var/www/html/* então
descompactamos os dois arquivos em */var/www/html/*

Editamos o arquivo

    /var/www/html/google-plus-php-starter-1.0/index.php

Alteramos as linhas:

    require_once 'google-api-php-client/src/apiClient.php';
    require_once 'google-api-php-client/src/contrib/apiPlusService.php';

para:

    require_once '../google-api-php-client/src/apiClient.php';
    require_once '../google-api-php-client/src/contrib/apiPlusService.php';

Descomentamos as linhas abaixo e colocamos os dados da sua conta de
desenvolvedor. Esses dados podem ser obtidos no endereço
[https://code.google.com/apis/console](https://code.google.com/apis/console) 
clicando na opção API Access (você deve estar logado no Google para isso)

Era assim:

    // $client->setClientId('insert_your_oauth2_client_id');
    // $client->setClientSecret('insert_your_oauth2_client_secret');
    // $client->setRedirectUri('insert_your_oauth2_redirect_uri');
    // $client->setDeveloperKey('insert_your_developer_key');

Ficou assim:

    $client->setClientId('xxxxxxxxxxxx.apps.googleusercontent.com'); //código que está em Client ID:
    $client->setClientSecret('xxxxxxxxxxxxxxxxxxxxxxx'); //código que está em Client secret:
    $client->setRedirectUri('http://localhost/google-plus-php-starter-1.0/'); // url de call back, deve estar em Redirect URIs:
    $client->setDeveloperKey('xXxXxXxxXXxxXXXxxXx'); //código que deve estar em API key:

## Testando

Pronto, só acessar http://localhost/google-plus-php-starter-1.0/ e testar.

<img class="img-responsive img-thumbnail" title="Testando API G+" alt="Testando API G+" src='/assets/images/testandogplus1.png' />
<img class="img-responsive img-thumbnail" title="Testando API G+" alt="Testando API G+" src='/assets/images/testandogplus2.png' />
<img class="img-responsive img-thumbnail" title="Testando API G+" alt="Testando API G+" src='/assets/images/testandogplus3.png' />

Inté a próxima
