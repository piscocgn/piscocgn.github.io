---
{"dg-publish":true,"dg-permalink":"linux/xfce/notizen","permalink":"/linux/xfce/notizen/"}
---

#xfce #desktop
##  Xorg - Composite-Effekte für Xfce 
* siehe http://wiki.ubuntuusers.de/Xfce_Composite-Effekte
###  xorg.conf 

```bash
Section "Device"
...
        Option          "RenderAccel"           "true"
        Option          "AllowGLXWithComposite" "true"
EndSection

...

Section "Extensions"
         Option  "Composite" "Enable"
EndSection
```


##  Tomboy Default Browser 
Wenn Tomoby mit dem *xfce4-xfapplet-plugin* unter *Xfce* betrieben wird, kann es vorkommen dass beim Klick auf eine URL mit http oder https zu einer Meldung kommt:

{{ linux:xfce:tomboy_url_handler_msg.png|}}
<html>
Der Ort kann nicht geöffnet werden
...
Die Vorgabeaktion unterstützt dieses Protokoll nicht
</html>

Tomboy bezieht die Konfiguration aus den GNOME URL-Handlern, welche mit dem gconf-editor angepasst werden können:
* den gconf-editor öffnen
* den Zweig * /desktop/gnome/url-handlers/http/ * aufsuchen
    * den Eintrag "command" auf "exo-open --launch WebBrowser %s"((*exo-open --launch WebBrowser* öffnet den in *Einstellungen -> Bevorzugte Anwendungen* voreingestellten Browser)) oder direkt ein Programmnamen wie "firefox %s" eingeben.
    * den Eintrag "enabled" auf true setzen.
* gegebenenfalls die gleichen Einstellungen bei *https* vornemhen

##  Zuletzt geöffnete Dokumente / Recent Files 
Oft ist ein Menü mit den zuletzt geöffneten Dokumenten recht nützlich. Auf der  von [Sven Müller](http://www.thelightmaker.de/) gibt es ein entsprechende [Anleitung](http://www.thelightmaker.de/knowhow/xfce_recent_files), basierend auf ein Skript welches die entsprechenden Informationen aus der Desktop-Suchmaschine [[http://www.beagle-project.org/Beagle\|http://www.beagle-project.org/Beagle]] herausholt.
<note tip>
Für Debian muss das Paket *libgnomevfs2-bin* installiert sein, und naturlich *beagle* selbst.
</note>
