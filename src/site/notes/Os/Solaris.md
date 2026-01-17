---
{"dg-publish":true,"dg-permalink":"os/solaris","permalink":"/os/solaris/"}
---

##  zusätzliche Software

Oracle (ehem. Sun) bietet eine CD mit Freeware an, welche einiges an Open Source Software enthält. Die Programme sind aber recht alt (Stand 02/2011)
* http://www.sun.com/software/solaris/freeware/
###  Repositories 
* http://www.blastwave.org/
* http://www.opencsw.org/
    * http://www.opencsw.org/get-it/pkg-get/
* http://www.sunfreeware.com/
* http://www.openpkg.org/

##  Diverses 
###  Ein ISO-Image einbinden 

```bash
  lofiadm -a /path/to/iso_image.iso /dev/lofi/1
  mount -F hsfs -o ro /dev/lofi/1 /mnt/iso
```

###  Drucker einbinden 
* das grafische Toll ''printmgr''
* PPD-Dateien einbinden mit ''ppdmgr''
* ''lpadmin''

###  Konsole 
* ''less'' als Pager für ''man''

für Bourne-Shell: ''~/.profile''
```bash
PAGER=/usr/bin/less
export PAGER
```

###  Dienste 
Solaris setzt ab Version 10 auf die [[wpde>Service Management Facility\|wpde>Service Management Facility]] (SMF) und ersetzt damit in weiten Teilen [[wpde>System V\|wpde>System V]]. Einer der wenigen Dienste die sich auf traditionellem Wege starten und stoppen lassen ist [[wpde>sendmail\|wpde>sendmail]], ws auch nur ein Wrapper um SMF ist.

###  VIM 
Der weiterentwickelte Editor [[VI-VIM \| VIM]] ist auf der [[ \| Solaris Companion CD]] oder Software [[]] wie http://opencsw.org zu finden. Um *Syntax Hilightning* zu nutzen muss die Variable TERM entsprechend gesetzt werden:

  `TERM=sun-color`
###  Prozessverwaltung 
* http://developers.sun.com/solaris/articles/prstat.html
##  Fehlermeldungen 
###  xscreensaver 
Und sowas von einem 'kommerzeiellen' Produkt in der Standard-Installation:

  Beim Starten des Bildschirmschoners ist ein Fehler aufgetreten:
  Der untergeordnete Prozess "xscreensaver" (Datei oder Verzeichnis nicht gefunden) konnte nicht ausgeführt werden.
  Der Bildschirmschoner ist in dieser Sitzung nicht verfügbar.

![solaris10_xscreensaver.png.png](/img/user/media/os/solaris10_xscreensaver.png.png)
####  Lösung 
Das Programm *xscreensaver* kann nicht aufgerufen werden, da das Programm in ''/usr/openwin/bin'' lokalisiert ist. Die Variable PATH muss in ''/etc/profile'' oder ''~/.profile'' um ''/usr/openwin/bin'' erweitert werden.

###  apropos cmd 
Das Kommando ''apropos'' liefert keine Ergebnisse sondern folgende Meldung:

```bash
/usr/dt/man/windex: Datei oder Verzeichnis nicht gefunden
/usr/man/windex: Datei oder Verzeichnis nicht gefunden
/usr/openwin/share/man/windex: Datei oder Verzeichnis nicht gefunden
```
####  Lösung

''man apropos'' verweist auf catman um die Indexes anzulegen. ''catman -w'' ausführen.
#unix