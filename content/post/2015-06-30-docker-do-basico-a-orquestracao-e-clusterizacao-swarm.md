+++
title = "Docker, do Básico a Orquestração e Clusterização - 6. Swarm"
slug = "docker-do-basico-a-orquestracao-e-clusterizacao-swarm"
categories = ["Tecnologia"]
tags = ["Linux","Virtualização","Docker","Hub","Containers","Swarm","Machine","Compose"]
date = "2015-06-30T05:49:00-03:00"
+++

Nessa série de artigos estamos abordando tópicos para uma boa utilização do [Docker](http://www.docker.com/) .

<img class="img-responsive img-thumbnail pull-left" title="Swarm" alt="Swarm" src='/assets/images/docker-swarm.png' />

O docker-swarm é uma ferramenta que nos possibilita através do docker-machine (que vimos no último artigo) juntar vários hosts de docker montando um como se fosse um grande cluster onde podemos rodar nossos containers com nossas aplicações dentro.

Neste momento você deve estar perguntando: "Mas wsilva, que exemplo de aplicação precisa que eu monte um cluster em ambiente de desenvolvimento?" A resposta que sempre me vem a cabeça é nenhuma aplicação. Esse cluster é vantajoso quando precisamos que nossa aplicação permaneça disponível mesmo quanto um fornecedor de infraestrutura de TI estejá indisponível. "Como assim wsilva?" 

<!--continua-->

Imagine que toda sua infraestrutura está baseada nas soluções de cloud da Amazon e toda ela fique fora do ar, muito improvável né, ok. Então imagine que você queira simplesmente baixar seu custo utilizando o provedor de serviços mais barato em um determinado período. Com o swarm essas tarefas ficam simples. 

No exemplo que veremos a seguir vamos levantar VMs locais mas podemos fazer o mesmo exercício com VMs em provedores distintos, AWS, Azure, DigitalOcean, Hackspace, etc, etc, etc.

## Hands On

### Pegando toke no Dockerhub
Como estou usando MacOSX primeiro subi a VM usando o docker machine e exportei as variáveis para que consiga acessar o serviço do docker. Se vc estiver no Linux basta pegar o token do docker hub diretamente com o comando *docker run swarm create*. Apenas anote o token gerado, pois com ele que amarramos todos as VMs que vamos criar.
<script type="text/javascript" src="https://asciinema.org/a/22702.js" id="asciicast-22702" async data-autoplay="true" data-loop="true"></script>

### Criandos o nó central
Aqui chamo o VM principal de abelha-rainha:
<script type="text/javascript" src="https://asciinema.org/a/22703.js" id="asciicast-22703" async data-autoplay="true" data-loop="true"></script>

### Criando outros nós
Aqui criamos 2 nós que chamamos de abelhas operárias, o comando é o mesmo só não temos a flag de *--swarm-master*:
<script type="text/javascript" src="https://asciinema.org/a/22704.js" id="asciicast-22704" async data-autoplay="true" data-loop="true"></script>

### Quarto passo: Rodar uma aplicação no enxame
Aqui nós apontamos nossas variáveis do Docker para o nó mestre do enxame, digamos que para a abelha rainha. Subimos uma instância de redis em um dos containers e escalamos ele para rodar 4 instâncias no total. Quando escalamos percebemos que cada instância do redis foi distribuida e executada nas abelhas-operarias e também na abelha rainha.

<img class="img-responsive img-thumbnail center-block" title="Abelha rainha" alt="Abelha rainha" src='/assets/images/bee-movie.jpg' />

Podemos perceber com o comando *docker ps* que a abelha operária 1 ficou executando o redis 1 e o redis 4 e as demais abelhas rodando redis 2 e 3, também notamos os ips e as portas onde são executados, na abelha operária 1 por exemplo podemos ver que 2 portas (32769 e 32768) são mapeadas para dentro dos containers ao contrário das demais abelhas com apenas uma porta mapeada. 
<script type="text/javascript" src="https://asciinema.org/a/22707.js" id="asciicast-22707" async data-autoplay="true" data-loop="true"></script>

## Concluindo

Com o Docker Swarm podemos montar e comunicar diversas VMs rodando Docker e permitir uma alta disponibilidade de nossas aplicações. Esse exemplo rodamos em VMs num ambiente local. Porém mudando o driver do docker machine (docker-machine create opção -d) podemos criar nossas VMs na AWS, DigitalOcean, Azure, e outras. 

O mundo docker está evoluindo muito e bem rápido, é bom acompanhar as novidades sempre no [blog](http://blog.docker.com/)  deles e nos meet ups das comunidades, quando comecei escrever esse artigo foi antes do DockerCon 2015 e na versão 0.2 do docker-machine e do docker-swarm. Hoje já estamos na 0.3 de ambos. Uma boa novidade do swarm é a integração com serviços de descuberta e gerenciamento de containers como Mesos e Marathon que apresentadas na conference. Vale dar uma olhada quando eles liberarem todos os videos da Dockercon 2015. 

Aqui encerramos a série do básico a orquestração de containers, mas testarei novas ferramentas e customizações no mundo Docker.

Té +
