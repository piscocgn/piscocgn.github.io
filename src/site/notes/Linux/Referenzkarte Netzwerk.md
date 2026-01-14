---
{"dg-publish":true,"dg-permalink":"linux/referenzkarte_netzwerk","permalink":"/linux/referenzkarte_netzwerk/"}
---

#linux #lpi #netzwerk #unvollständig 

##  Abstract 
Eine Schnellreferenz-Karte für tägliche Netzwerkaufgaben.  Syntax 
  cmd notwendiger_Parameter [optionaler Parameter]
##  Grundlagen 
###  CIDR 

```
00000000.00000000.00000000.00000000 = 32
|-------|--------|--------|--------|
    8       8        8          8      
<- Leseweise 128 64 32 16 8 4 2 1 . 128 64 ...
```
* 255.255.255.0 /24 253 IP Adresse (255 - Netzwerkadresse .0 - Broadcast .255 = 253
* zum Umrechnen kann [ipcalc](http://jodies.de/ipcalc) hinzugezogen werden

###  Klasse A 
* 255.0.0.0 /8, also 8 Netzwerk-Bits und 24 Host-Bits (32 - 8 = 24)
* 128 Netzwerke, 2 sind reseviert (0.0.0.0 Default-Route, 127.0.0.1 Loop-Device mit 127.0.0.1 Localhost)
* 2hoch 24 - 2 (netzwerkadresse und Broadcast) Host möglich.
###  Klasse B 
* begint binaer mit 10  ( == 128 siehe oben))
*  Adressbereich von 128.0.0.0 bis 191.255.255.255
###  Klasse C 
* beginnt mit 110 ( 128 + 64 = 192, siehe oben)
* Adressbereich von 192.0.0.0 bis 223.255.255.255
###  Klasse D und E 
* sehr selten, experimentell, reserviert als Multicast für Rechnergruppen und haben keinen Netzwerkanteil
* beginnt mit 1110 ( 128 + 64 + 32 = 224, siehe oben)
* Adressbereich 224.0.0.0 bis 238.255.255.255.0

###  Private Adressbereiche 
Außerhalb des Internets
* Klasse A  10.0.0.0, Netzwerkmaske /8 bzw. 255.0.0.0
    * nur ein netz
* Klasse B 172.16.0.0 bis 172.31.255.255, Netzwerkmaske /16 bzw. 255.255.0.0
    * insgesamt 16 Netze 
* Klasse C 192.168.0.0 bis 192.168.255.255, Netzwerkmaske /8 bzw. 255.255.255.0
    * insgesamt 256 Netze ( 2 * 128 = 256)

##  Interfaces 

  ifconfig [interface] [adresse] [[optionen\|optionen]]
  ip link [show|list|ls] [device]

###  Beispiele 
* Rosetta Stone ifconfig / ip

| Aktion | ip | ifconfig / traditionell |
|---|---|---|
|Interface ein- oder ausschalten | ifconfig [device] up''|''down | ip link set [device] up''|''down |
|Setzen einer IP-Adresse 192.168.3.1 Netzmaske 255.255.255.0  und Boadcast (wird in beiden Fällen automatisch zugewiesen)  für das Interface eth0 | ip addr add 192.168.3.1/24 dev eth0 brd + | ifconfig eth0 192.168.3.1 netmask 255.255.255.0 |
|Erzeugen eines virtuellen Interfaces | ip addr add 192.168.3.1/24 dev eth0:0 brd + label eth0:1 ((die Anweisung *label* ist nur zur Kompatilität mit ifconfig))| ifconfig eth0:0 192.168.3.1 netmask 255.255.255.0 |
| Löschen eines virtuellen Interfaces | ip addr del 192.168.3.1/24 dev eth0:0 | ifconfig eth0:0 del 192.168.3.1 |
| Routing |||
|---|---|---|
| Routing-Tabelle anzeigen | ip route | route -n ''|'' netstat -r |
| ein Standard-Gateway hinzufügen | ip route add default via [host IP''|''name] | route add default gw [host IP''|''name] |
|eine Standard-Gateway löschen | ip route del default | route del default |
| eine Route in ein Netz hinzufügen | ip route add [network]/[CIDR] via [gateway] | route add -net [network IP] [netmask] [device] |
| eine Route in ein Netz löschen | ip route del [network]/[CIDR] via [gateway] | route del -net [network IP] [netmask] [device] |

###  LowLevel 
Cache der ARP Tabelle ausgeben, die Option -n verhindert die Namensauflösung.
  cat /proc/net/arp
  arp -n
Einträge bearbeiten:
  arp -d IP
  arp -s IP MAC

###  Geschwindigkeit prüfen und einstellen 
  mii-tool eth0
  ethtool eth0

  mii-tool -F 100baseTx-FD
  mii-tool -F 10baseT-HD

  ethtool -s eth0 speed 100 duplex full
  ethtool -s eth0 speed 10 duplex half

##  Wireles LAN 
  wlanconfig
  watch -n1 'cat /proc/net/wireless'

##  DNS 
* Abfrage Hostname zu IP

  host hostname [IP des Nameservers anstatt resolv.conf]
  nslookup hostname [IP des Nameservers anstatt resolv.conf]
  dig [@IP des Nameservers anstatt resolv.conf] [-t type] hostname

* Abfrage IP zu Hostname

  host IP [IP des Nameservers anstatt resolv.conf]
  nslookup IP [IP des Nameservers anstatt resolv.conf]
  nslookup [@IP des Nameservers anstatt resolv.conf] -x IP

, whois ~~nsquery~~

##  Bridging 
* [Dokumentation](http://www.linuxfoundation.org/en/Net%3ABridge), siehe auch ''/usr/share/doc/bridge-utils''
  brctl addbr br0
  brctl addif br0 eth0

Anschließend die IP-Adresse von eth0 auf br0 übertragen, eth0 auf 0.0.0.0 promisc setzen.

##  IP-Tables 
Eine einfache Regel für NAT
  iptables -t nat -A POSTROUTING -s 192.168.3.0/24 -o eth0 -j MASQUERADE
<note>
Das Kommando
  echo 1 > /proc/sys/net/ipv4/ip_forward
aktiviert das Weiterleiten von IP-Verkehr. Feste Einstellung ist über /etc/sysctl.conf oder bei Debian */etc/network/options* *ip_forward=yes* möglich
</note>

##  Monitoring 
Alle lokalen TCP- und UDP-Dienste des Hosts anzeigen, *-c* für aktualisierende Anzeige (continuous, 1sec), *n* zeigt numerische Werte statt Namen
  netstat -ltu[c][n]
oder eine schönere Ausgabe
  watch -n0,5 'netstat -ltu[n]'
Aktuele Verbindungen und die zugehörigen Programme anzeigen((root-Rechte erforderlich))
  watch -n0,5 'netstat -ptu[n]'
Geöffnete Dateien von Netzwerkverbindungen anzeigen (IPv4 und IPv6) ((zeigt nur eigene Verbindungen an, sonst mit root-Rechten aufrufen))
  lsof -i

Netzwerkverkehr ansehen, optional Einfärben mit ccze
  tcpdump -vvv [| ccze]
oder mit dem Programm *multitail*. Die Option -A zeigt de Inhalt von Paketen an.

##  Testen 
###  Simple Chat 
* ''netcat -l -p 1234''
* auf der anderen Seite ''netcat [ip|hostname] 1234''
###  Dateiübertragung 
* ''cat [file] | netcat -l -p 1234''
* auf der anderen Seite ''netcat [ip|hostname] 1234 > [file]''

##  Sicherheit 
Überwachung der Kombination IP- und MAC-Adresse.
  aprwatch
###  SSH 
* [[Linux/ssh\|ssh]]
###  OpenVPN 
TODO

