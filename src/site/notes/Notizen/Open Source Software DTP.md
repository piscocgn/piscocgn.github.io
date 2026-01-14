---
{"dg-publish":true,"dg-permalink":"/notizen/oos-dtp","permalink":"/notizen/oos-dtp/"}
---

#dtp #linux 
## Bildbearbeitung (Raster) 

###  Gimp 
* http://www.gimp.org , [[wpde>GIMP\|wpde>GIMP]]
* [Dokumentation](http://docs.gimp.org/de/)
* [Photoshop-Plugins unter GIMP](http://www.gimp.org/~tml/gimp/win32/pspi.html) - auch für Linux
* CMYK in GIMP
    * [separate](http://www.blackfiveservices.co.uk/separate.shtml)
    * [separate+](http://cue.yellowmagic.info/softwares/separate.html), Weiterentwicklung von separate
* [Liquid Rescale Plugin](http://liquidrescale.wikidot.com/)
* RAW Format
    * [gimp-ufraw](http://ufraw.sourceforge.net/) (Nikon)
    * [gimp-dcraw](http://www.lebsanft.org/blog/index.php?cat=9)
####  Sonderformen / Forks 
* [GIMPshop](http://www.gimpshop.com/) - GIMP im Photoshop 'Look & Feel' (Debian-Pakete verfügbar)
    * Im wesentlichen sieht GIMPShop auf dem ersten Blick kaum anders aus. Die Terminologie und die Anordnung der Menüs und deren Punkte entspricht denen in Photoshop, und macht es Umsteigern etwas leichter.
<note warning>
Die Verwendung von GIMPshop ist insofern problematisch, dass man hier durch die abweichende Terminologie schwer in der Lage ist GIMP-Tutorials zu folgen und Support durch die Community zu bekommen.
</note>
* [[wpde>Cinepaint\|wpde>Cinepaint]]
###  Panorama-Tools 
*  [Hugin](http://hugin.sourceforge.net/) - ein Meta-Tool für
    * [autopano-sift](http://user.cs.tu-berlin.de/~nowozin/autopano-sift/) für die automatische Suche von Kontrollpunkten in Bildern
    * [enblend](http://enblend.sourceforge.net/) zum Ausgleichen unterschiedlicher Belichtungsverhältnisse (Blending)
* ~~[HOWTO](http://exolucere.ca/articles/create-panorama) für hugin, autopanosift und enblend~~((Link ist leider ungültig))
* [HOWTO](http://www.panoclub.de/hugin_tut/index.html) für Hugin in Deutsch

##  Gestaltung / Illustration 

###  Inkscape 
####  Informationen 
* Freehand
* Guide to Inkscape http://tavmjong.free.fr/INKSCAPE/
* Mouse and Keys http://www.inkscape.org/doc/keys.html
* inoffizielles Handbuch http://wiki.inkscape-forum.de/
* Tips and Tricks http://www.inkscape.org/doc/tips/tutorial-tips.html
* Manual http://tavmjong.free.fr/INKSCAPE/MANUAL/html/index.html
    * Import http://tavmjong.free.fr/INKSCAPE/MANUAL/html/ch03s02.html
* Feautre Screenshots http://inkscape.org/screenshots/index.php?version=0.44
* PDF Präsentationen http://volition.wustl.edu/slides/
* Animation
    * http://tavmjong.free.fr/INKSCAPE/MANUAL/html/Web-Animation.html
    * Beispiel http://tavmjong.free.fr/INKSCAPE/DRAWINGS/clock_plain.svg
* Präsentationen (mit Animationen)
    * sozi http://sozi.baierouge.fr/wiki/
    * JessyInk http://code.google.com/p/jessyink/
* http://www.unicode.org/charts/
* Beispiele http://inkscape.svn.sourceforge.net/viewvc/inkscape/inkscape/trunk/share/examples/
* Inkscape to TikZ exporter (SVG2LaTeX) http://code.google.com/p/inkscape2tikz/

###  Vektorisieren 
    * [potrace](http://potrace.sf.net/)
    * [autotrace](http://autotrace.sf.net/)
    * [Vector Magigc](http://vectormagic.stanford.edu/) - The Online Tool for Precision Vectorization (Stanford University)
####  GUI 
* ~~http://delineate.sourceforge.net/~~((Entwicklung steht still, Probleme mit Java aktueller als J2SE 1.4.1 oder 1.4.2)) autotrace, potrace
* http://potracegui.sourceforge.net/ für potrace und autotrace (QT)

#### = SVG 
* [[Impress#Sonstiges \| OpenOffice.org SVG Support]]

##  Layout / Druck 

###  Scribus 
* http://www.scribus.net/ , [[wpde>Scribus\|wpde>Scribus]]
* Debian http://debian.scribus.net
* [Farbprofile](http://www.scribus.net/index.php?name=Downloads&req=viewdownload&cid=16)
* [Dokumentation](http://docs.scribus.net/) (englisch)
* [Wiki](http://wiki.scribus.net/index.php/Main_Page)

###  Color Management System 
* Monitor einstellen, Beleuchtung http://www.normankoren.com/makingfineprints1A.html
* Little Color Management System Profiler [LPROF](http://lprof.sourceforge.net/)
* [QuickGamma](http://quickgamma.de/) (Windows), für Linux [GAMMApage](http://www.pcbypaul.com/software/GAMMApage.html) (Python/Gtk) oder [Monica](http://www.pcbypaul.com/software/monica.html)
* [Heise c't Screen](http://www.heise.de/ct/ctscreen/#CTSCREEN) (Java Applet)
* [PANTONE: Kalibrierung](http://www.pantone.de/pages/pantone/pantone.aspx?ca=2)

##  Schriften / Fonts 
* Wissenswertes über den [Drucker-Jargon](http://www.blog.druckerey.de/index.php?id=40) aus dem Blog von Martin Z. Schröder
* [[wpde>TrueType\|wpde>TrueType]] Datiendung .TTF, [[wpde>Type1-Font\|wpde>Type1-Font]] Dateiendung .PFB ist die Schrift und .PFM enthält Informationen zur [[wpde>Unterschneidung (Typografie) \| Unterschneidung]] (Kerning)
* als Einzelbenutzer ~/.fonts/, systemweit /usr/share/fonts/[truetype|type1]/
<note tip>
Falls die Schriften nicht angezeigt werden nachdem sie als Benutzer oder systemweit installiert worden muss ggf. fc-cache ausgeführt werden, bzw. sudo fc-cache für systemweite Installation.
</note> 
* Inkscape: [Installing Fonts as a User](http://wiki.inkscape.org/wiki/index.php/Installing_Fonts_as_a_User)
* Scribus: [Installing additional fonts](http://wiki.scribus.net/index.php/Installing_additional_fonts)

###  Installation 
* TrueType & Type 1 http://wiki.ubuntuusers.de/Schriften

###  Management 
* [gfontview](http://gfontview.sourceforge.net/)
* [Fonty Python](https://savannah.nongnu.org/projects/fontypython), verteilte Fonts zu 'pogs' sichten, zusammenfassen, installieren und entfernen (truetype)
* [FONTPage](http://www.pcbypaul.com/software/FONTpage.html)

##  PDF 
* pdftk
* [pdfjam](http://www2.warwick.ac.uk/fac/sci/statistics/staff/academic/firth/software/pdfjam/)
* [Multivalent Tools & Browser](http://multivalent.sourceforge.net/index.html) (Java) - erstellen von Notizen zu PDF-Dokumenten (werden zusätzlich in einem XML-Format gespeichert.

##  Werkzeuge 
* [pstoedit](http://www.pstoedit.net/): PostScript und PDF's zu editierbaren Formaten umwandeln (SVG, WMF, Adobe Illustrator)
* eps2svg http://www.montefiore.ulg.ac.be/~piater/personal/software/
* [uniconvertor](http://sk1project.org/modules.php?name=Products&product=uniconvertor)((in Debian [python-uniconvertor](http://packages.debian.org/search?suite=default&section=all&arch=any&searchon=names&keywords=python-uniconvertor))): wandelt diverse, unter anderem proprietäre Vektor-Formate wie Corel .cdr oder Adobe Illustrator .ai bis 9.x und Windiws Meta File .wmf, in gängige Freie Formate wie SVG um.

##  3D 
* Blender((http://orange.blender.org/))
* Material Editor für Lightflow: MATSpider ((nur für Windows, mit wine nicht getestet)) http://homepage3.nifty.com/escargot/DownLoadFrm.html
* Lightflow http://www.lightflowtech.com/
* YafRay http://www.yafray.org/
