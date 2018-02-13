+++
title = "Gerar arquivo de deploy usando Git"
categorias = ["Tecnologia"]
tags = ["Deploy","Git"]
date = "2012-08-19T10:50:00-03:00"
+++

Saudações. Primeiramente vamos explicar alguns termos:

- **Deploy**: a tradução mais próxima para o sentido de deploy que vamos usar seria implantação.
- **Git**: Sistema de controle de versão distribuido desenvolvido por Linus Torvalds.
- **Commit**: Grava as mudanças feitas no repositório como uma estágio atual.
- **SHA1**: Id único que identifica um commit.
- **Integração contínua**: Nome dado à rotina de integração de alterações ao sistema ou software desenvolvido incluindo testes que verificam se houve quebra de funcionalidades.

Vamos deixar claro que o ideal para implantações é sempre usar algum sistema
de integração contínua, o Git foi inventado para fazer apenas o controle de
versão e faz isso muito bem.

<!--continua-->

Há técnicas muito boas usando hooks como do
[Thiago Belem](http://blog.thiagobelem.net/automatizando-a-instalacao-deploy-e-atualizacao-de-sites-com-git/) 
que aceleram esse deploy, ou servidores como
o [Heroku](http://heroku.com)  onde você simplesmente
executa um push e suas alterações vão para o
servidor de testes.

Para situações onde você tem que criar um arquivo ou escrever uma rotina
para que alguém faça a implantação utilize os comandos a seguir para
facilitar sua rotina.

## Lista de arquivos removidos entre dois commits

    git diff –name-status SHA1INI SHA1FIN |grep -e "^D" |cut -b 3- |xargs  echo > /DIR/para_remover.txt

Explicando:
Lista os arquivos e status de alteração entre dois commits (SHA1INI e SHA1FIN).

    git diff –name-status SHA1INI SHA1FIN

Dos arquivos listados pega apenas as linhas que começam com D.

    grep -e "^D"

Da listagem remove os primeiros caracteres deixando apenas a lista de arquivos.

    cut -b 3-

O xargs passa a lista como parâmetros para o comando echo que por sua vez
direciona sua saída para o arquivo em */DIR/para_remover.txt*

    xargs echo > /DIR/para_remover.txt

Com essa lista de arquivos você pode montar instruções ou um shell script
para removê-los no servidor onde será feito o deploy.

## Para gerar uma lista de arquivos adicionados ou alterados e empacota

    git diff –name-status SHA1INI SHA1FIN |grep -e "^[^D]" |cut -b 3- | xargs tar cvfz /DIR/fontes.tar.gz

Explicando:
Lista os arquivos e status de alteração entre dois commits (SHA1INI e SHA1FIN).

    git diff –name-status SHA1INI SHA1FIN

Dos arquivos listados pega apenas as linhas que não começam com D.

    grep -e "^[^D]"

Da listagem remove os primeiros caracteres deixando apenas a lista de arquivos.

    cut -b 3-

O xargs passa a lista como parâmetros para o comando tar que por sua vez
empacota e compacta no arquivo */DIR/fontes.tar.gz*

    xargs tar cvfz /DIR/fontes.tar.gz

Com esse pacote a basta a pessoa que for aplicar o deploy subir para o
servidor e descompactar na pasta raiz do projeto sobre escrevendo os
arquivos que lá estiverem.

Se precisar pegar as mensagens de commit entre 2 commits específicos
precisamos de script mais elaborado:

    #!/bin/bash
    #definindo os SHA1 inicial e final e o diretório onde vamos gerar o arquivo com as mensagens
    SHA1INI="informe aqui o sha1 do commit inicial"
    SHA1FIN="informe aqui o sha1 do commit final"
    DIR="/informe/aqui/o/caminho/para/o/arquivo/a/ser/gerado/"
    # pegando os SHA1 curtos cortando os SHA1 inicial e final originais
    SHA1INICURTO=`echo "$SHA1INI" | cut -c 1-7`
    SHA1FINCURTO=`echo "$SHA1FIN" | cut -c 1-7`
    #criando o arquivo que vai guardar a listagem
    rm $DIR/msgcommits.txt && touch $DIR/msgcommits.txt
    git log –oneline $SHA1FINCURTO | while read linha
    do
        SHA1CURTO=`echo $linha | cut -c 1-7`
        MSG=`echo $linha | cut -c 9-`
        echo " - $MSG" >> $DIR/msgcommits.txt
        if [ "$SHA1CURTO" == "$SHA1INICURTO" ]; then
            exit 0
        fi
    done

No arquivo msgcommits.txt vamos ter todas as mensagens que cada usuário
informou ao realizar o commit.

Com um pouquinho mais de shell script, criatividade e tempo dá para
evoluir e combinar esses comandos e montar um belo pacote para deploy.

Mas como disse anteriormente esse método de montar arquivo implantação
é mais destinado para servidores que você não tenha acesso, onde algum
terceiro vai implantar, ou para ser enviado para uma auditoria.

De maneira geral o caminho mais recomendado seria uma integração
contínua, geralmente economiza tempo de implantação e retestes.

Agradecimentos especiais a Fernando Uchiyama e André Tannus que
começaram a desenvolver algumas dessas concatenações de comandos
que hoje com algumas adaptações utilizo para automatizar a geração
de pacotes de implantação.

Té +
