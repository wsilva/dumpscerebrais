+++
title = "Docker, do Básico a Orquestração e Clusterização - 3. Montando containers"
slug="docker-do-basico-a-orquestracao-e-clusterizacao-montando-containers"
categories = ["Tecnologia"]
tags = ["Linux","Virtualização","Docker","Hub","Containers","Build"]
date = "2015-03-07T23:57:00-03:00"
thumbnail = "/assets/images/docker-uhu.jpg"
+++

<!-- <img class="img-responsive img-thumbnail pull-left" title="Docker Logo" alt="Docker Logo" src='/assets/images/docker-uhu.jpg' /> -->
Nessa série de artigos estamos abordando tópicos para uma boa utilização do [Docker](http://www.docker.com/) .

Dando continuidade ao artigo anterior vamos abordar a criação "on the fly" de containers para rodar sua aplicação, a criação utilizando receitas em arquivos Dockerfile e algumas dicas para montar um bom Dockerfile para sua aplicação.

<!--continua-->

## Montando container "na unha"

Primeiramente vamos para a montagem de um container. O jeito mais simples onde você consegue ver de maneira direta o que está acontecendo é criando na hora, "on the fly", passo a passo até o container estar pronto.

Seu serviço do docker tem que estar rodando
Para iniciar no Linux:

~~~ bash
$ sudo service docker start
~~~

Para iniciar no Mac e no Windows

~~~ bash
$ boot2docker start
~~~

*Antes que me perguntem sobre o docker machine, escreverei sobre ele em breve abordando as novidades do mundo docker.*

Nesse exemplo vamos começar baixando um container do Debian 7:

~~~ bash
$ docker pull debian
debian:latest: The image you are pulling has been verified
511136ea3c5a: Already exists
30d39e59ffe2: Already exists
c90d655b99b2: Already exists
Status: Image is up to date for debian:latest
~~~

Entramos no container para começar a "montagem":

~~~ bash
$ docker run -ti debian bash
root@ba11f3c8394c:/#
~~~

Podemos perceber que estamos agora no bash dentro do container do Debian:

~~~bash
root@ba11f3c8394c:/# cat /etc/issue
Debian GNU/Linux 7 \n \l
~~~

Vamos agora cria o diretório para os fontes, atualizar os fontes, instalar o wget, instalar os repositórios dotdeb, instalar nginx, php5.4, o fpm tudo com os comandos que estamos acostumados para subir um servidor:

~~~bash
root@ba11f3c8394c:/# mkdir /src && apt-get update && apt-get install -y wget
root@ba11f3c8394c:/# echo "deb http://packages.dotdeb.org wheezy all" > /etc/apt/sources.list.d/dotdeb.list && echo "deb-src http://packages.dotdeb.org wheezy all" >> /etc/apt/sources.list.d/dotdeb.list && wget -O - http://www.dotdeb.org/dotdeb.gpg |apt-key add -
root@ba11f3c8394c:/# apt-get install php5-cli php5-fpm php5-mysql php5-intl php5-xdebug php5-recode php5-snmp php5-mcrypt php5-memcache php5-memcached php5-imagick php5-curl php5-xsl php5-snmp php5-dev php5-tidy php5-xmlrpc php5-gd php5-pspell php-pear nginx
2 upgraded, 162 newly installed, 0 to remove and 6 not upgraded.
Need to get 103 MB of archives.
After this operation, 289 MB of additional disk space will be used.
Do you want to continue [Y/n]? Y
~~~

Depois de tudo instalado vamos configurar o php.ini o fpm e o nginx:

~~~ bash
root@ba11f3c8394c:/# sed -i "s/;date.timezone =/date.timezone = America\/Sao_Paulo/" /etc/php5/cli/php.ini \
    && sed -i "s/;date.timezone =/date.timezone = America\/Sao_Paulo/" /etc/php5/fpm/php.ini \
    && sed -i "s/short_open_tag = On/short_open_tag = Off/" /etc/php5/cli/php.ini \
    && sed -i "s/short_open_tag = On/short_open_tag = Off/" /etc/php5/fpm/php.ini \
    && sed -i "s/error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT/error_reporting = E_ALL/" /etc/php5/cli/php.ini \
    && sed -i "s/error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT/error_reporting = E_ALL/" /etc/php5/fpm/php.ini \
    && sed -i "s/display_errors = Off/display_errors = On/" /etc/php5/cli/php.ini \
    && sed -i "s/display_errors = Off/display_errors = On/" /etc/php5/fpm/php.ini \
    && sed -i "s/display_startup_errors = Off/display_startup_errors = On/" /etc/php5/cli/php.ini \
    && sed -i "s/display_startup_errors = Off/display_startup_errors = On/" /etc/php5/fpm/php.ini
~~~

*Se quiser pode instalar o vi ou nano para editar os arquivos de configurações que mencionamos.*

Por fim criamos um virtual host teste.dev

~~~bash
root@ba11f3c8394c:/# cat << FIM >  /etc/nginx/sites-available/teste.dev.conf
server {

  server_name teste.dev;

  client_max_body_size  5m;
  client_header_timeout 1200;
  client_body_timeout   1200;
  send_timeout          1200;
  keepalive_timeout     1200;

  root        /src/;
  try_files   $uri $uri/ /index.php?$args;
  index       index.php;

  location ~ \.php$ {
    fastcgi_pass    unix:/var/run/php5-fpm.sock;
    fastcgi_index   index.php;
    include         fastcgi_params;

    fastcgi_connect_timeout 1200;
    fastcgi_send_timeout    7200;
    fastcgi_read_timeout    7200;
    fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name;
  }
}
FIM
~~~

E ativamos ele nos sites enable:

~~~bash
ln -sf /etc/nginx/sites-available/teste.dev.conf /etc/nginx/sites-enabled/teste.dev.conf
~~~


### Daemon vs no Daemon
Agora um passo que apanhei bastante e consegui resolver com a ajuda do Mário Rezende. Quando iniciamos um container ele executa o comando que informamos e ao encerrar o comando ele finaliza esse container.

Você pode confirmar fazendo o seguinte teste. Abra 2 terminais, e um execute:

~~~bash
$ docker run -it debian sleep 40 && echo fim
~~~

Isso vai subir o container debian, vai aguardar 40 segundos vai imprimir fim e vai encerrar o container

Antes de acabar os 40 segundos em outro terminal execute várias vezes o comando *docker ps*

~~~bash
$ docker ps
CONTAINER ID        IMAGE                COMMAND             CREATED             STATUS              PORTS               NAMES
a233a963c968        debian:latest        "sleep 40"          2 hours ago         Up 34 seconds
~~~

Primeiro aparece o container em execução depois ele some:

~~~bash
$ docker ps
CONTAINER ID        IMAGE                COMMAND             CREATED             STATUS              PORTS               NAMES
~~~

Podemos listar o último container executado com o parâmetro -l:

~~~bash
$ docker ps -l
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
a233a963c968        debian:latest       "sleep 40"          2 hours ago         Exited (0) 9 seconds ago                       nostalgic_wozniak
~~~

Com isso na cabeça podemos entender porque quando subimos um container logo após o Nginx iniciar com sucesso ele finaliza (o container todo finaliza).

Sabendo disso vamos rodar o Nginx como um executável normal, não como daemon e para isso no nosso container, que ainda está rodando porque no comando inicial mandamos rodar o bash (e o bash ainda está rodando), vamos mudar uma configuração no nginx.conf:

~~~bash
root@ba11f3c8394c:/# sed -i "s/www-data;/www-data;\\ndaemon off;/g" /etc/nginx/nginx.conf
~~~

Também vamos criar um script básico, maroto para rodar nosso container:

~~~bash
root@ba11f3c8394c:/# cat << FIM >  /run.sh
#!/bin/bash
/etc/init.d/php5-fpm restart && nginx
FIM
root@ba11f3c8394c:/# chmod a+x /run.sh # tornando executável
~~~

Container pronto vamos pegar o container id, gerar uma nova imagem e enviar para o docker hub:

~~~bash
root@ba11f3c8394c:/# exit
$ docker ps -l
CONTAINER ID        IMAGE                         COMMAND             CREATED             STATUS                       PORTS               NAMES
ba11f3c8394c        wfsilva/nginx-phpfpm:latest   "bash"              2 minutes ago
$ docker commit ba11f3c8394c wfsilva/nginx-phpfpm
aca05520d01b7057baacd4f7f9e9342c1c888ed063c4a1fd557a6d206999c65c
$ docker push wfsilva/nginx-phpfpm
~~~

Pronto sempre que quiser essa imagem é só rodar:

~~~bash
docker run -p 80:80 -v /caminho/para/meu/projeto/php:/src wfsilva/nginx-phpfpm /run.sh
~~~

Mesmo que o container não esteja na minha máquina ele será baixado do meu repositório no docker hub.

O caminho para os fontes do projeto (opção -v) vai ser montada em */src* que também foi configurado como path para nosso virtualhost. A porta 80 foi mapeada para que possamos acessar e o comando que executamos foi o script *run.sh* que inicia o php-fpm e o Nginx.

Podemos adicionar o nosso virtualhost em nosso arquivo de hosts
Se for mac sugiro usar o boot2docker ip ou o docker-machine ip:

~~~bash
echo "`boot2docker ip` teste.dev" | sudo tee -a  /etc/hosts
echo "`docker-machine ip` teste.dev" | sudo tee -a  /etc/hosts
~~~

Se for linux:

~~~bash
echo "127.0.0.1 teste.dev" | sudo tee -a  /etc/hosts
~~~

Se for Windows, bom você deve saber onde fica seu arquivo de hosts. Se não souber pode perguntar.

Acessando *http://testes.dev* batemos em nosso container que responde acessando os arquivos mapeados em nossa maquina "host".

## Montando container com Dockerfile

<img class="img-responsive img-thumbnail pull-left" title="Cool meme" alt="Cool meme" src='/assets/images/cool.png' />

Para mim esse é o jeito mais legal porque com um simples arquivo e com instruções simples montamos um container. Todos os comandos possíveis, e não são muitos, num Dockerfile estão em [https://docs.docker.com/reference/builder/](https://docs.docker.com/reference/builder/) .

Também acho legal porque posso montar um repositório no Github ou no Bitbucket com meu Dockerfile e demais arquivos que ele precisa para "buildar" (eu e esses neologismos gringos, aff) e criar um repositório no Dockerhub apontando para esse repositório Git.
O Dockerhub vai procurar esses arquivos e tentar buildar um container, e a cada push feito para seu Github ou Bitbucket criado o Dockerhub vai tentar buildar novamente.

Como exemplo segue esse Dockerfile disponível em [https://github.com/wsilva/nginx-php-fpm-docker](https://github.com/wsilva/nginx-php-fpm-docker) 

~~~
FROM    debian
MAINTAINER Wellington Silva <****@***.br>

# Keep upstart from complaining
RUN dpkg-divert --local --rename --add /sbin/initctl && ln -sf /bin/true /sbin/initctl

# Let the conatiner know that there is no tty
ENV DEBIAN_FRONTEND noninteractive

# installing wget and creating /src path
RUN apt-get update && apt-get install -y wget && mkdir /src

# add dotdeb source list
RUN echo "deb http://packages.dotdeb.org wheezy all" > /etc/apt/sources.list.d/dotdeb.list \
    && echo "deb-src http://packages.dotdeb.org wheezy all" >> /etc/apt/sources.list.d/dotdeb.list \
    && wget -O - http://www.dotdeb.org/dotdeb.gpg |apt-key add -

# installing php stuff
RUN apt-get update && apt-get install -y php5-cli php5-fpm php5-mysql php5-intl php5-xdebug php5-recode \
    php5-snmp php5-mcrypt php5-memcache php5-memcached php5-imagick php5-curl php5-xsl php5-snmp \
    php5-dev php5-tidy php5-xmlrpc php5-gd php5-pspell php-pear php-apc nginx

# setting up php.ini, fpm pool conf and nginx.conf
RUN sed -i "s/;date.timezone =/date.timezone = America\/Sao_Paulo/" /etc/php5/cli/php.ini \
    && sed -i "s/;date.timezone =/date.timezone = America\/Sao_Paulo/" /etc/php5/fpm/php.ini \
    && sed -i "s/short_open_tag = On/short_open_tag = Off/" /etc/php5/cli/php.ini \
    && sed -i "s/short_open_tag = On/short_open_tag = Off/" /etc/php5/fpm/php.ini \
    && sed -i "s/error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT/error_reporting = E_ALL/" /etc/php5/cli/php.ini \
    && sed -i "s/error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT/error_reporting = E_ALL/" /etc/php5/fpm/php.ini \
    && sed -i "s/display_errors = Off/display_errors = On/" /etc/php5/cli/php.ini \
    && sed -i "s/display_errors = Off/display_errors = On/" /etc/php5/fpm/php.ini \
    && sed -i "s/display_startup_errors = Off/display_startup_errors = On/" /etc/php5/cli/php.ini \
    && sed -i "s/display_startup_errors = Off/display_startup_errors = On/" /etc/php5/fpm/php.ini \
    && sed -i "s/www-data;/www-data;\\ndaemon off;/g" /etc/nginx/nginx.conf

COPY run.sh /run.sh
RUN chmod a+x /run.sh

# virtualhosts configuration
COPY teste.dev.conf /etc/nginx/sites-available/teste.dev.conf

RUN ln -sf /etc/nginx/sites-available/teste.dev.conf /etc/nginx/sites-enabled/teste.dev.conf

# removing deb files
RUN apt-get clean

EXPOSE 80:80

ENTRYPOINT ["/run.sh"]
~~~

Esse é o run.sh que é copiado para dentro do container durante o build, também disponível no Github:

~~~
#!/bin/bash
/etc/init.d/php5-fpm restart && nginx
~~~

E esse o teste.dev.conf, também copiado para dentro:

~~~
server {

  server_name teste.dev;

  client_max_body_size  5m;
  client_header_timeout 1200;
  client_body_timeout   1200;
  send_timeout          1200;
  keepalive_timeout     1200;

  root        /src/;
  try_files   $uri $uri/ /index.php?$args;
  index       index.php;

  location ~ \.php$ {
    fastcgi_pass    unix:/var/run/php5-fpm.sock;
    fastcgi_index   index.php;
    include         fastcgi_params;

    fastcgi_connect_timeout 1200;
    fastcgi_send_timeout    7200;
    fastcgi_read_timeout    7200;
    fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name;
  }
}
~~~

O repositório no Dockerhub que monitora esse github é o [https://registry.hub.docker.com/u/wfsilva/nginx-php-fpm-docker/](https://registry.hub.docker.com/u/wfsilva/nginx-php-fpm-docker/).

Se você não quiser utilizar o Dockerhub basta ir até a pasta onde gravou o seu Dockerfile e rodar o comando:

~~~bash
$ docker build -t wfsilva/nginx-php-fpm-home-build ./
~~~

A opção -t é para criar a tag, e como pode imaginar a parte *wfsilva/nginx-php-fpm-home-build* foi por minha conta. E para a minha conta (no dockerhub).


### Dicas

 - Sempre use tags para construir seus container, senão você ficará louco com os hashes:

~~~ bash
$ docker build -t yabadaba/dooo ./path/to/wherever/dockerfile/is
~~~

 - Sempre use o cache de imagens - cada instrução do Dockerfile é reutilizada no build, portanto o build vai ser praticamente instantâneo até o ponto onde foi feita a mudança no seu dockerfile.

 - EXPOSE-ing ports - tentar não mapear portas públicas para evitar colisão com portas de serviços que estejam rodando no host. Ao invés de *-p 80:80* ou EXPOSE 80:80 utilizemos -p 80 ou EXPOSE 80

 - CMD e ENTRYPOINT -  sempre usar a sintaxe de array: *CMD ["/bin/echo"]* porque quando colocamos *CMD /bin/echo* automaticamente no container vai ser colocado um /bin/sh -c antes do comando.

 - Tentar usar CMD e ENTRYPOINT juntos - o entry point é tipo um binário, se nenhum parâmetro for passado o cmd seguinte será chamado. ex:

~~~ bash
ENTRYPOINT ["/usr/bin/script"]
CMD ["--help"]
~~~

Ao rodar o container só chamando o script */usr/bin/script* ele será executado com a opção *--help*. Se executarmos com alguma outra opção qualquer, *--verbose* por exemplo, o *--help* será ignorado.

 - Se você se cansar de testar e quiser fazer uma limpa geral *docker rm $(docker ps -a -q)* para remover todos containers parados e *docker rmi $(docker images -q)* para remover todas imagens.

 - Para mostrar histórico das imagens: *docker images --tree*.
 - docker run - inicia / executa o container.
 - docker exec - executa um outro comando ou entra em um container que já esteja rodando.
 - docker attach - te anexa no shell do container que esteja rodando
 - Reforçando Container vs Imagem - Container é a "imagem que está rodando", não vai esquecer mais.

## To Be Continued
Ainda abordaremos como utilizar o docker em ambiente Windows e Mac (boot2docker e docker-machine), orquestração de containers usando fig (agora docker-composer), e as novidades (já em beta e liberadas para teste) anunciadas na Dockercon em dezembro de 2014 para a próxima versão do docker: (docker-machine, docker-composer, docker-swarm).

Té +
