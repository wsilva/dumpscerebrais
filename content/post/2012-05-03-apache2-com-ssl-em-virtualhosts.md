+++
title = "Apache2 com SSL em VirtualHosts"
categorias = ["Tecnologia"]
tags = ["Apache","Fedora","Linux","PHP","SSL","VirtualHost"]
date = "2012-05-03T20:29:00-03:00"
+++

Saudações Só pra não ficar sem escrever hoje precisei de SSL em apenas
um projeto para testar algumas funcionalidades e segue o que fiz.
Pesquisando no google achei um how to muito bom neste site. Foi só seguir
a risca a criança. Passo a passo:

<!--continua-->

Saudações

Só pra não ficar sem escrever hoje precisei de SSL em apenas um projeto
para testar algumas funcionalidades e segue o que fiz.

Pesquisando no google achei um how to muito bom neste site.
Foi só seguir a risca a criança.

## Passo a passo:

Instalando o módulo:

    yum install mod_ssl

Criando o diretório para armazenar as chaves

    mkdir /etc/httpd/ssl

Gerando uma chave local com uma validade de 365 dias

    openssl req -new -x509 -days 365 -nodes -out /etc/httpd/ssl/nome_do_projeto.pem -keyout /etc/httpd/ssl/nome_do_projeto.key

Responda as informações solicitadas sem utilizar acentuação e se quiser
deixar o campo em branco digite . (ponto)

Exemplo:

    Country Name (2 letter code) [XX]:BR
    State or Province Name (full name) [Some-State]:Sao Paulo
    Locality Name (eg, city) []:Sao Paulo
    Organization Name (eg, company) [Internet Widgits Pty Ltd]:Dumps Cerebrais Inc
    Organizational Unit Name (eg, section) []:Diretoria de servicos
    Common Name (eg, YOUR name) []:testedessl.meudominio.inet
    Email Address []:seu@email.com

Alterando a configuração do apache

    vim /etc/httpd/conf/httpd.conf

Se já estiver trabalhando com Virtual Host a linha abaixo deve estar
descomentada, senão descomente.

    NameVirtualHost *:80

Adicione a linha logo abaixo como no exemplo:

    NameVirtualHost *:80
    NameVirtualHost *:443

Grave a alteração e altere a configuração do virtual host com o domínio
que você aplicar o SSL

    vim /etc/httpd/conf.d/vhost-testedessl.meudominio.inet.conf

Geralmente eu crio em arquivos separados com o domínio no nome do arquivo
como podemos perceber acima.

Meu arquivo estava assim:

    <VirtualHost *:80>
        ServerName testedessl.meudominio.inet
        DocumentRoot /var/www/html/testedessl/
        <Directory /var/www/html/testedessl/>
            Options -Indexes FollowSymLinks
            DirectoryIndex index.php index.html
            AllowOverride All
            Order allow,deny
            Allow from all
        <Directory>
        CustomLog /etc/httpd/logs/testedessl.meudominio.inet.log combined
        ErrorLog /etc/httpd/logs/testedessl.meudominio.inet.error.log
    <VirtualHost>

Acrescentei as linhas abaixo:

    <VirtualHost *:443>
        ServerName testedessl.meudominio.inet
        SSLEngine On
        SSLCertificateFile /etc/httpd/ssl/nome_do_projeto.pem
        SSLCertificateKeyFile /etc/httpd/ssl/nome_do_projeto.key
        DocumentRoot /var/www/html/testedessl/
        <Directory /var/www/html/testedessl/>
            Options -Indexes FollowSymLinks
            DirectoryIndex index.php index.html
            AllowOverride All
            Order allow,deny
            Allow from all
        <Directory>
        CustomLog /etc/httpd/logs/testedessl.meudominio.inet.log combined
        ErrorLog /etc/httpd/logs/testedessl.meudominio.inet.error.log
    <VirtualHost>

Pronto, só reiniciar o apache

    service httpd restart

<img class="img-responsive img-thumbnail" title="no firefox" alt="no firefox" src='/assets/images/ssl.png' />
<img class="img-responsive img-thumbnail" title="adicionando exception" alt="adicionando exception" src='/assets/images/ssl2.png' />
<img class="img-responsive img-thumbnail" title="pop up adicionando exception" alt="pop up adicionando exception" src='/assets/images/ssl3.png' />
<img class="img-responsive img-thumbnail" title="carregado" alt="carregado" src='/assets/images/ssl4.png' />
<img class="img-responsive img-thumbnail" title="no chrome" alt="no chrome" src='/assets/images/ssl5.png' />
<img class="img-responsive img-thumbnail" title="após adicionar exception" alt="após adicionar exception" src='/assets/images/ssl6.png' />

Se quiser que seu certificado seja reconhecido pelas agências
certificadoras tipo **Verisign**, **Thawte**, **Globalsign** ou **Comodo**
os passos são um pouco diferentes.

Naquele [site](http://library.linode.com/web-servers/apache/ssl-guides/fedora-14){:targe="_blank"}
que informei explica melhor esses detalhes como a necessidade de um
IP dedicado, permissionamento do arquivo a ser enviado, etc.

Inté a próxima…
