---
{"dg-publish":true,"dg-permalink":"os/debian","permalink":"/os/debian/"}
---


#debian #linux 
##  Packaging & Repository 
###  Backports: frische Software für stable 
Aus den sogenannten Backports kann -- auf eigene Gefahr ™ -- aktuelle Software auf einen stable System installiert werden. Statt *squeeze* die entsprechende Version eintragen, und statt *package* das Paket verwenden. Ob das Paket überhaupt als Backport vorliegt kann mit *apt-cache* überprüft werden.

```bash
echo "deb http://backports.debian.org/debian-backports squeeze-backports main" >> /etc/apt/sources.list
aptitude update # alternativ apt-get update
apt-cache policy "package"
aptitude -t squeeze-backports install "package" # alternativ apt-get -t squeeze-backports install "package"
```
* http://backports.debian.org/Instructions/
###  Entwicklung 

* Lucas Nussbaum: [An Introduction to Debian Packaging](http://git.debian.org/?p=users/lucas/packaging-tutorial.git;a=blob_plain;f=packaging-tutorial.pdf;hb=refs/heads/pdf) (PDF Download)
* Debian Administration: [Setting up your own APT repository with upload support](http://www.debian-administration.org/articles/286)
* apt-get install debian-wizard (Raphaël Hertzog): [http://raphaelhertzog.com/2010/12/15/howto-to-rebuild-debian-packages/](http://raphaelhertzog.com/2010/12/15/howto-to-rebuild-debian-packages/)
    * Bei lokalen Änderungen ''dch --local myname'' um einen Eintrag in debian/changelog zu setzten
* schneller bauen mit cowpuilder http://wiki.debian.org/cowbuilder
##  Debian Etch/Lenny: Treiber für ATI Grafikkarten 
* Treiber von [ATI](http://ati.amd.com/support/driver.html) auswählen und herunterladen, z.B. in /tmp

```bash
aptitude install module-assistant debhelper libstdc++5
m-a prepare
chmod +x ./ati-driver-installer-*
./ati-driver-installer-* --buildpkg Debian/lenny
dpkg -i *.deb
aticonfig --intial --input /etc/X11/xorg.conf
```
* m-a prepare zieht fast alles was benötigt wird (z.B. build-essential)
* libstdc++6 kommt automatisch mit libstdc++5
* gegebenenfalls den ati-driver-installer-* mit ''--listpkg'' abfragen, ob die jeweilige Version unterstützt wird und was einzutragen ist (Debian/lenny usw.)
##  Langsame Netzwerkverbindung 
Gegebenenfalls IPv6 deaktivieren, siehe [hier](http://www.debian-administration.org/articles/409)

##  Dropbox für Debian Lenny/Squeeze 
* Auf der Homepage von Dropbox das Paket für Ubuntu 7.10 [herunterladen](https://www.dropbox.com/downloading?os=lnx) und mit

  `dpkg - i ...`

installieren.