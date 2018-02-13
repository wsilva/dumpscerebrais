+++
title = "Nginx não repassa os parâmetros query string para o framework"
categorias = ["Tecnologia"]
tags = ["Nginx","Framework"]
date = "2012-08-23T23:56:00-03:00"
+++

Saudações, A dica do dia é habilitar o envio de query string para o
framework (zend, codeigniter, etc...) para que possa receber os parâmetros
através de uma requisição GET. O problema foi detectado usando zend
framework e ao chamar uma action usando query string para passar parâmetros.

(http://exemplo.inet/teste/index?var1=bla&var2=yey) as variáveis vinham
vazias e usando a notação de barras no próprio zend
(http://exemplo.inet/teste/index/var1/bla/var2/yey) funcionava normalmente.

A configuração desse host no nginx estava parecido com o abaixo:

<!--continua-->

    server {

        root /var/www/nginx/exemplo;
        index index.php;
        ...
        try_files $uri $uri/ /index.php;
        ...
        location ~ \.php$ {
            include /etc/nginx/fastcgi_params;
            fastcgi_pass  127.0.0.1:9001;
            fastcgi_index index.php;
            ...
        }
    }

Após pesquisar e achar o post desse
cara: [http://kfalck.net/2011/06/19/fix-empty-nginx-fastcgi-query-strings](http://kfalck.net/2011/06/19/fix-empty-nginx-fastcgi-query-strings) 
alterei para:

    server {

        root /var/www/nginx/exemplo;
        index index.php;
        ...
        try_files $uri $uri/ /index.php?$query_string;
        ...
        location ~ \.php$ {
                include /etc/nginx/fastcgi_params;
                fastcgi_pass  127.0.0.1:9001;
                fastcgi_index index.php;
                ...
        }
    }

Reiniciado o Nginx tudo voltou ao normal.

Té +
