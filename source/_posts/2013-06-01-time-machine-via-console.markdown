---
layout: post
title: "Time machine via console"
date: 2013-06-01 18:14
comments: true
categories: mac
---

Estive pesquisando sobre como deletar manualmente backups antigos do time machine e descobri o comando tmutil, ele pode ser utilizado para realizar essa tarefa via console, mas não é só isto, o tmutil permite manipular de forma direta o time machine, veja abaixo alguns comandos interessantes.

Para listar os backups existentes em seu disco

    tmutil listbackups
    
Acompanhe a saída

```
/Volumes/GutoWD1TB/Backups.backupdb/kaiten/2013-04-02-085203
/Volumes/GutoWD1TB/Backups.backupdb/kaiten/2013-04-02-134033
/Volumes/GutoWD1TB/Backups.backupdb/kaiten/2013-04-19-125352
```

Para saber qual o backup mais recente

    tmutil latestbackup

Acompanhe a saída

    /Volumes/GutoWD1TB/Backups.backupdb/kaiten/2013-04-19-125352

Para habilitar backups automáticos
    
    tmutil enable
    
Para desabilitar backups automáticos

    tmutil disable

Para habilitar snapshots locais do time machine (no mesmo disco)

    tmutil enablelocal
    
Para desabilitar snapshots locais do time machine

    tmutil disablelocal

Para iniciar um backup manualmente

    tmutil startbackup
    
Para interromper um backup em progresso

    tmutil stopbackup

Para criar um snapshot local (no mesmo disco)

    tmutil snapshot

Para deletar todos os backups de um dispositivo

    tmutil delete /Volumes/drive_name/Backups.backupdb/old_mac_name

Para deletar um snapshot específico de um dispositivo específico

    tmutil delete /Volumes/drive_name/Backups.backupdb/mac_name/YYYY-MM-DD-hhmmss
    
Para associar um disco ao time machine

    tmutil setdestination /Volumes/backupdrive
    
Para associar um ponto de montagem afp via rede ao time machine

    tmutil setdestination afp://user:password@server-address/directory
    
Para adicionar um diretório a lista de exclusão

    tmutil addexclusion /Downloads
    
Para remover um diretório da lista de exclusão

    tmutil removeexclusion /Downloads
    
Para comparar os arquivos de backup com os arquivos atuais em seu disco

    tmutil compare
    
Acompanhe a saída

```
- 181B                          /Volumes/GutoWD1TB/Backups.backupdb/kaiten/2013-06-01-201328/OSX/.com.apple.backupd.mvlist.plist
+ 0B                            /.dbfseventsd
!         (mtime)               /.DocumentRevisions-V100/.cs
! 12.0M   (size, mtime)         /.DocumentRevisions-V100/.cs/ChunkStorage/0/0/0/18
! 2.4M    (size, mtime)         /.DocumentRevisions-V100/.cs/ChunkStoreDatabase-wal
!         (mtime)               /.DocumentRevisions-V100/ChunkTemp
!         (mtime)               /.DocumentRevisions-V100/PerUID/501/110/com.apple.documentVersions
+ 1.2K                          /.DocumentRevisions-V100/PerUID/501/110/com.apple.documentVersions/B849E3FA-CBC3-4E33-8859-E278C049B78A.markdown
! 2.7M    (size, mtime)         /.DocumentRevisions-V100/db-V1/db.sqlite-wal
!         (mtime)               /.DocumentRevisions-V100/staging
```

Entenda as legendas

    + significa arquivo novo
    - significa arquivo removido
    ! significa arquivo modificado

Para comparar apenas o tamanho dos arquivos do backup com os atuais em disco

    tmutil -s
    
Para comparar arquivos do disco com um backup mais antigo

    tmutil compare /Volumes/TimeMachineDriveName/Backups.backupdb/mac_name/AAAA-MM-DD-hhmmss
    
Para saber mais

    tmutil --help
    tmutil help <verb>
    man tmutil
    
[s]<br>
Guto 
