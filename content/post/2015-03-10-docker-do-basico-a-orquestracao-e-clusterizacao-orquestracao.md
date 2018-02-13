+++
title = "Docker, do Básico a Orquestração e Clusterização - 4. Orquestração"
slug = "docker-do-basico-a-orquestracao-e-clusterizacao-orquestracao"
categories = ["Tecnologia"]
tags = ["Linux","Virtualização","Docker","Hub","Containers","Fig","Build"]
date = "2015-03-10T05:48:00-03:00"
+++

Nessa série de artigos estamos abordando tópicos para uma boa utilização do [Docker](http://www.docker.com/) .
<img class="img-responsive img-thumbnail pull-left" title="Thug Orchestra" alt="Thug Orchestra" src='/assets/images/thug-orchestra.jpg' />

No artigo anterior abordamos duas maneiras de construir um container, algumas dicas para montagem e utilização. Agora nos perguntamos montamos um "containerzão" com todos os serviços que minha aplicação precisa para rodar ou montamos vários "containerzinhos" um para cada serviço da aplicação.

Sugiro que sempre monte de acordo com a sua arquitetura em produção, quanto mais "live/production" nosso ambiente de desenvolvimento está, menos surpresas teremos em nossas entregas.

<!--continua-->

Quando você está trabalhando em uma aplicação cheia de serviços acoplados como banco, memória, busca elástica entre outros serviços acabamos montando tudo no mesmo Virtualbox, ou no mesmo Vagrant, por que se subirmos uma VM para cada serviço nossa máquina host vai pro Goiás. Toda hora vai ficar travando e coisa do gênero.

Com Docker a coisa muda de figura, conseguimos subir vários container na mesma máquina e ligamos eles à nossa maneira na mesma rede. Por exemplo podemos ter um web server que deva se comunicar diretamente com um servidor de banco de dados, então eles devem se enxergar dentro da rede.

Por sua vez o sevidor de banco de dados não precisa se comunicar com uma aplicação de elastic search por exemplo.

## Orquestração Vida Loka

A orquestração na unha dá muito trabalho para manter.
Neste exemplo vamos subir um container rodando um banco de dados MySQL.

~~~ bash
$ docker run -d --name mysql wfsilva/mysql
~~~

Antes de rodar o container de Web vamos apagar o antigo

~~~ bash
$ docker rm -f web
~~~

Agora sim podemos subir o container da aplicação com link para o container de banco de dados:

~~~bash
docker run -it --name web --link mysql:mysql wfsilva/nginx-php-fpm-docker /run.sh
~~~

Neste momento temos 2 containers rodando *wfsilva/nginx-php-fpm-docker* e *wfsilva/mysql*.

Se entrarmos no container "web" conseguiremos acessar o container "mysql" como se fossem 2 servidores na mesma rede:

~~~bash
$ docker exec -ti wfsilva/nginx-php-fpm-docker bash
# ping mysql
PING localhost (172.17.0.5): 48 data bytes
56 bytes from 172.17.0.5: icmp_seq=0 ttl=64 time=0.064 ms
56 bytes from 172.17.0.5: icmp_seq=1 ttl=64 time=0.125 ms
56 bytes from 172.17.0.5: icmp_seq=2 ttl=64 time=0.092 ms
56 bytes from 172.17.0.5: icmp_seq=3 ttl=64 time=0.096 ms
^C
~~~

## Orquestração com o Fig (Hoje internalizado como Docker Compose)
<img class="img-responsive img-thumbnail pull-right" title="Fig Logo" alt="Fig Logo" src='/assets/images/fig-logo.png' />

O [Fig](http://www.fig.sh/)  é um carinha feito em python que foi inventado para facilitar o uso de vários containers interligados. Ele foi internalizado pelo pessoal do Docker e agora se chama [docker-compose](http://docs.docker.com/compose/) . Ele junto com o docker-machine e o docker-swarm são as grandes novidades anunciadas na DockerCon EU 2014 que ocorreu em dezembro de 2014.

Para instalar bastava rodar o comando:

~~~bash
$ pip install fig
~~~

Agora temos as seguintes opções:

~~~bash
$ curl -L https://github.com/docker/compose/releases/download/1.1.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
~~~

ou

~~~bash
$ pip install -U docker-compose
~~~

Com o fig / docker-compose todos os containers que rodam sua aplicação ficam descritos em um arquivo yml onde os nós principais são os containers e os nós dentro de cada container representam suas propriedades.

Ex. de docker-compose.yml:

~~~yml
web:
  build: .
  command: /run.sh
  ports:
    - "80:80"
  volumes:
    - .:/src
  links:
    - mysql
mysql:
  image: mysql
  ports: "3306:3306"
~~~

O arquivo fala por si só. Ele descreve 2 containers nomeados web e mysql.

O container web vai contruir a partir de um dockerfile que steja localizado no mesmo path onde o arquivo docker-compose.yml está. Vai rodar o run.sh quando iniciar. Vai mapear a porta 80 para a porta 80 do docker host, desta maneira podemos acessá-lo como se fosse um serviço local através do ip 127.0.0.1 . Vai montar o diretório que está para dentro do /src no container e vai ligar com o container mysql.

O container mysql será baixado pronto do repositório no dockerhub e vai mapear a porta 3306 para a porta 3306 do docker host.

Para contruir um container rodamos fig build <nome do container> ou docker-compose <nome do container>. Ex:

~~~bash
$ fig build web
$ # ou
$ docker-compose build web
~~~

Para contruir todos os containers do arquivo yml basta rodar sem parâmetros:

~~~bash
$ fig build
$ # ou
$ docker-compose build
~~~

Para iniciar ou encerrar um ou mais containers basta passar os nomes:

~~~bash
$ fig start web mysql
$ fig stop mysql
$ # ou
$ docker-compose start web mysql
$ docker-compose stop mysql
~~~

Para subir o grupo todo utilizamos o parâmetro up que além de subir os containers vai mostrar os logs de todos os containers na tela. Para finalizar basta dar um ctrl+c. Ou ao iniciar passamos o parâmetro *-d* que não mostrará os logs na tela.

~~~bash
$ fig up -d
$ # ou
$ docker-compose up -d
~~~

Para acessar os logs de execução dos containers:

~~~bash
$ fig logs
$ # ou
$ docker-compose logs
~~~

Outra coisa bem legal é o scale. Se temos um container que só expõe a porta (EXPOSE 80) e não mapeia (EXPOSE 80:80) podemos escalar ele:

~~~bash
$ fig scale web=8
$ # ou
$ docker-compose scale web=8
~~~

Teremos 8 instâncias do container web rodando. Cada uma em um ip diferente e expondo a porta 80. No docker host essas portas 80 estarão mapeadas em portas altas.

Abaixo veremos um exemplo de yml onde utilizo um container chamado proxy que vi implementado por [Jason Wilder](https://github.com/jwilder) .

Nesse container ele mapeia o */var/run/docker.sock* da docker host para */tmp/docker.sock* dentro do container. No container também tem um binário forego, um script em go que monitora esse arquivo mapeado */tmp/docker.sock* e se há alterações ele usa o docker-gen para reconstruir um nginx.conf, se baseando num arquivo de template.

Por fim ele faz um reload do Nginx server e com isso esse Nginx acaba fazendo proxy pass e load balance para os containers monitorados.

O que acham de um yml desse:

~~~yml
proxy:
    build: ./proxy
    volumes:
        - /var/run/docker.sock:/tmp/docker.sock
    ports:
        - "80:80"
redis:
    build: ./redis
    privileged: true
    volumes:
        - ./redis:/data
    ports:
        - "6379:6379"
    entrypoint: redis-server
rabbit:
    image: tutum/rabbitmq
    environment:
        - RABBITMQ_PASS=teste
    ports:
        - "5672:5672"
        - "15672:15672"
solr:
    build: ./solr
    privileged: true
    volumes:
        - ./src/solr:/src/solr
    ports:
        - "8983:8983"
    entrypoint: /start-solr.sh
mysql:
    build: ./mysql
    privileged: true
    ports:
        - "3306:3306"
    environment:
        - MYSQL_ROOT_PASSWORD=teste
        - MYSQL_USER=teste
        - MYSQL_PASSWORD=teste
        - MYSQL_DATABASE=teste_db
web:
    build: ./web
    environment:
        - VIRTUAL_HOST=teste.dev
    links:
        - redis:redis
        - rabbit:rabbit
        - solr:solr
        - mysql:mysql
    privileged: true
    volumes:
        - ./src:/src
    ports:
        - "80"
    entrypoint: /run.sh
~~~

Pois é, se rodamos o comando:

~~~bash
$ fig scale web=5
$ # ou
$ docker-compose scale web=5
~~~

O proxy será regenerado e começa a fazer um load balance dos requests feitos para o domínio *teste.dev* entre todos o containers web que estiverem rodando.

Isso é ótimo para testar se a aplicação suporta load balance entre diversos servidores.

Mais detalhes dessa suruba de containers desse arquivo yml no meu [github](https://github.com/wsilva/figpoc) .


## To Be Continued
Ainda abordaremos como utilizar o docker em ambiente Windows e Mac (boot2docker e docker-compose) e o gerenciador de clusters de containers: docker-swarm.

Té +
