+++
date = "2016-09-01T03:36:49-03:00"
draft = false
slug = ""
categories = ["Tecnologia"]
tags = ["Linux","Bash","Docker", "Machine", "Compose", "Auto-complete"]
title = "Auto complete para Docker Engine, Machine e Compose"
+++


Se você é como eu então adora o auto complete do bash e provavelmente digita sempre com o dedo anelar esquerdo sempre meio levantado. Por que wsilva? Porque a cada 2 ou 3 letras de um comando já uso o tab para auto completar ou pelo menos sugerir uma continuação.

Usando Docker o auto completion é vital, existem diversos subcomandos e cada subcomando tem seus parâmetros, um jeito de descobrir é digitar o seguinte comando para ver os subcomandos ou opções possíveis.

```
$ docker --help 
```

No fim do output exite a sugestão:

```Run 'docker COMMAND --help' for more information on a command.```

Ou seja, para ver as opções possíveis para o comando run fazemos de maneira similar:

```
$ docker run --help
```

Mas acho isso muito chato eu prefiro começar a digitar `docker run --vol...[tab][tab]` e o autocomplete fazer o trabalho:

```
$ docker run --volume
--volume         --volume-driver  --volumes-from
```

## Instalando o Bash Completion

### No Mac OS X
O processo é simples, estando no Mac OS X temos que instalar o auto completion usando o brew:

```
$ brew install bash-completion
```

E em seguida adicionar as linhas em nosso arquivo de profile ($HOME/.bash_profile ou /Users/<seu usuário>/.bash_profile)

```
if [ -f $(brew --prefix)/etc/bash_completion ]; then
    . $(brew --prefix)/etc/bash_completion
fi
```

Dá para fazer com os seguintes comandos que devem ser usados nesta sequência:

```
$ echo "if [ -f $(brew --prefix)/etc/bash_completion ]; then" | tee -a $HOME/.bash_profile
$ echo "    . $(brew --prefix)/etc/bash_completion" | tee -a $HOME/.bash_profile
$ echo "fi" | tee -a $HOME/.bash_profile
```

Atenção: Se não temos o *homebrew* instalado em http://brew.sh/ encontramos mais informações.

### No Linux

No Linux como sempre mais simples é só instalar via gerenciador de pacotes da sua distribuição. Se for Debian like (Ubuntu, Mint, etc) usamos o *apt-get*:

```
$ sudo apt-get install bash-completion
```

Se for Red Hat like (RHEL, CentOS, Fedora, etc) então usamos *yum*:

```
$ sudo yum install bash-completion
```

## Completion do Docker Engine

O arquivo está disponível em https://github.com/docker/docker/blob/master/contrib/completion/bash/docker
portanto sempre que fizer uma atualização é bom baixar novamente esse arquivo para refletir os comandos e parâmetros novos disponibilizados pelo Docker.

Ao baixar devemos colocá-lo na pasta  `/etc/bash_completion.d/` se estivermos no Linux ou devemos acrescentar a pasta do homebrew antes se estivermos em um Mac OS X: `$(brew --prefix)/etc/bash_completion.d`.

### No Mac OS X

```
$ curl -L https://raw.githubusercontent.com/docker/docker/master/contrib/completion/bash/docker > $(brew --prefix)/etc/bash_completion.d/docker
```

### No Linux

```
$ curl -L https://raw.githubusercontent.com/docker/docker/master/contrib/completion/bash/docker > /etc/bash_completion.d/docker
```

## Completion do Docker Compose

De maneira similar o arquivo está disponível em https://github.com/docker/compose/blob/master/contrib/completion/bash/docker-compose . Reforçamos que sempre que fizer uma atualização é bom baixar novamente esse arquivo para refletir os comandos e parâmetros novos disponibilizados pelo Docker Compose.

Ao baixar também devemos colocá-lo na pasta  `/etc/bash_completion.d/` se estivermos no Linux ou devemos acrescentar a pasta do homebrew antes se estivermos em um Mac OS X: `$(brew --prefix)/etc/bash_completion.d`.

### No Mac OS X

```
$ curl -L https://raw.githubusercontent.com/docker/compose/master/contrib/completion/bash/docker-compose > $(brew --prefix)/etc/bash_completion.d/docker-compose
```

### No Linux

```
$ curl -L https://raw.githubusercontent.com/docker/compose/master/contrib/completion/bash/docker-compose > /etc/bash_completion.d/docker-compose
```

## Completion do Docker Machine

O processo é praticamente o mesmo porém o docker-machine possui 3 arquivos de auto complete. Os arquivos estão disponíveis em https://github.com/docker/machine/tree/master/contrib/completion/bash . Reforçamos que sempre que fizer uma atualização é bom baixar novamente esse arquivo para refletir os comandos e parâmetros novos disponibilizados pelo Docker Machine.

Como dito anteriormente devemos colocá-los na pasta  `/etc/bash_completion.d/` se estivermos no Linux ou devemos acrescentar a pasta do homebrew antes se estivermos em um Mac OS X: `$(brew --prefix)/etc/bash_completion.d`.

Como são mais arquivos recomendo utilizarmos um loop para baixarmos todos em sequência utilizando apenas um comando.

### No Mac OS X

```
machineVersion=`docker-machine --version | tr -ds ',' ' ' | awk 'NR==1{print $(3)}'`
files=(docker-machine docker-machine-wrapper docker-machine-prompt)
for file in "${files[@]}"; do
  curl -L https://raw.githubusercontent.com/docker/machine/v$machineVersion/contrib/completion/bash/$file.bash > `brew --prefix`/etc/bash_completion.d/$file
done
unset machineVersion
```

### No Linux

```
machineVersion=`docker-machine --version | tr -ds ',' ' ' | awk 'NR==1{print $(3)}'`
files=(docker-machine docker-machine-wrapper docker-machine-prompt)
for file in "${files[@]}"; do
  curl -L https://raw.githubusercontent.com/docker/machine/v$machineVersion/contrib/completion/bash/$file.bash > /etc/bash_completion.d/$file
done
unset machineVersion
```

## Considerações

### Versões diferentes

Se você utilizar alguma versão específica de uma dessas ferramentas basta baixar os arquivos da pasta correspondente. 

Por exemplo o docker engine, da maneira que mostramos vamos pegar o arquivo que está na branch *master*, atualmente *versão 1.12.1*, para pegar na *versão 1.11.0* por exemplo o arquivo fica em https://github.com/docker/docker/blob/v1.11.0/contrib/completion/bash/docker

### Utilizo outro shell

O Docker disponibiliza alguns arquivos de completion para outros shells. O Docker Engine disponibiliza *bash*, *zsh*, *fish* e *powershell* como podemos ver em https://github.com/docker/docker/tree/master/contrib/completion .

O Docker Compose e o Docker Machine disponibilizam apenas para *bash* e *zsh*  como vemos nas urls https://github.com/docker/compose/tree/master/contrib/completion e https://github.com/docker/machine/tree/master/contrib/completion .

### Demo

Segue um demo de como instalar o bash completion para o docker, para o docker-compose e para o docker-machine e como utilizá-los no Mac OS X.


<script type="text/javascript" src="https://asciinema.org/a/84494.js" id="asciicast-84494" async data-autoplay="true" data-loop="true"></script>

É isso ae, curtiu, use, comente, compartilhe e contribua. Não curtiu, também comente e participe. 

Abç