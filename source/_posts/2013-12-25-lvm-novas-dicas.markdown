---
layout: post
title: "LVM novas dicas"
date: 2013-12-25 19:05
comments: true
toc: true
categories: console
---

## 1. Novas dicas

Já mostrei algumas dicas de LVM aqui neste [post](http://gutocarvalho.net/octopress/2013/05/17/lvm-dicas-rapidas/), mas essa semana eu passei por alguns apertos que me ajudaram a descobrir novos macetes que compartilho aqui com vocês.

O Zabbix vinha alertando que alguns servidores estavam com pouco de espaço em algumas partições, aproveitando o final do ano partimos para realizar alguns ajustes e expandir algumas partições.

Abaixo respondo algumas perguntas comuns e compartilho as soluções encontradas.

### 1.1 Linux Single vs LiveCD

Devo usar linux single para fazer a manutenção de uma partição ou um livecd?

Na realidade depende, se você quer redimensionar uma partição que não seja a / ou /usr, fazendo o procedimento da forma recomendada, verificando os discos antes e depois de redimensionar (e2fsck) é bem possível utilizar o linux single, desmontar a partição e depois utilizar as ferramentas LVM para aumentá-la ou diminuí-la, dependendo de sua necessidade.

Se precisa mexer no / ou /usr, e vai seguir as recomendações verificando as partições antes e depois de redimensionar (e2fsck), prefira utilizar sempre um livecd pois em modo single você não vai conseguir desmontar o / ou /usr.

Vou pode também fazer resize **on-the-fly**, sem verificar os volumes, veremos isto mais a frente.

### 1.2 Devo adicionar um novo disco ou expandir o disco existente em caso de máquina virtual?

Se você está utilizando um hypervisor - como o VMWARE - que permite a utilização de discos inteligentes, é possível apenas aumentar o disco (inclusive com a vm ligada), criar uma nova partição com o espaço livre que acrescentou, forçar a releitura da geometria do disco, transformá-la em PV, adicioná-la ao VG e depois utilizar o novo espaço livre no VG para expandir o LV que necessita de disco.

#### 1.2.1 Vários discos em uma VM (VMWARE)

Já me perguntaram se tem algum problema ter vários discos em uma VM, eu particularmente não vejo, mas o pessoal especializado em VMWARE alerta que ter vários discos pode aumentar a complexidade de administração e a VM terá maior custo no cluster HA do VMWARE, vale o alerta. Outro detalhe é que perder algum destes discos significará perder o VG, por isso sempre esteja com o seu backup em dia.

### 1.3 Eu dei boot em modo rescue utilizando o CD/DVD da distribuição, porém no shell os comandos pvdisplay, vgdisplay e lvdisplay não mostram nada.

Isso pode acontecer, inclusive aconteceu comigo recentemente e foi o que me motivou a escrever este post.

No caso do CentOS 6.x e Debian 7.x, o modo rescue não está ativando os volumes LVM por default, então neste caso precisamos fazer isto manualmente.

#### 1.3.1 Resolvendo a questão

Você pode forçar a detecção desta forma no shell do livecd

    # lvm pvscan
    # lvm vgscan
    # lvm lvscan
   
No meu caso ele até detectou o PV e até encontrou o VG, contudo não vi os LVs. Para ativar os VGs e LVs precisamos utilizar o seguinte comando:
 
    # lvm vgchange -a y
    
A partir daí você conseguirá enxergar e manipular os volumes lógicos, fica a dica.

### 1.4 Devo fazer resize on-the-fly de partições ext3/ext4?

O recurso existe e funciona muito bem desde o kernel 2.6, você pode ver neste [post](http://theducks.org/2009/11/expanding-lvm-on-boot-disk-under-vmware-3-5-without-rebooting/) um exemplo disto, contudo você não rodará o e2fsck para checar sua partição antes e depois de fazer o resize, e caso o resize2fs reclame você terá de fazer o resize da forma tradicional.

Eu acho arriscado dependendo da criticidade e do tamanho do volume, prefiro estar em um ambiente LiveCD com as partições desmontadas para fazer a manutenção. Em determinados cenários eu particularmente acho que o risco de corromper a partição é maior do que o benefício de fazer o resize on-the-fly, mas em determinados cenários pode ser a salvação da pátria, então vamos entender como isso funciona.

### 1.4.1 Método on-the-fly adicionando nova partição

Primeiro aumente o tamanho do disco no VMWARE com a VM ligada mesmo, isto só é possível com discos THIN.

Depois vá no OS, se você aumentou por exemplo espaço do disco SDA, entre nas configurações deste disco

    # cfdisk /dev/sda
 
Se você não está enxergando o espaço que você adicionou, saia do cfdisk e rode o comando abaixo

    # echo '1' > /sys/class/scsi_disk/0\:0\:0\:0/device/rescan
    
Entre novamente no cfdisk e verifique se está enxergando o espaço livre

    # cfdisk /dev/sda 
        
Agora crie uma nova partição do tipo LVM como espaço em disco que você adicionou, depois saia do cfdisk e transforme essa nova partição em PV (volume físico)

    # pvcreate /dev/sdaX

Se o sei OS não deixou você rodar o comando e está pendido reboot, apenas rode o comando abaixo

     # partprobe -s

Tente novamente, agora vai funcionar

    # pvcreate /dev/sdaX

Adicione o PV ao VG existente

    # vgextent vg_nome /dev/sdaX
        
Expanda seu LV (exemplo /opt)

    # lvextent -L +100G /dev/mapper/vg_nome_opt

Faça o resize da partição

    # resiz2fs -p /dev/mappaer/vg_nome_opt 

Rode o DF para verificar se a partição aumentou

    # df -h

E a magia on-the-fly aconteceu.

Eu fiz em dois servidores em produção, sem downtime, sem reboot, sem umount, funciona mesmo.

### 1.4.2 Método on-the-fly expandindo um PV existente

É possível fazer o resize do PV existente ao invés de criar um novo PV, neste [post](http://theducks.org/2009/11/expanding-lvm-partitions-in-vmware-on-the-fly) você verá esse procedimento de forma detalhada, contudo, isto envolve deletar e recriar uma partição de seu disco, e ao recriar você precisa marcar o setor exato de início e fim, segundo o tutorial funciona, mas eu acho arriscado, e só funcionará de forma fácil se o seu PV for a última partição do disco, enfim, é mais uma alternativa, fica a dica.

## 2. Referências

Oficiais:

  * http://www.tldp.org/HOWTO/LVM-HOWTO/activatevgs.html
  * http://www.redhat.com/archives/anaconda-devel-list/2012-April/msg00210.html
  
Blogs:  
  
  * http://theducks.org/2009/11/expanding-lvm-partitions-in-vmware-on-the-fly
  * http://theducks.org/2009/11/expanding-lvm-on-boot-disk-under-vmware-3-5-without-rebooting/
  * http://www.linuxuser.co.uk/features/resize-your-disks-on-the-fly-with-lvm
  * http://dailypackage.fedorabook.com/index.php?/archives/159-System-Recovery-Week-Using-LVM-In-Rescue-Mode.html=
  * http://grokbase.com/t/centos/centos/11187s4xr4/livecd-system-recovery-mounting-lvm
