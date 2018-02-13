+++
title = "PHP com Notices? Warnings? pode ser o output_buffering..."
slug = "php-com-notices-warnings-pode-ser-o-output_buffering"
categorias = ["Tecnologia"]
tags = ["Erros","PHP"]
date = "2012-04-14T22:52:00-03:00"
+++

Saudações

Esse post vai ser curto, mesmo porque o intuito principal é não esquecer e
resolver rápido na próxima.

Estava eu trabalhando esses dias quando um desenvolvedor de uma outra equipe
me falou que algo estranho estava acontecendo, que na máquina dele um
fomulário estava funcionando normal mas na máquina do designer estava dando
pau.

<!--continua-->

Fui lá conferir e a mensagem era mais ou menos assim:
**Notice: Undefined index bla bla bla... Line x, bla bla bla...** e logo abaixo:
**Fatal Error, bla bla bla... Headers Already Sent, bla bla bla...**

Para resolver isso é fácil, só ir no arquivo que gerou o erro durante a
execução na linha que o Notice reclamou e verificar que algum Array ou
Objeto foi chamado numa posição que não existe o índice.

Exemplo:

    echo $var[3];

Mas se não existe a posição 3 dessa variável o PHP vai lançar um Notice e
continuar a execução.

Então comecei o checklist com o cara:

    - Os arquivos fontes nas duas máquinas estão iguais?
    - Sim, foram obtidos pelo servidor de controle de versão.

    - O banco de dados usado está igual?
    - Sim, ambas as máquinas estavam consumindo o mesmo servidor MySQL.

    - Então sua máquina está com a diretiva error_reporting da configuração no
php.ini ignorando notices e a do designer não está ignorando.
    - Na minha máquina está ligado.

Verificando as duas máquinas estavam com o mesma configuração:

    error_reporting = E_ALL | E_STRICT

<img class="img-responsive img-thumbnail" title="Error Reporting" src='/assets/images/errorreporting.png' />

Para mim algum código estava redefinindo essa diretiva durante a execução e
tasquei uns var_dumps / die no código para ver. Mesma coisa.

Então dei meu braço a torcer e pesquisei aproximadamente 1h no Google e
finalmente descobri.

A persistência dos dados deste formulário passava por uma validação dentro
do "controller", e se tudo desse certo gravava no banco usando o "model".
No final da execução lançava uma "flash message" com a mensagem de sucesso
ou fracasso seguida de um redirect para um outro método do sistema que
exibia essa flash message.

Com a diretiva **output_buffering** ligada a máquina do desenvolvedor lançava o
Notice, continuava a execução e redirecionava normalmente pois só
descarregava o código HTML do buffer no final da execução.

Já a máquina do designer com a diretiva **output_buffering** desligada lançava
o Notice no código HTML e tentava fazer o redirecionamento. Como o header
(cabeçalho) da página já tinha sido montado e despachado ele não conseguia
despachar de novo o outro header e parava a execução lançando um **"Fatal Error"**

<img class="img-responsive img-thumbnail" title="Output Buffering" src='/assets/images/outputbuffer.png' />

Moral da história, desconfie de tudo. Se as mensagens de
**Erros, Warnings e Notices** do PHP não estiverem aparecendo desconfie do
**error_reporting** mas
também desconfie do **output_buffering**.

Abçs e inté
