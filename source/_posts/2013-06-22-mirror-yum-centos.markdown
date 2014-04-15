---
layout: post
title: "Mirror Yum CentOS"
date: 2013-06-22 11:15
comments: true
categories: tecnologia
---

Em um cliente me foi solicitado criar um mirror local para pacotes CentOS 5 e 6, abaixo compartilho os scripts que foram criados, vai que te ajuda ;)

Os scripts utilizam rsync para pegar os pacotes.

## 1. Scripts

### 1.1. CentOS Base/Updates/Extras/Media

Este baixa pacotes oficiais CentOS para CentOS 5 e 6.

```
#!/bin/bash

LOCKFILE="/var/lock/subsys/rsync_centos_updates"
CENTOSDIR="/srv/mirror/centos"
MIRROR6="rsync://mirror.steadfast.net/centos/6.4"
MIRROR5="rsync://mirror.steadfast.net/centos/5.9"
if [ -f $LOCKFILE ]; then
    echo "Updates via rsync already running."
    exit 0
else
	if [ -d $CENTOSDIR ] ; then
	   touch $LOCKFILE
           rsync -avSHP $MIRROR6 --delete --exclude 'local*' --exclude isos --exclude SRPMS $CENTOSDIR
           rsync -avSHP $MIRROR5 --delete --exclude 'local*' --exclude isos --exclude SRPMS $CENTOSDIR
	   rm -f $LOCKFILE
	else
	    echo "Target directory $CENTOSDIR not present."
	fi
fi
```

### 1.2. Epel

Este baixa pacotes EPEL para CentOS 5 e 6.

```
!/bin/bash

LOCKFILE="/var/lock/subsys/rsync_updates_epel"

EPELDIR6="/srv/mirror/epel/centos/6/x86_64"
MIRROR6="rsync://fedora.mirror.nexicom.net/Fedora-EPEL/6/x86_64/"

EPELDIR5="/srv/mirror/epel/centos/5/x86_64"
MIRROR5="rsync://fedora.mirror.nexicom.net/Fedora-EPEL/5/x86_64/"

if [ -f $LOCKFILE ]; then
    echo "Updates via rsync already running."
    exit 0
else
	if [ -d $EPELDIR6 ] && [ -d $EPELDIR5 ] ; then
	   touch $LOCKFILE
           rsync -avSHP $MIRROR6 --delete --exclude SRPMS $EPELDIR6/
           rsync -avSHP $MIRROR5 --delete --exclude SRPMS $EPELDIR5/
	   rm -f $LOCKFILE
	else
	    echo "Target directory $EPELDIR6 or $EPELDIR5 not present."
	fi
fi
```

### 1.3. PuppetLabs

Este baixa pacotes oficiais Puppet para CentOS 5 e 6.

```
#!/bin/bash

LOCKFILE="/var/lock/subsys/rsync_updates_puppet"

PUPPETDIR5="/srv/mirror/puppet/products/5/x86_64"
MIRROR5="rsync://yum.puppetlabs.com/packages/yum/el/5/products/x86_64/"

DEPPUPPETDIR5="/srv/mirror/puppet/dependencies/5/x86_64"
DEPMIRROR5="rsync://yum.puppetlabs.com/packages/yum/el/5/dependencies/x86_64/"

PUPPETDIR6="/srv/mirror/puppet/products/6/x86_64"
MIRROR6="rsync://yum.puppetlabs.com/packages/yum/el/6/products/x86_64/"

DEPPUPPETDIR6="/srv/mirror/puppet/dependencies/6/x86_64"
DEPMIRROR6="rsync://yum.puppetlabs.com/packages/yum/el/6/dependencies/x86_64/"

PARAMS="-av --exclude SRPMS --delete"

if [ -f $LOCKFILE ]; then
    echo "Updates via rsync already running."
    exit 0
else
	if [ -d $PUPPETDIR5 ] && [ -d $PUPPETDIR6 ] && [ -d $DEPPUPPETDIR5 ] && [ -d $DEPPUPPETDIR6 ]   ; then
	   touch $LOCKFILE
	   rsync $PARAMS $MIRROR6 $PUPPETDIR6
	   rsync $PARAMS $MIRROR5 $PUPPETDIR5
	   rsync $PARAMS $DEPMIRROR6 $DEPPUPPETDIR6
	   rsync $PARAMS $DEPMIRROR5 $DEPPUPPETDIR5
	   rm -f $LOCKFILE
	else
	    echo "Target directory $PUPPETDIR5 or $PUPPETDIR6 not present."
	fi
fi
```

### 1.3. Zabbix

Este baixa pacotes oficiais Zabbix para CentOS 5 e 6.

```
#!/bin/bash

LOCKFILE="/var/lock/subsys/rsync_updates_zabbix"

ZBXDIR5="/srv/mirror/zabbix/centos/5/x86_64"
MIRROR5="rsync://repo.zabbixzone.com/centos/5/x86_64/"

ZBXDIR6="/srv/mirror/zabbix/centos/6/x86_64"
MIRROR6="rsync://repo.zabbixzone.com/centos/6/x86_64/"

PARAMS="-av --exclude SRPMS --delete"

if [ -f $LOCKFILE ]; then
    echo "Updates via rsync already running."
    exit 0
else
        if [ -d $ZBXDIR5 ] && [ -d $ZBXDIR6 ] ; then
           touch $LOCKFILE
           rsync $PARAMS $MIRROR6 $ZBXDIR6
           rsync $PARAMS $MIRROR5 $ZBXDIR5
           rm -f $LOCKFILE
        else
            echo "Target directory $ZBXDIR5 or $ZBXDIR6 not present."
        fi
fi
```

## 2. Apache Config

No apache apenas criei um arquivo chamado mirror.conf dentro de /etc/httpd/conf.d com o conteúdo abaixo:

```
#
# repositorio
#

Alias /mirror /srv/mirror

<Directory /srv/mirror>
	Options Indexes
	Options +FollowSymLinks
	AllowOverride none
	Order Allow,Deny
	Allow from all
</Directory>

```

## 3. YUM Config

### 3.1 Para CentOS 6

```
# repositorio centos

[base]
name=CentOS 6.4 Base x86_64
baseurl=http://mirror.local/mirror/centos/6.4/os/x86_64
enabled=1
gpgcheck=0
exclude=puppet*,zabbix*

[updates]
name=CentOS 6.4 Updates x86_64
baseurl=http://mirror.local/mirror/centos/6.4/updates/x86_64
enabled=1
gpgcheck=0
exclude=puppet*,zabbix*

# repositorio epel, pacotes extras enterprise

[epel]
name=CentOS 6 EPEL x86_64
baseurl=http://mirror.local/epel/centos/6/x86_64
enabled=1
gpgcheck=0
exclude=puppet*

# puppet-agent packages from http://yum.puppetlabs.com/

[puppet-products]
name=CentOS 6 PuppetLabs Packages x86_64
baseurl=http://mirror.local/mirror/puppet/products/6/x86_64
enabled=1
gpgcheck=0

[puppet-dependencies]
name=CentOS 6 PuppetLabs Packages x86_64
baseurl=http://mirror.local/mirror/puppet/dependencies/6/x86_64
enabled=1
gpgcheck=0
```
### 3.1 Para CentOS 5

```
[base]
name=CentOS 5 Base x86_64
baseurl=http://mirror.local/mirror/centos/5.9/os/x86_64
enabled=1
gpgcheck=0
exclude=puppet*,zabbix*

[updates]
name=CentOS 5 Updates x86_64
baseurl=http://mirror.local/mirror/centos/5.9/updates/x86_64
enabled=1
gpgcheck=0
exclude=puppet*,zabbix*

# repositorio epel, pacotes extras enterprise

[epel]
name=CentOS 5 EPEL x86_64
baseurl=http://mirror.local/epel/centos/5/x86_64
enabled=1
gpgcheck=0
exclude=puppet*

# puppet-agent packages from http://yum.puppetlabs.com/

[puppet-products]
name=CentOS 5 PuppetLabs Packages x86_64
baseurl=http://mirror.local/mirror/puppet/products/5/x86_64
enabled=1
gpgcheck=0

[puppet-dependencies]
name=CentOS 5 PuppetLabs Packages x86_64
baseurl=http://mirror.local/mirror/puppet/dependencies/5/x86_64
enabled=1
gpgcheck=0
```

[s]<br>
Guto