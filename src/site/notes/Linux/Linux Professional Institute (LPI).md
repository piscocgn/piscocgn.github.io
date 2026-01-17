---
{"dg-publish":true,"dg-permalink":"linux/zertifizierung","permalink":"/linux/zertifizierung/"}
---

#linux
##  Vorbereitungsmöglichkeiten 
* webbasierte Prüfungssimulation, Study Guides für die einzelnen Prüfungen und Themen
    * http://www.lpi-test.de/ ((derzeit keine Vorbereitungsmöglichkeiten für LPIC-2, Stand 05.09.2006))
* Prüfungssimulator lpisim, Handhabung mit tclkit ist recht umständlich, zumindest auf Debian oder Ubuntu. Die Fragen kommen auch bei http://www.lpi-test.de/ zum Einsatz
    * lipsim http://lpi-buch.linupfront.de/simulator.html
* Sehr gutes Schulungsmaterial von IBM ('Big Blue'), man merkt das die Jungs bei Unix zu Hause sind. Zu den Themen gibt es jeweils praxisnahe Übungen.
    * http://www-128.ibm.com/developerworks/linux/lpi/index.html ((Registrierung notwendig))
* freie Online-Ausgabe von 'Wie werde ich UNIX-Guru?' von Arnold Willemer, vieles ist unmittelbar auf Linux anwendbar.
    * http://www.galileocomputing.de/openbook/unix_guru/
* SelfLinux eignet sich ebenso hervorragend um einzelne Themenbereiche zu festigen. Besonderes Augenmerk sollte auf die Herleitung Netzmaske zu IP-Adressenbereich gerichtet werden.
    * http://www.selflinux.org/
* verschiedene Links rund um das LPI
    * http://linuxwiki.org/LPI
* LPI-Acadamy: Prüfungssimulation von Anselm Lingnau
    * [[http://www.lpi-academy.de/wiki/home/\|http://www.lpi-academy.de/wiki/home/]]
###  Empfehlung 
* Bücher
    * Open Source Press Verlag [LPIC-1](http://www.amazon.de/gp/redirect.html?link_code=ur2&tag=polyformal-21&camp=1638&creative=6742&location=%2FLPIC-1-Vorbereitung-Pr%25fcfung-Professional-Institute%2Fdp%2F393751421X%2Fsr%3D8-1%2Fqid%3D1157457221%2Fref%3Dpd_ka_1%3Fie%3DUTF8%26s%3Dgateway), [LPIC-2](http://www.amazon.de/gp/redirect.html?link_code=ur2&tag=polyformal-21&camp=1638&creative=6742&location=%2FLPIC-2-Vorbereitung-Pr%25fcfung-Professional-Institute%2Fdp%2F3937514201%2Fsr%3D8-1%2Fqid%3D1157457315%2Fref%3Dpd_ka_1%3Fie%3DUTF8%26s%3Dgateway)
* Wer Debian, Ubuntu oder andere 'Nicht-RPM'-Distributionen verwendet, für den ist es zum Vorbereiten recht hilfreich z.B. über ~~VMWare Player~~ VirtualBox ein OpenSUSE((wenn schon rpm Distribution dann besser CentOS, das nahezu 100% kompatibel zu Red Hat (lpisim nicht getestet) )) einzusetzen, zumal dort der lpisim recht unproblematisch funktioniert.
    * http://www.vmware.com/products/player/
    * KDE on SUSE prebuild appliance for VMWare Player http://developer.kde.org/~binner/vmware/
* Moderne Distributionen setzten in der Regel auf X.org, Prüfungsstoff ist jedoch XFree86. Zu empfehlen für 101 X-Windows ist Debian Sarge.

###  Abnahme der Prüfung 
Die Prüfung kann in Köln zum Beispiel im [Com Center](http://www.com-training.com/de/zertifizierungen/linux.html) abgelegt werden.

##  LPIC-2 
###  201 
####  213 
#####  1) Automating tasks using scripts 
* awk
    * IBM developers: [Awk by example, Part 1](http://www.ibm.com/developerworks/linux/library/l-awk1.html?ca=drs-)