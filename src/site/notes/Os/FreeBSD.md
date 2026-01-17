---
{"dg-publish":true,"dg-permalink":"/os/freebsd","permalink":"/os/freebsd/"}
---

#gnome #unix

* [Das FreeBSD Handbuch](http://www.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/index.html)
## Administration 
Für viele Administrative Aufgaben steht *root* das Programm ''sysinstall'' zur Verfügung.
### Softwareverwaltung 
#### pkg_add
##### Proxy 
FreeBSD benutzt im Auslieferungszustand die *csh*, hier muss ein Proxy wie folgt in der Umgebung definiert werden:

  `setenv http_proxy http://ip.proxy[:port]`

##### Abweichenden Softwarespiegel verwenden, z.B. nur HTTP
Wenn der Proxy kein FTP zulässt, dann muss ein Spiegel ausgewählt werden, der HTTP unterstützt. Das ist nicht bei allen Spiegeln der Fall. Eine [Übersicht](http://www.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/mirrors-ftp.html) der Spiegel ist im FreeBSD Handbuch zu finden.

`setenv PACKAGESITE http://ftp1.de.freebsd.org/freebsd/ports/i386/packages-8.1-release/Latest/ `

Dann wie gewohnt Pakete mit

`pkg_add -r foo`

instalieren.

##### Eigener partieller Spiegel 
Eine Liste mit rsync-Spiegeln ist im FreeBSD [Handbuch](http://www.freebsd.org/doc/handbook/mirrors-rsync.html) zu finden.

```bash
MIRROR=ftp.nl.FreeBSD.org
ARCH=i386
RELEASE=x.x
rsync -auv --delete --progress $MIRROR::FreeBSD/ports/$ARCH/packages-$RELEASE-release/ FreeBSD-$RELEASE-$ARCH
```
### su - ermöglichen =
Ein *normaler* Benutzer muss Mitglied der Gruppe *wheel* sein, um ''su -'' durchführen zu können:
  pw user mod foo -G wheel
## Gnome 
* gnome2, gdm und xorg installieren
* Falls im GDM keine Anmeldung möglich ist: /proc fehlt http://www.freebsd.org/gnome/docs/faq2.html#procfs

/etc/fstab

```bash
proc           /proc       procfs  rw  0   0
```

* gdm/Gnome beim Booten starten: 

```bash
/etc/rc.conf: gnome_enable="YES"
```
### CUPS 
Damit die in CUPS eingerichteten Drucker auch in den Druckdialogen von GNOME-Anwendungen angezeigt werden müssen die Pakete *libgnomeprint* und *gtk20* (Stand FreeBSD 8.1) erneut mit der Option ''WITH_CUPS=yes'' übersetzt werden, siehe [CUPS im FreeBSD Wiki](http://www.freebsdwiki.net/index.php/CUPS#CUPS_and_Gnome_Warning). Alternatik kann mittels des Paketes ''gtklp'' in den Druckdialogen gleichnamiger Befehl als Druck-Kommando verwendet werden.

* Probelm ''htmlview not found''
    *  ''cd /usr/ports/print/cups-base && make config'' und XDG_OPEN wählen ((http://forums.freebsd.org/showthread.php?t=1291))
### gdm3 
#### Benutzerliste deaktivieren 
* Hierzu muss dem System-Account *gdm* kurzfristig eine Login-Shell zugewiesen werden, und mit ''gconftool-2'' die entsprechenden Einstellungen vorgenommen werden.

```bash
chsh -s /bin/sh gdm
su - gdm gconftool-2 --set --type boolean /apps/gdm/simple-greeter/disable_user_list true
chsh -s /usr/sbin/nologin gdm
```
#### gdm Anmeldedialog Standard auf Deutsch 
in der Datei ''/etc/rc.conf''

```bash
gdm_lang="de_DE.UTF-8"
```

## CUPS 
### Remote Administration 
Beim Zuschalten der Option *Allow remote administration* verweigert CUPS spätestens bei Authentifizierung das Upgrade auf https.

var/log/cups/error_log
```bash
Unable to create SSL server key file "/usr/local/etc/cups/ssl/server.key" - no such file or directory
```

Wie die Fehlermeldung aussagt scheint der Pfad nicht in Ordnung zu sein. Beim Überprüfen stellt sich heraus, dass das Verzeichnis ''/usr/local/etc/cups/ssl/'' noch nicht existiert.
