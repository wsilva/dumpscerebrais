+++
date = "2017-02-15T02:22:49-03:00"
draft = false
slug = "como-trabalhar-com-secrets-no-docker-1-13-x"
categories = ["Tecnologia"]
tags = ["Docker","Secret","Senha","Segurança","Swarm"]
title = "Como trabalhar com Secrets no Docker 1.13.x"
+++

No dia 18 de janeiro de 2017, há menos de um mês foi lançada a versão *1.13* do Docker com uma porrada de novas e matadoras funcionalidades. Uma das que eu mais gostei foi a possibilidade de gerenciar *secrets* no *Swarm Mode* e é sobre ela que vou discorrer nesse artigo.

## O que é o secret?

Quando estamos trabalhando em um projeto e precisamos passar informações sensíveis para o ambiente, tais como senhas, chaves privadas, tokens, chaves de APIs e afins sempre passamos pelo problema de não podermos deixar no controle de versão e devemos sempre utilizar uma maneira segura de trafegar esses segredos.

Muitas vezes acabamos trabalhando com variáveis de ambiente para guardar essas informações, o que não é recomendado por alguns motivos. 

[Diogo Monica](http://twitter.com/diogomonica) que é um dos engenheiros de software da Docker mencionou em um [comentário](https://github.com/docker/docker/pull/9176#issuecomment-99542089) que variáveis de ambiente quebram o princípio de "least surprise" e podem levar a eventuais vazamentos de segredos já que estão acessíveis de várias maneiras, tais como linked containers, através do *docker inspect*, de  processos filhos e até de arquivos de logs já que em caso de exceptions da aplicação muitos frameworks fazem o dump do contexto, inclusive o valor das variáveis de ambiente no arquivo de log.


## Um pouco da história

Em Janeiro de 2015 houve uma proposta de adicionar o comando `docker vault` numa alusão ao [Vault Project](https://www.vaultproject.io) da Hashicorp para fazer a gerencia de segredos dentro do próprio Docker. Segue o link para a issue [10310](https://github.com/docker/docker/issues/10310) no GitHub.

A discussão evoluiu e virou a issue [13490](https://github.com/docker/docker/issues/13490) onde trataram do roadmap para o atual Docker Secrets.


## Como funciona?

O Docker *Secrets* funciona como um cofre onde você pode colocar coisas sensíveis lá e só que tem a chave do cofre consegue utilizar, no caso essa chave é designada aos nós dos serviços que a chave for atribuída.

### Dicas

Só funciona no *Swarm Mode* onde toda a comunicação entre os nós é por padrão encriptada.

Se utiliza do algoritmo de RAFT para persistir o segredo de forma encriptada por todos os *nós* managers e distribuir aos contêineres que fizerem parte do serviço ao qual a chave for atribuida.

Segue diagrama da própria Docker: 

<img class="img-responsive img-thumbnail" title="secrets" alt="secrets" src='/assets/images/secrets.png' />

### Criando um secret

Podemos criar um *secret* de duas maneiras, usando o STDIN:

	$ echo "ConteudoDoSecret" | docker secret create um-secret -

Ou lendo um arquivo:
	
	$ docker secret create novo-secret $HOME/senhas.txt

### Listando, removendo e demais opções

Para listar as *secrets* disponíveis:

```bash
$ docker secrets ls
ID                        NAME        CREATED             UPDATED
8ulrzh4i1kdlxeypgh8hx5imt um-secret   3 minutes ago       3 minutes ago
n95fprwd2trpqnjooojmpsh6z novo-secret About an hour ago   About an hour ago
```

Demais opções do que fazer com os *secrets* como remover, inspecionar, etc podem ser listadas com o *help*:

	$ docker secret --help


### Usando o secret criado

Podemos criar um serviço usando um *secret* criado com o comando:

	$ docker service create --name demo --secret um-secret mysql:5.7	
	
Podemos remover um *secret* de algum serviço existente

	$ docker service update --secret-rm um-secret demo
	
Ou podemos adicionar um *secret* a algum service que esteja de pé: 

	$ docker service update --secret-add novo-secret demo

Quando atrelamos um *secret* a um service podemos então acessar qualquer um dos contêineres que estejam rodando nesse service no path */run/secrets*. Dentro do contêiner nesse path vai existir um arquivo plain text com o nome igual ao definido no nome do *secret* e com o conteúdo desejado, o *secret* em plain text. No nosso caso seria no path */run/secrets/novo-secret*.
	

### Exemplo
	
Segue exemplo onde criamos e utilizando um *secret* como senha para um banco de dados MySQL, ao final nós conectamos ao MYSQL usando a mesma senha passada como secret para a criação do serviço.

<script type="text/javascript" src="https://asciinema.org/a/103254.js" id="asciicast-103254" async data-autoplay="true" data-loop="true"></script>	
	
Até a próxima.