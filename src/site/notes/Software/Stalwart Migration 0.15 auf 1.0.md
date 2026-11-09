---
{"dg-publish":true,"dg-permalink":"software/stalwart-migration","permalink":"/software/stalwart-migration/","title":"Stalwart Mailserver: Migration von 0.15 auf 1.0","metatags":{"description":"Erfahrungsbericht: Stalwart-Mailserver mit Docker und OpenLDAP von 0.15.5 über 0.16 auf 1.0.3 migrieren. Ablauf, Fallstricke und Checkliste."},"dg-note-properties":{"title":"Stalwart Mailserver: Migration von 0.15 auf 1.0"}}
---

#stalwart #mail #migration #docker #ldap

Erfahrungsbericht: Migration eines kleinen Stalwart-Mailservers von 0.15.5 auf 1.0.3. Das Setup: Docker Compose, RocksDB, OpenLDAP als Verzeichnis, Reverse Proxy davor, weniger als 10 Konten, rund 13 GB Daten.

Die Migration erfolgte im November 2026 mit `stalwart:v0.16.27`, `stalwart-migrate:1.0.6`, `stalwart:v1.0.3` und `stalwart-cli` 1.0.14. Objekt- und Feldnamen können sich in neueren Versionen ändern. Alle Namen sind Platzhalter (`example.com`, `<...>`). Die offizielle Upgrade-Dokumentation hat Vorrang vor dieser Seite.

**English version:** [[En/Software/Stalwart Migration 0.15 to 1.0\|Stalwart Migration 0.15 to 1.0]]

## Das Wichtigste vorab

* Es gibt keinen direkten Weg: 0.15 → 0.16 → `stalwart-migrate` → 1.0.
* Der erste Start von 0.16 verändert die Daten unumkehrbar. Einstellungen und Verzeichniseinträge werden dabei gelöscht. Deshalb nur auf einer Kopie arbeiten und vorher einen Dump ziehen.
* `:latest` zeigt inzwischen auf 1.0. Das Image der alten Version vor dem Umstieg fest pinnen, sonst startet ein versehentliches Pull die falsche Version auf den alten Daten.
* Die eigentliche Migration lief problemlos. Die Probleme kamen danach und betrafen Rollen, IP-Sperren, Logging und Zertifikate. Die Checkliste unten deckt sie ab.
* Eine Generalprobe auf einer Kopie lohnt sich sehr, aber nur, wenn sie am Ende auch echte Logins prüft. Bei uns fehlte genau das, und der schwerste Fehler zeigte sich deshalb erst im Betrieb.

## Ergebnis

| Phase | Dauer |
|---|---|
| Datenkopie 13 GB | 2 Minuten |
| 0.16 starten, Konfiguration einspielen | 1 Minute |
| `stalwart-migrate` inklusive vollständiger Prüfung | 5 Minuten |
| Ausfall gesamt bis 1.0 öffentlich läuft | 9 Minuten |
| Logins danach gestört | rund 13 Minuten, ein Admin-Konto rund 25 Minuten |

`verify --full` meldete alle Konten ohne Abweichung. Die Login-Störung wäre mit der Checkliste unten vermeidbar gewesen.

## Vorbereitung

1. Image pinnen und `pull_policy: never` setzen:
```yaml
image: stalwart-local:pre-upgrade-0.15.5   # lokaler Tag des laufenden Images
pull_policy: never
```
2. Generalprobe auf einer Kopie der Daten durchführen. Die Kopie isoliert betreiben (`docker network create --internal`), damit sie keine Mail zustellt.
3. Prüfen, welche Dienste den Mailserver intern nutzen. Webmail kann Ports erreichen, die nach außen gar nicht veröffentlicht sind, zum Beispiel 465 und 4190.
4. Inhalte vergleichen: Anzahl Mails und Summe der Größen je Konto per JMAP. Der Quota-Zähler taugt dafür nicht. In 0.15 war er bei uns um mehr als den Faktor 2 zu niedrig, die echte Summe stimmte nach der Migration auf das Byte.
5. Klären, wer Admin wird. Mit aktivem LDAP können lokale Konten keine Zugangsdaten haben. Ohne LDAP bleibt nur `STALWART_RECOVERY_ADMIN` als Notzugang.
6. Benutzer informieren. OAuth-Clients müssen sich neu anmelden, CalDAV, CardDAV und WebDAV laden ihre Daten neu.

## Ablauf

### 1. Dump aus 0.15

Das Skript `migrate_v016.py` aus der Upgrade-Dokumentation exportiert Einstellungen und Principals über die API. Das geht im laufenden Betrieb:

```sh
python migrate_v016.py dump --url https://mail.example.com --username admin \
  --settings settings.json --principals principals.json

python migrate_v016.py convert --settings settings.json --principals principals.json \
  --config config.json --output export.json \
  --patch-paths /opt/stalwart=/var/lib/stalwart --unmigrated-output unmigrated.txt
```

`unmigrated.txt` ansehen. Bei uns wurde der Großteil der Keys nicht übernommen, darunter die vielen automatisch angelegten IP-Sperren. Das ist normal, aber die Liste lohnt einen Blick.

`export.json` ist NDJSON. Veraltete Domains, DKIM-Schlüssel oder Aliase lassen sich vor dem Import herausfiltern. Danach prüfen, dass keine Referenzen wie `#create-N` ins Leere zeigen.

### 2. Stoppen und kopieren

```sh
docker stop -t 60 mailserver
cp -a <0.15-daten> <work016>
chown -R 2000:2000 <work016>     # neue Images laufen als UID 2000
```

Die Originaldaten bleiben unangetastet und sind der Rollback.

### 3. 0.16 im Recovery-Modus

`config.json` für 0.16:

```json
{ "@type": "RocksDb", "path": "/var/lib/stalwart/data" }
```

```sh
docker run -d --name stalwart-016 --network <internes-netz> \
  -e STALWART_RECOVERY_ADMIN=admin:<passwort> -e STALWART_RECOVERY_MODE=1 \
  -v <c016>:/etc/stalwart -v <work016>:/var/lib/stalwart/data \
  stalwartlabs/stalwart:v0.16.27
```

Die CLI läuft im Netz-Namensraum des Containers:

```sh
docker run --rm --network container:stalwart-016 \
  -e STALWART_URL=http://127.0.0.1:8080 -e STALWART_USER=admin -e STALWART_PASSWORD=<passwort> \
  -v <ordner>:/d:ro ghcr.io/stalwartlabs/cli:latest apply --file /d/export.json
```

### 4. LDAP-Verzeichnis anlegen

Set-Felder sind Objekte, keine Listen, also `{"mail": true}` statt `["mail"]`. Listen enden in `invalidPatch` beziehungsweise `invalid value object`.

```json
{
  "description": "LDAP",
  "url": "ldap://ldap.example.internal:1389",
  "baseDn": "dc=example,dc=com",
  "bindDn": "cn=admin,dc=example,dc=com",
  "bindSecret": "<geheim>",
  "bindAuthentication": false,
  "filterLogin": "(&(objectClass=inetOrgPerson)(|(mail=?)(uid=?)))",
  "filterMailbox": "(&(objectClass=inetOrgPerson)(|(mail=?)(mailLocalAddress=?)))",
  "attrClass": {"objectClass": true},
  "attrEmail": {"mail": true},
  "attrEmailAlias": {"mailLocalAddress": true}
}
```

```sh
stalwart-cli create Directory/Ldap --stdin < directory.json
stalwart-cli update Authentication singleton --field directoryId=<id>
```

* `bindAuthentication: false` bedeutet Hash-Vergleich, das funktionierte mit `{SSHA}`.
* Weder 0.16.27 noch 1.0.3 ergänzen bei LDAP-Logins die Standard-Domain. Damit Logins ohne Domain weiter gehen, muss `filterLogin` auch `uid` abdecken.
* `filterMailbox` aus 0.16 heißt in 1.0 `filterRecipient`, `stalwart-migrate` wandelt das um.

### 5. stalwart-migrate

0.16 stoppen. Zielkonfiguration für 1.0:

```json
{ "@type": "RocksDb", "path": "/var/lib/stalwart-v1/" }
```

```sh
A="--source /etc/stalwart/config.json --target /etc/stalwart-v1/config.json --state /var/lib/stalwart-migrate"
migrate prepare --non-interactive --source /etc/stalwart/config.json --state /var/lib/stalwart-migrate
migrate run $A
migrate run $A --final --yes
migrate verify --full $A
```

`migrate` steht hier für `docker run --rm -it --ulimit nofile=65536:65536 --network none` mit den passenden Volumes und `stalwartlabs/stalwart-migrate:1.0.6`. Das Tool braucht ein TTY, deshalb die Kette in `screen` laufen lassen. Für `--non-interactive` muss `answers.toml` im State-Ordner liegen. Die Datei schreibt `prepare` selbst, sie lässt sich also aus der Generalprobe übernehmen.

### 6. 1.0 erst isoliert einrichten

Dieser Schritt ist die wichtigste Abweichung vom naheliegenden Weg. 1.0 zuerst ohne veröffentlichte Ports starten, alles einrichten und erst dann öffentlich machen. So läuft der Server nie halb konfiguriert im Netz.

Den ersten Start einmal mit `STALWART_UPDATE_ROLE_PERMISSIONS=true`, danach die Variable wieder entfernen. Die Einstellungen im Einzelnen stehen in der Checkliste.

### 7. Umschalten

```yaml
mailserver:
  image: stalwartlabs/stalwart:v1.0.3
  environment:
    - STALWART_PUBLIC_URL=https://mail.example.com
  volumes:
    - ./v1/config:/etc/stalwart:ro
    - ./v1/data:/var/lib/stalwart-v1
    - ./v1/logs:/var/log/stalwart
    - ./certs:/data/certs:ro
```

## Checkliste vor dem öffentlichen Start

Diese Punkte haben bei uns alle gefehlt oder Probleme gemacht.

### Default-Rollen setzen

**Das war der schwerste Fehler.** Nach der Migration waren alle `Default Roles` leer. Konten mit `{"@type":"User"}` hatten damit keinerlei Rechte.

Symptom: Das Log zeigt `Authentication successful` und direkt danach `Unauthorized access` mit `reason = "authenticate"`. Clients melden nur einen fehlgeschlagenen Login. Das sieht nach einem Passwort- oder LDAP-Problem aus, ist aber keines.

```sh
stalwart-cli query Role --fields id,description
stalwart-cli update Authentication singleton \
  --field 'defaultUserRoleIds={"<user>":true}' \
  --field 'defaultGroupRoleIds={"<group>":true}' \
  --field 'defaultTenantRoleIds={"<tenant>":true}' \
  --field 'defaultAdminRoleIds={"<admin>":true,"<user>":true}'
```

Die Rollen-IDs unterscheiden sich je nach Installation. Warum die Felder leer waren, haben wir nicht geklärt. Unbedingt vor dem öffentlichen Start prüfen, denn ohne Rollen fehlt auch das Recht `emailReceive`. Port 25 nahm Mails zwar an, ob sie in dieser Zeit zugestellt wurden, ließ sich wegen des fehlenden Logs nicht mehr klären.

### Admin braucht zwei Rollen

Die Rolle `System Administrator` enthält keine IMAP-Rechte. Ein Admin-Konto kommt damit in die Verwaltung, aber nicht ins Postfach. Das Log zeigt `Unauthorized access` mit `imapAuthenticate`. Deshalb trägt `defaultAdminRoleIds` oben beide Rollen. Danach bekommt das Konto die Rolle:

```sh
stalwart-cli update Account <id> --field 'roles={"@type":"Admin"}'
```

### Reverse Proxy von der automatischen Sperre ausnehmen

Der Webzugriff antwortete plötzlich mit HTTP 502. Direkt am Container kam korrekt 401. Ursache: Bots rufen Pfade wie `*.php` oder `*/wp-*` auf, die Scan-Erkennung sieht als Quelle nur den Proxy und sperrt ihn, und zwar unbefristet. Die Regel griff bei uns wenige Minuten nach dem Start.

```sh
stalwart-cli query BlockedIp --fields id,address,reason
stalwart-cli create AllowedIp --json '{"address":"<internes-subnetz>","reason":"Reverse Proxy"}'
stalwart-cli delete BlockedIp --ids <id>
```

Danach neu starten, die Sperre blieb sonst im Cache aktiv. Ein Subnetz ist robuster als eine einzelne IP, weil sich Container-IPs beim Neuanlegen ändern können.

### Listener prüfen

Bei uns fehlten nach der Migration `imap` (143) und `submission` (587), die übrigen waren da:

```sh
stalwart-cli query NetworkListener --fields id,name,bind,protocol
stalwart-cli create NetworkListener --json '{"name":"imap","bind":{"[::]:143":true},"protocol":"imap"}'
stalwart-cli create NetworkListener --json '{"name":"submission","bind":{"[::]:587":true},"protocol":"smtp"}'
```

### Zertifikat

```sh
stalwart-cli create Certificate --json '{"certificate":{"@type":"File","filePath":"/data/certs/mail.example.com/cert.pem"},"privateKey":{"@type":"File","filePath":"/data/certs/mail.example.com/key.pem"}}'
stalwart-cli update SystemSettings singleton --field defaultCertificateId=<id>
```

* Das neue Image läuft als UID 2000. Ein `key.pem` mit `root:root 600` ist nicht lesbar, und TLS fällt aus.
* Wer `traefik-certs-dumper` nutzt: `OVERRIDE_UID=2000` und `OVERRIDE_GID=2000` wirken nicht auf vorhandene, unveränderte Dateien. Die Besitzrechte also einmalig selbst setzen und nach der nächsten Erneuerung prüfen. Letzteres steht bei uns noch aus.
* Zertifikatsdateien werden nur beim Start oder per `stalwart-cli create Action/ReloadTlsCertificates` neu gelesen. Wir starten den Container einmal pro Woche per Cron neu, dafür muss kein Passwort auf dem Server liegen.

### Logging prüfen

Nach der Migration blieb `docker logs` komplett leer, und der migrierte Tracer schrieb nach `/var/log/stalwart`, das im Container nicht existiert. Ein leeres Log bedeutet also nicht, dass es keine Fehler gibt. Abhilfe: den Ordner als Volume einhängen, Besitzer UID 2000.

Für die Fehlersuche hilft ein temporärer Tracer auf der Konsole. Er wird nach einem Neustart aktiv:

```sh
stalwart-cli create Tracer/Stdout --json '{"enable":true,"level":"debug","eventsPolicy":"exclude","events":{}}'
```

### CLI aktuell halten

`stalwart-cli` 1.0.13 konnte das Schema von 1.0.3 nicht lesen, erst 1.0.14 funktionierte. Vor dem Fenster das CLI-Image neu ziehen.

## Tests nach dem Start

Von außen testen, nicht nur am Container:

* Ports 25, 143, 465, 587, 993, je nach Setup auch 4190
* Login mit Kurzname und mit voller Adresse, über IMAP und SMTP
* Webmail, auch nach jedem weiteren Neustart oder jeder Rollenänderung
* Admin-Oberfläche, ebenso die Postfächer des Admin-Kontos
* Mail in beide Richtungen, DKIM-Prüfung beim Empfänger
* Unbekannte Adresse und entfernte Domains müssen abgelehnt werden

Ein Test mit absichtlich falschem Passwort zeigt schnell, ob der Web-Pfad stimmt: Korrekt ist 401, nicht 502.

## Rollback

Solange 1.0 keine neue Mail angenommen hat: Compose zurück auf das gepinnte Image und die alten Volumes. Die 0.15-Daten bleiben dafür unverändert liegen. Später angekommene Mail müsste man von Hand übernehmen. Daher bei Problemen früh entscheiden.

Die Arbeitskopie von 0.16 kann weg, sobald 1.0 bestätigt läuft. Die 0.15-Daten haben wir noch etwa eine Woche aufgehoben.

## Sonstige Beobachtungen

* Die Spam-Trainingsdaten waren nach 0.16 weg, das war zu erwarten. 1.0 lud beim Start ein Spam-Modell. Ob es dem alten entspricht, haben wir nicht geprüft.
* Die `answers.toml` enthält Geheimnisse im Klartext, wenn welche abgefragt wurden. Auch Dumps und `export.json` enthalten Hashes und DKIM-Schlüssel. Solche Dateien mit Rechten `600` ablegen und nach dem Umstieg löschen.
* `STALWART_RECOVERY_ADMIN` funktioniert auch im Normalbetrieb bei ausgefallenem LDAP. Nach Gebrauch wieder entfernen.
* In Shell-Skripten verschluckt `docker run -i` den Rest des Skripts, wenn es über stdin läuft. `</dev/null` anhängen.
* Account-Objekte niemals löschen, sie tragen die Zuordnung zu den Postfächern.
