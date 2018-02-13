+++
title = "Ruby on Rails rodando no Apache com Passenger"
categorias = ["Tecnologia"]
tags = ["Apache","Fedora","Linux","Passenger","Rails","Ruby","RVM"]
date = "2012-04-22T09:04:00-03:00"
+++

Antes de mais nada devo lembrar que nosso apache já está rodando e
funcionando com PHP e que já fizemos a [instalação do Ruby on Rails usando RVM](/2012/04/instalacao-do-ruby-on-rails-no-fedora-16-usando-rvm) .
Outra coisa que devo lembrar é que foi muito fácil.
Agora sim hands on.

<!--continua-->

Indo para o Home

    cd ~/

Instalamos a Gem Passenger

    gem install passenger

Instalamos o módulo do apache 2 seguindo os passos

    passenger-install-apache2-module -a

Criamos um arquivo de configuração no apache...

    sudo vim /etc/httpd/conf.d/passenger.conf

... com o conteúdo que o comando de instalação do módulo do apache retornou,
no nosso caso:

    LoadModule passenger_module /home/wsilva/.rvm/gems/ruby-1.9.3-p194/gems/passenger-3.0.12/ext/apache2/mod_passenger.so
    PassengerRoot /home/wsilva/.rvm/gems/ruby-1.9.3-p194/gems/passenger-3.0.12
    PassengerRuby /home/wsilva/.rvm/wrappers/ruby-1.9.3-p194/ruby

Pronto está feito. Só reiniciar o apache:

    sudo service httpd restart

Criamos um projeto de teste no diretório do apache (seu usuário tem que
ter permissão de escrita)

    cd /var/www/html
    rails new testandoapache

Criamos um VirtualHost para testar a aplicação:

    sudo vim /etc/httpd/conf.d/vhost-testandoapache.inet.conf

Com o conteúdo:

    <VirtualHost *:80>
        ServerName testandoapache.inet
        DocumentRoot /var/www/html/testandoapache/public
        <Directory /var/www/html/olarails/testandoapache>
            AllowOverride all
            Options -MultiViews +Indexes
        <Directory>
        RailsEnv development
        CustomLog /etc/httpd/logs/testandoapache.inet.log combined
        ErrorLog /etc/httpd/logs/testandoapache.inet.error.log
    <VirtualHost>

Reiniciamos novamente o apache:

    sudo service httpd restart

Alteramos o arquivo de host para apontar para o domínio usado no Virtual Host:

    sudo vim /etc/hosts

Adicionando a linha abaixo no final do arquivo:

    127.0.0.1           testandoapache.inet

Pronto, abra o seu navegador e acesse *http://testandoapache.inet*

<img class="img-responsive img-thumbnail" title="Welcome Screenshot" src='/assets/images/ror_welcome_aboard.png' />

A página que carrega tem conteúdo estático, clicando em
*About your application's environment* você vai ver as informações
pertinentes a sua aplicação que foram geradas pelo ruby on rails.

Té mais
