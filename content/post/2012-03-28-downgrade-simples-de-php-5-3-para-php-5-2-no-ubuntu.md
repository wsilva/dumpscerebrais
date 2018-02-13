+++
title = "Downgrade simples de PHP 5.3 para PHP 5.2 no Ubuntu"
slug = "downgrade-simples-de-php-5-3-para-php-5-2-no-ubuntu"
categories = ["Tecnologia"]
tags = ["Downgrade","Karmic","Linux","PHP","Ubuntu"]
date = "2012-03-28T00:15:00-03:00"
+++

Vamo que vamo que to empolgado. Motivos? vários:
 - Acabo de chegar em casa mais cedo do que de costume, na aula de desafios de
 programação consguimos tirar um 10 na atividade (nem lembro do último 10 que
tirei na vida);
 - Consegui rodar várias versões e configurações de PHP numa mesma máquina
 separada por virtual hosts (mas esse artigo fica pra próxima, preciso fechar
alguns estes amanhã);

<!--continua-->

 - E por fim sabadão vamos dar uma parada, uma miniférias creio que merecida;

Desta vez vou escrever algo que sempre esquecia e sempre usava: downgrade da
versão de PHP para 5.2 numa máquina usando as últimas versões de Ubuntu.

Ah mais esxistem vários artigos postados por aí! Sim é verdade, mas nunca
consigo achar de maneira fácil ou sempre tem pelo menos um furo no passo a
passo.

First of All e antes de mais nada: Instalar o aptitude para bloquear os pacotes
php depois de instalado, senão eles vão encher o saco pedindo para fazer o
upgrade.

~~~ bash
    sudo apt-get install aptitude
~~~

Na sequência gravamos em uma variável os pacotes já instalados

~~~ bash
    $pacotesinstalados=`dpkg -l | grep php| awk ‘{print $2}’ |tr "\n" " "`
~~~

Feio né, mas é simples: o *dpkg -l* lista pacotes instalados, o *grep php*
filtra apenas os pacotes com php no nome, *o awk '{print $2}'* é para pegar
somente a coluna com o nome dos pacotes e o *tr "\n" " "* troca as quebras de
linha por espaços.

Removemos esses pacotes

~~~ bash
    sudo aptitude purge $pacotesinstalados
~~~

Agora preparamos o apt para buscar os pacotes php no repositório do Ubuntu 9.10
(Karmic), último Ubuntu com PHP 5.2.

Criamos um arquivo de preferences só para pacotes php usando uma lista de
pacotes disponíveis para php.

~~~ bash
    sudo apt-cache search php |grep php |awk '{print "Package:", $1,"\nPin: release a=karmic\nPin-Priority: 991\n"}' |sudo tee -a /etc/apt/preferences.d/php52 > /dev/null
~~~

Traduzindo:
- *apt-cache search php* vai listar pacotes php disponíveis;
- *grep php* vai filtrar só os que tem php no nome;
- *awk '{print "Package:", $1,"\nPin: release a=karmic\nPin-Priority: 991\n"}'*
vai criar um texto para cada pacote com o Pin e os atributos deste pacote;
- *tee -a /etc/apt/preferences.d/php52* divide a saída da tela e direciona para o
arquivo /etc/apt/preferences.d/php52;
- o *> /dev/null* pega a saida e direciona para um dispositivo nulo, não
existente. (Várias vezes já rodei o comando *cat /minha/vida/coisas_ruins > dev/null* mas
nunca deu certo)

Proximo passo criamos o karmic.list baseando-se no source.list original.

~~~ bash
    sudo egrep '(main restricted|universe|multiverse)' /etc/apt/sources.list |grep -v "#"| sed s/`lsb_release -a 2> /dev/null | grep Codename | cut  -f 2`/karmic/g | sed s/us\.archive/old-releases/g | sed s/br\.archive/old-releases/g | sed s/security\.ubuntu/old-releases\.ubuntu/g | sudo tee /etc/apt/sources.list.d/karmic.list > /dev/null
~~~

Esse é ninja vamos lá jack, por partes:
- *egrep '(main restricted|universe|multiverse)' /etc/apt/sources.list* pega as
linhas do source.list original onde temos uma das palavras;
- *grep -v "#"* ignora as linhas com comentários;
- Vamos abstrair o trecho *sed s/\`lsb_release -a 2> /dev/null | grep Codename | cut  -f 2\`/karmic/g*  para
*sed s/alguma_coisa/karmic/g* e isso sim é inteligivel, estamos trocando o
"alguma_coisa" por karmic em todas as ocorrências;
- Sendo o *lsb_release -a 2 > /dev/null | grep Codename | cut  -f 2* a tal da
coisa podemos ler assim:
*lsb_release -a 2* pega os detalhes da distribuição Ubuntu instalada;
- *> /dev/null*, vc já sabe
- *grep Codename*, filtra apenas as linhas com Codename (que é apenas uma)
- e *cut  -f 2*, pega só a coluna com o apelido da distribuição (lucid, ou maverick,
ou natty por exemplo)

Voltando, os trechos *sed s/us\.archive/old-releases/g* e
*sed s/br\.archive/old-releases/g* tentam trocar o domínio do source.list para
old-releases, já que o karmic não é mais suportado pelo Ubuntu então os pacotes
ficam no repositório de distribuições antigas.

De maneira análoga o trecho  *sed s/security\.ubuntu/old-releases\.ubuntu/g*
também faz troca para old-releases

Por fim o trecho  *tee /etc/apt/sources.list.d/karmic.list > /dev/null* direciona
a saída de tudo isso para o arquivo karmic.list

Próximo passo atualizamos fontes

~~~ bash
    sudo aptitude update
~~~

Reinstalando os pacotes

~~~ bash
    sudo apt-get install $pacotesinstalados
~~~

Travando os pacotes para que o update-manager pare de reclamar que estão
desatualizados

~~~ bash
    sudo aptitude hold `dpkg -l | grep php5| awk '{print $2}' |tr "\n" " "`
~~~

É isso aí, acho que amanhã escrevo como subir 2 ou mais sites num mesmo
servidor com versões e compilações de php diferentes.
