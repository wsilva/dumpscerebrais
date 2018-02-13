+++
title = "Camera como leitor de código de barras"
slug = "camera-como-leitor-de-codigos-de-barra"
categorias = ["Tecnologia"]
tags = ["Linux","Webcam","Camera","Código de Barras","Zbar","Crikey"]
date = "2014-12-09T12:10:00-03:00"
+++

Aproveitando que mudei o paradigma desse site para static blogging usando
Jekyll (ver último post) vamos escrever sobre algo muito útil (pelo menos para
mim myself and I). Lendo um código de barras usando a webcam do meu notebook.

<!--continua-->

Um tempo atrás eu queria facilitar o pagamento de minhas contas usando a webcam
do meu notebook. Usando Debian na época instalei as dependências e compilei o
Zbar, que hoje já é disponibilizado nos sources das principais distribuições de
Linux. Depois de sofrer um pouco acabei desistindo.

Semana passada me mostraram um app da Kinoni que enviava o código de barras via
bluetooth ou wifi (desde que na mesma subnet) mas o carinha que deveria ser
instalado do lado do desktop só funcionava no mac ou no windows, sem contar que
instalei o app e ele só funcionava em trial, depois pedia para comprar a versão
pro. O importante é que isso me fez pesquisar de novo.

## Hands On

Basicamente instalei o :

~~~ bash
    sudo yum install zbar
~~~

Nos Debian/Ubuntu deve ser com *sudo apt-get install zbar*

E fiz alguns testes com os comandos *zbarcam* e *zbarimg* que
respectivamente "scaneiam" (cara, como odeio esses neologismos gringos, mas
não achei melhor) e geram códigos de barra cada um com seus parâmetros.


O que mais gostei foi:

~~~ bash
    zbarcam --raw /dev/video0
~~~

Que abre a webcam faz o scan (me recuso a usar a palavra "scaneia" de novo) e gera o
output na própria linha de comando.

<img class="img-responsive img-thumbnail" title="zbarcam" alt="zbarcam" src='/assets/images/zbarcam.png' />

## Jogando o output do zbar no browser

Meu objetivo primário era facilitar o pagamento de contas e desta maneira (com o output no terminal) teria que continuar a copiar os números e colar no campo de formulário dentro dessas páginas de pagamentos de conta.

Num forum achei o [crikey](http://www.shallowsky.com/software/crikey/) 
 (Conveniently Repeated Input Key) que simula eventos de teclado

Baixei a última versão crikey-0.8.3.tar.gz (versão no dia que escrevi isso)
e decompactei.

~~~ bash
    cd ~/Downloads
    wget -c http://www.shallowsky.com/software/crikey/crikey-0.8.3.tar.gz
    tar -zxvf crikey-0.8.3.tar.gz
~~~

Gerou a saída:

~~~ bash
  crikey-0.8.3/
  crikey-0.8.3/Makefile
  crikey-0.8.3/crikey.c
  crikey-0.8.3/TESTING
~~~

Para instalar tentei:

~~~ bash
  cd crikey-0.8.3
  make
~~~

Mas deu um erro por causa de uma dependência:

~~~ bash
  gcc -Wall -Wstrict-prototypes -g -O2   -c -o crikey.o crikey.c
  crikey.c:29:34: fatal error: X11/extensions/XTest.h: No such file or directory
   #include <X11/extensions/XTest.h>
                                    ^
  compilation terminated.
  make: *** [crikey.o] Error 1
~~~

Desconfiei que fosse dependencia do pacote *libX11-devel* mas ela já estava
instalada. Um busca rápida vi que faltava a *libXtst-devel*:

~~~ bash
  sudo yum install libXtst-devel
~~~

Nos Debians/Ubuntus deve ser:

~~~ bash
  sudo apt-get install libxtst-dev
~~~

Aí sim conseguimos compilar:

~~~ bash
  cd crikey-0.8.3
  make
~~~

Output da compilação:

~~~ bash
  gcc -Wall -Wstrict-prototypes -g -O2   -c -o crikey.o crikey.c
  gcc -o crikey crikey.o -L/usr/X11R6/lib -lX11 -lXtst -lXext
~~~

E instalar:

~~~ bash
  sudo cp crikey /usr/local/bin
~~~

Um teste rápido que fiz foi rodar o comando

~~~ bash
  crikey -s 5 'Olá mundo!!!\nEscrevendo via crikey!!!'
~~~

<!-- <img class="img-responsive img-thumbnail" title="testando-crikey" alt="testando-crikey" src='/assets/images/testando-crikey.jpg' /> -->

que manda o texto depois de 5 segundos de delay para onde seu cursor estiver
focado. No caso abri o gedit, rodei o comando no console e voltei pro gedit.
Depois de exatos 5 segundos qe rodei o comando o output apareceu no meu gedit.

## Pagando as contas

Agora quando vou pagar uma conta, rodo o comando no meu terminal:

~~~ bash
  zbarcam --raw /dev/video0 | crikey -i
~~~

Onde o zbar abre minha webcam, lê o código de barras, gera um output de texto
puro que é direcionado para a entrada interativa do crikey que manda para onde
você estiver com o cursor.

Depois de rodar esse comando vou até meu internet banking, abro o pagamento de
conta, posiciono o cursor no campo do código de barras e posiciono o código de
barras de frente a webcam até ele conseguir ler. Magicamente os números aparecem
no campo.

<!-- <img class="img-responsive img-thumbnail" title="pagando-conta" alt="pagando-conta" src='/assets/images/pagando-conta.jpg' /> -->


Divirtam-se e até a próxima...

;-)
