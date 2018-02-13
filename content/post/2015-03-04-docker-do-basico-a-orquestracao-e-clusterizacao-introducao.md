+++
title = "Docker, do Básico a Orquestração e Clusterização - 1. Introdução"
slug = "docker-do-basico-a-orquestracao-e-clusterizacao-introducao"
categories = ["Tecnologia"]
tags = ["Linux","Virtualização","Docker","Vagrant","Virtual Box","Cluster","Orquestração"]
date = "2015-03-04T18:27:00-03:00"
+++

Nessa série de artigos abordaremos tópicos para uma boa utilização do [Docker](http://www.docker.com/) .
<img class="img-responsive img-thumbnail pull-left" title="Docker Logo" alt="Docker Logo" src='/assets/images/docker-logo.jpg' />

Se você desenvolve ou desenvolveu para web deve ter esbarrado em problemas de configuração de ambiente ou padronização de ambientes para desenvolvedores, homologação e produção. Se pesquisou a respeito então provavelmente já ouviu falar sobre Docker, [Vagrant](https://www.vagrantup.com/), ou pelo menos [Virtualbox](https://www.virtualbox.org/).

Primeiramente vamos definir alguns conceitos.

<!--continua-->

## Virtualização

Sem muitos rodeios é quando criamos um ambiente que roda sobre outro, e pode ser montado direto no hardware (Bare Metal) como no Xen, VMware, Hyper-V, ou via software (Hosted) como no Virtualbox, Paralel Desktops.
Resumindo seja é um carinha que virtualiza a arquitetura de um PC e você pode literalmente instalar um sistema operacional em cima dele.

## Host / Guest
Na vitualização tipo **Hosted**, o **Host** é o sistema operacional que está rodando no hardware e **Guest** é o sistema instalado.

## Virtualbox
No mundo *Dev o Software de Virtualização mais popular e ultilizado é o Virtualbox*, que *roda* em sistemas Linux, Windows e MacOS entre outros. No Virtualbox podemos criar uma *maquina-virtual* e literalmente instalar qualquer *Distro Linux* ou *versão do Windows* nele.

## Vagrant
Para tornar o uso do Virtualbox mais fácil para os desenvolvedores web o Vagrant, com ou sem ajuda de ferramentas como o puppet ou o chef, torna mais fácil provisionarmos um ambiente que seja idêntico ao ambiente que vai rodar em produção e compartilhar o código produzido na máquina host com a máquina guest (normalmente servidor linux)

## Tá, mas e o Docker?
O Docker é um cara que entra onde a virtualização é um problema, na performance. Para ficar mais fácil de explicar veja a imagem abaixo.

<img class="img-responsive img-thumbnail" title="Virtualização vs LXC" alt="Virtualização vs LXC" src='/assets/images/virtualizaca-vs-containers.png' />

O Docker utiliza o esquema de LXC (Linux Containers) para rodar somente a parte que interessa do Guest OS e compartilhar parte dos recursos do kernel do sistema operacional Host.

Há quem diga que LXC é um tipo de virtualização, e outros mais ortodoxos dizem que não. Se quiser estude e me conta, porque na verdade isso no momento não me interessa muito, o que me interessa é que desse jeito subir um sistema é absurdamente rápido e leve.

O LXC permite que recursos, libs e configurações sejam "aprisionados" (é isso aí, e tem caras falam enjaulados tbm)
em um path e isolados através de namespaces e chroot e rode como um processo.

Com a ajuda do AuFS (sistema de arquivos em camadas e que gerencia recursos de rede também) o Docker funciona como uma camada sobre o LXC, praticamente uma API.

## Vantagens:
 * Sobe muuuuiiiitttoooo rápido - não precisamos do Guest OS todo.
 * Isola o sistema - chroot e processo único.
 * Fácil replicação - é tudo linux, é só pegar a imagem, subir em outro ambiente e mandar rodar o container dessa imagem.
 * Facilita a utilização dos Linux Containers - Subir um container LXC na unha dá muito mais trabalho que no Docker.

## Desvantagens:
 * Overhead de entrada e saída - Demanda muita escrita e leitura.
 * Não tão isolado - há recursos do Kernel do sistema Host que são compartilhados, por mais seguro que seja pode ser passível de algum ataque bem sofisticado.
 * Só Linux: desvantagem que considero questionável já que são raros os webservers rodando Windows, e também nunca vi webservers rodando Unix, Solaris, Mac (imagina o preço de um servidor Mac).
(obs: Se você usa Mac ou Windows como maquinha de Dev, é possível usar o Docker com o Boot2Docker ou o Docker-Machine. Veremos mais desse assunto no próximo artigo).

## Adequando o vocabulário
No mundo LXC e do Docker não dizemos que temos um Guest OS, dizemos que temos um container. Também não temos snapshots, temos imagens.

## To Be Continued
No próximo artigo vamos mostrar o básico do mundo dos containers.
