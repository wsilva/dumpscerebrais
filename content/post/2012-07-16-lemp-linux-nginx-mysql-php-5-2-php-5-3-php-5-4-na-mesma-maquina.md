+++
title = "LEMP (Linux + Nginx + MySQL + PHP 5.2 + PHP 5.3 + PHP 5.4) na mesma máquina."
slug = "lemp-linux-nginx-mysql-php-5-2-php-5-3-php-5-4-na-mesma-maquina"
categorias = ["Tecnologia"]
tags = ["APC","Fastcgi","Fedora","LEMP","Linux","Memcache","MySQL","Nginx","PHP","Stemmer","Xdebug","Zend"]
date = "2012-07-16T18:32:00-03:00"
+++

Nesse pequeno (#not) artigo vamos preparar um ambiente com Nginx rodando
versões diferentes de PHP.

Ao final também explicamos como utilizar o
**XDebug**, o **APC**, o **Memcached** e o **PHP Stemmer** em cada uma das
versões diferentes instaladas. Primeiramente levantamos o servidor **LEMP**
normalemente seguindo o passo a passo descrito
[aqui](/2012/07/lemp-linux-nginx-mysql-php-no-fedora-17) .

Resumidamente seguimos 2 passos:

<!--continua-->

1 - Instalamos, levantamos o serviço e configuramos o servidor de MySQL.

    sudo yum install mysql mysql-server
    systemctl start mysqld.service
    /usr/bin/mysql_secure_installation

Respondendo as perguntas da instalação:

    Definir a senha do usuário root:
    Remove anonymous users? [Y/n] Y
    Disallow root login remotely? [Y/n] Y
    Remove test database and access to it? [Y/n] Y
    Reload privilege tables now? [Y/n] Y

2 - Instalamos o Nginx, o PHP e o suporte FastCGI para o PHP.

    sudo yum install php php-pecl-apc php-common php-cli php-pear php-pdo \
    php-mysql memcached php-pecl-memcache php-pecl-memcached php-gd \
    php-mbstring php-mcrypt php-xml php-pecl-xdebug nginx php-fpm

Como voltei a usar o Fedora 16 a versão instalada foi a 5.3.14 do PHP.

## Vamos colocar a mão na massa.
Baixamos os fontes das versões 5.4.4 (última versão estável) e 5.2.17
(última versão estável do 5.2)

    cd ~
    wget -c http://us3.php.net/get/php-5.2.17.tar.gz/from/br.php.net/mirror
    wget -c http://us3.php.net/get/php-5.4.4.tar.gz/from/br.php.net/mirror

Baixamos o patch para fazer o a versão 5.2.17 suportar fastcgi através do FPM

    wget -c http://php-fpm.org/downloads/php-5.2.17-fpm-0.5.14.diff.gz

Baixamos os componentes adicionais (xdebug, apc, e memcache)

    pear download pecl/xdebug
    pear download pecl/memcache
    pear download pecl/memcached
    pear download pecl/apc

Baixamos o PHP Stemmer

    wget -c http://php-stemmer.googlecode.com/files/php-stemmer-0.7.0.tar.gz

Para compilar os fontes do PHP com todas as opções que queremos
precisamos instalar os pacotes abaixo:

    sudo yum install libtool httpd-devel aspell-devel apr-devel gcc \
    libxml2-devel bzip2-devel zlib-devel curl-devel libmcrypt-devel \
    libjpeg-devel libpng-devel gd-devel libtool http-devel apr-devel \
    mysql-devel libmhash mhash-devel libpspell pspell-devel aspell-devel \
    openssl-devel libpng-devel freetype-devel libmcrypt-devel pam-devel \
    freetype-devel libtidy-devel expat-devel libxslt-devel \
    libc-client-devel mm-devel libX11-devel mod_fcgid libtool-ltdl-devel \
    t1lib-devel readline-devel snmp++-devel net-snmp-devel \
    libmemcached-devel

Descompactamos o fonte da versão 5.2.17:

    tar -xvzf php-5.2.17.tar.gz

Aplicamos o patch:

    gzip -cd php-5.2.17-fpm-0.5.14.diff.gz | patch -d php-5.2.17 -p1

Criamos um arquivo builder para configurar com as opções que desejamos:

    vim php-5.2.17/builder-php5.2.17.sh

Com o conteúdo:

    #!/bin/bash
    ./configure --prefix=/opt/php5.2.17 --enable-fastcgi --with-curl \
    --with-curlwrappers --with-freetype-dir=/usr --with-imap \
    --with-imap-ssl=/usr --with-ttf --enable-mbstring --enable-shmop \
    --with-pdo-mysql=shared --with-pdo-sqlite=shared --with-sqlite=shared \
    --with-tidy --with-ttf --with-pic --enable-pdo=shared --enable-safe-mode \
    --with-pcre-regex --with-mcrypt --with-mhash --with-libdir=lib64 \
    --with-libexpat-dir=/usr --with-config-file-path=/opt/php5.2.17 \
    --with-config-file-scan-dir=/opt/php5.2.17/php.d --enable-bcmath \
    --enable-calendar --enable-dbase --enable-dba --enable-ftp \
    --enable-pcntl --enable-soap --enable-sqlite-utf8 --enable-wddx \
    --enable-sysvsem --enable-sysvshm --enable-sysvmsg --enable-zend-multibyte \
    --enable-zip --with-bz2 --enable-cli --enable-fpm \
    --enable-inline-optimization --with-pear --with-openssl \
    --with-openssl=/usr --with-iconv --with-mysql --with-mysqli \
    --enable-exif --with-jpeg-dir=/usr --with-kerberos --with-zlib \
    --with-zlib-dir --with-png-dir --with-png-dir=/usr --with-gd \
    --with-gettext --with-readline --enable-gd-native-ttf --enable-libxml \
    --with-xmlrpc --with-xpm-dir --with-xsl --with-pspell --with-mime-magic \
    --with-mm --enable-magic-quotes --enable-sockets --enable-force-cgi-redirect \
    --with-t1lib --with-fpm-user=nginx --with-fpm-group=nginx --with-snmp

Dando permissão de execução ao arquivo:

    chmod a+x php-5.2.17/builder-php5.2.17.sh

Eu preferi rodar ele mais inchado com mais módulos para não precisara
recompilar se for necessário usar um novo módulo em algum projeto.
Se preferir rode o comando abaixo para mais opções:

    ./configure --help | less

De maneira análoga mas sem precisar aplicar patch fizemos com os fontes
da versão 5.4.4:

    tar -zxvf php-5.4.4.tar.gz
    vim php-5.4.4/builder-php-5.4.4.sh

Conteúdo:

    #!/bin/bash
    ./configure --prefix=/opt/php5.4.4 --enable-fastcgi --with-curl \
    --with-curlwrappers --with-freetype-dir=/usr --with-imap \
    --with-imap-ssl=/usr --with-ttf --enable-mbstring --enable-shmop \
    --with-pdo-mysql=shared --with-pdo-sqlite=shared --with-sqlite=shared \
    --with-tidy --with-ttf --with-pic --enable-pdo=shared \
    --enable-safe-mode --with-pcre-regex --with-mcrypt --with-mhash \
    --with-libdir=lib64 --with-libexpat-dir=/usr \
    --with-config-file-path=/opt/php5.4.4 \
    --with-config-file-scan-dir=/opt/php5.4.4/php.d \
    --enable-bcmath --enable-calendar --enable-dbase --enable-dba \
    --enable-ftp --enable-pcntl --enable-soap --enable-sqlite-utf8 \
    --enable-wddx --enable-sysvsem --enable-sysvshm --enable-sysvmsg \
    --enable-zend-multibyte --enable-zip --with-bz2 --enable-cli \
    --enable-fpm --enable-inline-optimization --with-pear \
    --with-openssl --with-openssl=/usr --with-iconv --with-mysql \
    --with-mysqli --enable-exif --with-jpeg-dir=/usr --with-kerberos \
    --with-zlib --with-zlib-dir --with-png-dir --with-png-dir=/usr \
    --with-gd --with-gettext --with-readline --enable-gd-native-ttf \
    --enable-libxml --with-xmlrpc --with-xpm-dir --with-xsl \
    --with-pspell --with-mime-magic --with-mm --enable-magic-quotes \
    --enable-sockets --enable-force-cgi-redirect --with-t1lib \
    --with-fpm-user=nginx --with-fpm-group=nginx --with-snmp

Permissão de execução:

    chmod a+x php-5.4.4/builder-php5.4.4.sh

## Hora do vamos ver.

Entramos na pasta dos fontes

    cd ~/php-5.2.17

Configuramos:

    ./builder-php5.2.17.sh

Compilamos:

    make

Instalamos:

    sudo make install

Copiamos o arquivo ini:

    sudo cp ./php.ini-recommended /opt/php5.2.17/php.ini

Editamos o php.ini (Atenção, as confs abaixo são para ambiente de
desenvolvimento, onde precisamos melhorar apresentação de mensagens
de erros, warnings e notices, e com isso fazer as devidas correções):

    sudo vim /opt/php5.2.17/php.ini

Descomentar e alterar short_open_tag para Off:

    short_open_tag = Off

Descomentar e alterar error_reporting para E_ALL:

    error_reporting = E_ALL

Descomentar e alterar display_errors para ON:

    display_errors = On

Descomentar e alterar html_errors para On:

    html_errors = On

Comentar a linha extension_dir:

    ;extension_dir = "./"

Por fim criamos a pasta onde vão ficar os arquivos ini adicionais como definimos no ./configure:

    sudo mkdir /opt/php5.2.17/php.d


Voltamos para a pasta dos fontes da versão 5.4.4:

    cd ~/php-5.4.4

Configuramos:

    ./builder-php5.4.4.sh

Compilamos:

    make

Instalamos:

    sudo make install

Também copiamos o php.ini:

    sudo cp ./php.ini-development /opt/php5.4.4/php.ini

Também editamos preparando como uma máquina de desenvolvimento:

    sudo vim /opt/php5.4.4/php.ini

Descomentar e alterar short_open_tag para Off:

    short_open_tag = Off

Descomentar e alterar error_reporting para E_ALL:

    error_reporting = E_ALL

Descomentar e alterar display_errors para ON:

    display_errors = On

Descomentar e alterar html_errors para On:

    html_errors = On

Comentar a linha extension_dir:

    ;extension_dir = "./"

Por fim também criamos a pasta onde vão ficar os arquivos ini adicionais
como definimos no ./configure:

    sudo mkdir /opt/php5.4.4/php.d

Cada um dos serviçõs de PHP-FPM deve rodar em uma porta diferente para
não termos conflito. Vamos aproveitar e liberar a porta 9000 para poder
rodar o XDebug.

Para o PHP-FPM instalado via yum:

    sudo vim /etc/php-fpm.d/www.conf

Alterar a linha de listen de

    listen = 127.0.0.1:9000

para

    listen = 127.0.0.1:9001

Para o PHP-FPM da versão 5.2:

    sudo vim /opt/php5.2.17/etc/php-fpm.conf

Alteramos a linha de listen_address de

    127.0.0.1:9000

para

    127.0.0.1:9002

Descomentamos as linhas de usuário e grupo e definimos como nginx e nginx o usuário e o grupo:

    nginx
    nginx

Para o PHP-FPM da versão 5.4 renomeamos:

    sudo mv /opt/php5.4.4/etc/php-fpm.conf.default /opt/php5.4.4/etc/php-fpm.conf

Editamos o arquivo:

    sudo vim /opt/php5.4.4/etc/php-fpm.conf

Alterar a linha de listen de

    listen = 127.0.0.1:9000

para

    listen = 127.0.0.1:9003

E verifique se o usuário e o grupo que vai rodar o processo são nginx:

    user = nginx
    group = nginx

## Tudo instalado, hora de testar.

Criamos as pastas para 3 virtual hosts:

    mkdir -p /var/www/nginx/php52
    mkdir -p /var/www/nginx/php53
    mkdir -p /var/www/nginx/php54

Criamos 3 arquivos para visualizar as informações do PHP:

    echo "" > /var/www/nginx/php52/info.php
    echo "" > /var/www/nginx/php53/info.php
    echo "" > /var/www/nginx/php54/info.php

Criamos os arquivos de configuração dos hosts:

Para o PHP 5.2:

    sudo vim /etc/nginx/sites-available/php52.inet

Conteúdo:

    server
    {
        server_name php52.inet;
        root /var/www/nginx/php52;
        index index.php index.html index.htm;
        if ($request_uri ~* index/?$)
        {
            rewrite ^/(.*)/index/?$ /$1 permanent;
        }
        if (!-e $request_filename)
        {
            rewrite ^/(.*)$ /index.php?/$1 last;
            break;
        }
        location ~* \.(jpg|jpeg|gif|png|css|js|ico|xml)$
        {
            access_log off;
            log_not_found off;
            expires 360d;
        }
        location ~* \.php$
        {
            server_tokens off;
            client_max_body_size 20m;
            client_body_buffer_size 128k;
            include /etc/nginx/fastcgi_params;
            try_files $uri /index.php;
            fastcgi_pass  127.0.0.1:9002;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param SCRIPT_NAME $fastcgi_script_name;
        }
        location ~ /\.
        {
            access_log off;
            log_not_found off;
            deny all;
        }
    }

Para o PHP 5.3:

    sudo vim /etc/nginx/sites-available/php53.inet

Conteúdo:

    server
    {
        server_name php53.inet;
        root /var/www/nginx/php53;
        index index.php index.html index.htm;
        if ($request_uri ~* index/?$)
        {
            rewrite ^/(.*)/index/?$ /$1 permanent;
        }
        if (!-e $request_filename)
        {
            rewrite ^/(.*)$ /index.php?/$1 last;
            break;
        }
        location ~* \.(jpg|jpeg|gif|png|css|js|ico|xml)$
        {
            access_log off;
            log_not_found off;
            expires 360d;
        }
        location ~* \.php$
        {
            server_tokens off;
            client_max_body_size 20m;
            client_body_buffer_size 128k;
            include /etc/nginx/fastcgi_params;
            try_files $uri /index.php;
            fastcgi_pass  127.0.0.1:9001;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param SCRIPT_NAME $fastcgi_script_name;
        }
        location ~ /\.
        {
            access_log off;
            log_not_found off;
            deny all;
        }
    }

Para o PHP 5.4:

    sudo vim /etc/nginx/sites-available/php54.inet

Conteúdo:

    server
    {
        server_name php54.inet;
        root /var/www/nginx/php54;
        index index.php index.html index.htm;
        if ($request_uri ~* index/?$)
        {
            rewrite ^/(.*)/index/?$ /$1 permanent;
        }
        if (!-e $request_filename)
        {
            rewrite ^/(.*)$ /index.php?/$1 last;
            break;
        }
        location ~* \.(jpg|jpeg|gif|png|css|js|ico|xml)$
        {
            access_log off;
            log_not_found off;
            expires 360d;
        }
        location ~* \.php$
        {
            server_tokens off;
            client_max_body_size 20m;
            client_body_buffer_size 128k;
            include /etc/nginx/fastcgi_params;
            try_files $uri /index.php;
            fastcgi_pass  127.0.0.1:9003;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param SCRIPT_NAME $fastcgi_script_name;
        }
        location ~ /\.
        {
            access_log off;
            log_not_found off;
            deny all;
        }
    }

Ativamos os hosts criando links simbólicos para a pasta enabled

    sudo ln -s /etc/nginx/sites-available/php52.inet /etc/nginx/sites-enabled/
    sudo ln -s /etc/nginx/sites-available/php53.inet /etc/nginx/sites-enabled/
    sudo ln -s /etc/nginx/sites-available/php54.inet /etc/nginx/sites-enabled/

Adicionamos as entradas no arquivo de hosts:

    sudo vim /etc/hosts

Conteúdo a ser adicionado (no final do arquivo):

    127.0.0.1    php52.inet
    127.0.0.1    php53.inet
    127.0.0.1    php54.inet

Levantamos os serviços:

    sudo service nginx start
    sudo service php-fpm start
    sudo /opt/php5.2.17/sbin/php-fpm start
    sudo /opt/php5.4.4/sbin/php-fpm

Para parar o php-fpm 5.4.4 temos que matar o processo com o comando:

    kill -9 {id_do_processo}

Para descobrir o id do processo execute:

    ps aux |grep php-fpm

Os demais podemos parar com a opção stop no lugar da opção start:

    service php-fpm stop
    /opt/php5.2.17/sbin/php-fpm stop

Para testar basta carregar os sites no navegador.

<img class="img-responsive img-thumbnail" title="php 5.2.17 + nginx + php-fpm (compilado)" alt="php 5.2.17 + nginx + php-fpm (compilado)" src='/assets/images/php52.png' />
<img class="img-responsive img-thumbnail" title="php 5.3.14 + nginx + php-fpm (instalado via yum)" alt="php 5.3.14 + nginx + php-fpm (instalado via yum)" src='/assets/images/php53.png' />
<img class="img-responsive img-thumbnail" title="php 5.4.4 + nginx + php-fpm (compilado)" alt="php 5.4.4 + nginx + php-fpm (compilado)" src='/assets/images/php54.png' />

## Instalando o XDebug

Voltamos para a pasta home:

    cd ~

Descompactamos:

    tar -xzvf xdebug-x.x.x.tgz  # (quando testamos era a xdebug-2.2.1.tgz)

Entramos no diretório e preparamos usando o phpize da versão que vamos
instalar:

    cd xdebug-x.x.x
    /opt/php5.2.17/bin/phpize

Configuramos com a opção do diretório com o nosso php-config:

    ./configure –with-php-config=/opt/php5.2.17/bin/php-config

Compilamos:

    make

E instalamos:

    sudo make install

Verificamos qual a pasta onde foi instalado a biblioteca 'so':

    find /opt/php5.2.17/ | grep xdebug.so

Vamos usar este diretório no parâmetro zend_extension do arquivo de
configuração xdebug.ini

Criamos / Editamos o arquivo de configuração:

    sudo vim /opt/php5.2.17/php.d/xdebug.ini

Exemplo de conteúdo:

    ; Enable xdebug extension module
    zend_extension=/opt/php5.2.17/lib/php/extensions/no-debug-non-zts-20060613/xdebug.so
    xdebug.remote_enable=1
    xdebug.remote_handler=dbgp
    xdebug.remote_mode=req
    xdebug.remote_host=127.0.0.1
    xdebug.remote_port=9000

Repetimos para a versão 5.4.4 do PHP.

Voltamos para a pasta home e removemos o diretório gerado:

    cd ~
    rm -rf xdebug-x.x.x

Descompactamos novamente:

    tar -xzvf xdebug-x.x.x.tgz  # (quando testamos era a xdebug-2.2.1.tgz)

Entramos no diretório e preparamos usando o phpize da versão que vamos
instalar:

    cd xdebug-x.x.x
    /opt/php5.4.4/bin/phpize

Configuramos com a opção do diretório com o nosso php-config:

    ./configure –with-php-config=/opt/php5.4.4/bin/php-config

Compilamos:

    make

E instalamos:

    sudo make install

Verificamos qual a pasta onde foi instalado a biblioteca 'so':

    find /opt/php5.4.4/ |grep xdebug.so

Vamos usar este diretório no parâmetro zend_extension do arquivo de
configuração xdebug.ini

Criamos / Editamos o arquivo de configuração:

    sudo vim /opt/php5.4.4/php.d/xdebug.ini

Exemplo de conteúdo:

    ; Enable xdebug extension module
    zend_extension=/opt/php5.4.4/lib/php/extensions/no-debug-non-zts-20100525/xdebug.so
    xdebug.remote_enable=1
    xdebug.remote_handler=dbgp
    xdebug.remote_mode=req
    xdebug.remote_host=127.0.0.1
    xdebug.remote_port=9000

## Instalando suporte ao Memcache

### Com php-memcache

Voltamos para a pasta home

    cd ~

Descompactamos:

    tar -vzxf memcache-x.x.x.tgz # (quando testamos era a memcache-3.0.6.tgz)

Entramos no diretório descompactamos e preparamos a instalação:

    cd memcache-x.x.x
    /opt/php5.2.17/bin/phpize

Configuramos com o diretório do php-config:

    ./configure –with-php-config=/opt/php5.2.17/bin/php-config

Compilamos:

    make

E instalamos:

    sudo make install

Criamos/Editamos o arquivo ini:

    sudo vim /opt/php5.2.17/php.d/memcache.ini

Colocamos o conteúdo:

    extension=memcache.so

Voltamos para a pasta home e removemos o diretório gerado:

    cd ~
    rm -rf memcache-x.x.x

Descompactamos:

    tar -vzxf memcache-x.x.x.tgz # (quando testamos era a memcache-3.0.6.tgz)

Entramos no diretório descompactadoe preparamos a instalação:

    cd memcache-x.x.x
    /opt/php5.4.4/bin/phpize

Configuramos com o diretório do php-config:

    ./configure –with-php-config=/opt/php5.4.4/bin/php-config

Compilamos:

    make

E instalamos:

    sudo make install

Criamos/Editamos o arquivo ini:

    sudo vim /opt/php5.4.4/php.d/memcache.ini

Colocamos o conteúdo:

    extension=memcache.so

### Com php-memcached

Voltamos para a pasta home

    cd ~

Descompactamos:

    tar -vzxf memcached-x.x.x.tgz # (quando testamos era a memcached-2.0.1.tgz)

Entramos no diretório descompactadoe preparamos a instalação:

    cd memcached-x.x.x
    /opt/php5.2.17/bin/phpize

Configuramos com o diretório do php-config:

    ./configure –with-php-config=/opt/php5.2.17/bin/php-config

Compilamos:

    make

E instalamos:

    sudo make install

Criamos/Editamos o arquivo ini:

    sudo vim /opt/php5.2.17/php.d/memcached.ini

Colocamos o conteúdo:

    extension=memcached.so

Voltamos para a pasta home e removemos o diretório gerado:

    cd ~
    rm -rf memcached-x.x.x

Descompactamos:

    tar -vzxf memcached-x.x.x.tgz # (quando testamos era a memcached-2.0.1.tgz)

Entramos no diretório descompactadoe preparamos a instalação:

    cd memcached-x.x.x
    /opt/php5.4.4/bin/phpize

Configuramos com o diretório do php-config:

    ./configure –with-php-config=/opt/php5.4.4/bin/php-config

Compilamos:

    make

E instalamos:

    sudo make install

Criamos/Editamos o arquivo ini:

    sudo vim /opt/php5.4.4/php.d/memcached.ini

Colocamos o conteúdo:

    extension=memcached.so

## Instalando APC

Voltamos para a pasta home

    cd ~

Descompactamos:

    tar -zxvf APC-x.x.x.tgz # (quando testamos era a versão APC-3.1.9.tgz)

Entrar e preparar:

    cd APC-x.x.x/
    /opt/php5.2.17/bin/phpize

Configurar com o path do php-config:

    ./configure –with-php-config=/opt/php5.2.17/bin/php-config

Compilar:

    make

Instalar:

    make install

E criar o arquivo ini:

    sudo vim /opt/php5.2.17/php.d/apc.ini

Colocamos o conteúdo:

    extension=apc.so
    apc.enabled=1
    apc.shm_size=128M
    apc.ttl=7200
    apc.user_ttl=7200
    apc.enable_cli=1

Voltamos para a pasta home e removemos a pasta gerada:

    cd ~
    rm -rf APC-x.x.x

Descompactamos novamente:

    tar -zxvf APC-x.x.x.tgz # (quando testamos era a versão APC-3.1.9.tgz)

Entrar e preparar:

    cd APC-x.x.x/
    /opt/php5.4.4/bin/phpize

Configurar com o path do php-config:

    ./configure –with-php-config=/opt/php5.4.4/bin/php-config

Compilar:

    make

Instalar:

    make install

E criar o arquivo ini:

    sudo vim /opt/php5.4.4/php.d/apc.ini

Colocamos o conteúdo:

    extension=apc.so
    apc.enabled=1
    apc.shm_size=128M
    apc.ttl=7200
    apc.user_ttl=7200
    apc.enable_cli=1

## Instalando o PHP Stemmer.

Voltamos para a pasta home

    cd ~

Descompactamos:

    tar -zxf php-stemmer-0.7.0.tar.gz

Para servidores baseados em x64 editamos o arquivo libstemmer_c/Makefile,
procuramos pela linha:

    'CFLAGS=-Iinclude'

e modificamos para

    CFLAGS=-Iinclude -fPIC

Preparamos

    /opt/php5.2.17/bin/phpize

Configurar com o path do php-config:

    ./configure –with-php-config=/opt/php5.2.17/bin/php-config

Entramos no libstemmer:

    cd libstemmer_c

Compilamos:

    make

Voltamos a pasta do php-stemmer:

    cd ..

Compilamos:

    make

Instalamos

    sudo make install

Criamos o arquivo ini:

    sudo vim /opt/php5.4.4/php.d/php-stemmer.ini

Com o conteúdo:

    extension=stemmer.so

Voltamos para a pasta home e removemos a pasta gerada:

    cd ~
    rm -rf php-stemmer-0.7.0

Descompactamos novamente:

    tar -zxf php-stemmer-0.7.0.tar.gz

Para servidores baseados em x64 editamos o arquivo libstemmer_c/Makefile, procuramos pela linha

    'CFLAGS=-Iinclude'

e modificamos para

    CFLAGS=-Iinclude -fPIC

Preparamos

    /opt/php5.4.4/bin/phpize

Configurar com o path do php-config:

    ./configure –with-php-config=/opt/php5.4.4/bin/php-config

Entramos no libstemmer:

    cd libstemmer_c

Compilamos:

    make

Voltamos a pasta do php-stemmer:

    cd ..

Compilamos:

    make

Instalamos

    sudo make install

Criamos o arquivo ini:

    sudo vim /opt/php5.4.4/php.d/php-stemmer.ini

Com o conteúdo:

    extension=stemmer.so

## Algumas referências:

- [http://ondrejsimek.com/blog/running-multiple-php-versions-is-so-easy-with-fastcgi/](http://ondrejsimek.com/blog/running-multiple-php-versions-is-so-easy-with-fastcgi/) 
- [http://www.vladgh.com/blog/install-nginx-and-php-php-fpm-mysql-and-apc](http://www.vladgh.com/blog/install-nginx-and-php-php-fpm-mysql-and-apc) 
- [http://www.x83.net/nginx-php-5-2-17-php-fpm/](http://www.x83.net/nginx-php-5-2-17-php-fpm/) 
- [http://www.if-not-true-then-false.com/2011/lemp-on-fedora-centos-red-hat-rhel-linux-nginx-mysql-php-fpm/](http://www.if-not-true-then-false.com/2011/lemp-on-fedora-centos-red-hat-rhel-linux-nginx-mysql-php-fpm/) 
- [http://code.google.com/p/php-stemmer/](http://code.google.com/p/php-stemmer/) 

Inté +

Atenção galera de plantão, se vc usa Ubuntu segue o
[link](http://www.tekniq.com.br/blog/?p=19)  para o post
do [@ehriq](http://twitter.com/ehriq)  explicando como
fazer essa maluquice.
