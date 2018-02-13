+++
title = "Docker no Mac OSX com melhorias no Boot2docker"
categorias = ["Tecnologia"]
tags = ["Linux","Mac","OSX","Virtualbox","Docker","Conteiner","Boot2docker","Machine"]
date = "2015-07-02T05:20:00-03:00"
+++

Se você utiliza *Mac OSX* para trabalhar e se aventurou tentando fazer seus projetos rodaram no *Docker* deve ter percebido que o serviço não roda nativamente, que temos algumas alternativas como boot2docker, ou docker-machine, ou alguma variação desses carinhas.

Eu tenho alguns projetos e a maioria rodando em PHP, na meu *Fedora 19* eu instalei o *Docker*, subi o serviço, montei alguns conteiners de acordo com cada projeto e cada vez que começo a "codar" basta subir os conteiners do projeto que estou trabalhando e acessar como se fossem serviços rodando na minha máquina (e realmente são), fácil, tudo no 127.0.0.1. Já no meu *Mac OSX* a história foi um pouco diferente.

Vou explicar rapidamente como funciona o boot2docker no *Mac OSX* e como customizar o boot2docker ou como personalizar o boot2docker (quem me conhece sabe que não sou muito fã de neologismos fracos como em "customizar" que vem de custom) para melhorar a experiência de uso.

<!--continua-->


## Como funciona o *Docker* no *Mac OSX*
<img class="img-responsive img-thumbnail pull-left" title="como funciona" alt="como funciona" src='/assets/images/como-funciona.jpeg' />
Nos *Mac OSX* todas as soluções se baseam em subir uma máquina virtual linux com serviço do *Docker* rodando para servir de host para os containers, assim temos
algumas camadas a mais como o hypervisor, no nosso caso o Virtualbox e o Linux para ser nosso host Linux. Praticamente perderíamos as vantagens do *Docker* se fosse necessário ter uma máquina virtual para cada container que fossemos rodar.

Apesar de ter algumas alternativas como o [Debian2docker](https://github.com/unclejack/debian2docker)  ou o próprio *docker-machine* a instalação oficial de *Docker* para *Mac OSX* na [documentação](https://docs.docker.com/installation/mac/) do site é o *Boot2docker*.


## Problemas conhecidos

Por ter que rodar os containers em uma VM linux temos alguns problemas comuns:

1. Ter que trabalhar na VM - Se optarmos por subir uma VM linux qualquer (*Ubuntu*, *Fedora*, *Debian*, *Arch*) os containers trabalharão com os arquivos que estiverem na VM e todos os comandos de docker devem ser executados dentro da VM. Há a possibilidade de montar shared folders para trabalhar com os arquivos no Mac mas o trabalho de acertar as permissões não compensam

2. Lentidão - Mesmo utilizando a instalação oficial para *Mac OSX* com *Boot2docker* ou *Docker Machine* temos o problema que os arquivos da pasta home do seu usuário no Mac são montadas via shared folder do Virtualbox para dentro da VM e acabam ficando disponíveis para os seus conteiners através das tradicionais montagem de volumes. As shared folders são conhecidas já por serem muito lentas no gerenciamento de arquivos

3. Fora da home sem chance - Se os arquivos de seu projeto ficam em uma pasta fora da home do seu usuário eles não estarão disponíveis na VM e nem para os conteiners já que apenas a pasta /Users é montada como shared folder.


## Soluções
Dizem que quando você busca uma coisa acaba achando outra. Quando tinha parado de procurar alternativas de uso vi um tweet do Fabien Potencier (@fabpot), o cara do framework symfony, com um link que explica como a backfire utiliza docker lá dentro.

 - tweet: [https://twitter.com/fabpot/status/593393566916878336](https://twitter.com/fabpot/status/593393566916878336) 
 - link da blackfire.io [http://blog.blackfire.io/how-we-use-docker.html](http://blog.blackfire.io/how-we-use-docker.html) 

Inspirado nas customizações citadas nesse post comecei a testar e automatizar a criação da VM do boot2docker.


## Automatizando uma parte
Apesar de não ter um exímio dominio de shell script fiz e disponibilizei no [github](https://github.com/wsilva/docker-stuff/blob/master/boot2docker-customizations.sh)  um script que faz algumas melhorias no boot2docker:

 - backup da configuração de perfil atual do seu boot2docker (~/.boot2docker/profile);
 - cria um novo;
 - travar o endereço ip da VM;
 - dá um nome novo mais personalizado para a VM;
 - backup da sua configuração atual de *NFS* (/etc/exports);
 - configura o seu home do Usuário no mac para acesso via *NFS* (nfsd);
 - desmonta o compartilhamento padrão da shared folder no virtualbox (dentro da vm em /var/lib/boot2docker/bootlocal.sh que é carregado em todo o boot);
 - monta o compartilhamento de arquivo entre o *Mac OSX* e os containers via *NFS* (também em /var/lib/boot2docker/bootlocal.sh - muuuiiito mais rápido);
 - corrige um bug de certificados e rede que está rolando a partir da ultima versão do *Docker*, 1.7.0 (também dentro da VM em /var/lib/boot2docker/profile).


## Demonstração:
<script type="text/javascript" src="https://asciinema.org/a/22829.js" id="asciicast-22829" async data-autoplay="true" data-loop="true"></script>


## Conclusão
Com essas customizações consigo trabalhar bem melhor no *Mac OSX* atualmente. Você deve estar se perguntando: Mas wsilva e o docker-machine? O docker machine quando usado com drivers de nuvem (AWS, DigitalOcean, etc) provisiona uma VM com a opção de linux selecionada na configuração, quando usado com o drivers de hypervisors locais como Virtualbox e VMWare cria uma VM com boot2docker. Na *Dockercon 2015* na semana passada foi lançado o *docker-machine 0.3.0* com algumas [funcionalidades](https://blog.docker.com/2015/06/docker-machine-0-3-0-deep-dive/)  como suporte a outras imagens como o boot2docker personalizado:

~~~bash
$ docker-machine create -d virtualbox --virtualbox-import-boot2docker-vm boot2docker-blablabla-vm dev
~~~

Ou até mesmo outros SOs como o *RancherOS*:

~~~bash
$ docker-machine create -d virtualbox --virtualbox-boot2docker-url https://github.com/rancherio/os/releases/download/v0.3.1/machine-rancheros.iso rancheros
~~~




Té +
