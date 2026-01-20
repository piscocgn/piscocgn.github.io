---
{"dg-publish":true,"dg-permalink":"software/remind","permalink":"/software/remind/"}
---

* [Homepage](http://www.roaringpenguin.com/products/remind)
* zahlreiche Informationen im wiki von [43folders.com](http://wiki.43folders.com/index.php/Remind)
* [rem2ics](https://launchpad.net/rem2ics) - Konvertierung von Remind zu ICS

##  Feiertage 

Deutsche Feiertage für remind
```bash
# you have to include .reminders_setting

MSG Feiertage:%"%"%
## Feiertage
OMIT     Jan  1         MSG %"Neujahr%"%
REM	 Jan  6		MSG %"Heilige Drei Könige (regionaler Feiertag)%"%
REM      Feb 14         MSG %"Valentinsitag%"%
OMIT     Dec 31         MSG %"Silvester%"%
REM      Mar  8         MSG %"Internationaler Frauentag%"%

# Ostern
SET SaveTrig $NumTrig
SET easter EASTERDATE(YEAR(TODAY()))
REM  [TRIGGER(easter-48)] MSG %"Rosenmontag%"%
REM  [TRIGGER(easter-46)] MSG %"Aschermittwoch%"%
OMIT [TRIGGER(easter-2)]  MSG %"Karfreitag%"%
OMIT [TRIGGER(easter)]    MSG %"Ostersonntag%"%
OMIT [TRIGGER(easter+1)]    MSG %"Ostermontag%"%
OMIT [TRIGGER(easter+39)] MSG %"Christi Himmelfahrt%"%
OMIT [TRIGGER(easter+49)] MSG %"Pfingsten%"%
OMIT [TRIGGER(easter+50)] MSG %"Pfingstmontag%"%
REM [TRIGGER(easter+60)]  MSG %"Fronleichnam (regionaler Feiertag)%"%

REM      Apr  1         MSG %"1. April :)%"%
OMIT  	 May  1		MSG %"1. Mai, Tag der Arbeit%"%
REM Sun  May 8 MSG %"Muttertag%"%
REM	 Aug  8 	MSG %"Augsburger Friedensfest%"%
REM 	 Aug 15		MSG %"Mariä Himmelfahrt (regionaler Feiertag)%"%

# The DST rules are accurate for most locations in
# North America
REM Sun [_last(Mar)] 	MSG %"Umstellung auf Sommerzeit%"%
REM Sun [_last(Oct)] 	MSG %"Umstellung auf Winterzeit%"%

OMIT  	 Oct  3		MSG %"Tag der Deutschen Einheit%"%

REM 	Oct	31	MSG %"Reformationstag (regionaler Feiertag)%"%
REM	Nov	1	MSG %"Allerheiligen (regionaler Feiertag)%"%
REM	Wed Nov 23 --7	MSG %"Buß- und Bettag (regionaler Feiertag)%"%
#TotenSonntag
REM	Sun Dec 24 --35 MSG %"Totensonntag%"%
#Advent
REM	Sun Dec 24 --28 MSG %"1.Advent%"%
REM	Sun Dec 24 --21 MSG %"2.Advent%"%
REM	Sun Dec 24 --14 MSG %"3.Advent%"%
REM	Sun Dec 24 --7 	MSG %"4.Advent%"%
#Weihnachten
REM	Dec  6		MSG %"Nikolaus%"%
OMIT	Dec  24		MSG %"Heiliger Abend%"%
OMIT	Dec  25		MSG %"1. Weihnachtstag%"%
OMIT	Dec  26		MSG %"2. Weihnachtstag%"%
```

###  GUIs für remind 
###  wxremind (wxpython) 
* http://www.duke.edu/~dgraham/wxRemind/
* benötigt ''python-wxgtk2.8 python-dateutil'' (Debian/Squeeze)
###  wyrd (ncurses) 
* [Projekt-Seite](http://pessimization.com/software/wyrd/)
* [Wyrd](http://wiki.43folders.com/index.php/Wyrd) bei [[http://www.43folders.com/\|http://www.43folders.com/]]
* {{software:wyrd-1.4.4_german.patch.gz|}}

##  Remember The Milk zu remind 
* basiert auf einem Tip aus dem [RTM-Forum](http://www.rememberthemilk.com/forums/tips/5178/)
* Voraussetzungen: [ICal2Rem](http://wiki.43folders.com/index.php/ICal2Rem)
* benötige Perl-Pakete Debian Squeeze
    * ''libical-parser-perl''
`rtm2ical.sh`
```bash
#!/bin/bash
TMP_FILE=$(mktemp)
# Benutzername und Passwort müssen in der ~/.netrc hinterlegt sein:
# machine www.rememberthemilk.com
# login rtm_username
# password rtm_password
RTM_USER=[rtm_user]
RTM_URL="https://www.rememberthemilk.com/icalendar/$RTM_USER/events/"
wget -qO $TMP_FILE RTM_URL && ical2rem.pl $TMP_FILE  > ~/.remind/.reminders_rtm && rm $TMP_FILE
```