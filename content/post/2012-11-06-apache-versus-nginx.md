+++
title = "Apache versus Nginx"
categorias = ["Tecnologia"]
tags = ["Apache","Fastcgi","LAMP","LEMP","Nginx","PHP","Benchmark"]
date = "2012-11-06T00:46:00-03:00"
+++

Depois de uma fase conturbada e sem artigos novos vamos retornar
escrevendo um comparativo entre Nginx e Apache.

<img class="img-responsive img-thumbnail" title="Apache vs Nginx" alt="Apache vs Nginx" src='/assets/images/apache_vs_nginx.png' />

Para elaborar os testes criamos quatro arquivos de naturezas diferentes para
serem servido pelo Apache e posteriormente pelo Nginx para que
possamos comparar.

<!--continua-->

Para medir a capacidade de cada servidor utilizamos um outro computador
conectado na mesma rede do servidor testado e com a rede isolada
efetuamos os testes usando ab (Apache Benchmark).

## Os artefatos

Como artefatos utilizamos um arquivo html estático nomeado
*estatica.html* com o conteúdo:

    <h1>Testando</h1>
    <p>Isto é uma página estática</p>

Como podemos ver aqui:

<img class="img-responsive img-thumbnail" title="Página Estática" alt="Página Estática" src='/assets/images/apache-estatica.png' />

Um arquivo dinamica.php com um conteúdo:

~~~ php
    <?php
    echo "<h1>Isto é uma página dinâmica</h1>";
    for($i=0; $i<10; $i++)
    {
        for($j=0; $j<10; $j++)
        {
                echo "$i.$j";
                echo PHP_EOL;
        }
    }
    ?>
~~~

Como vemos aqui:

<img class="img-responsive img-thumbnail" title="Página Dinâmica" alt="Página Dinâmica" src='/assets/images/apache-dinamica.png' />

Um arquivo info.php que gera as configurações do php muito utilizado para
testes com o conteúdo:

~~~ php
    <?php
    phpinfo();
~~~

E um arquivo de imagem de 80KB:

<img class="img-responsive img-thumbnail" title="Imagem Genérica" alt="Imagem Genérica" src='/assets/images/imagem.jpg' />

## Montando o ambiente

Primeiro instalamos o tradicional LAMP, com exceção do MySQL, onde o
php roda sobre o Apache Handler como podemos ver nas informações do phpinfo:

<img class="img-responsive img-thumbnail" title="phpinfo apache" alt="phpinfo apache" src='/assets/images/info-apache.png' />

Fizemos um teste acessando os arquivos e em seguida desligamos o
serviço do apache:

~~~ bash
    sudo service httpd stop
~~~

Istalamos o chamado LEMP, também com exceção do MySQL, onde o php roda
sobre o php-fpm que serve as páginas através do fastcgi como podemos ver:

<img class="img-responsive img-thumbnail" title="phpinfo nginx" alt="phpinfo nginx" src='/assets/images/nginx-info.png' />

Testamos novamente para ver se nossas configurações funcionaram e em
seguida desligamos o serviço do Nginx e do PHP-FPM:

~~~ bash
    sudo service nginx stop
    sudo service php-fpm stop
~~~

Para maiores informações sobre a instalação do LAMP e do LEMP
acesse outros [artigos](/index.html) 
meus neste blog em mais antigos ou pelas [tags / marcadores](/categorias/tecnologia).

## O Teste
O ambiente usado como servidor foi um Intel Core i5 com 4GB de memória
rodando *Fedora 17* com *Apache 2.2.22*, *Nginx 1.0.15* e *PHP 5.4.7*.

A ferramenta que utilizamos para medição foi o *Apache Benchmark*
e rodamos através do comando:

~~~ bash
    ab -kc 10 -t 30 http://url-de-destino/arquivo.ext
~~~

Onde:

- k – é keep alive
- c – é quantidade de concorrência
- t – é tempo em segundos

Com o servidor Apache ligado executamos o comando apontando para
cada arquivo testado (estático, dinâmico, phpinfo e imagem)
coletando os resultados.

Depois desligamos o servidor Apache, ligamos o servidor Nginx
junto com o PHP-FPM e repetimos os testes coletando novamente
os resultados

## Os resultados

Como podemos ver o Nginx teve uma melhor performance
servindo os arquivos testados.

**9871 requisições** completadas no *Nginx* contra **4561 requisições**
no *Apache* para a página *dinamica.php*

<img class="img-responsive img-thumbnail" title="Comparativo de Página Dinâmica" alt="Comparativo de Página Dinâmica" src='/assets/images/comp-dinamica.png' />

**21231 requisições** completadas no *Nginx* contra **10127 requisições** no
*Apache* para a página *estatica.html*

<img class="img-responsive img-thumbnail" title="Comparativo de Página Estática" alt="Comparativo de Página Estática" src='/assets/images/comp-estatica.png' />

**639 requisições** completadas no *Nginx* contra **621 requisições** no
*Apache* para o arquivo de *imagem*.

<img class="img-responsive img-thumbnail" title="Comparativo de Comparativo" alt="Comparativo de Comparativo" src='/assets/images/comp-imagem.png' />

**734 requisições** completadas no *Nginx* contra **501 requisições** no
Apache para o arquivo *info.php*,

<img class="img-responsive img-thumbnail" title="Comparativo de Página Dinâmica de Informações" alt="Comparativo de Página Dinâmica de Informações" src='/assets/images/comp-info.png' />

## Concluindo:
Com **Nginx** tivemos performances de até **116%** maior que **Apache2** para
o mesmo ambiente.

Divirtam-se e até a próxima...
