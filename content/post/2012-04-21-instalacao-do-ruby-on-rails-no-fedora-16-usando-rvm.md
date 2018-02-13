+++
title = "Instalação do Ruby on Rails no Fedora 16 usando RVM"
categories = ["Tecnologia"]
tags = ["Fedora","Gems","Instalação","Linux","Rails","Ruby","RVM"]
date = "2012-04-21T19:09:00-03:00"
+++

Olá cambada O que eu vou guar dar pra não esquecer hoje é a instalação do
Ruby on Rails no meu Fedora 16 usando o [RVM](http://rvm.io/) 
(Ruby Version Manager). Foi bem simples, primeiro temos que ter o CURL
instalado, se não tiver, baba:

    sudo yum install curl

Depois é só seguir as instruções do quick install no site:

<!--continua-->

No terminal executamos como usuário normal

    curl -L get.rvm.io | bash -s stable

Após instalar recarregamos o shell com o comando:

    source ~/.rvm/scripts/'rvm'

...ou fechamos e abrimos novamente a janela de terminal.
Rodamos o comando:

    rvm requirements

Ele vai listar o que é necessário instalar antes de continuar. Basicamente:

    yum install  gcc-c++ patch readline readline-devel zlib zlib-devel \
    libyaml-devel libffi-devel openssl-devel make bzip2 autoconf \
    automake libtool bison iconv-devel

Após instalar e configurar o RVM rodamos comando:

    type rvm | head -1

Deve retornar a mensagem:

    rvm is a function

Se não retornar temos que editar nosso profile e essa parte não entendi muito
bem qual deve ser editado. Na dúvida adicionei a linha abaixo...

    [[ -s "$HOME/.rvm/scripts/rvm" ]] && source "$HOME/.rvm/scripts/rvm"

no final dos arquivos *~/.bash_profile* e  *~/.bashrc*
Após alterar fechamos o terminal, abrimos de novo e executamos:

    type rvm | head -1

...que agora deve retornar:

    rvm is a function

Em seguida só alegria:

    rvm install 1.9.3
    rvm use 1.9.3 –default
    gem update
    gem install rails

Pequeno parenteses, se for usar o banco sqlite instale antes os pacotes
para dev com o comando:

    sudo yum install sqlite-devel

E em seguida instale a gem

    gem install sqlite3-ruby

Pronto, só criar seus projetos.
Abaixo segue um exemplo, o famoso HELLO WORLD:

Para acesar seu home

    cd ~/

Criando o projeto

    rails new olamundorails

Entrando no projeto

    cd olamundorails/

Atualizando gems

    bundle install

Criando o controller *mandaoi*

    rails generate controller mandaoi

Editamos o controller

    vi app/controllers/mandaoi_controller.rb

Inserimos o conteúdo

    class SalutationController < ApplicationController
            def ola
                    @msg = 'Ola cambada!!!'
            end
    end

Criamos a view

    vi app/views/mandaoi/ola.html.erb

Com o conteúdo:

    <html>
        <body>
            <h2><%=@msg%></h2>
        </body>
    </html>

Liberamos a rota editando o arquivo:

    vi config/routes.rb

... e descomentando a linha:

    match ':controller(/:action(/:id))(.:format)'

Para ver sua primeira aplicação:

    rails server

E no browser acesse *http://localhost:3000/mandaoi/ola*

De volta ao terminal use *CTRL+C* para parar o servidor.

Té + cambada!!!
