+++
date = "2017-04-08T08:57:50-03:00"
draft = false
slug = "aumentando-a-seguranca-do-docker-swarm-com-autolock"
categorias = ["Tecnologia"]
tags = ["Docker","Swarm","Autolock","Secret","Segurança"]
title = "Aumentando a segurança do Docker Swarm com Auto Lock"
+++

O **Auto Locking** é uma das grandes funcionalidades adicionadas na versão *1.13.0* do Docker e presente nas novas versões *1.13.1*, *17.03.0-ce*, *17.03.1-ce* e *17.04.0-ce* (versões disponíveis até a data de publicação desse artigo). 

Você deve estar se perguntando: "Versões *1.12*, *1.13* e agora *17.03.x-ce*? Que zona é essa?" Na verdade não é zona, é reorganização, na versão 1.13 uma das novidades foi a reorganização dos comandos no CLI, agora a Docker está reorganizando os release e versões. Ela lançou um novo ciclo de releases, renomeou as versões baseando-se em datas (ano e mês) e separou a parte open source, Docker CE (community edition) da parte proprietária com licensa e com planos de suporte da própria Docker, o EE (enterprise edition). Resumindo reorganização. Mas essa é uma história que podemos detalhar em outro artigo, nosso foco aqui será mostrar o que é e como funciona o **Auto Locking**

## A motivação por trás do Auto Locking

Tanto a comunicação entre os nós do *Swarm* quanto os logs do *Raft consensus* são por padrão encriptados por chaves TLS que ficam nos discos das máquinas managers. Tá, mas o que isso quer dizer?

Vamos imaginar que uma máquina manager reinicia, tanto a chave TLS usada para a encriptação da comunicação entre os nós, quanto a chave TLS usada para encriptar os logs do *Raft* são carregados na sua memória. Isso por um lado é bom pois permite que um nó seja reiniciado sem intervenção humana e sistema continua funcionando normalmente. Por outro lado se alguém conseguir essas chaves seja por um backup que vaze ou disco corrompido essa pessoa vai conseguir acesso aos logs do *Raft* e também às informações sensíveis como onde estão rodando os serviços, às configurações de acesso dos serviços e inclusive aos *secrets* dessas aplicações rodando nesse cluster.

## Como funciona o Auto Locking.

O *auto locking* é uma camada de proteção a mais e é super simples. Quando você iniciar ou atualizar um cluster Swarm passando a flag `--autolock` uma chave é gerada para encriptar as demais chaves, porém essa nova chave é responsabilidade nossa e deve ser mantida por nós, devs e operadores. Seja via password manager, seja guardando de cabeça (o que duvido), seja anotando em papel de pão.

Com o *auto lock* ativo se por um acaso a máquina reiniciar ninguém conseguirá rodar comandos nela sem antes desbloqueá-la e para desbloquear aquela chave de nossa responsabilidade será solicitada.

Podemos iniciar um cluster já utilizando a funcionalidade de *auto lock*:

```bash
$ docker swarm init --autolock
Swarm initialized: current node (drr4hrzngqoh90cpp0t5lnuv1) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-1infi7fei781xasqoh1q1k5i7p47ocvyptaoq1r6dmmp1bwz0p-e37ividr81a45qzhq9ptkb836 \
    192.168.65.2:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

To unlock a swarm manager after it restarts, run the `docker swarm unlock`
command and provide the following key:

    SWMKEY-1-nly8Gmulj+xuPwFF4ks4evN6OxdUtVFdD1QlIhdVvto

Please remember to store this key in a password manager, since without it you
will not be able to restart the manager.
```

Ou adicionar a um cluster já em produção:

```bash
$ docker swarm update --autolock
Swarm updated.
To unlock a swarm manager after it restarts, run the `docker swarm unlock`
command and provide the following key:

    SWMKEY-1-av+McYBAyjy6o1RMYsDqUsS8lkjH05t5HMnP7USjk+4

Please remember to store this key in a password manager, since without it you
will not be able to restart the manager.
```

Podemos perceber o token gerado nos dois casos, ambos começam com *SWMKEY*, essa é a chave que devemos guardar, ela não é persistida no cluster.

Se a máquina manager não estiver travada você consegue recuperar essa chave com seguinte comando

```bash
$ docker swarm unlock-key
To unlock a swarm manager after it restarts, run the `docker swarm unlock`
command and provide the following key:

    SWMKEY-1-av+McYBAyjy6o1RMYsDqUsS8lkjH05t5HMnP7USjk+4

Please remember to store this key in a password manager, since without it you
will not be able to restart the manager.
```

Se a máquina estiver travada qualquer comando administrativo não será executado e vai retornar a seguinte mensagem:


```bash
$ docker node ls
Error response from daemon: Swarm is encrypted and needs to be unlocked before it can be used. Please use "docker swarm unlock" to unlock it.
```

Para mudarmos a key que destrava o Swarm usamos o seguinte comando:


```bash
$ docker swarm unlock-key --rotate
Successfully rotated manager unlock key.

To unlock a swarm manager after it restarts, run the `docker swarm unlock`
command and provide the following key:

    SWMKEY-1-ECkuyhW8Zgc9nZthmkMpZd5f+mxPuOEC5K6CyA89xNk

Please remember to store this key in a password manager, since without it you
will not be able to restart the manager.
```

## Exemplo
	
Segue exemplo onde mostro como ativar o *auto lock*, como rotacionar a chave e o comportamento quando o cluster está travado.

<script type="text/javascript" src="https://asciinema.org/a/111981.js" id="asciicast-111981" async data-autoplay="true" data-loop="true"></script>	
Uma demostração similar disponível neste [video](https://youtu.be/sPwvz_gsLGw) foi feita para um dos meetups de Docker em São Paulo do qual ajudo organizar.

Mais informações sobre *Swarm Auto Locking* podemos encontrar na documentação oficial nesse [link](https://docs.docker.com/engine/swarm/swarm_manager_locking/) e sobre os meetups de Docker em São Paulo acesse outro [link](https://www.meetup.com/Docker-Sao-Paulo/).


Até a próxima.