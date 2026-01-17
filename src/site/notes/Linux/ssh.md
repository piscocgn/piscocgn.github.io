---
{"dg-publish":true,"permalink":"/linux/ssh/"}
---

##  SSH-Tunnel 
Beispiel für eine MySQL-Verbindung: einen SSH-Tunnel auf den lokalen Port 3307 vom entferneten Rechner host.example.com Port 3306 (MySql Server) erstellen:

  `ssh -fN -L 3307:localhost:3306 host.example.com`

Anschließend die Verbindung aufbauen mit:
  `mysql -u user -p -P3307 -h 127.0.0.1`

*localhost* kann auch ein Host im Netz hinter *host.example.com* sein, also kann entsprechend die IP-Adresse oder der Hostname aus dem entfernten Netz angegeben werden. Natürlich kann auch ein abweichender Benutzername mit `user@host.example.com` angegeben werden.

###  Remote-Forward 
Beispiel für eine VNC-Verbindung: den lokalen Port 5900 auf den entfernten Rechner host.example.com Port 12345 legen.

  `ssh -fN -R 12345:localhost:5900 host.example.com`

Anschließend kann die Verbindung von host.example.com aus aufgebaut werden
  `vncviewer localhost:12345`
Natürlich kann sich die Gegenstelle auch den Port 12345 von host.example.com lokal tunneln; siehe oben.

##  rsync über ssh mit abweichendem Port 
Wenn der SSH-Dienst auf einem anderen Port als 22 läuft muss es bei rsync entsprechend mit angegeben werden.

  `rsync -auvH -e 'ssh -p 22123' [Quelle] [Ziel]`

#netzwerk #mysql #linux
  