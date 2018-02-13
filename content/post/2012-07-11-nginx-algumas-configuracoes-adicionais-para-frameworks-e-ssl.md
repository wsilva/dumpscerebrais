+++
title = "Nginx: algumas configurações adicionais para frameworks e SSL"
categorias = ["Tecnologia"]
tags = ["Codeigniter","Fastcgi","Fedora","Framework","LEMP","Linux","Nginx","PHP","SSL","Xdebug","Zend"]
date = "2012-07-11T20:02:00-03:00"
+++

Saudações, dando continuidade ao [último post](/2012/07/lemp-linux-nginx-mysql-php-no-fedora-17) 
segue uma série de configurações para o Nginx.

SSL, Gerar as chaves com o comando

    openssl req -new -x509 -days 365 -nodes -out /etc/httpd/ssl/teste.pem -keyout /etc/httpd/ssl/teste.key

<!--continua-->

(Em caso de dúvida ver esse [post](/2012/05/apache2-com-ssl-em-virtualhosts) .)

Na configuração do VirtualHost que coloquei em
*/etc/nginx/sites-available/teste.inet* colocar as linhas:

    server
    {
        ...
        listen 443;
        ssl on;
        ssl_certificate /etc/httpd/ssl/teste.pem;
        ssl_certificate_key /etc/httpd/ssl/teste.key;
        ssl_session_timeout  5m;
        ssl_protocols  SSLv2 SSLv3 TLSv1;
        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers   on;
        ...
    }


Se o framework trata páginas não encontradas (erro 404) como o Zend
Framework, o Codeigniter entre outros então direcionamos a requisição
para a index.php

    server
    {
        ...
        error_page 404 /index.php;
        ...
    }

Frameworks como o Codeigniter e o Zend Framework por exemplo tem
requisições direcionadas para controllers então devemos definir
uma regra para a rota principal.

    server
    {
        ...
        if ($request_uri ~* ^(/bemvindo(/index)?|/index(.php)?)/?$)
        {
            rewrite ^(.*)$ / permanent;
        }
        ...
    }

Neste caso quando acessarmos *http://teste.inet* ele vai direcionar de
maneira adequada para *http://teste.inet/bemvindo/index* além de tratar
as chamadas diretas em cima dos métodos
(exemplo: *http://teste.inet/auth/login*) de maneira correta.

Remove o método padrão index quando um método de um controller não é
invocado.

    server
    {
        ...
        if ($request_uri ~* index/?$)
        {
            rewrite ^/(.*)/index/?$ /$1 permanent;
        }
        ...
    }

Negando acesso aos arquivos .htaccess do apache.

    server
    {
        ...
        location ~ /\.ht
        {
            deny all;
        }
        ...
    }

Se a chamada não for em arquivos estáticos como imagem, scripts,
ou folhas de estilos direcionamos para o index do framework


    server
    {
        ...
        if (!-e $request_filename)
        {
            rewrite ^/(.*)$ /index.php?/$1 last;
            break;
        }
        ...
    }

Com essas regrinhas acima já consegui trabalhar tranquilo com os
frameworsks Zend, Codeigniter e Kohana.

Próximos passos:

- fazer debug de código e profile usando Xdebug;
- testar funcionamento de módulos do php como php-stemmer;
- separar versões diferentes de php em virtual hosts diferentes.

Até a próxima
