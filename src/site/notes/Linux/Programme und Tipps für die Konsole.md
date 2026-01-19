---
{"dg-publish":true,"dg-permalink":"linux/console","permalink":"/linux/console/"}
---

#linux #console

* [linux.com: Top 10 Linux console applications](http://www.linux.com/feature/44366)

##  Z-Shell (zsh) 
Die Z-Shell ist eine sehr leistungsfähige Alternative zur Standard-Shell vieler Distributionen, der Bash. Jedoch zeigt sich die *zsh* nach der Installation nicht sehr 'kooperativ'. Ein guter Anfang können die Konfigurationsdateien der Distribution [[wpde>Grml\|wpde>Grml]] sein ((Da grml eine Debian-basierte Distribution ist, sind einige Debian-spezifische Fuktionen enthalten. Die angesprochenen Fuktionen werdenauf anderen Systemen nicht weiter stören, da sie einfach nicht funktionieren werden.)):

  wget -O .zshrc http://git.grml.org/f/grml-etc-core/etc/zsh/zshrc

Die Fuktionen sind auf der [grml-zsh-refcard (PDF)](http://grml.org/zsh/grml-zsh-refcard.pdf) gut dokumentiert, ausdrucken lohnt sich.

* http://grml.org/zsh/

Folgende Zeile - vorzugsweise in der ~/.zshrc.local - zeigt in der *killall [tab-completition]* alle Prozesse des Benutzers an, auch der X-Sitzung.
  zstyle ':completion:*:processes-names' command 'ps c -u ${USER} -o command | uniq'

##  Screen 
Auf für *Screen* hält *grml* eine durchaus sinnvolle Konfiguration bereit:
  wget -O ~/.screenrc     'http://git.grml.org/?p=grml-etc-core.git;a=blob_plain;f=etc/grml/screenrc;hb=HEAD'
Allerdings sollte man folgende Zeilen auskommentieren, sofern man nicht die Programme *cpu-screen* und *ip-screen* aus von *grml* auf dem System hat((oder installieren via .deb Paket von [[http://deb.grml.org/pool/main/g/grml-scripts/\|http://deb.grml.org/pool/main/g/grml-scripts/]])):
  #backtick 1 0 60   /usr/bin/cpu-screen
  #backtick 2 0 60   /usr/bin/ip-screen

##  Diverses 
* Calc http://home3.inet.tele.dk/frda/src/calc_tip.html
* http://www.pimpmyshell.de/
####  Kalender 
* [Wyrd](http://www.eecs.umich.edu/~pelzlpj/wyrd/) -  Ein grafisches Frontend für reminder
    * [iCal2rem](http://wiki.43folders.com/index.php/ICal2Rem) - Ein Tool zu konvertieren vom iCal- zum reminder-Format

##  Kommunikation 

###  Mutt 
* Mutt Info's und Hacks, auch nntp http://www.michael-prokop.at/computer/tools_mutt.html
    * http://www.fefe.de/muttfaq/faq.html
* http://wiki.mutt.org/index.cgi?MuttWiki
* http://www.muttrcbuilder.org/
* [GTD mit mutt](http://www.footils.org/cms/show/59)
* [E-Mails mit mutt 'Verschlagworten'](http://auriga.wearlab.de/~alb/other/mutt-labels/)
* [goobook: Google Kontakte in mutt](http://castrojo.wordpress.com/2009/01/28/gmail-contacts-with-mutt/)
####  Dokumentation 
* [Dokumentation](http://www.uni-koeln.de/rrzk/mail/software/mutt/doc/manual-de/manual-de.html#toc1) vom RRZ der Uni Köln
* hilfreiche [Hinweise](http://firmen-links.net/index.php/SSMTP) über die Konfiguration SSMTP((simple SMTP))

###  Newsreader & RSS 
####  Newsreader 
* srln oder gepatchter mutt
#####  slrn 
* [Offizielle Dokumentation](http://www.slrn.org/manual/slrn-manual.html)
* [Deutsche Einführung](http://alfie.ist.org/projects/slrn/quick.de.html) von Alfie, auch mit weiteren Ressourcen((http://alfie.ist.org/projects/slrn/)).
* HOWTO kompakt http://ugweb.cs.ualberta.ca/howtos/slrn.html
* [Ressourcen](http://www.michael-prokop.at/computer/tools_slrn.html) von Michael Prokop (grml Projekt)
* [slrnpull](http://strcat.de/eigenes/slrnpull.html)
* [Introduction to Usenet News and the slrn Newsreader](http://alcor.concordia.ca/topics/netnews/slrn/intro/)
* slrn: Problem mit Umplauten, vim und UTF-8 / iso-8859-(1|15)
    * eine [Lösung](http://malison.org/archives/899-slrn-%2B-vim-%2B-utf-8.html) von Carsten Müller (tut es bei mit nicht)
    * über luit, siehe http://de.gentoo-wiki.com/Utf8
####  RSS 
    * [Snownews](http://kiza.kcore.de/software/snownews/)
    * [Newsbeuter](http://synflood.at/newsbeuter.html)

###  Instant Messaging 
* bitlbee Multi IM Gateway http://www.bitlbee.org/main.php/intro.html
* Scripts für irssi und bitlbee http://the-timing.nl/stuff/irssi-bitlbee
###  SSH 
* [Steve Friedl: An Illustrated Guide to SSH Agent Forwarding](http://www.unixwiz.net/techtips/ssh-agent-forwarding.html)

##  Editoren 

* [[Software/VI-VIM\|VI-VIM]]

##  Sys-Admin's Tools 
* multitail, ccze - Logfiles übersichtlich
<note tip>
ccze & less
  [output cames from here] | ccze -A | less -R
</note>
* colordiff - Unterschiede in Dateien farbig darstellen
<box round 98%| aliases e.g. .profile >
```
# nice functions for VCS 
# usage: (hg|svn|cvs)diff [file or dir]
# or     diffless [file1] [file2]

if [ -x "$(which colordiff)" ] ; then 
        if [ -x "$(which less)" ] ; then
                lessdiff () { diff -ruN $1 $2 | colordiff | less -R }
        fi

        if [ -x "$(which cvs)" ] ; then
                cvsdiff () { cvs diff -uN "${@}" | colordiff | less -R }
        fi

        if [ -x "$(which svn)" ] ; then
                svndiff () { svn diff "${@}" | colordiff | less -R }
        fi

        if [ -x "$(which hg)" ] ; then
                hgdiff () { hg diff "${@}" | colordiff | less -R }
        fi
fi
```</box>
* iftop - Monitoring von Netzwerktraffic
* ncdu - 'grafische' Oberfläche für du (ab Debian lenny)
* pwgen - sichere Passwörter, alternativ
<html>head -c12 /dev/random | uuencode -m - | sed -n '2s/=*$//;2p'</html>
* [colored man pages](http://nion.modprobe.de/blog/archives/569-colored-manpages.html)
* tree - Verzeichnissinhalte als Baum

##  Grafik / PDF 
* [fbi/fbgs](http://linux.bytesex.org/fbida/) - Pixelgrafiken und PDF auf der Konsole ansehen (nicht unter X-Terminals, nur echte tty's)
    * [Debian Pakete](http://packages.debian.org/search?suite=default&section=all&arch=any&searchon=names&keywords=fbi)
* für andere Terminals (X, SSH)

  pdftotext pdf-file.pdf - | less
  oder 
  lesspipe

##  Linux Logo 
* http://www.linux-user.de/ausgabe/2007/08/086-zubefehl/index.html?print=y
* http://www.deater.net/weave/vmwprod/linux_logo/USAGE
Um das Logo über MOTD anzuzeigen müssen bei Debian noch folgende Änderungen durchgeführt werden:
<box 98% round|Änderungen für Anzeige über MOTD>
```
Index: /etc/init.d/bootmisc.sh
===================================================================
--- /etc/init.d/bootmisc.sh     (Revision 24)
+++ /etc/init.d/bootmisc.sh     (Arbeitskopie)
@@ -40,7 +40,7 @@
        fi

        # Update motd
-       uname -snrvm > /var/run/motd
+       /usr/bin/linux_logo > /var/run/motd
        [ -f /etc/motd.tail ] && cat /etc/motd.tail >> /var/run/motd

        # Save kernel messages in /var/log/dmesg
```</box>
##  Werkzeugkiste 
* mit sed - Stream Editor rekursiv Suchen und ersetzen

  find . -type f -exec sed -i 's/search/replace/g' {} +

* Lange, umgebrochene dn's in einer langen Zeile darstellen ((http://www.computing.net/answers/linux/remove-leading-space-and-join-previous-line/31227.html))

  sed -e :a -e '$!N;s/n //;ta' -e 'P;D' file.ldif

