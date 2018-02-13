+++
title = "Como melhorar a segurança do seu servidor na internet"
categories = ["Tecnologia"]
tags = ["Linux","Servidor","Web","Internet","Segurança"]
date = "2015-03-01T05:20:00-03:00"
draft = true
+++

# Melhorando a segurança em seus servidores

Então você contratou seu serviço de cloud, entrou em http://cloud.locaweb.com.br com seu usuário e senha e lá estão os servidores contratados.

Com a senha do usuário root em mãos vamos levantar nossos serviços e colocar nossa máquina para operar, certo? Errado, sempre devemos nos preocupar com segurança e aqui vamos passar algumas dicas para deixar nossos ambientes mais seguros.


## 1. Não permitir acesso com usuário root.

Nunca deixe o acesso feito ao seu servidor Linux ser feito via usuário *root*, prefira sempre acesssar com um usuário comum e assumir acesso *root* apenas depois de entrar em seu servidor usando o comando `sudo su`, ou com o comando  `sudo -i` se nosso usuário comum esta no arquivo de sudoers, ou com o comando `su` se não estiver. A diferença é que com o comando su será necessário a senha do usuário *root*, com o comando sudo basta a senha do usuário comum.

Uma animação que demonstra bem o que é um servidor ligado direto na internet é a disponível no endereço 

![http://devopsreactions.tumblr.com/post/44611270530/checking-the-logs-on-a-server-thats-directly](http://33.media.tumblr.com/050e4da00a1d01ddbeb171d588ae2ab1/tumblr_inline_miss6hkQ6s1qz4rgp.gif)

Antes de bloquearmos o acesso ssh para o usuário *root* devemos ter acesso com algum usuário comum, se não temos um usuário comum podemos criar com os seguintes passos:

```
root@cpro36320:~# adduser wsilva
Adding user `wsilva' ...
Adding new group `wsilva' (1000) ...
Adding new user `wsilva' (1000) with group `wsilva' ...
Creating home directory `/home/wsilva' ...
Copying files from `/etc/skel' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for wsilva
Enter the new value, or press ENTER for the default
    Full Name []: Wellington
    Room Number []:
    Work Phone []:
    Home Phone []:
    Other []:
Is the information correct? [Y/n] Y
root@cpro36320:~#
```

Para bloquear o acesso como root devemos editar o arquivo `/etc/ssh/sshd_config`. Devemos prestar atenção aqui e não confundir. Devemos editar o arquivo *sshd_config* e não o arquivo *ssh_config*.


```
root@cpro36320:~# vim /etc/ssh/sshd_config
```

A linha com o conteúdo `PermitRootLogin yes` deve ser alterada para `PermitRootLogin no`
Gravar e sair do arquivo de configuração e em seguida reiniciar o serviço de ssh

```
root@cpro36320:~# sudo service ssh restart
ssh stop/waiting
ssh start/running, process 30751
root@cpro36320:~#
```


## 2. Mudar porta do serviço SSH

Apesar de ser uma mudança muito fácil de se fazer um bypass, o simples fato de alterar a porta padrão que roda um serviço já dificulta a ação de robos mais simplistas e cada passo que adicionamos para dificultar um acesso não autorizado já ajuda.
Um paralelo que podemos fazer é imaginar dois carros idênticos, um trancado e com alarme e outro aberto e com chaves no contato. O que tem maior probabilidade de ser furtado é que dê menos trabalho, ou seja o que já está aberto e com chaves no contato.

Para mudar a porta padrão do serviço de SSH basta alterarmos a diretiva `Port` no arquivo `/etc/ssh/sshd_config` citado no dica anterior. Para alterarmos por exemplo para o serviço de SSH responder na porta 2222 basta alterarmos no arquivo e reiniciarmos o serviço.

```
root@cpro36320:~# vim /etc/ssh/sshd_config

# What ports, IPs and protocols we listen for
Port 2222
# Port 22

root@cpro36320:~# service ssh restart
ssh stop/waiting
ssh start/running, process 28867
root@cpro36320:~#
```


## 3. Colocar mensagem para inibir acesso
É uma outra técnica que não surte muito efeito mas assusta um atacante inexperiente.

No arquivo `/etc/ssh/sshd_config` citado anteriormente devemos descomentar e definir a opção Banner:

```
root@cpro36320:~# vim /etc/ssh/sshd_config
Banner /etc/issue.net
```

No arquivo `/etc/issue.net` colocamos o texto e ascii art se tivermos.

```
root@cpro36320:~# cat /etc/issue.net
################################################################
# All connections are monitored here.                          #
# Disconnect IMMEDIATELY if you are not an authorized user.    #
# LAWS will be applied in case of RULES VIOLATION.             #
################################################################
```

Em seguida podemos reiniciar o serviço SSH e tentar acessar para visualizar a mensagem:

```
root@cpro36320:~# service ssh restart
ssh stop/waiting
ssh start/running, process 29877
root@cpro36320:~#
root@cpro36320:~# exit
exit
wsilva@cpro36320:~$ exit
logout
Connection to cpro36320.publiccloud.com.br closed.
[wsilva@localhost ~]$ ssh wsilva@cpro36320.publiccloud.com.br
################################################################
# All connections are monitored here                           #
# Disconnect IMMEDIATELY if you are not an authorized user.    #
# Laws will be applied in case of rules violation.             #
################################################################
Welcome to Ubuntu 14.04.4 LTS (GNU/Linux 3.13.0-79-generic x86_64)

 * Documentation:  https://help.ubuntu.com/
Last login: Mon Mar 27 16:18:22 2016 from 200.205.195.2
```


## 4. Bloquear ataques de bruteforce.

Para bloquear ataques de bruteforce exitem algumas ferramentas como o *denyhosts* e uma eu pessoalmente gosto que é o *fail2ban*. Eles bloqueiam temporariamente o endereço IP do atacante para qualquer tentativa de acesso via ssh. O *fail2ban* também pode ser configurado para bloquear acesso a outros serviços como apache, asterisk e mysql-auth entre outros. Também podemos personalizar as ações a serem tomadas além de simplesmente bloquear o atacante no iptables, como enviar e-mails e notificacões de alertas por exemplo.

Podemos instalar o fail2ban utilizando o gerenciador de pacotes de nossa distribuição Linux.

Para CentOS, RedHat, Fedora e similares:

```
root@cpro36320:~# yum install fail2ban
```

Para Debian, Ubuntu, Mint e similares:

```
root@cpro36320:~# apt-get install fail2ban
```

O arquivo de configuração do fail2ban fica em `/etc/fail2ban/jail.conf`. Devemos ter atenção aos seguintes parâmetros:

 - *ignoreip*: define que redes serão ignoradas do monitoramento. Deve ser declarado no formato de CIDR. Ex.: 192.168.1.0/255.255.255.0 ou 192.168.1.0/24;
 - *bantime*: é o tempo em segundos que o atacante será banido;
 - *maxretry*: define o máximo de tentativas permitidas;
 - *banaction*: define qual será a ação que o fail2ban vai tomar, o padrão é bloquear o acesso a todas as portas via iptables.

Essas configurações são para qualquer serviço e ficam dentro de `[DEFAULT]`, as diretivas específicas para acesso SSH estão agrupadas pela marcação `[ssh]` e dentre elas destacamos as seguintes:

 - *enable*: habilita o serviço;
 - *port*: define a porta a ser monitorada para o serviço;
 - *filter*: define o filtro que será usado pelo ao analisar os arquivos de logs;
 - *logpath*: define o caminho para o arquivo de log que será usado durante o monitoramento;
 - *maxretry*: usado para sobreescrever o valor padrão de tentativas global.

O mais interessante é que conseguimos ver os endereços IPs sendo bloqueados e desbloqueados no log:

```
root@cpro36320:~# tail -f /var/log/fail2ban.log
2016-03-27 14:43:05,638 fail2ban.server : INFO   Exiting Fail2ban
2016-03-27 14:43:06,214 fail2ban.server : INFO   Changed logging target to /var/log/fail2ban.log for Fail2ban v0.8.11
2016-03-27 14:43:06,215 fail2ban.jail   : INFO   Creating new jail 'ssh'
2016-03-27 14:43:06,276 fail2ban.jail   : INFO   Jail 'ssh' uses pyinotify
2016-03-27 14:43:06,325 fail2ban.jail   : INFO   Initiated 'pyinotify' backend
2016-03-27 14:43:06,328 fail2ban.filter : INFO   Added logfile = /var/log/auth.log
2016-03-27 14:43:06,329 fail2ban.filter : INFO   Set maxRetry = 3
2016-03-27 14:43:06,330 fail2ban.filter : INFO   Set findtime = 600
2016-03-27 14:43:06,331 fail2ban.actions: INFO   Set banTime = 600
2016-03-27 14:43:06,391 fail2ban.jail   : INFO   Jail 'ssh' started
2016-03-27 14:45:41,838 fail2ban.actions: WARNING [ssh] Ban 58.218.211.11
2016-03-27 14:55:41,014 fail2ban.actions: WARNING [ssh] Unban 58.218.211.11
2016-03-27 14:56:45,047 fail2ban.actions: WARNING [ssh] Ban 188.214.58.170
2016-03-27 15:06:45,227 fail2ban.actions: WARNING [ssh] Unban 188.214.58.170
2016-03-27 15:22:03,276 fail2ban.actions: WARNING [ssh] Ban 188.214.58.170
2016-03-27 15:22:25,331 fail2ban.actions: WARNING [ssh] Ban 58.218.204.30
2016-03-27 15:32:03,483 fail2ban.actions: WARNING [ssh] Unban 188.214.58.170
2016-03-27 15:32:20,533 fail2ban.actions: WARNING [ssh] Ban 188.214.58.170
2016-03-27 15:32:25,568 fail2ban.actions: WARNING [ssh] Unban 58.218.204.30
2016-03-27 15:42:20,739 fail2ban.actions: WARNING [ssh] Unban 188.214.58.170
2016-03-27 15:45:02,145 fail2ban.actions: WARNING [ssh] Ban 125.88.146.116
2016-03-27 15:55:02,322 fail2ban.actions: WARNING [ssh] Unban 125.88.146.116
2016-03-27 16:20:09,796 fail2ban.actions: WARNING [ssh] Ban 125.88.146.116
2016-03-27 16:30:09,977 fail2ban.actions: WARNING [ssh] Unban 125.88.146.116
2016-03-27 16:29:44,572 fail2ban.actions: WARNING [ssh] Ban 46.172.71.249
2016-03-27 16:39:44,749 fail2ban.actions: WARNING [ssh] Unban 46.172.71.249
```


## 5. Acesso através de chave (sem senha)

Essa técnica consiste em gerar um para de chaves (uma chave pública e uma chave privada) e enviar a chave pública para o servidor. Ao acessarmos o servidor não será mais necessário digitar nossa senha.

Primeiro passo é gerar um par de chaves em nossa máquina local caso ainda não tenhamos. Basta executar o seguinte comando e seguir as instruções da tela tais como onde gravar o arquivo da chave e senha para criptografar e gerar a chave.

```
[wsilva@localhost ~]$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/wsilva/.ssh/id_rsa): /Users/wsilva/.ssh/id_rsa_teste
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/wsilva/.ssh/id_rsa_teste.
Your public key has been saved in /Users/wsilva/.ssh/id_rsa_teste.pub.
The key fingerprint is:
SHA256:wELug3EAUQ/w2pRf8YY/5hwKXBWdOx2Mf1/RMdfVjdM wsilva@localhost
The key's randomart image is:
+---[RSA 2048]----+
|+=+ . . oo +   =X|
| . B . =  + o o.E|
|  = = * o  + . ..|
| + B + +  o o . .|
|. o *   S  . . ..|
|     o = o      .|
|      . o        |
|                 |
|                 |
+----[SHA256]-----+
[wsilva@localhost ~]$
```

Com a chave gerada agora podemos enviá-la para o servidor, será necessário digitar nossa senha criada ao gerar as chaves:

```
[wsilva@localhost ~]$ ssh-copy-id -i /Users/wsilva/.ssh/id_rsa_teste.pub cpro36320.publiccloud.com.br
/usr/local/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/Users/wsilva/.ssh/id_rsa_teste.pub"
/usr/local/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/local/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
################################################################
# All connections are monitored here                           #
# Disconnect IMMEDIATELY if you are not an authorized user.    #
# Laws will be applied in case of rules violation.             #
################################################################
wsilva@cpro36320.publiccloud.com.br's password:

Number of key(s) added:        1

Now try logging into the machine, with:   "ssh 'cpro36320.publiccloud.com.br'"
and check to make sure that only the key(s) you wanted were added.

[wsilva@localhost ~]$
```

Vamos testar acessando nosso servidor:

```
[wsilva@localhost ~]$ ssh wsilva@cpro36320.publiccloud.com.br
################################################################
# All connections are monitored here                           #
# Disconnect IMMEDIATELY if you are not an authorized user.    #
# Laws will be applied in case of rules violation.             #
################################################################
Welcome to Ubuntu 14.04.4 LTS (GNU/Linux 3.13.0-79-generic x86_64)

 * Documentation:  https://help.ubuntu.com/
Last login: Mon Mar 27 16:30:22 2016 from 200.205.195.2
wsilva@cpro36320:~$
```


## 6. Redes privadas.

Uma outra técnica muito eficaz é colocar nossos servidores em uma rede privada inacessível, nessa mesma rede deixamos um servidor exclusivo para acesso chamado bastion. O acesso a qualquer servidor é feito via bastion.

![Diagrama com bastion](http://i.imgur.com/dBCGSPx.png)


## 7. Mas meu servidores são Windows.

Algumas dicas se baseam em técnicas que podem ser aplicadas também em servidores Windows, porém devemos ter atenção com outras brechas de segurança também.

### Manter servidor atualizado
Sempre instale as atualizações de segurança disponibilizadas pela Microsoft, pelo menos uma vez por mês, elas normalmente corrigem falhas que podem ser exploradas por atacantes mal intencionados.

Basta ir em "Control Panel", "System and Security", "Windows Update" para verificar se existe alguma atualização disponível, ver o que será alterado com a atualização e instalar as atualizações disponíveis.

![Windows Update](http://i.imgur.com/ListDxK.png)
![Windows Update](http://i.imgur.com/a70ZXgl.png)


### Bloqueie o acesso remoto para administradores

Usuários administradores tem muitos privilégios, não é seguro acessar diretamente com esse tipo de usuário. Assim como no Linux é recomendado bloquear o acesso. Se necessário faça login com um usuário comum e execute os programas que precisa como administrador. Lembrando que também é possível acesso através do painel de administração da Locaweb. 

No Windows para remover este privilégio de acesso remoto devemos acessar "Computer Configuration", "Windows Settings", "Security Settings", "Local Policies", "User Rights Assignment"

![Removendo Administradores do acesso remoto](http://i.imgur.com/1BWKs9p.png)
![Removendo Administradores do acesso remoto](http://i.imgur.com/REH3cp9.png)


### Utilize Firewalls

Trabalhe com o firewall habilitado e libere somente as portas que são necessárias para que a aplicação rodando no servidor seja acessada.

Podemos acessar o Firewall do Windows 2012 Server pelo "Server Manager", "Tools", "Windows Firewall with Advanced Security".

![Windows Firewall](http://i.imgur.com/PsvXovT.png)
![Windows Firewall](http://i.imgur.com/obd5Uja.png)


### DMZ

Assim como no Linux mostramos a técnica de acesso via Bastion no Windows também é recomendado que seu servidor esteja em uma rede DMZ, protegendo sua rede interna ou privada.


## Conclusão

Esse artigo tem o objetivo de mostrar algumas técnicas para aumentar a segurança em seus servidores, somente essas dicas não garantem 100% de segurança, suas aplicações rodando sobre esses servidores também devem ser projetadas para suportar testes de penetração.

Além das técnicas de proteger acesso aos servidores também sugerimos trabalhar com monitoramento de atividades e notificação. Assim podemos tomar ações antes que uma catástrofe aconteça, porém monitoramento envolve uma série de outros detalhes sendo um bom assunto para outro artigo.

## Fontes

 - Fail2ban: http://www.fail2ban.org/wiki/index.php/Main_Page
 - DenyHosts: http://denyhosts.sourceforge.net/faq.html
 - Bastion: https://en.wikipedia.org/wiki/Bastion_host
 - SSH: http://www.openssh.com/
 - Windows 2012 Server: https://technet.microsoft.com/en-us/library/hh801901.aspx


