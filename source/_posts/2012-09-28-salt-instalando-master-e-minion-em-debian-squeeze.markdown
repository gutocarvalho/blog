---
layout: post
title: "Salt: Instalando master e minion em debian squeeze"
date: 2012-09-28 16:41
comments: true
categories: [ tecnologia, salt ]
---

Vou abordar como instalar rapidamente o salt-master e o salt-minion em debian squeeze, em seguida vou mostrar como executar comandos remotamente.

### Instalando salt-master em debian squeeze

Habilite o repositório backports e atualize os índices

    root@saltmaster:~# aptitude update

Instale o salt-master

    root@saltmaster:~# aptitude install salt-master


Veja o seu endereço IPv4

```
root@puppetmaster:~# ifconfig
eth0      Link encap:Ethernet  HWaddr 00:0c:29:12:bb:0f  
          inet addr:192.168.17.130  Bcast:192.168.17.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fe12:bb0f/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:38987 errors:0 dropped:0 overruns:0 frame:0
          TX packets:20024 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:28844823 (27.5 MiB)  TX bytes:2181661 (2.0 MiB)
```

Edite o aquivo /etc/salt/master e vá até a linha abaixo

    #interface 0.0.0.0

Modifique a linha descomentando e inserindo o IP de sua interface

    interface: 192.168.17.130

Reinicie o salt master

    root@saltmaster:~# /etc/init.d/salt-master restart

Ótimo, master pronto e instalado, agora vamos para o minion.

### Instalando salt-minion em debian squeeze

No minion que será gerenciado pelo salt-master, habilite o repositório backports e instale o pacote através do comando abaixo.

    root@saltminion:~# aptitude install salt-minion

Após instalar edite o arquivo /etc/salt/minion vá aré a linha abaixo

    #master: salt

Modifique a linha descomentando e inserindo o IP do master

    master: 192.168.17.130
    
Reinicie o salt

    root@saltminion:~# /etc/init.d/salt-minion restart
 
### Autorizando o minion a usar o master
 
Agora vá ao salt master e digite

    root@saltmaster:~# salt-key -L
    Unaccepted Keys:
    saltminion.hacklab
    Accepted Keys:
    Rejected:
   
Veja que o minion saltminion precisa ser autorizado, use o comando abaixo para autorizar o uso do salt-master.

    root@saltmaster:~# salt-key -a saltminion.hacklab
    Key for salminion.hacklab accepted.

### Executando comandos remotamente a partir do master

Vamos pedir para o salt testar se os minions estão ligados

    root@saltmaster:~# salt '*' test.ping
     saltminion.hacklab: True
     
Veja que usamos * para orientar o teste em todos os minions

    Para executar em minions de um domínio use: '*.dominio'
    Para executar em um minion específico use: 'minion.dominio'
     
Vamos executar o comando uname em todos os nodes
    
    root@saltmaster:~# salt '*' cmd.run 'uname -a'
    saltminion.hacklab: Linux saltminion.hacklab 2.6.32-5-amd64 #1 SMP Sat May 5     01:12:59 UTC 2012 x86_64 GNU/Linux

Executando comando free

    root@saltmaster:~# salt '*' cmd.run 'free'
    saltminion.hacklab:              total       used       free     shared    buffers     cached
    saltminion.hacklab: Mem:        114048     110828       3220          0       1500      62180
    saltminion.hacklab: -/+ buffers/cache:      47148      66900
    saltminion.hacklab: Swap:       217080       9608     207472

Executando comando vmstat

    root@saltmaster:~# salt '*' cmd.run 'vmstat'
    saltminion.hacklab: procs -----------memory---------- ---swap-- -----io---- -system-- ----cpu----
    saltminion.hacklab:  r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa
    saltminion.hacklab:  0  0   9608   3288   1500  62192    1    1    11     5    9    8  0  0 100  0

Executando código em python

    root@saltmaster:~#salt '*' cmd.exec_code python 'import sys; print sys.version'
    {'saltminion.hacklab': '2.6.6 (r266:84292, Dec 26 2010, 22:31:48) \n[GCC 4.4.5]'}

Executando código em bash

    root@saltmaster:~# salt '*' cmd.exec_code bash 'if [ -e /etc/resolv.conf ];then echo true;else echo false;fi'
    {'saltminion.hacklab': 'true'}

Veja que seu funcionamento é bastante simples e eficiente, eu gostei particularmente dos recursos **cmd.exec_code** e **cmd.run**.

Sigo estudando e compartilhando o que for aprendendo.

[s]<br>
Guto
