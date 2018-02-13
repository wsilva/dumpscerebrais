+++
title = "Docker, do Básico a Orquestração e Clusterização - 2. Básico"
slug = "docker-do-basico-a-orquestracao-e-clusterizacao-basico"
categories = ["Tecnologia"]
tags = ["Linux","Virtualização","Docker","Hub","Containers"]
date = "2015-03-05T02:07:00-03:00"
+++

Nessa série de artigos estamos abordando tópicos para uma boa utilização do [Docker](http://www.docker.com/) .
<img class="img-responsive img-thumbnail pull-left" title="Docker Logo" alt="Docker Logo" src='/assets/images/lxc.png' />

Dando continuidade ao artigo anterior com introdução sobre o que é docker e como funciona agora abordaremos como utilizar o Docker: baixar imagens, rodar containers, monitorar, versionar, manipular imagens criadas, criar imagens com base em arquivo com receitas (Dockerfiles), principais comandos e dicas de boas práticas baseando em dificuldades e gotchas que tenho enfrentado.

<!--continua-->

## Instalação
Não vou abordar a instalação aqui, cada sistema operacional tem sua maneira de instalar, alguns tem mais de uma maneira.

Ubuntu por exemplo tem o docker dos sources lists próprios mas o recomendado é utilizar o source list do próprio repositório do Docker.

Algumas versões de Fedora tem tweaks por causa do SELinux.

Para MacOS e Windows temos o boot2docker mas novidades estão vindo e teremos na próxima versão o docker-machine (novidade serão abordadas nos próximos posts).

Novidades a parte, acesse a [página de instalação do Docker](https://docs.docker.com/installation/) , escolha o sistema operacional onde você vai instalar e siga a risca as instruções. Preste atenção na parte de permissionamento de usuários ou você terá que sempre rodar os comandos docker com sudo. Ex.:

~~~bash
$ sudo docker ps
~~~

Existe um tutorial básico e guiado onde podemos rodar os comandos no próprio site. Recomendo fazê-lo [aqui](https://www.docker.com/tryit/){:target:'_blank'}. São 8 passos onde buscamos uma imagem, baixamos, rodamos, alteramos ela, inspecionamos e enviamos de volta ao repositório do dockerhub.

## O Flow
Similar ao git onde podemos baixar (pull), alterar (commit) e enviar (push), no docker também podemos baixar, alterar e enviar imagens. *Mas enviar pra onde?* Assim como no git temos repositórios remotos como gitorious, bitbucket e o mais famoso github, no docker temos o [dockerhub](https://hub.docker.com/){:target='blank'}. Podemos montar um repositório privado nosso também.

Os comandos mais básicos são o **docker images** que lista as imagens que já baixamos ou construímos:

~~~bash
docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
debian              latest              c90d655b99b2        5 days ago          85.01 MB
debian              wheezy              c90d655b99b2        5 days ago          85.01 MB
ubuntu              latest              2d24f826cb16        12 days ago         188.3 MB
~~~

Percebemos que temos duas entradas de Debian com tags diferentes (latest e wheezy) mas com o mesmo Image ID, ou seja é a mesma imagem.

E o comando **docker ps** que lista os containers que estão, ou estiveram rodando.

~~~bash
docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
~~~

## Exercício (similar ao do tutorial do docker)

<img class="img-responsive img-thumbnail" title="Exercícios" alt="Exercícios" src='/assets/images/exercicios.jpg' />

Baixando um container básico do dockerhub:

~~~bash
$ docker pull debian
debian:latest: The image you are pulling has been verified
511136ea3c5a: Pulling fs layer
30d39e59ffe2: Download complete
c90d655b99b2: Download complete
...
~~~

Criando um arquivo teste dentro dele:

~~~bash
$ docker run debian touch /tmp/teste
~~~

Quando fazemos isso o container roda faz a alteração e pára.

Listando o último container para pegar o Container ID:

~~~bash
$ docker ps -l
CONTAINER ID        IMAGE               COMMAND              CREATED             STATUS                      PORTS               NAMES
a04e63f87349        debian:latest       "touch /tmp/teste"   13 minutes ago      Exited (0) 13 minutes ago                       happy_mayer
~~~

Gerando uma nova imagem com essa alteração (usando o container ID):

~~~bash
$ docker commit a04e63f87349 wfsilva/teste1
7e74615a398990c8af6d2e20c39d447f81ca23af257db19680419fc52d6fed03
~~~

Enviando essa imagem para sua conta (a minha é wfsilva, por isso dei o nome de wfsilva/teste1) no docker hub:

~~~bash
$ docker push wfsilva/teste1
The push refers to a repository [wfsilva/teste1] (len: 1)
Sending image list

Please login prior to push:
Username: wfsilva
Password:
Email: ****@***.br
Login Succeeded
The push refers to a repository [wfsilva/teste1] (len: 1)
Sending image list
Pushing repository wfsilva/teste1 (1 tags)
c90d655b99b2: Image already pushed, skipping
...
7e74615a3989: Image successfully pushed
Pushing tag for rev [7e74615a3989] on {https://cdn-registry-1.docker.io/v1/repositories/wfsilva/teste1/tags/latest}
~~~

**Fim de exercício**

## Dicas
### Containers vs Images
Sempre confundi mas pensando que imagens são estáticas e os container são dinâmicos ficou fácil. Quando rodamos uma imagem temos um container. Quando gravamos as alterações de um container temos uma nova imagem.

### Alterando um container:
Uma outra maneira de alterarmos um container é rodarmos passando o comando bash:

~~~bash
$ docker run -it debian bash
root@504451ee512c:/#
~~~

Esse comando vai te retornar o bash do container onde você fica mais livre para fazer suas alterações com a vantagem do shell interativo (opção i), do tty (opção t) e do auto complete. Depois saia do container (com o comando exit ou atalho ctrl + d) e poderá gerar uma nova imagem da mesma maneira.

### Layers:
O Docker trabalha com camadas de imagens, cada alteração gravada é uma nova imagem

Nos exemplos acima se rodarmos a imagem alterada teremos o container com o arquivo de teste lá:

~~~bash
$ docker run wfsilva/teste1 ls /tmp
teste
~~~

Mas se rodarmos a base image do debian as alterações são perdidas:

~~~bash
$ docker run debian ls /tmp

~~~

### Emagrecendo Containers

<img class="img-responsive img-thumbnail" title="Emagrecendo Container" alt="Emagrecendo Container" src='/assets/images/flattern-container.jpg' />

O comando save grava uma imagem em arquivo e o load carrega de volta preservando todo histórico:

~~~bash
$ docker save <IMAGE NAME> > /tmp/image.tar
$ docker load < /tmp/image.tar
~~~

O comando export exporta um container em arquivo e o import carrega de volta forçando a perda do histórico:

~~~bash
$ docker export <CONTAINER ID> > /tmp/container.tar
$ cat /tmp/container.tar | docker import - novo-nome:latest
~~~

A vantagem de perder o histórico é que o o container fica mais leve, o que podemos conferir com o comando:

~~~bash
$ docker images --tree
~~~

### Limpando testes
Para remover containers utilizamos o comando:

~~~bash
$ docker rm <CONTAINER ID>
~~~

E para remover imagens utilizamos:

~~~bash
$ docker rmi <IMAGE ID>
~~~

### Vida quase real:

<img class="img-responsive img-thumbnail" title="Vida Real" alt="Vida Real" src='/assets/images/real-life.jpg' />

~~~bash
$ docker run -it --rm --name yabadabadoo -p 80:80 -v /path/para/aplicacao:/var/www/html php:5.6-apache --privileged
~~~

Esse comando vai baixar a imagem base do container php, tag 5.6-apache, vai rodar de maneira interativa e com terminal, vai montar o caminho do meu projeto para dentro de /var/www/html que está dentro do container, vai rodar com flag privileged para adicinar permissões de escrita, vai mapear a porta 80 do container para a porta 80 da máquina host e quando o container for finalizado as alterações feitas no container serão removidas.

Mais próximo da realidade só montando um parque de containers com diversos serviços se comunicando, e isso será assunto para os próximos artigos.

## To Be Continued
Ainda abordaremos como utilizar o docker em ambiente Windows e Mac, como contruir um container através de um arquivo com receita (Dockerfile), orquestração de containers usando fig, e as novidades (já em beta e liberadas para teste) anunciadas na Dockercon em dezembro de 2014 para a próxima versão do docker.

Té +


