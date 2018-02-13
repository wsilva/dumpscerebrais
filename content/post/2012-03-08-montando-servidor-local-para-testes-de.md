+++
title = "Montando servidor local para testes de SSL"
slug = "montando-servidor-local-para-testes-de"
categories = ["Tecnologia"]
tags = ["Apache","Developer","Linux","PHP","SSL","Ubuntu"]
date = "2012-03-08T21:10:00-03:00"
+++

Hoje precisei debugar um problema que só acontece se acessarmos um servidor com
SSL.

O protocolo SSL provê a privacidade e a integridade de dados entre duas
aplicações que comuniquem pela Internet. Isto ocorre através da autenticação das
partes envolvidas e da cifra dos dados transmitidos entre as partes.

Esse protocolo ajuda a prevenir que intermediários entre as duas pontas da
comunicação tenham acesso indevido ou falsifiquem os dados transmitidos.
(via <a href='http://pt.wikipedia.org/wiki/Transport_Layer_Security' target='_blank'>wikipedia</a>)

<!--continua-->

Bom para reproduzir o pepino segui os seguintes (pausa: "seguir o seguinte" é
quase um trava lingua) passos:

1) Eu sabia que já tinha instalado o openssl mas mesmo assim rodei:

~~~ bash
    $sudo apt-get update && sudo apt-get install openssl
~~~

2) Ativei o módulo ssl

~~~ bash
    sudo a2enmod ssl
~~~

3) Criar um entrada de virtual host para o site com ssl seguindo o exemplo:

~~~ bash
    <VirtualHost *:443>
        ServerName      sitecomproblema.inet
        DocumentRoot    /var/www/html/sitecomproblema
        <Directory /var/www/html/sitecomproblema>
                Options Indexes FollowSymLinks MultiViews
                AllowOverride All
                Order allow,deny
                Allow from all
        </Directory>
        CustomLog       /var/log/apache2/sitecomproblema-ssl.inet.log combined
        ErrorLog        /var/log/apache2/sitecomproblema-ssl.inet.error.log
        SSLEngine on
        SSLCertificateFile    /etc/ssl/certs/ssl-cert-snakeoil.pem
        SSLCertificateKeyFile /etc/ssl/private/ssl-cert-snakeoil.key
    </VirtualHost>
~~~

4) Ativei o virtual host com o comando

~~~ bash
    sudo a2ensite sitecomproblema
~~~

5) Reiniciei o websever

~~~ bash
    sudo service apache2 restart
~~~

6) Ao acessar https://sitecomproblema.inet o navegador informou conexão não
confiável, adicionei a excessão e boa, carregou bonito.

7) Alterei a url do site no netbeans para https

E na hora de sair debugando, adivinha, surgiu uma nova demanda...

té +
