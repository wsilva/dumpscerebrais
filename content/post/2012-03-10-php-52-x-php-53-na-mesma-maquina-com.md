+++
title = "PHP 5.2 x PHP 5.3 na mesma máquina com xdebug e virtualbox"
slug = "php-52-x-php-53-na-mesma-maquina-com"
categories = ["Tecnologia"]
tags = ["Developer","Linux","PHP","SSH","Virtual Box","Xdebug"]
date = "2012-03-10T12:02:00-03:00"
+++

Saudações

Pra falar a verdade eu não gostei desta solução mas algumas me pessoas pediram,
esse site ainda está meio sem conteúdo então vou descrever a solução porca que
estou usando para rodar um PHP 5.2 e um PHP 5.3 na mesma máquina.

<!--continua-->

Tudo começou quando fui chamado para coordenar uma equipe de desenvolvimento e
alguns dos projetos rodavam usando PHP 5.2 e outros PHP 5.3 (das máquinas
utilizadas para desenvolver umas rodam Ubuntu 11.10 outras 10.04 e algumas o
9.10).

Se precisar ficar meter a mão no código de um projeto PHP 5.2 é fácil, só fazer
o downgrade e boa eu também já sabia como fazer o downgrade (outra hora eu
publico como fiz mas por enquanto tentem o [google](https://www.google.com.br/search?hl=pt-BR&q=downgrade+php5.2+ubuntu+karmic+source.list))
publicado [aqui](/2012/03/downgrade-simples-de-php-53-para-php-52). Se precisar
meter a mão no código de um projeto PHP 5.3 também era fácil, tirava as travas e
 upgrade na instalação. Porém é um saco ficar configurando a toda hora que muda
 a instalação do PHP: php.ini, variáveis debug (erros, notices, warnings),
 tamanhos de upload, memória utilizada, memcache, apc, gd, ssl xdebug, curl,
 cli, etc, etc, etc...

Eu e outro coordenador tentamos juntos durante alguns dias compilar fontes e
montar diferentes soluções onde instalações de php bem personalizadas e definir
em cada virtual host qual versão de interpretador seria utilizado, e funcionou,
mas só em ubuntu 32 bits.

Apelamos e fizemos o seguinte:

Baixamos o pacote  de instalação do VirtualBox direto pelo site
(no dia a versão era 4.1.8).

Instalamos o VirtualBox

~~~ bash
    dpkg -i virtualbox-4.1_4.1.8-75467~Ubuntu~oneiric_amd64.deb
~~~

Adicionamos o usuário da máquina ao grupo do virtual box

~~~ bash
    sudo adduser nomedeusuario vboxusers
~~~

Baixamos a versão server do Ubuntu 9.10 (Karmic)

~~~ bash
    wget -c http://old-releases.ubuntu.com/releases/karmic/ubuntu-9.10-server-amd64.iso
~~~

é, usamos a de 64 bits mesmo.

Criamos uma máquina virtual no VirtualBox com a imagem baixada e ligamos ela.

Como o Karmic é uma versão antiga não mais suportada então alteramos o
source.list para apontar para os servidores antigos. Na máquina virtual usamos
o comando

~~~ bash
    sed -i "s/br\.archive/old-releases/g" /etc/apt/source.list
~~~

(atenção se você não instalou a localidade como brasil altere manualmente o arquivo)
e

~~~ bash
    sed -i "s/security\.ubuntu/old-releases\.ubuntu/g" /etc/apt/source.list
~~~

para corrigir os fontes.
Basicamente trocamos os fontes de br.archive.ubuntu para br.archive.ubuntu
para old-releases.ubuntu e  security.ubuntu para old-releases.ubuntu

Atualizamos os fontes

~~~ bash
    sudo apt-get update
~~~

Instalamos os pacotes

~~~ bash
    sudo apt-get install aptitude build-essential ssh openssl curl memcached apache2 php5 php5-cli php5-mysql php5-memcache php5-memcached php-apc php5-xdebug php5-gd php5-curl php5-dev
~~~

Apesar de usar bancos MySQL nos projetos não instalamos na máquina virtual,
ficou mais fácil apontar os fontes para buscar o banco na instalação da
máquina real.

Durante esse último passo um nespresso vai muito bem.

Após a instalação criamos uma pasta na máquina virtual para colocar os projetos

~~~ bash
    sudo mkdir -p /var/www/php52
~~~

Alteramos o dono da pasta

~~~ bash
    sudo chown nomedeusuario:www-data /var/www/php52
~~~

Alteramos o php.ini com as configurações que precisávamos, verifique quais as
opções que você vai precisar nesse passo.

~~~ bash
    sudo vim /etc/php5/apache2/php.ini
~~~

As opções que alteramos foram: `short_open_tag = Off, memory_limit = 128M,
error_reporting = E_ALL, display_errors = On, log_errors = On,
html_errors = On`, entre outros...

Preparamos o VirtualHost do projeto que vai rodar nesse servidor "virtual".
Normalmente eu crio um arquivo para cada VirtualHost mas isso é uma preferencia
pessoal.

~~~ bash
    sudo vim /etc/apache2/sites-available/site_a_ser_instalado
~~~

com o conteúdo:

~~~ bash
    <VirtualHost *:80>
        ServerAdmin     email@dominio.tst
        ServerName      dominiodositequeinstalei.inet
        ServerAlias     www.dominiodositequeinstalei.inet
        DocumentRoot    /var/www/php52/dominiodositequeinstalei
        <Directory /var/www/php52/dominiodositequeinstalei/>
                Options FollowSymLinks
                AllowOverride   All
        </Directory>
        ErrorLog        ${APACHE_LOG_DIR}/dominiodositequeinstalei.inet.error.log
        CustomLog       ${APACHE_LOG_DIR}/dominiodositequeinstalei.inet.access.log combined
    </VirtualHost>
~~~

Ativamos o site
~~~ bash
    sudo a2nsite site_a_ser_instalado
~~~

Ativamos o módulos que precisaremos no projeto.

~~~ bash
    sudo a2enmod rewrite
~~~

Neste projeto em particular só utilizamos o mod rewrite

Neste momento anotamos os ips da máquina real e da máquina virtual.
Vamos supor que 1.1.1.7 seja da máquina real e 1.1.1.8 seja a máquina virtual.

Na máquina virtual alteramos a configuração do xdebug

~~~ bash
    sudo vim /etc/php5/apache2/conf.d/xdebug.ini
~~~

Deixando da seguinte maneira:

~~~ bash
    zend_extension=/onde/esta/o/arquivo/xdebug.so
    xdebug.remote_enable=1
    xdebug.remote_host=1.1.1.7
    xdebug.remote_port=9000
    xdebug.remote_handler=dbgp
~~~

A primeira linha normalmente já vem com o caminho correto, é só adicionar as
demais linhas colocando o ip da máquina real na linha do remote host.

Por último na máquina virtual é só reiniciar os serviços, no nosso caso apache
e memcache, mas é provável que só o apache bastaria

~~~ bash
    sudo /etc/init.d/apache2 restart
~~~

Na máquina real alteramos o arquivo de host para encaminhar as chamadas no
domínio criado
~~~ bash
    sudo vim /etc/hosts
~~~

No final do arquivo adicionamos a linha abaixo com o ip da máquina virtual e o
domínio que criamos no virtualhost:

~~~ bash
    1.1.1.8 dominiodositequeinstalei.inet
~~~

Ainda na máquina real criamos uma pasta para manipular o site

~~~ bash
    sudo mkdir -p /var/www/php52
~~~

Alteramos o dono da pasta

~~~ bash
    sudo chown nomedeusuario:www-data /var/www/php52
~~~

Instalamos suporte para o sshfs

~~~ bash
    sudo apt-get install sshfs
~~~

Carregamos o módulo fuse

~~~ bash
    sudo modprobe fuse
~~~

Adicionamos o usuário ao grupo fuse

~~~ bash
    sudo adduser nomedeusuario fuse
~~~

Alteramos o dono do dispositivo fuse

~~~ bash
    sudo chown root:fuse /dev/fuse
~~~

E montamos a pasta da máquina virtual na máquina real

~~~ bash
    sshfs nomedeusuario@1.1.1.8:/var/www/php52 /var/www/php52
~~~

Atenção para montar a pasta da máquina real TEM que estar vazia, senão vai dar
merda.

Pronto coloquei o projeto dentro da pasta php52
(/var/www/php52/dominiodositequeinstalei) e comecei a trabalhar nas correções.
Nem precisei mudar nada no netbeans, na hora que rodo o debug o navegador
carrega o site hospedado na máquina virtual e o xdebug da máquina virtual faz o
meio campo me devolvendo linha a linha no netbeans da máquina real usando a
porta 9000.

Ainda vou tentar fazer rodar versões diferentes de PHP na mesma máquina sem
precisar de máquina vitual, afinal como dizia meu supervisor de estágio
Ricardinho anos atrás: "- Isso não é gambiarra, é um ajuste técnico."

té +


Se quiser xingar, xinga nos comments aqui em baixo
