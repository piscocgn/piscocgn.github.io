---
{"dg-publish":true,"dg-permalink":"os/fedora","permalink":"/os/fedora/"}
---

#linux #fedora

##  Informationen 
* Homepage
* Vergleich Red Hat <-> Fedora http://fedora.redhat.com/about/rhel.html
* Download, ältere Versionen unter core/ http://download.fedora.redhat.com/pub/fedora/linux/
* [Extra Packages for Enterprise Linux (EPEL)](http://fedoraproject.org/wiki/EPEL)

##  Dokumentation 

* http://fedoraproject.org/wiki/Docs
    * http://fedoraproject.org/wiki/Docs/Drafts
* wikibooks.org - [Fedora And Red Hat System Administration: Working with repositories](http://en.wikibooks.org/wiki/Fedora_And_Red_Hat_System_Administration%3AWorking_with_repositories#_note-2)
##  Installation 
* Alternate method: at boot CD prompt: linux askmethod

##  Kernel 
* http://www.brandonhutchinson.com/Recompiling_the_Fedora_Core_kernel.html

###  Netinstall / Mirrors 
* Mirrors http://mirrors.fedoraproject.org/publiclist
* Pfad .../fedora-core/5/i386/os/ 
* [Creating a Local Update Repository for FC6](http://news.softpedia.com/news/Creating-a-Local-Update-Repository-for-FC6-43559.shtml)
* [Mirroring a Repository](http://www.phy.duke.edu/~rgb/General/yum_article/yum_article/node15.html)

```bash
#!/bin/sh
# select a rsync host close to your location from
# http://mirrors.fedoraproject.org/publiclist
MIRROR_PATH=./fedora
MIRROR_HOST=ftp-stud.hs-esslingen.de
SECTION=core
VERSION=4
ARCH=i386
EXCLUDE="--exclude=debug/ --exclude=iso/ --exclude=repodata/ --exclude=*debuginfo*"
# append excludes below to exclode something more ...
# --exclude=*i18* --exclude=*langpack*
OPTIONS="-auvH --progress --delay-updates --numeric-ids --delete-after"

rsync rsync://$MIRROR_HOST/fedora/linux/$SECTION/$VERSION/$ARCH $OPTIONS $EXCLUDE $MIRROR_PATH
```

#  Paketmanagement 
##  yum 
* http://fedoraproject.org/wiki/Docs/Drafts/SoftwareManagementGuide
* http://en.wikibooks.org/wiki/Fedora_And_Red_Hat_System_Administration:Working_with_repositories
* http://www.fedorafaq.org/#yumconf
* http://linux.duke.edu/projects/yum/
* http://www.phy.duke.edu/~rgb/General/yum_HOWTO/yum_HOWTO/
* Lokale DVD
<html>
name=Installations_DVD
baseurl=file:///mnt/DVD/
enable=1 
</html>
##  Post-Installation (optional)
* terminus-fonts(( /lib/kbd/consolefonts/ter* ))
```bash
yum install terminus-font-console
nano /etc/sysconfig/i18n
  SYSFONT "ter-i16n"
setsysfonf
```