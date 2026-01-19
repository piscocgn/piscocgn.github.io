---
{"dg-publish":true,"dg-permalink":"linux/umts","permalink":"/linux/umts/"}
---

* [UMTS](https://de.wikipedia.org/wiki/Universal_Mobile_Telecommunications_System)

#linux 
### Abfrage der Netzabdeckung 
* [T-Mobile](http://www.t-mobile.de/funkversorgung/inland)
* [Vodafone](http://www.vodafone.de/hilfe-support/netz-uebertragung-netzabdeckung/108099.html)
* [E-Plus](http://eis03sn1.eplus-online.de/evportal/portal/gsm)

### Übertragungsraten 
Bis dato konnte ich eine Spitze von 290 KB/sec messen (Download eines DVD-Images via bittorrent).

## T-Mobile web'n'walk Stick III 
* [Informationen](http://www.polyformal.de/de/systemarbeit/t_mobile_webnwalk_stick_iii_unter_linux.html) zum Gerät und Tarif
* die für andere Karten [nozomi](http://www.pharscape.org/)-Teiber werden hier **nicht** benötigt

![wen_n_walk_stick_3.jpg](/img/user/media/linux/wen_n_walk_stick_3.jpg)
### Identifikation der Karte 
Bei dem Stick handelt es sich um das [Model E17X](http://www.huawei.com/mobileweb/en/products/view.do?id=940) vom Hersteller [Huawei](http://www.huawei.com). Über minicom können Details zum Gerät abgefragt werden (die letzten 8 Ziffern der IMEI wurden durch X ersetzt).

minicom auf /dev/ttyUSB0, Ausgabe des Befehls ATI
```
ATI
Manufacturer: huawei
Model: E17X
Revision: 11.306.07.01.55
IMEI: 3592980XXXXXXXX
+GCAP: +CGSM,+DS,+ES
```
###  Installation 

> [!NOTE] 
> Unter Debian Lenny ( a.k.a Testing, getestet mit 2.6.24-1-686-bigmem ) ist der unten genannte Umweg mit *huaweiAktBbo.c* und udev-Rules nicht mehr nötig. Die Devices */dev/ttyUSB0* und */dev/ttyUSB1* werden sofort verfügbar gemacht.

Leider werden abhängig von Distribution und Kernelversion die zur Kommunikation notwendigen seriellen Schnittstellen */dev/ttyUSB0* und */dev/ttyUSB1* nicht in jedem Fall automatisch erkannt. Statt dessen zeigt *dmesg* nur das CD-ROM Device mit den Windows-Treiber. Eine gute [Beschreibung der Problematik](http://linux.frankenberger.at/Huawei_E220.html) bei Frankenberger, hauptsächlich geht es um das Helfer-Programm [huaweiAktBbo](http://www.kanoistika.sk/bobovsky/archiv/umts/) von  bobovsky und die entsprechenden Einträge in der Datei */etc/udev/rules.d/50-huawei-e220.rules*
* libusb-dev muss installiert sein (aptitude install libusb-dev)
* cd /usr/src
* mkdir umts && cd umts
* wget http://www.kanoistika.sk/bobovsky/archiv/umts/huaweiAktBbo.c
* cc huaweiAktBbo.c -lusb -o huaweiAktBbo
* chown root:root huaweiAktBbo
* mv huaweiAktBbo /sbin
Anschließend muss folgende Datei angelegt werden (root-Rechte erfolderlich)
/etc/udev/rules.d/50-huawei-e220.rules
```bash
SUBSYSTEM=="usb", SYSFS{idProduct}=="1003", SYSFS{idVendor}=="12d1", RUN+="/sbin/huaweiAktBbo"
SUBSYSTEM=="usb", SYSFS{idProduct}=="1003", SYSFS{idVendor}=="12d1", RUN+="/bin/sleep 5"
SUBSYSTEM=="usb", SYSFS{idProduct}=="1003", SYSFS{idVendor}=="12d1", RUN+="/sbin/modprobe usbserial vendor=0x12d1 product=0x1003"
```

Jetzt kann der UMTS-Stick eingesteckt werden und *dmesg* müsste in etwa folgendes ausgeben:

Ausgabe von dmesg
```bash
option 2-2.5:1.0: GSM modem (1-port) converter detected
usb 2-2.5: GSM modem (1-port) converter now attached to ttyUSB0
option 2-2.5:1.1: GSM modem (1-port) converter detected
usb 2-2.5: GSM modem (1-port) converter now attached to ttyUSB1
scsi20 : SCSI emulation for USB Mass Storage devices
usb-storage: device found at 33
usb-storage: waiting for device to settle before scanning
usb-storage: device scan complete
scsi 20:0:0:0: CD-ROM            HUAWEI   Mass Storage     2.31 PQ: 0 ANSI: 2
sr1: scsi-1 drive
sr 20:0:0:0: Attached scsi CD-ROM sr1
sr 20:0:0:0: Attached scsi generic sg0 type 5
```
##  Aufbau einer Verbindung 
* pppd bringt alles mit um eine Verbindung aufzubauen
* auch wvdial kann verwendet werden (nicht getestet)
* gnome-ppp und auch umtsmon (siehe unten) können als GUI für den Desktop verwendet werden

###  PIN-Code eingeben
Möchte man seinen PIN-code nicht in Skripten angeben kann man das auf de Kommandozeile mit 
  echo 'AT+CPIN="xxxx"' > /dev/ttyUSB0

Besser ist es aber die PIN wie oben über über minicom einzugeben.

###  Verbindung mit ppp 
/etc/ppp/peers/umts
```bash
hide-password
noauth
connect "/usr/sbin/chat -v -f /etc/chatscripts/umts"
debug
/dev/ttyUSB0
115200
defaultroute
noipdefault
user "t-mobile"
password "internet"
usepeerdns
ipcp-accept-remote
ipcp-accept-local
lock
```

/etc/chatscripts/umts
```bash
'TIMEOUT' '10'
#abortstring
ABORT BUSY
ABORT VOICE
ABORT 'NO DIALTONE'
ABORT 'NO DIAL TONE'
ABORT 'NO ANSWER'
ABORT '+CPIN: SIM PIN'
ABORT DELAYED
# modeminit
'' 'ATZ'
'OK' 'AT&F'
'OK' 'ATE1'
SAY 'Checking pin lock
'
'OK' 'AT+CPIN?'
SAY 'Check Quality 
'
'OK' 'AT+CSQ'
SAY 'Setting APN
'
'OK' 'AT+CGDCONT=1,"IP","internet.t-mobile"'
# ispnumber
'OK' 'ATDT*99#'
# ispconnect
'CONNECT'
```
Nach dem die PIN eingegeben wurde, kann mittels *pon umts* eine Verbindung aufgebaut werden. Um die PIN-Abfrage dauerhaft zu deaktivieren kann eines der AT-Kommandos verwendet werden (siehe nächster Abschnitt).

##  weitere Informationen 
* Einige [hilfreiche AT-Befehle](http://de.gentoo-wiki.com/GPRS/UMTS#Konfiguration_des_Modems) zum Anfragen von Daten des UMTS-Modems  aus dem Gentoo-wiki
* [umtsmon](http://umtsmon.sourceforge.net/) - eine GUI((basiert auf Qt)) für UMTS-Verbindungen