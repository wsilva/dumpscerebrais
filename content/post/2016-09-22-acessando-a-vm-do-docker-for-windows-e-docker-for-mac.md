+++
date = "2016-09-22T23:22:49-03:00"
draft = false
slug = ""
categories = ["Tecnologia"]
tags = ["Docker","Microsoft","Windows","Mac","OS X","Nativo", "Xhyve","Hyper-V","VM","Debug"]
title = "Acessando a VM do Docker for Windows e Docker For Mac"
+++

Tenho usado o *Docker for Mac* desde que lançaram seu beta, já nas versões *1.11.\** e agradeci muito por facilitarem minha vida no OS X, só que o processo de Debug nas VMs ficou um pouco mais oneroso.


## VMs Debian e Ubuntu
Quando comecei com o Docker tive que subir muita VM na mão: Ubuntu, Debian, CentOS. Depois passei a usar o *boot2docker* e seu client que facilitava apontar o Docker Client para o Docker Daemon rodando dentro da VM.

## Com Vagrant
Os problemas de performance me levaram a customizar o *boot2docker* com arquivos shell script para montar as shared folders manualmente e usando NFS. Acabei usando Docker por um tempo dentro do Vagrant para automatizar este processo.

## Docker Machine
O *Docker Machine* foi uma baita invenção, com ele conseguia usar mais de uma VM na mesma máquina de maneira mais fácil e customizava cada uma delas conforme minha necessidade, além de poder rodar Docker remotamente em máquinas na Cloud usando minhas credenciais da Amazon Web Service ou meu token da Digital Ocean por exemplo.

## Docker for Mac e Windows
Após este post da Docker [https://blog.docker.com/2016/06/docker-mac-windows-public-beta](https://blog.docker.com/2016/06/docker-mac-windows-public-beta) rodar no OS X era bem parecido com rodar no Linux, mas alguns detalhes como a url para testar *(local.docker)* ou acesso apenas a pasta Home para montagem de volumes sempre me lembravam que estávamos em uma VM.

Na segunda frase deste mesmo post eles mencionam: 

>"Our major goal was to bring a native Docker experience to Mac and Windows, making it easier for developers to work with Docker in their own environments". 

Naquele momento a cabeça de muito Dev achava que o Docker estava rodando nativo no OS X e no Windows, o que na minha opinião seria uma mágica já que o kernel do OS X, o Darvin, e o kernel do Windows são fechados e não temos ideia se existe recurso para isolar recursos da máquina host. 

Quando questionados como eles funcionavam a Docker sempre abriu o jogo e contou que havia uma máquina virtual rodando Alpine Linux mas que os aplicativos e a experiência de uso eram nativos. Nativos também eram os hypervisors utilizados para rodar essa mini VM com Linux e Docker, Hyper-V no Windows e xhyve no OS X.

## Como debugar

Umb belo dia estava tentando fazer rodar um container que permitia acesso via http e não consegui acessar a porta da aplicação no navegador. 

Depois de muito bater a cabeça achei que poderia ser algum problema na VM rodando o Docker, já tinha passado por algo parecido e dentro da VM rodando um simples *curl* no endereço da aplicação ela respondia. Mas como acessá-la para fazer esse teste? Com o Docker Machine bastava rodar o comando:

~~~bash
docker-machine ssh nome-da-vm
~~~

e bum, estava no shell da VM. Ai era só instalar os pacotes do *boot2docker* que precisava para debugar e testar. Por exemplo o *htop*:

~~~bash
tce-load -wi htop
~~~

### No OS X
Com o *Docker for Mac* tive que buscar na documentação e acabei achando uma maneira de acessar a VM na página  [https://beta.docker.com/docs/mac/troubleshoot/](https://beta.docker.com/docs/mac/troubleshoot/) que já não existe mais, somos redirecionados para a página [https://docs.docker.com/docker-for-mac/troubleshoot/](https://docs.docker.com/docker-for-mac/troubleshoot/) desde que encerrou o beta público e o Docker for Mac e Windows foram liberaram para o público em geral, nesta página a informação de acesso a VM foi removida.

Para acessar a VM no OS X basta utilizar o comando *screen* para anexar (attach) o terminal da VM:

~~~bash
screen $HOME/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/tty
~~~

Para sair devemos rodar os seguintes comandos em outro terminal para desanexar e limpar as sessões de screen terminadas.

~~~bash
screen -D 
screen -wipe
~~~

Exemplo mostrando o acesso:

<script type="text/javascript" src="https://asciinema.org/a/87312.js" id="asciicast-87312" async data-autoplay="true" data-loop="true"></script>

### No Windows

No Windows até hoje não achei uma maneira fácil, tipo *ssh* ou *screen* então apelei por usar um container Docker e compartilhar a maioria dos namespaces do host:

~~~bash
docker run --net=host --ipc=host --uts=host --pid=host -it --security-opt=seccomp=unconfined --privileged --rm -v /:/host alpine /bin/sh
~~~

Em seguida assumimos o controle do host (VM) rodando Docker.

~~~bash
chroot /host
~~~

Vale lembrar que esta maneira também funciona no OS X como no exemplo a seguir.

<script type="text/javascript" src="https://asciinema.org/a/87313.js" id="asciicast-87313" async data-autoplay="true" data-loop="true"></script>

É isso ae galera, em algum próximo post explico melhor esse lance de namespaces e cgroups que faz toda a mágica de isolamento dos containers. 

Bons debugs !!!
___

Vídeos com exemplos adicionados em 28 de setembro.