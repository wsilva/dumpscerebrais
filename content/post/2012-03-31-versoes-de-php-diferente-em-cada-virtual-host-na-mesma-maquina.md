+++
title = "Versões de PHP diferente em cada virtual host na mesma máquina."
slug = "versoes-de-php-diferente-em-cada-virtual-host-na-mesma-maquina"
categories = ["Tecnologia"]
tags = ["Apache","APC","Developer","Fastcgi","Fedora","Linux","Memcache","PHP","VirtualHost","Xdebug"]
date = "2012-03-28T22:46:00-03:00"
+++

Um belo dia precisei dar suporte em dois projetos rodando PHP, legal mas um
projeto roda usando PHP 5.2 e tinha que ter os módulos de memcache, APC e um
módulo de radicalização de palavras chamado
[php-stemmer](http://code.google.com/p/php-stemmer/) .
O outro tinha que
rodar em PHP 5.3 com os módulos mod_rewrite e mcrypt. Ok mas eu tenho uma
máquina só, como faz?

<!--continua-->

Estratégia: usar o ambiente com as instalações padrões ou seja, com PHP 5.3
e módulos instalados normalmente com apt-get install, ou aptitude install, ou
yum install. E uma instalação de PHP 5.2 compilada na unha com os módulos e
tudo mais rodando em CGI ou Fast CGI.

Após tentativas incessantes de compilar o PHP 5.2.7 no Ubuntu 11.10 de
diversas maneiras diferentes além de perder alguns acabei ficando muito
irritado ao ponto de desistir por algum tempo.

Meu brother Erick Müller mais conhecido lá no escritório como o Nave
conseguiu compilar e rodar versões diferentes de PHP usando CGI mas em uma
instalação de 32bits. Como precisava trabalhar montei uma Virtual Machine com
ubuntu server, fiz downgrade da instalação do PHP para 5.2 e montei localmente
com SSHFS (SSH FileSystem).

Graças à insistência do Erick tentamos refazer isso no Fedora 16 e
impressionantemente foi uma manteiga.

Partindo nessa aventura voltei a usar Fedora depois de muitos anos (a última
vez que usei algo padrão Red Hat foi o próprio Red Hat 7.3 ...  Na verdade
cheguei a usar o Fedora Core 6 uma ou duas vezes na faculdade)

Talk is cheap, show me the code, então vamo que vamos:

Assumindo papel de root:

~~~ bash
    su -
~~~


Supondo que pegamos um Fedora 16 zerado novinho, instalamos pacotes necessários
para compilação:

~~~ bash
    yum install automake gcc
~~~


Instalamos o Apache2, o PHP 5.3 e os módulos do PHP necessários para uso
nos projetos PHP 5.3 através dos repositórios:

~~~ bash
    yum install php php-pecl-apc php-common php-cli php-pear php-pdo php-mysql memcached php-pecl-memcache php-pecl-memcached php-gd php-mbstring php-mcrypt php-xml php-pecl-xdebug
~~~


Instalamos o MySQL Server:

~~~ bash
    yum install mysql mysql-server
~~~


Na sequência já subimos o serviço, colocamos na inicialização do sistema e
fizemos a primeira configuração para usar o MySQL:

~~~ bash
    systemctl start mysqld.service
    systemctl enable mysqld.service
    /usr/bin/mysql_secure_installation

    Definir a senha do usuário root:
    Remove anonymous users? [Y/n] Y
    Disallow root login remotely? [Y/n] Y
    Remove test database and access to it? [Y/n] Y
    Reload privilege tables now? [Y/n] Y
~~~


Voltando ao PHP instalamos os pacotes necessários para compilação:

~~~ bash
    yum install libtool httpd-devel aspell-devel apr-devel gcc libxml2-devel bzip2-devel zlib-devel curl-devel libmcrypt-devel libjpeg-devel libpng-devel gd-devel libtool http-devel apr-devel mysql-devel libmhash mhash-devel libpspell pspell-devel aspell-devel openssl-devel libpng-devel freetype-devel libmcrypt-devel pam-devel freetype-devel libtidy-devel expat-devel libxslt-devel libc-client-devel mm-devel libX11-devel mod_fcgid libtool-ltdl-devel
~~~


Adicionamos o usuário da máquina ao grupo apache. Supondo que o usuario da
máquina seja zezinho fica:

~~~ bash
    usermod –append –groups apache zezinho
~~~


Em seguida baixamos os pacotes na pasta home (neste caso a home do usuário
root):

~~~ bash
    cd ~
    wget -c http://us3.php.net/get/php-5.2.17.tar.gz/from/br.php.net/mirror
    wget -c http://us3.php.net/get/php-5.4.0.tar.gz/from/br.php.net/mirror
~~~


Descompactamos o php52 e o php 54:

~~~ bash
    cd ~
    tar -zxvf php-5.2.17.tar.gz
    tar -zxvf php-5.4.0.tar.gz
~~~


Primeiro entramos na pasta do PHP 5.2.17:

~~~ bash
    cd php-5.2.17
~~~


Preparamos o configurator com o pequeno comandinho (sugestão, coloque dentro
de um arquivo sh, dê permissão de execussão e execute-o):

~~~ bash
    ./configure --prefix=/opt/php5.2.17 --enable-fastcgi --with-curl \
    --with-curlwrappers --with-freetype-dir=/usr \
    --with-imap=/opt/php_with_imap_client/ --with-imap-ssl=/usr \
    --with-gettext --with-ttf --enable-mbstring --with-openssl \
    --enable-shmop --enable-sockets --with-pear --with-pdo-mysql=shared \
    --with-pdo-sqlite=shared --with-sqlite=shared --with-tidy --with-ttf \
    --with-pic --enable-pdo=shared --enable-safe-mode --with-png-dir \
    --with-pcre-regex --with-mcrypt --with-mhash --with-libdir=lib64 \
    --with-libexpat-dir=/usr --with-config-file-path=/opt/php5.2.17 \
    --with-config-file-scan-dir=/opt/php5.2.17/php.d --enable-bcmath \
    --enable-calendar --enable-dbase --enable-exif --enable-ftp \
    --enable-mbstring --enable-pcntl --enable-soap --enable-sockets \
    --enable-sqlite-utf8 --enable-wddx --enable-zend-multibyte \
    --enable-zip --with-bz2 --with-zlib --with-gettext --enable-cli \
    --with-pear --with-openssl=/usr --with-iconv --with-mysql --with-mysqli \
    --enable-mbstring --enable-exif --with-jpeg-dir=/usr --with-kerberos \
    --with-zlib --with-zlib-dir --with-png-dir=/usr --with-gd --with-gettext \
    --enable-gd-native-ttf --enable-libxml --with-xmlrpc --with-xpm-dir \
    --with-xsl --with-mhash --with-pspell --with-mcrypt --enable-bcmath \
    --with-mime-magic --with-mm --enable-magic-quotes --enable-sockets \
    --enable-soap --enable-calendar --enable-force-cgi-redirect
~~~


Compilamos:

~~~ bash
    make
~~~


E instalamos em /opt/php5.2.17:

~~~ bash
    make install
~~~


Copiamos o arquivo ini:

~~~ bash
    cp ./php.ini-recommended /opt/php5.2.17/php.ini
~~~


Editamos o php.ini (Atenção, as confs abaixo são para ambiente de desenvolvimento, onde precisamos melhorar apresentação de mensagens de erros, warnings e notices, e com isso fazer as devidas correções):

~~~ bash
    vim /opt/php5.2.17/php.ini
~~~


Descomentar e alterar short_open_tag para Off:

~~~ bash
    short_open_tag = Off
~~~


Descomentar e alterar error_reporting para E_ALL:

~~~ bash
    error_reporting = E_ALL
~~~


Descomentar e alterar display_errors para ON:

~~~ bash
    display_errors = On
~~~


Descomentar e alterar html_errors para On:

~~~ bash
    html_errors = On
~~~


Comentar a linha extension_dir:

~~~ bash
    ;extension_dir = "./"
~~~


Por fim criamos a pasta onde vão ficar os arquivos ini adicionais como definimos no ./configure:

~~~ bash
    mkdir /opt/php5.2.17/php.d
~~~


De maneira análoga vamos preparar o PHP 5.4.0:

~~~ bash
    cd ~/php-5.4.0

    ./configure --prefix=/opt/php5.4.0 --enable-fastcgi --with-curl \
    --with-curlwrappers --with-freetype-dir=/usr \
    --with-imap=/opt/php_with_imap_client/ --with-imap-ssl=/usr \
    --with-gettext --with-ttf --enable-mbstring --with-openssl \
    --enable-shmop --enable-sockets --with-pear --with-pdo-mysql=shared \
    --with-pdo-sqlite=shared --with-sqlite=shared --with-tidy --with-ttf \
    --with-pic --enable-pdo=shared --enable-safe-mode --with-png-dir \
    --with-pcre-regex --with-mcrypt --with-mhash --with-libdir=lib64 \
    --with-libexpat-dir=/usr --with-config-file-path=/opt/php5.4.0 \
    --with-config-file-scan-dir=/opt/php5.4.0/php.d --enable-bcmath \
    --enable-calendar --enable-dbase --enable-exif --enable-ftp \
    --enable-mbstring --enable-pcntl --enable-soap --enable-sockets \
    --enable-sqlite-utf8 --enable-wddx --enable-zend-multibyte --enable-zip \
    --with-bz2 --with-zlib --with-gettext --enable-cli --with-pear \
    --with-openssl=/usr --with-iconv --with-mysql --with-mysqli \
    --enable-mbstring --enable-exif --with-jpeg-dir=/usr --with-kerberos \
    --with-zlib --with-zlib-dir --with-png-dir=/usr --with-gd --with-gettext \
    --enable-gd-native-ttf --enable-libxml --with-xmlrpc --with-xpm-dir \
    --with-xsl --with-mhash --with-pspell --with-mcrypt --enable-bcmath \
    --with-mime-magic --with-mm --enable-magic-quotes --enable-sockets \
    --enable-soap --enable-calendar --enable-force-cgi-redirect
~~~


Compilamos:

~~~ bash
    make
~~~


E instalamos em /opt/php5.4.0:

~~~ bash
    make install
~~~


Também copiamos o php.ini:

~~~ bash
    cp ./php.ini-development /opt/php5.4.0/php.ini
~~~


Também editamos preparando como uma máquina de desenvolvimento:

~~~ bash
    vim /opt/php5.4.0/php.ini
~~~


Descomentar e alterar short_open_tag para Off:

~~~ bash
    short_open_tag = Off
~~~


Descomentar e alterar error_reporting para E_ALL:

~~~ bash
    error_reporting = E_ALL
~~~


Descomentar e alterar display_errors para ON:

~~~ bash
    display_errors = On
~~~


Descomentar e alterar html_errors para On:

~~~ bash
    html_errors = On
~~~


Comentar a linha extension_dir:

~~~ bash
    ;extension_dir = “./”
~~~


Por fim também criamos a pasta onde vão ficar os arquivos ini adicionais
como definimos no ./configure

~~~ bash
    mkdir /opt/php5.4.0/php.d
~~~


Fontes já compilados e instalados agora falta configurar o fastcgi:

~~~ bash
    vim /var/www/cgi-bin/php5.2.17.fcgi
~~~


Inserimos o conteúdo:

~~~ bash
    #!/bin/bash
    PHP_CGI=/opt/php5.2.17/bin/php-cgi
    PHP_FCGI_CHILDREN=4
    PHP_FCGI_MAX_REQUESTS=1000
    export PHP_FCGI_CHILDREN
    export PHP_FCGI_MAX_REQUESTS
    exec $PHP_CGI
~~~


Também fazemos com o PHP 5.4:

~~~ bash
    vim /var/www/cgi-bin/php5.4.0.fcgi
~~~


Com o conteúdo:

~~~ bash
    #!/bin/bash
    PHP_CGI=/opt/php5.4.0/bin/php-cgi
    PHP_FCGI_CHILDREN=4
    PHP_FCGI_MAX_REQUESTS=1000
    export PHP_FCGI_CHILDREN
    export PHP_FCGI_MAX_REQUESTS
    exec $PHP_CGI
~~~


Corrigimos o dono e o permissionamento dos arquivos fastcgi:

~~~ bash
    chmod 744 /var/www/cgi-bin/php5.2.17.fcgi
    chmod 744 /var/www/cgi-bin/php5.4.0.fcgi
    chown apache:apache /var/www/cgi-bin/php5.2.17.fcgi
    chown apache:apache /var/www/cgi-bin/php5.4.0.fcgi
~~~


Criamos as pastas onde ficarão os arquivos de cada virtualhost:

~~~ bash
    mkdir /var/www/html/php52
    mkdir /var/www/html/php53
    mkdir /var/www/html/php54
~~~


Alteramos o dono dessas pastas para seu usuário. Vamos supor que seu
usuário seja zezinho:

~~~ bash
    chown zezinho:apache /var/www/html/php52
    chown zezinho:apache /var/www/html/php53
    chown zezinho:apache /var/www/html/php54
~~~


Preparamos o Apache para trabalhar com VirtualHosts:

~~~ bash
    vim /etc/httpd/conf/httpd.conf
~~~


E descomentamos a linha:

~~~ bash
    NameVirtualHost *:80
~~~


Criamos os VirtualHosts:

~~~ bash
    vim /etc/httpd/conf.d/vhost-php52.inet.conf
    vim /etc/httpd/conf.d/vhost-php53.inet.conf
    vim /etc/httpd/conf.d/vhost-php54.inet.conf
~~~


O conteúdo do vhost-php52.inet.conf vai ficar assim:

~~~ bash
    <VirtualHost *:80>
            ServerName php52.inet
            DocumentRoot /var/www/html/php52
            <Directory /var/www/html/php52>
                    Options -Indexes FollowSymLinks +ExecCGI
                    AllowOverride AuthConfig FileInfo
                    AddHandler php5-fastcgi .php
                    Action php5-fastcgi /cgi-bin/php5.2.17.fcgi
                    DirectoryIndex index.php index.html
                    AllowOverride All
                    Order allow,deny
                    Allow from all
            </Directory>
            CustomLog /etc/httpd/logs/php52.inet.log combined
            ErrorLog /etc/httpd/logs/php52.inet.error.log
    </VirtualHost>
~~~


O conteúdo do  vhost-php54.inet.conf vai ficar assim:

~~~ bash
    <VirtualHost *:80>
            ServerName php54.inet
            DocumentRoot /var/www/html/php54
            <Directory /var/www/html/php54>
                    Options -Indexes FollowSymLinks +ExecCGI
                    AllowOverride AuthConfig FileInfo
                    AddHandler php5-fastcgi .php
                    Action php5-fastcgi /cgi-bin/php5.4.0.fcgi
                    DirectoryIndex index.php index.html
                    AllowOverride All
                    Order allow,deny
                    Allow from all
            </Directory>
            CustomLog /etc/httpd/logs/php54.inet.log combined
            ErrorLog /etc/httpd/logs/php54.inet.error.log
    </VirtualHost>
~~~


O conteúdo do  vhost-php53.inet.conf vai ficar um pouco diferente:

~~~ bash
    <VirtualHost *:80>
            ServerName php53.inet
            DocumentRoot /var/www/html/php54
            <Directory /var/www/html/php54>
                    Options -Indexes FollowSymLinks
                    DirectoryIndex index.php index.html
                    AllowOverride All
                    Order allow,deny
                    Allow from all
            </Directory>
            CustomLog /etc/httpd/logs/php53.inet.log combined
            ErrorLog /etc/httpd/logs/php53.inet.error.log
    </VirtualHost>
~~~


Para testar precisamos ter esse VirtualHosts colocados na nossa lista de hosts:

~~~ bash
    vim /etc/hosts
~~~


E inserimos as linhas abaixo no final:

~~~ bash
    127.0.0.1    php52.inet
    127.0.0.1    php53.inet
    127.0.0.1    php54.inet
~~~


Para mostrar a instalação funcionando usamos um arquivo com o comando phpinfo();

Para criar os arquivos deslogamos o usuário root:

~~~ bash
    logout
~~~


Assim voltamos a ser o usuário da máquina, aqui no nosso exemplo zezinho. Como zezinho vamos criar o arquivo info.php:

~~~ bash
    vim /var/www/html/php52/info.php
~~~


E colocamos o conteúdo:

~~~ php
     <?php phpinfo(); ?>
~~~


Copiamos o arquivo para as pastas dos outros VirtualHosts:

~~~ bash
    cp /var/www/html/php52/info.php /var/www/html/php53/info.php
    cp /var/www/html/php52/info.php /var/www/html/php54/info.php
~~~


Reiniciamos o apache:

~~~ bash
    service httpd restart
~~~


Taaadaaammmm:

<img class="img-responsive img-thumbnail" title="phpinfo 5.2" src='/assets/images/phpinfo52.png' />
<img class="img-responsive img-thumbnail" title="phpinfo 5.3" src='/assets/images/phpinfo53.png' />
<img class="img-responsive img-thumbnail" title="phpinfo 5.4" src='/assets/images/phpinfo54.png' />


Para colocar os módulos devemos compilá-los um a um usando o phpize da versão
que instalamos e o ./configure com o parâmetro –with-php-config= apontando para o path onde está o binário php-config. E aí? Mãos na massa?

Bom para o xdebug fizemos assim. Assumimos o papel de root novamente:

~~~ bash
    su -
~~~


Baixamos o fonte do Xdebug:

~~~ bash
    cd ~
    pear download pecl/xdebug
~~~


Descompactamos:

~~~ bash
    tar -xzvf xdebug-x.x.x.tgz  # (quando testamos era a xdebug-2.2.0RC1.tgz)
~~~


Entramos no diretório e preparamos usando o phpize da versão que vamos instalar:

~~~ bash
    cd xdebug-x.x.x
    /opt/php5.2.17/bin/phpize
~~~


Configuramos com a opção do diretório com o nosso php-config:

~~~ bash
    ./configure –with-php-config=/opt/php5.2.17/bin/php-config
~~~


Compilamos:

~~~ bash
    make
~~~


E Instalamos:

~~~ bash
    make install
~~~


Criamos / Editamos o arquivo de configuração:

~~~ bash
    vim /opt/php5.2.17/php.d/xdebug.ini
~~~


Com o conteúdo:

~~~ bash
    ; Enable xdebug extension module
    zend_extension=/opt/php5.2.17/lib/php/extensions/no-debug-non-zts-20060613/xdebug.so
    xdebug.remote_enable=1
    xdebug.remote_handler=dbgp
    xdebug.remote_mode=req
    xdebug.remote_host=127.0.0.1
    xdebug.remote_port=9000
~~~


Para o módulo de Memcache fizemos similar. Baixamos o fonte:

~~~ bash
    cd ~
    pear download pecl/memcache
~~~


Descompactamos:

~~~ bash
    tar -vzxf memcache-2.2.6.tgz
~~~


Entramos no diretório descompactadoe preparamos a instalação:

~~~ bash
    cd memcache-2.2.6
    /opt/php5.2.17/bin/phpize
~~~


Configuramos com o diretório do php-config:

~~~ bash
    ./configure –with-php-config=/opt/php5.2.17/bin/php-config
~~~


Compilamos:

~~~ bash
    make
~~~


E instalamos:

~~~ bash
    make install
~~~


Criamos/Editamos o arquivo ini:

~~~ bash
    vim /opt/php5.2.17/php.d/memcache.ini
~~~


E colocamos o conteúdo:

~~~ bash
    extension=memcache.so
~~~


Para o módulo de APC também bem similar. Download:

~~~ bash
    cd ~
    pear download pecl/apc
~~~


Descompactar:

~~~ bash
    tar -zxvf APC-3.1.9.tgz
~~~


Entrar e preparar:

~~~ bash
    cd APC-3.1.9/
    /opt/php5.2.17/bin/phpize
~~~


Configurar com o path do php-config:

~~~ bash
    ./configure –with-php-config=/opt/php5.2.17/bin/php-config
~~~


Compilar:

~~~ bash
    make
~~~


Instalar:

~~~ bash
    make install
~~~


E criar o arquivo ini:

~~~ bash
    vim /opt/php5.2.17/php.d/apc.ini
~~~


Nesse colocamos o conteúdo:

~~~ bash
    extension=apc.so
    apc.enabled=1
    apc.shm_size=128M
    apc.ttl=7200
    apc.user_ttl=7200
    apc.enable_cli=1
~~~

É isso ai cambada, comigo está funcionando bem aqui.
Erick, dá uma zoiada. Se esqueci de algo me avisa que editamos.

Sugestões e xingamentos abaixo

té +
