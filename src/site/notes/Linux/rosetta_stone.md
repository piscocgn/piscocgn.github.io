---
{"dg-publish":true,"permalink":"/linux/rosetta-stone/"}
---

[[En/Linux/rosetta_stone\|En/Linux/rosetta_stone]]
## Rosetta Stone Linux (oder Inter-Linux)
Folgende Übersicht soll eine Gegenüberstellung von Befehlen der einzelnen Distributionen zu diversen Aufgaben darstellen, um Unterschiede aufzuzeigen oder Hilfestellung bei einem Umstieg zu bieten.
<flattr>
title=Rosetta Stone Linux (oder Inter-Linux)
uid=30553
description=Eine Tabelle für Distributions-Hopper
category=software
align=left
</flattr>

##  Paketmanagement 

|Aktion |Debian((http://debiananwenderhandbuch.de/aptitude.html)) | Red Hat/CentOS/Fedora((http://docs.fedoraproject.org/yum/en/)) | openSUSE(([Zypper](http://en.opensuse.org/Zypper) seit 10.2 Beta1, basiert auf libzypp)) | Gentoo((http://www.gentoo.de/doc/de/handbook/handbook-x86.xml?part=2&chap=1)) |
|---|---|---|---|---|
|Quellen aktualisieren | aptitude update|  --  | zypper refresh | emerge --sync |
|vorhandene Updates einspielen | aptitude safe-upgrade| yum update | zypper update | emerge -u[p]D world |
|Paket suchen | aptitude search [Paketname]| yum list [Paketname]| zypper search [Paketname] | emerge -s [Paketname] |
|Paketbeschreibung anzeigen | aptitude show [Paketname]| yum info [paketname]| zypper info [Paketname] | emerge -s [Paketname] |
|Paket installieren | aptitude install [Paketname]| yum install [Paketname]| yast2 install / -i [Paketname]  zypper install | emerge [Paketname] |
|Die Zugehörigkeit einer Datei zu einem  installierten Paket anzeigen|dpkg -S [Dateiname] (( ein spezielles Kommando kann über $(which [command]) abgefragt werden ))|  rpm -qf [Dateiname] (( ein spezielles Kommando kann über $(which [command]) abgefragt werden ))  || equery (( Der Befehl *equery* stammt aus dem Paket *app-portage/gentoolkit* )) belongs [Dateiname] (( ein spezielles Kommando kann über $(which [command]) abgefragt werden )) |
|Die Zugehörigkeit einer Datei zu einem  nicht-installierten Paket anzeigen|apt-file find [Dateiname] ((Bei dem ersten Einsatz muss apt-file update ausgeführt werden, um den Cache aufzubauen))  | yum whatprovides [Dateiname]  | |  --((Gentoo wird direkt aus dem Quellcode der einzelnen Projekte kompiliert. Daher ist eine Zuordnung von Dateien bei nicht installiertem Paket nicht möglich.))  |
|Den Inhalt eines nicht-installierten  Paketes aus den Repositorien anzeigen | apt-file -F list [Paketname] ((Bei dem ersten Einsatz muss apt-file update ausgeführt werden, um den Cache aufzubauen)) | | | |
|Informationen eines vorliegenden  Paketarchives anzeigen | dpkg -I [Paketname.deb]((-I oder --info)) |  rpm -qpi [Paketname.rpm]  || |
|Den Inhalt eines vorliegenden  Paketarchives anzeigen | dpkg -c [Paketname.deb]((-c oder --contents)) |  rpm -qpl [Paketname.rpm]  || |
|Den Zustand installierter  Pakete abfragen und anzeigen| debsums [Paketname]((Die MD5-Datenbank muss vorher mit *debsums_init* initialisert werden.))  debsums -as|  rpm -V [Paketname]  rpm -Va  ||  equery((Der Befehl *equery* stammt aus dem Paket *app-portage/gentoolkit*)) check [Dateiname]  |
|Paketinhalt ohne Installation  entpacken | dpkg -x [Paketname.deb] [Zielverzeichnis]((-x oder --extract, verbose mit -X oder --vextract))  oder  1. ar xv [Paketname].deb  2. tar -xzvf data.tar.gz  |  <html>rpm2cpio [Paketname].rpm | cpio -idmv</html>  ||  --((Da Gentoo aus dem Quellcode der jeweiligen Projekte installiert wird, kann man ein ''emerge -f [Paketname]'' machen, das Archiv aus ${DISTDIR} (e.g. ''/usr/portage/distfiles'') kopieren und dann mit ''tar'' auspacken.))  |
|Paket aus den Quellen bauen | apt-build install --rebuild [Paketname]((ggf. vorher apt-get build-dep [Paketname] um vorher nötige Bibliotheken zu installieren)) |  rpm -i [Paketname].src.rpm  cd /usr/src/packages/SPEC  rpmbuild -ba [Paketname].spec  || ist bei Gentoo die Standard-Vorgehensweise|
| Kernel komplilieren  |||||
|---|---|---|---|---|
|notwendige Tools installieren | aptitude install build-essential kernel-package | | zypper install -t pattern devel_kernel | |
|Kernel kompilieren und  als Paket erstellen | sudo fakeroot make-kpkg   --revision $REVISION --initrd   kernel_image modules_image | make oldconfig   ggf. vorher ''rm *.spec''((bei neuen Revisionen))  make binrpm-pkg   rpm -i /usr/src/packages/RPMS/i386 ..   mkinitrd -k /boot/Kernel-file -i /boot/initrdfile   ||  |

##  Benutzer- und Gruppenverwaltung 

|Aktion |Debian((http://debiananwenderhandbuch.de/benutzerverw.html)) | Red Hat/CentOS/Fedora(([Dokumentation CentOS](http://www.centos.org/docs/5/html/Deployment_Guide-en-US/ch-users-groups.html))) | openSUSE | Gentoo |
|---|---|---|---|---|
|Benutzer anlegen |  useradd [-m][-g Primäre Gruppe((Die primäre Gruppe wird auch *initial group* oder *login group* genannt))] Benutzer  ||||
|Primäre Gruppe((Die primäre Gruppe wird auch *initial group* oder *login group* genannt)) eines Benutzers ändern |  usermod -g Gruppe Benutzer  ||||
|Benutzer löschen |  userdel [-r] Benutzer  ||||
|Gruppe anlegen |  groupadd Gruppe  ||||
|Benutzer zu einer Gruppe hinzufügen | adduser Benutzer Gruppe((Debian-spezifisch))  gpasswd -a Benutzer Gruppe | usermod -aG Gruppe[,Gruppe] Benutzer  gpasswd -a Benutzer Gruppe |groupmod -A Benutzer Gruppe | usermod -aG Gruppe[,Gruppe] Benutzer  gpasswd -a Benutzer Gruppe |
|Benutzer von einer Gruppe entfernen | deluser Benutzer Gruppe((Debian-spezifisch))  gpasswd -d Benutzer Gruppe | gpasswd -d Benutzer Gruppe |groupmod -R Benutzer Gruppe | gpasswd -d Benutzer Gruppe |
|Gruppenadministrator ernennen| gpasswd -A Benutzer,... Gruppe ||(seit 10.x?) nicht mehr implementiert((über LDAP))| gpasswd -A Benutzer,... Gruppe |
|eine Liste von Benutzern  einer Gruppe hinzufügen((bestehende Mitgliedschaften werden ersetzt))|  gpasswd -M Benutzer,... Gruppe  ||||

##  Einstellungen 
|Aktion |Debian | Red Hat/CentOS/Fedora | openSUSE | Gentoo |
|---|---|---|---|---|
| umask System-weit ändern | /etc/profile |  |  /etc/login.defs | |
|Proxy, systemweit| /etc/profiles:  ''export http_proxy=http://user:password@proxy[port]'' | | | |
|Dienste für (Default-) Runlevel aktivieren | update-rc.d [name] defaults |  insserv [name] ''|'' chkconfig -a name(([Unofficial SUSEFAQ - Starting and Stoping Services: insserv](http://susefaq.sourceforge.net/faq/services.html))) || |


#linux #unvollständig 