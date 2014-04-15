---
layout: post
title: "LVM dicas rápidas"
date: 2013-05-17 10:36
comments: true
toc: true
categories: tecnologia 
---

## 1. LVM

LVM significa "Logic Volume Manager", em português "Gerenciador de volume lógico", ele gerencia discos e dispositivos de armazenamento em massa. No LVM um volume é o equivalente a uma partição de um disco.

## 1.1 Usos do LVM

O LVM é muito utilizado em servidores linux por oferecer uma capacidade de ajuste dinâmico de seus volumes.

Se você por exemplo que refazer o desenho de partições de um disco, no método tradicional você precisaria fazer backup dos dados, apagar as partições, criar um novo layout de partições, formatar as partições, reinstalar o sistema operacional e depois ainda fazer o restore dos dados, algo chato e demorado.

Se você utilizar LVM estará administrando seu armazenamento em uma camada de abstração, você trabalhará com volumes físicos (PV), grupos de volumes (VG) e volumes lógicos (LV), guarde esses nomes.

Quando você cria uma partição do disco destinada a uso via LVM esta partição será um PV (Physical Volume), e fará parte de algum VG (Volume Group), já os LV são 'fatias' de algum VG.

Um VG pode ser criado com um ou mais PVs e o LVM lhe permite adicionar outros PVs a um VG para aumentar a capacidade de armazenamento quando for necessário.

Imagine um VG como se fosse um grande dispositivo de armazenamento composto por vários PVs, a capacidade total de armazenamento de um VG é a soma da capacidade dos PVs associados a ele.

LVs (Logical Volumes) são fatias do seu VG (volume group), imagine que você tem um VG com capacidade de 100 GB, você pode ter 10 LVs de 10GB ou dois LVs de 50GB, isso é configurável, para o sistema operacional linux, um LV equivale a uma partição de um disco, pode ser formatada e montada da mesma forma que uma partição de um disco comum.

A grande vantagem do LVM é que você pode redimensionar VGs e LVs, aumentando ou diminuindo seu tamanho, e se estiver utilizando um sistema de arquivos que suporte resize, algo como ext3 ou ext4, poderá também aumentar e diminuir o sistema de arquivos sem precisar reconstruir todas as partições e reinstalar seu ambiente.

### 1.2 Vantagens

Se o seu /var está quase cheio, e se você estiver utilizando LVM, bastará adicionar um novo PV (Physical Volume) ao VG (Volume Group) que possui o LV (Logical Volume) utilizado para o o ponto de montagem /VAR, após aumentar a capacidade do VG, você poderá com poucos comandos - em poucos minutos - aumentar o tamanho do /var sem grandes impactos em seu ambiente.

Especificamente para aumentar ou diminuir o /var, será necessário parar a máquina, mas dependendo da partição não será preciso.

Você pode também diminuir um LV que não esteja usando muito espaço para aumentar outro que esteja precisando de espaço, desde que estejam no mesmo VG.

O LVM torna a administração de partições algo muito flexível.

### 1.3 Desvantagens

Um VG é composto por PVs, se um PV quebrar você perde o VG e os LVs, isso pode ser um grande problema, portanto, é importante ter confiança nos discos envolvidos em um ambiente LVM.

Se possível é interessante utilizar RAID para poder perder um ou mais discos de um LVM sem perder seu VG inteiro.

Há quem diga também que por ser uma camada a mais em seu ambiente, haverá perda de performance em relação a gravação e leitura em um PV, contudo, acho que isso é imperceptível na grande maioria dos ambientes.

Normalmente o LVM versão 2 já vem instalado na maioria das distribuições linux, em Debian e CentOS está disponível desde a instalação.

## 2. Mão na massa

Vamos aprender a trabalhar com LVM, no estilo dicas rápidas.

### 2.1 Criando partição LVM

Vamos particionar o disco sdb como exemplo

    fdisk /dev/sdb

Acesse a ajuda
 
    Command (m for help): m
   
Acompanhe a saída

```
Command action
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition
   l   list known partition types
   m   print this menu
   n   add a new partition
   o   create a new empty DOS partition table
   p   print the partition table
   q   quit without saving changes
   s   create a new empty Sun disklabel
   t   change a partition's system id
   u   change display/entry units
   v   verify the partition table
   w   write table to disk and exit
   x   extra functionality (experts only)
```

Vamos criar uma nova partição

    Command (m for help): n

Escolha o tipo de partição

```
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended

```

Eu escolhi primária, as quatro primeiras partições de um disco podem ser primárias, se você pretende ter mais de quatro partições, crie 3 primárias e uma extendida.

    Select (default p): p

Esta será a primeira partição

    Partition number (1-4, default 1): 1

Agora o fdisk me pergunta em qual setor essa partição deve começar e depois ele também pergunta o tamanho, eu vou dar ENTER e depois ENTER para usar todo o disco a partir do primeiro setor.

```
First sector (2048-125829119, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-125829119, default 125829119):
Using default value 125829119
```

Agora vou imprimir as partições

    Command (m for help): p

Acompanhe a saída

```
Disk /dev/sdb: 64.4 GB, 64424509440 bytes
255 heads, 63 sectors/track, 7832 cylinders, total 125829120 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xe61fc21b

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048   125829119    62913536   83  Linux
```
Veja que uma partição foi criada, agora vamos definir o tipo da partição, veja os tipos disponíveis

    command (m for help): l

Acompanhe a saída

```
 0  Empty           24  NEC DOS         81  Minix / old Lin bf  Solaris
 1  FAT12           27  Hidden NTFS Win 82  Linux swap / So c1  DRDOS/sec (FAT-
 2  XENIX root      39  Plan 9          83  Linux           c4  DRDOS/sec (FAT-
 3  XENIX usr       3c  PartitionMagic  84  OS/2 hidden C:  c6  DRDOS/sec (FAT-
 4  FAT16 <32M      40  Venix 80286     85  Linux extended  c7  Syrinx
 5  Extended        41  PPC PReP Boot   86  NTFS volume set da  Non-FS data
 6  FAT16           42  SFS             87  NTFS volume set db  CP/M / CTOS / .
 7  HPFS/NTFS/exFAT 4d  QNX4.x          88  Linux plaintext de  Dell Utility
 8  AIX             4e  QNX4.x 2nd part 8e  Linux LVM       df  BootIt
 9  AIX bootable    4f  QNX4.x 3rd part 93  Amoeba          e1  DOS access
 a  OS/2 Boot Manag 50  OnTrack DM      94  Amoeba BBT      e3  DOS R/O
 b  W95 FAT32       51  OnTrack DM6 Aux 9f  BSD/OS          e4  SpeedStor
 c  W95 FAT32 (LBA) 52  CP/M            a0  IBM Thinkpad hi eb  BeOS fs
 e  W95 FAT16 (LBA) 53  OnTrack DM6 Aux a5  FreeBSD         ee  GPT
 f  W95 Ext'd (LBA) 54  OnTrackDM6      a6  OpenBSD         ef  EFI (FAT-12/16/
10  OPUS            55  EZ-Drive        a7  NeXTSTEP        f0  Linux/PA-RISC b
11  Hidden FAT12    56  Golden Bow      a8  Darwin UFS      f1  SpeedStor
12  Compaq diagnost 5c  Priam Edisk     a9  NetBSD          f4  SpeedStor
14  Hidden FAT16 <3 61  SpeedStor       ab  Darwin boot     f2  DOS secondary
16  Hidden FAT16    63  GNU HURD or Sys af  HFS / HFS+      fb  VMware VMFS
17  Hidden HPFS/NTF 64  Novell Netware  b7  BSDI fs         fc  VMware aMKCORE
18  AST SmartSleep  65  Novell Netware  b8  BSDI swap       fd  Linux raid auto
1b  Hidden W95 FAT3 70  DiskSecure Mult bb  Boot Wizard hid fe  LANstep
1c  Hidden W95 FAT3 75  PC/IX           be  Solaris boot    ff  BBT
1e  Hidden W95 FAT1 80  Old Minix
```

Agora vamos definir o tipo da partição                                                               

    Command (m for help): t

Escolha LVM (tipo 8e)    
    
    Selected partition 1
    Hex code (type L to list codes): 8e
    Changed system type of partition 1 to 8e (Linux LVM)

Vamos imprimir novamente a tabela de partições do disco

    command (m for help): p

Acompanhe a saída

```
Disk /dev/sdb: 64.4 GB, 64424509440 bytes
255 heads, 63 sectors/track, 7832 cylinders, total 125829120 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xe61fc21b

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048   125829119    62913536   8e  Linux LVM
```

Ótimo, tudo certo, agora vamos gravar essa configuração no disco

    Command (m for help): w

Acompanhe a saída

```
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

Agora já podemos sair do fdisk

    Command (m for help): q
    
Pronto, para ter certeza que está tudo certo rode o comando abaixo

    fdisk -l /dev/sdb
    
Acompanhe a saída

```
Disk /dev/sdb: 64.4 GB, 64424509440 bytes
128 heads, 39 sectors/track, 25206 cylinders, total 125829120 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xe61fc21b

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048   125829119    62913536   8e  Linux LVM
```

Veja que a partição sdb1 foi criada corretamente

### 2.2 Criando PV

Agora que já temos uma partição LVM, podemos criar um volume físico (PV)

    pvcreate /dev/sdb1

Vamos checar os PVs

    pvscan

Veja detalhes dos PVs

    pvdisplay
    
Acompanhe a saída
  
```    
"/dev/sdb1" is a new physical volume of "60.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb1
  VG Name
  PV Size               60.00 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               KtmCte-RHtK-XgTi-rLnd-qGL0-0OZe-8KHm4b
```

Beleza, agora podemos criar um VG

### 2.3 Criando VG

Depois de criar os volumes físicos, podemos criar os grupos de volume

    vgcreate fileserver /dev/sdb1

Verifique os VGs existentes

    vgscan

Veja detalhes dos VGs

    vgdisplay
    
Acompanhe a saída

```    
--- Volume group ---
  VG Name               fileserver
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               60.00 GiB
  PE Size               4.00 MiB
  Total PE              15359
  Alloc PE / Size       0 / 0
  Free  PE / Size       15359 / 60.00 GiB
  VG UUID               JYkvTQ-hZ3R-Mib0-tIzm-CXmr-7NWH-UPu8UT
```

### 2.4 Adicionando PV a VG

Se você quer expandir o seu VG adicionando outra partição use o comando abaixo

    vgextend fileserver /dev/sdc1

### 2.5 Criando LV

Agora que temos um grupo de volumes, vamos criar um volume lógico
    
    lvcreate --name public --size 40G fileserver
   
Verique os LVs

    lvscan
   
Veja detalhes dos LVs

    lvdisplay
    
Acompanhe a saída

```
  --- Logical volume ---
  LV Path                /dev/fileserver/fileserver-public
  LV Name                public
  VG Name                fileserver
  LV UUID                Qn8u3P-Olgx-whQE-eLPn-OiaH-eguo-jfM8Vg
  LV Write Access        read/write
  LV Creation host, time debian64, 2013-05-16 11:51:59 -0300
  LV Status              available
  # open                 1
  LV Size                40.00 GiB
  Current LE             39424
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           254:2
```   
    
### 2.6 Formatando a partição

Depois de criar o LV já podemos formatá-lo

    mkfs.ext4 /dev/mapper/fileserver-public
    
### 2.7 Montando a partição

    mkdir /srv/public
    mount /dev/mapper/fileserver-public /srv/public   
    
### 2.8 Configurando fstab

Verifique o identificador único das partições

    blkid /dev/sdb1

Acompanhe a saída

    /dev/mapper/fileserver-public: UUID="811782cc-ddc3-43ca-8ba8-f67efa80f229" TYPE="ext4"     

No fstab insira a seguinte linha

    UUID=811782cc-ddc3-43ca-8ba8-f67efa80f229 /srv/public               ext4    defaults 0       1

### 2.9 Aumentando LV (ext4)

Desmonte a partição

    umount /srv/public
 
Vamos estender o volume lógico adicionando mais 114 GB ao LV

    lvextend -L +114GB /dev/mapper/fileserver-public

Acompanhe a saída

```
Extending logical volume fileserver to 154.00 GiB
Logical volume mirrordata successfully resized 
```

E depois vamos passar o e2fsck para o verificar o LV

    e2fsck -f /dev/mapper/fileserver-public 
 
Agora vamos aumentar a partição ext4

    resize2fs -p /dev/mapper/fileserver-public
    
Rode novamente o e2fsck para checar a partição que fora estendida
    
    e2fsck -f /dev/mapper/fileserver-public

Monte a nova partição

    mount /srv/public
    
Pronto, partição estendida.

### 2.10 Diminuindo LV (ext4)

Desmonte o filesystem

    umount /srv/public

Antes de prosseguir verifique a partição

    e2fsck -f /dev/mapper/fileserver-public
 
Diminua o LV em 40GB

    resize2fs -p /dev/mapper/fileserver-public 40G

Diminua a partição ext4 em 40 gigas

    lvreduce -L 40G /dev/mapper/fileserver-public
 
Verifique a partição 
 
    e2fsck -f /dev/mapper/fileserver-public
 
Rode mais uma vez o o resize para se certificar que partição vai ficar do mesmo tamanho do LV

    resize2fs -p /dev/mapper/fileserver-public

Verifique a partição

    e2fsck -f /dev/mapper/fileserver-public

Monte a partição    
    
    mount /srv/public

Pronto.
    
## 3. Referências

* [http://www.tldp.org/HOWTO/LVM-HOWTO/](http://www.tldp.org/HOWTO/LVM-HOWTO/)

* [http://sourceware.org/lvm2/](http://sourceware.org/lvm2/)

* [http://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)](http://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux))

* [http://pubmem.wordpress.com/2010/09/16/how-to-resize-lvm-logical-volumes-with-ext4-as-filesystem/](http://pubmem.wordpress.com/2010/09/16/how-to-resize-lvm-logical-volumes-with-ext4-as-filesystem/)

* [http://www.howtoforge.com/linux_lvm_p2](http://www.howtoforge.com/linux_lvm_p2)
