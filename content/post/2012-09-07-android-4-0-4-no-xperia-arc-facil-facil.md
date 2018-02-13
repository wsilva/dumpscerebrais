+++
title = "Android 4.0.4 no Xperia Arc fácil fácil"
slug = "android-4-0-4-no-xperia-arc-facil-facil"
categorias = ["Tecnologia"]
tags = ["Android","FlashTool","Linux"]
date = "2012-09-07T16:33:00-03:00"
+++

Se você também tem um Xperia Arc ou Arc S e não aguenta esperar a atualização
para o Android 4 o ICS siga os passos.

<!--continua-->

## Baixando os pacotes

### Flashtool

Primeiro baixamos a última versão do FlashTool no site
[http://androxyde.github.com/](http://androxyde.github.com/) .
O projeto está bem ativo e com bastante atualizações. No dia que fiz o
upgrade no celular era a v0.9.2.0 mas no dia que escrevi o artigo
já era a v0.9.3.0. Há versões para windows e para linux.

### Imagem Flash

Baixamos também a imagem flash do Xperia Arc

- LT15i 4.1.B.0.431 Generic Global World: [http://uploaded.to/file/h6y4xlb2](http://uploaded.to/file/h6y4xlb2) 
- Se for o Arc S:
LT18i 4.1.B.0.431 Generic Global World: [http://uploaded.to/file/3a3izye6](http://uploaded.to/file/3a3izye6) 


### Instalando o FlashTool

Para instalar o Flashtool é necessário que já tenha instalado o java e
para Linux a libusb-1.0 tem que estar instalada. Para sistemas 64bits
tanto a libusb-1.0 de 64bit e a libusb-1.0 de 32bit devem ser instaladas.

#### Windows

Muito simples, só descompactar o arquivo, rodar o executável e seguir as
instruções do instalador. Preste atenção em que pasta o FlashTool foi instalado.

#### Linux

Também é simples, descompactamos o arquivo em uma pasta de instalação;
No meu caso foi em */opt/* criando o diretório */opt/FlashTool/*

Adicionar a regra abaixo para cada módulo nos arquivos do udev:

    SUBSYSTEM=="usb", ACTION=="add", SYSFS{idVendor}=="0fce", SYSFS{idProduct}=="*", MODE="0777"

Onde * é o id de cada módulo.

Para descobrir os módulos fizemos os seguintes passos:

Com o telefone ligado colocamos o cabo usb e rodamos o comando lsusb como abaixo:

    [wsilva@localhost ~]$ lsusb
    ...
    Bus 002 Device 005: ID 0fce:aaaa Sony Ericsson Mobile Communications AB
    ...

Depois com o telefone desligado rodamos novamente o comando:

    [wsilva@localhost ~]$ lsusb
    ...
    Bus 002 Device 006: ID 0fce:bbbb Sony Ericsson Mobile Communications AB
    ...

Depois ligamos o celular ao computador segurando o botão voltar para
colocá-lo em modo Flash e anotamos o lsusb novamente:

    [wsilva@localhost ~]$ lsusb
    ...
    Bus 002 Device 008: ID 0fce:cccc Sony Ericsson Mobile Communications AB Xperia Arc
    ...

Com os ids destacados em vermelho anotados criamos o arquivo /etc/udev/rules.d/98-flashtool.rules com o conteúdo:

    SUBSYSTEM=="usb", ACTION=="add", SYSFS{idVendor}=="0fce", SYSFS{idProduct}=="aaaa", MODE="0777"
    SUBSYSTEM=="usb", ACTION=="add", SYSFS{idVendor}=="0fce", SYSFS{idProduct}=="bbbb", MODE="0777"
    SUBSYSTEM=="usb", ACTION=="add", SYSFS{idVendor}=="0fce", SYSFS{idProduct}=="cccc", MODE="0777"

### Imagem Flash

Descompactamos a imagem *LT15i* ou *LT18i* baixada dentro da pasta firmwares
na pasta instalada. No nosso caso em */opt/FlashTool/firmwares/*

### Usando

Abrimos o Flashtool no nosso caso com o comando:

    sudo /opt/FlashTool/FlashTool

<img class="img-responsive img-thumbnail" title="FlashTool" alt="FlashTool" src='/assets/images/flashtool.png' />

Clicamos no botão flash e em seguida na janela que abriu na opção
flashmode e em ok.

<img class="img-responsive img-thumbnail" title="FlashTool Mode Selector" alt="FlashTool Mode Selector" src='/assets/images/flashtool2.png' />

Selecionamos a imagem que colocamos na pasta firmware e em ok.

<img class="img-responsive img-thumbnail" title="FlashTool Firmware Selection" alt="FlashTool Firmware Selection" src='/assets/images/flashtool3.png' />

Conforme a instrução colocamos ligamos o celular ao computador em modo
flash, ou seja segurando o botão voltar.

<img class="img-responsive img-thumbnail" title="FlashTool Flash Mode" alt="FlashTool Flash Mode" src='/assets/images/flashtool4.png' />

Em seguida apenas aguardamos que os arquivos sejam enviados.

Após o envio desconectamos o celular do computador ligamos o celular
e aguardamos (e essa primeira "ligada" demora demais).

Só seguir as instruções do celular e desfrutar as novidades:

<img class="img-responsive img-thumbnail" title="Print Screen" alt="Print Screen" src='/assets/images/flashtool5.png' />

*Para tirar screenshots pressionamos o botão power e o volume baixo
simultaneamente e seguramos por alguns segundos.*

## Referências

Créditos para
[Manjunatha Swamy P V](http://www.youtube.com/user/manjupvms) 
que fez o vídeo abaixo:

<iframe width="420" height="315" src="//www.youtube.com/embed/jd3dIY7XByw?rel=0" frameborder="0" allowfullscreen></iframe>

E para o fórum
[http://forum.xda-developers.com](http://forum.xda-developers.com) 
onde achei os firmwares, o FlashTool e o video do nosso amigo acima.

Té a próxima!
