---
{"dg-publish":true,"dg-permalink":"linux/gnome","permalink":"/linux/gnome/"}
---

#gnome #desktop #laptop
##  Dual Head Setup 
Falls X.org auf 2 Bildschirme konfiguriert ist, GNOME aber nur im Clone-Modus erscheint (auf beiden Bildschirmen das gleiche), dann könnte es an den gespeicherten Session Daten liegen. Abhilfe schafft das löschen folgender Dateien
  rm -fr ~/.gconf/desktop/gnome/screen/

##  gnome-power-manager 
Für die korrekte Arbeitsweise unter Debian muss der Benutzer Mitglied der Gruppe *powedev* sein((siehe /usr/share/doc/gnome-power-manager/README.Debian])).

Der gnome-power-manager ist unter anderem für das Abdunkeln des Bildschirms im Batteriebetrieb, sowie das Konfigurieren von Aktionen beim schließen des Deckels bei Laptops zuständig.


