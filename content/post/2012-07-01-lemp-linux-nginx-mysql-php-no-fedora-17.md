+++
title = "LEMP (Linux + Nginx + MySQL + PHP) no Fedora 17"
slug = "lemp-linux-nginx-mysql-php-no-fedora-17"
categories = ["Tecnologia"]
tags = ["Apache","Fedora","LAMP","LEMP","Linux","MySQL","Nginx","PHP","VirtualHost"]
date = "2012-07-01T22:27:00-03:00"
+++

Em férias escolares temporárias então de volta aos estudos caseiros. Resolvi
fazer um test drive do Nginx o web server que ultimamente tem
recebido boas críticas. Para instalação segui parcialmente o tutorial
do [if not true then false](http://www.if-not-true-then-false.com/2011/install-nginx-php-fpm-on-fedora-centos-red-hat-rhel/) .

Depois de fazer um backup, instalar do zero o Fedora 17, instalar algumas
ferramentas básicas, parti para o web server.

Primeiramente fiz a instalação que sempre fazia do LAMP (Linux +
Nginx + MySQL + PHP). Instalação que também pode ser conferida
[aqui](/2012/03/versoes-de-php-diferente-em-cada-virtual-host-na-mesma-maquina) .

No começo tudo normal. Assumindo papel de root:

<!--continua-->

    su -

Comecei com o MySQL Server. Instalando:

    yum install mysql mysql-server

Iniciando o serviço:

    systemctl start mysqld.service

Rodando o script de primeira inicialização:

    /usr/bin/mysql_secure_installation

E respondida as perguntas:

    Definir a senha do usuário root:
    Remove anonymous users? [Y/n] Y
    Disallow root login remotely? [Y/n] Y
    Remove test database and access to it? [Y/n] Y
    Reload privilege tables now? [Y/n] Y

Depois o PHP e algumas ferramentas adicionais:

    yum install php php-pecl-apc php-common php-cli php-pear php-pdo php-mysql memcached php-pecl-memcache php-pecl-memcached php-gd php-mbstring php-mcrypt php-xml php-pecl-xdebug

Acertei o arquivo de conf do apache...

    vim /etc/httpd/conf/httpd.conf

... descomentando a linha:

    NameVirtualHost *:80

Criei um Virtual Host...

    vim /etc/httpd/conf.d/vhost-teste.inet.conf

...com o conteúdo:

    <VirtualHost *:80>
        ServerName teste.inet
        DocumentRoot /var/www/html/teste/
        <Directory /var/www/html/teste/>
            Options -Indexes FollowSymLinks
            DirectoryIndex index.php index.html
            AllowOverride All
            Order allow,deny
            Allow from all
        <Directory>
        CustomLog /etc/httpd/logs/teste.inet.log combined
        ErrorLog /etc/httpd/logs/teste.inet.error.log
    <VirtualHost>

Criei a pasta:

    mkdir -p /var/www/html/teste

Criei o arquivo de teste...

    vim /var/www/html/teste/info.php

...com o conteúdo

    <?php
    phpinfo();

Alterei o dono:

    chown apache:apache -R /var/www/html/

E editei o arquivo de hosts...

    vim /etc/hosts

...adicionando a linha abaixo para poder testar:

    127.0.0.1         teste.inet

Ao terminar reiniciei o Apache e já tive a primeira surpresa,
o php que foi instalado já foi versão 5.4.4.

<img class="img-responsive img-thumbnail" title="PHP 5.4.4 rodando no Apache 2" alt="PHP 5.4.4 rodando no Apache 2" src='/assets/images/apache.png' />

Agora a parte legal. Parei o Apache:

    service httpd stop

Instalei o Nginx e o suporte Fastcgi para o PHP:

    yum install nginx php-fpm

Iniciei o Nginx e o suporte Fastcgi para o PHP:

    service nginx start
    service php-fpm start

Neste momento de configuração do Nginx pensei em usar a mesma pasta que
usamos para o apache (/var/www/html/teste) mas acabei criando uma nova
pasta porque teríamos um problema de permissões.

Para as páginas carregarem usando Apache o usuário apache tem que ter
pelo menos a permissão de leitura na pasta onde estão os arquivos, por
isso mudamos o dono das pastas e arquivos na instalação com o comando
chown. Da mesma maneira o Nginx precisa que o usuário nginx tenha pelo
menos a permissão de leitura nesta pasta, então toda hora que iniciar
ou o Apache ou o Nginx teríamos que alterar o usuário dono das
pastas e arquivos.

Criei a pasta para o site e também a pasta para os logs:

    mkdir -p /srv/www/teste.inet/public_html
    mkdir -p /srv/www/teste.inet/logs
    chown -R nginx:nginx /srv/www/teste.inet

Criei os diretórios para os virtual hosts:

    mkdir -p /etc/nginx/sites-available
    mkdir -p /etc/nginx/sites-enabled

Alterei o arquivo...

    vim /etc/nginx/nginx.conf

... inserindo a linha...

    include /etc/nginx/sites-enabled/* ;

...logo abaixo da linha abaixo ainda no bloco http:

    include /etc/nginx/conf.d/*.conf ;

Criei o arquivo:

    /etc/nginx/sites-available/teste.inet

Com o conteúdo:

    server {
        server_name teste.inet;
        access_log /srv/www/teste.inet/logs/access.log;
        error_log /srv/www/teste.inet/logs/error.log;
        root /srv/www/teste.inet/public_html;
        location / {
            index index.html index.htm index.php;
        }
        location ~ \.php$ {
            include /etc/nginx/fastcgi_params;
            fastcgi_pass  127.0.0.1:9000;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME /srv/www/teste.inet/public_html$fastcgi_script_name;
        }
    }

Criei um link simbólico e reiniciei o Nginx:

    cd /etc/nginx/sites-enabled/
    ln -s /etc/nginx/sites-available/teste.inet teste.inet
    service nginx restart

Criei o arquivo de teste...

    vim /srv/www/teste.inet/public_html/info.php

...com o conteúdo

    <?php
    phpinfo();

Alterei o dono:

    chown nginx:nginx -R /srv/www/teste.inet/public_html/

Ao testar também pude ver o PHP 5.4.4 rodando mas agora como fastcgi:

<img class="img-responsive img-thumbnail" title="PHP 5.4.4 rodando com Nginx e Fastcgi" alt="PHP 5.4.4 rodando com Nginx e Fastcgi" src='/assets/images/nginx.png' />

Agora vou testar mais coisas para depois escrever.

Até a próxima.
