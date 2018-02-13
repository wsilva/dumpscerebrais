+++
title = "Olá Mundo v2, agora com jekyll e github pages"
slug = "ola-mundo-agora-com-jekyll-e-github-pages"
categorias = ["Curiosidades"]
tags = ["blog","github","jekyll","html","css","hello-world"]
date = "2014-12-08T22:00:00-03:00"
+++

Depois de um tempo no Blogger e de tentar dar uma cara melhor usando wordpress
resolvi dar uma chance ao static blogging e de quebra testar o github pages.
Afinal estou praticamente há 2 anos sem escrever aqui. Motivos para static
blogging ao invés de outros CMSs? Muitos, abaixo alguns deles:

 - Performance do site (não há processamento)
 - Sem paus em gerenciadores de conteúdo (é só escrever em markdown)
 - Hospedagens baratas e algumas gratuitas (github pages)
 - Backup e deploys simples (git clone, altera, git push)

<!--continua-->

Apesar de ter alguns outros sites hospedados a locaweb me cansou com inúmeros
reparos de servidores, atualizações que quebravam alguns dos sites, péssima
qualidade de hospedagem e péssimo atendimento.

Não dá pra acreditar que ainda estavam com PHP 5.2 na "hospedagem profissional",
e que para atualizar para o 5.3 tinha que rolar umas maracutaias de tipo criar
um arquivo como flag pra sinalizar a versão. PHP 5.4, 5.5, ou 5.6? Esquece
Ruby e Python, piada, após diversos tutoriais e suportes via chat nunca
consegui subir um hello world. Minto, uma vez eles mudaram alguma coisa (por
causa de um chamado) e magicamente tinha uma aplicação usando rails, só que
toda quebrada por incompatibilidade de versões de algumas gems.

## Static blogging

Primeiro dei uma olhada nos caras mais famosos, Jekyll (ruby) e Hyde (python),
mas acabei juntando muita informação e demorando para fazer uma escolha, cheguei
até a cogitar usar algum escrito em php, se alguém quiser testar, esse carinha
https://www.staticgen.com/ tem uma lista boa de static sites generators.

## Qual?

Minha decisão se baseou na quantidade de documentação, tudo que eu queria que o
blog tivesse (google analytics, add sense, integração com git, ferramenta de
comentários, etc) eu encontrava fácil no Jekyll.

Para instalar e configurar segui o tutorial em http://jekyllrb.com/.
Utilizei o tema (livre e gratuito) do Scotte
https://github.com/scotte/jekyll-clean que precisei fazer poucos ajustes para
adequar meus gostos e necessidades.

## Onde?

A hospedagem fiquei na dúvida em usar o S3 da Amazon, ou o github pages.
Optei pelo Github por causa da gratuidade e pela facilidade de deploy, já que
não precisaria configurar hooks. É só seguir o tutorial em https://pages.github.com/.
Até para usar um domínio próprio a configuração necessária estava lá e clara, o problema
é que o site ficou fora do ar e até agora não sei se por causa dos banners ou por
causa da mudança de IPs do github (mudei para a configuração nova mas não funcionou).
Na verdadade creio que foi algum plugin do Jekyll que não funcionou.
No fim mudei para o S3.


## Comentários

Para comentários utilizei o consagrado Disqus que apesar de já ter a conta nele
nunca consegui fazer o Wordpress utilizar, ou seja, a enchurrada de spams
também parou.

## E os paranauês do google?

O tema do Scotte já tinha suporte para adicionar o Google Analytics só tive que
adaptar para mostrar também os banners do Add Sense e configurá-lo para as
novas divs onde seriam carregados os novos banners.

E foi isso aí, um git push e tudo no ar. Você deve se perguntar: "Sério?
Tudo no ar?"
E na verdade não, só tive que esperar propagar os DNSs.


Divirtam-se e até a próxima...

;-)
