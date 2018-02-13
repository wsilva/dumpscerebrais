+++
title = "Docker, do Básico a Orquestração e Clusterização - 5. Ambiente"
slug = "docker-do-basico-a-orquestracao-e-clusterizacao-machine"
categories = ["Tecnologia"]
tags = ["Linux","Virtualização","Docker","Hub","Containers","Windows","MacOS","Boot2docker"]
date = "2015-03-24T01:55:00-03:00"
thumbnail = "/assets/images/ship-shipping-ships.jpg"
+++

Nessa série de artigos estamos abordando tópicos para uma boa utilização do [Docker](http://www.docker.com/) .

<!-- <img class="img-responsive img-thumbnail pull-left" title="Ship shipping ships" alt="Ship shipping ships" src='/assets/images/ship-shipping-ships.jpg' /> -->

Para entender melhor a orquestração e antes de partirmos para a clusterização precisamos entender bem como usar o docker em nossa máquina de dia a dia, a nossa máquina dev, nosso ambiente de desenvolvimento.

Se sua máquina estiver rodando Linux, qualquer distribuição, tudo fica mais fácil. Por que wsilva? Porque como vimos nos artigos anteriores o Docker trabalha com LXC - Linux Containers. Bom, linux dispensa comentários mas container, fazendo um paralelo com a virtualização tradicional, seria o sistema operacional (Linux também no nosso caso) rodando como guest.

Pô wsilva mas então é Linux rodando em cima de Linux!?

Exatamente. E como também já vimos anteriormente a grande vantagem de usarmos LXC é não precisar do hypervisor (Virtualbox e afins), já que muitos recursos são compartilhados entre o Host e o Guest.

<!--continua-->

Aaaaa wsilva, então por isso que tem que ser Linux sobre Linux, senão esse compartilhamento não rola, certo!?

Certíssimo, rodar um linux container num host Windows acho quase impossível por causa da arquitetura, do Kernel diferente, etc. O mesmo também seria para o MacOS, mas devemos lembrar que MacOS vem do Unix. E o Linux vem do Minix que vem do Unix. Isso significa que por mais difícil que possa parecer pode pintar algum louco que consiga compartilhar o Kernel, libs e outros recursos de um host MacOS com containers Linux, ou até o contrário.

## No Linux

No linux tenho trabalhado apenas com o [Docker Compose](https://github.com/docker/compose) . Temos um fig.yml, que em breve teremos que renomear para docker-compose.yml, com os container, a instruções de cada container, os links e tudo mais.

Após rodar o fig up / docker-compose up nossos containers sobem como se estivessem em nossa máquina local. Podemos confirmar com o docker-compose ps.

~~~ bash
$ docker-compose up -d
fig.yml is deprecated and will not be supported in future. Please rename your config file to docker-compose.yml

Recreating projeto_proxy_1...
Recreating projeto_rabbit_1...
Recreating projeto_mysql_1...
Recreating projeto_solr_1...
Recreating projeto_redis_1...
Recreating projeto_web_1...
~~~

~~~ bash
$ docker-compose ps
fig.yml is deprecated and will not be supported in future. Please rename your config file to docker-compose.yml

     Name                Command          State                        Ports
--------------------------------------------------------------------------------------------------
projeto_mysql_1    /entrypoint.sh mysqld   Up      0.0.0.0:3306->3306/tcp
projeto_proxy_1    forego start -r         Up      443/tcp, 0.0.0.0:80->80/tcp
projeto_rabbit_1   /run.sh                 Up      0.0.0.0:15672->15672/tcp, 0.0.0.0:5672->5672/tcp
projeto_redis_1    redis-server            Up      0.0.0.0:6379->6379/tcp
projeto_solr_1     /start-solr.sh          Up      0.0.0.0:8983->8983/tcp
projeto_web_1      /start-web.sh           Up      0.0.0.0:49153->80/tcp
~~~

Podemos acessar por exemplo o dashboard do rabbit pelo ip de loopback: http://127.0.0.1:15672, o próprio painel do solr pelo http://127.0.0.1:8983/, ou até o redis sem passar o parâmetro com o host:

~~~ bash
redis-cli
127.0.0.1:6379>
~~~

## No MacOS

No MacOS (e creio que no Windows seja parecido) precisamos rodar uma máquina virtual linux com o serviço docker de pé e é esse linux que vai ser o docker host rodar os containers.

Até a postagem desse artigo a instalação oficial de docker para [MacOS](https://docs.docker.com/installation/mac/)  e para [Windows](https://docs.docker.com/installation/windows/)  era através do [boot2docker](http://boot2docker.io/) .

O boot2docker é um distribuição linux bem leve (aproximadamente 27MB) baseada no Tiny Core Linux que carrega em memória e já tem suporte para o file system (AUFS) requerido pelo docker e o próprio docker service.

Primeiro temos que criar nossa VM com o comando:

~~~ bash
$ boot2docker init
~~~

Depois podemos iniciar nossa VM com o comando

~~~ bash
$ boot2docker start
~~~

Sempre que iniciarmos uma janela de terminal nova no MacOS devemos exportar as variáveis de ambiente do Docker para que consigamos levantar nossos containers (que neste caso rodarão dentro do boot2docker). Basta rodar o comando:

~~~ bash
$ $(boot2docker shellinit)
~~~

Com as variáveis exportadas podemos subir nossos containers:

~~~ bash
$ docker-compose up -d
~~~

Para parar nossa VM:

~~~ bash
boot2docker stop
~~~

### Desvantagem
A desvantagem de não estar usando linux é que para acessar nossos containers temos que usar ip da VM, do boot2docker. Podemos pegar com o comando:

~~~ bash
$ boot2docker ip
~~~

E assim acessar nossos containers pelo ip. Ex.: http://{ip.do.boot2docker}:15672 (rabbit) ou http://{ip.do.boot2docker}:8983/ ou até concatenar comandos para acessar o container de redis por exemplo:

~~~ bash
redis-cli -h `boot2docker ip`
~~~

## Alternativas no MacOS

### Docker Machine

Quando disse que a até a data do post a instalação oficial era o boot2docker é porque agora temos uma "alternativa oficial" (sim entre aspas mesmo), o [docker-machine](https://docs.docker.com/machine/)  que não só provisiona VMs linux em nossa máquina dev através do Virtualbox e VMware ele também provisiona em outros ambientes Linux e em serviços cloud como o EC2 da Amazon, o Azure, a Digitalocean, o Google Engine, Openstack, Rackspace, Softlayer, etc.

Por que alternativa oficial entre aspas? Porque a VM que ele provisiona (pelo menos localmente) é um boot2docker, e sendo o docker-machine mais completo pode vir a substituir o processo de instalação em MacOS e Windows que atualmente na documentação é a instalação do boot2docker puro e simples.

Para criar uma VM na DigitalOcean por exemplo executamos:

~~~ bash
$ docker-machine create --driver digitalocean --digitalocean-access-token=tokenxptoyadaxptoyadatoken nome-da-vm
~~~

No nosso caso, para trabalhar localmente podemos criar uma VM no Virtual Box:

~~~ bash
$ docker-machine create -d virtualbox nome-da-vm
~~~

Se quisermos criar com mais recursos.

~~~ bash
$ docker-machine create --driver virtualbox --virtualbox-memory=2048 --virtualbox-disk-size=30000 nome-de-outra-vm
~~~

*Se não passados esses parâmetros os padrões adotados são 20000 MB de disco e 1024 MB de memória.*

Para listar as VMs criadas:

~~~ bash
$ docker-machine ls
NAME              ACTIVE   DRIVER       STATE     URL                         SWARM
nome-da-vm        *        virtualbox   Running   tcp://192.168.99.100:2376
nome-de-outra-vm           virtualbox   Running   tcp://192.168.99.101:2376
~~~

Para exportar as variáveis de ambiente em nosso shell:

~~~ bash
$ $(docker-machine env nome-da-vm)
~~~

Para parar e retormar uma vm. Se não passarmos o nome da VM ele entenderá que é a VM ativa:

~~~ bash
$ docker-machine stop nome-da-vm
$ docker-machine start nome-da-vm
~~~

Para mudar a VM ativa:

~~~ bash
$ docker-machine active nome-de-outra-vm
~~~

### Kitematic

Dois dias depois que escrevi a parte 4 desta série de posts, no dia 12 de março, o pessoal do Docker estava apresentando o [Kitematic](https://kitematic.com/)  como alternativa para Mac OS. Acompanhei o meet up online de apresentação do Kitematic no dia 17 de março e vou passar minhas impressões pessoais a respeito.

O Kitematic é um GUI (Graphical User Interface) baseado no docker-machine, escrito em Javascript. Parecido com o [Atom](https://atom.io/)  é baseado na engine do Google Chrome (da até para inspecionar elemento) e desta maneira pode ser portado para o Windows. Durante a parte das perguntas (Q&A) no meet up eles deixaram escapar que essa portabilidade já está no RoadMap deles (Além de diversas melhorias na integração com dockerhub).

Esse carinha facilita muito a criação e a manutenção de containers, e se tivermos uma orquestração via docker-compose, os container criados também aparecerão também na tela do Kitematic.

A interação se dá por botões simples e intuitivos. Temos a opção para entrar no console de um container que esteja rodando (por debaixo dos panos utiliza o comando *docker exec -ti container /bin/bash*), temos a opção de abrir no navegador a url, na verdade o ip, do container, temos a opção de alterar as configurações de execução de um containere outras opções.

Abrindo o Kitematic. A VM criada é iniciada por baixo dos panos:
<img class="img-responsive img-thumbnail center-block" title="Booting VM" alt="Booting VM" src='/assets/images/1-kitematic-booting.png' />

Criando container com base em imagens existentes no Dockerhub:
<img class="img-responsive img-thumbnail center-block" title="Creating Containers" alt="Creating Containers" src='/assets/images/2-creating-container.png' />

Iniciando o novo container:
<img class="img-responsive img-thumbnail center-block" title="Starting new container" alt="Starting new container" src='/assets/images/3-starting-new-container.png' />

Opções de visualizar log e preview no navegador:
<img class="img-responsive img-thumbnail center-block" title="Preview Log and in Browser" alt="Preview Log and in Browser" src='/assets/images/4-preview-log-browser.png' />

Configurações do container:
<img class="img-responsive img-thumbnail center-block" title="General Settings" alt="General Settings" src='/assets/images/5-general-settings.png' />

Configurações de exposição de portas:
<img class="img-responsive img-thumbnail center-block" title="Port Settings" alt="Port Settings" src='/assets/images/6-port-settings.png' />

Configurações de volumes a serem montados:
<img class="img-responsive img-thumbnail center-block" title="Volume Settings" alt="Volume Settings" src='/assets/images/7-volume-settings.png' />

### DVM
Alternativa para subir o docker no Mac OS [http://fnichol.github.io/dvm/](http://fnichol.github.io/dvm/) . Também é baseado numa VM rodando boot2docker e os containers dentro da VM.

## Ferramentas para gerenciamento de docker containers

São ferramentas para gerenciar a criação e até orquestração de containers

### Shipyard
É um container que quando iniciamos consegue gerenciar a criação de novos containers via navegador. Tem algumas ferramentas interessantes que permitem monitoramento e até orquestração. [http://shipyard-project.com/](http://shipyard-project.com/) .

### Docker-UI
Parecido com o Shipyard mas com menos recursos [https://github.com/crosbymichael/dockerui](https://github.com/crosbymichael/dockerui) .

### Panamax
Também parecido com Shipyard te dá a opção de subir um sistema multi containers [http://panamax.io/](http://panamax.io/) .

## Concluindo
Todas as ferramentas que tenho visto para usar containers em Mac OS e Windows esbarram em ter que rodar os containers sobre uma VM (boot2docker). E sempre que rodamos esses containers para conseguirmos acesso via web , logs, e acesso remoto temos que ter em mente que não adianta bater no ip de loopback (127.0.0.1). Temos que bater (ou apontar nossos virtual hosts para) endereço da VM.

O mundo docker está evoluindo muito e bem rápido, é bom acompanhar as novidades sempre no [blog](http://blog.docker.com/)  deles e nos meet ups das comunidades.

## To Be Continued
Para finalizar a série no próximo artigo falaremos sobre clusterização usando Docker Swarm

Té +
