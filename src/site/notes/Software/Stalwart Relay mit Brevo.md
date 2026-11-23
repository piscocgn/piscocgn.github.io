---
{"dg-publish":true,"dg-permalink":"software/stalwart-brevo-relay","permalink":"/software/stalwart-brevo-relay/","title":"Stalwart Mailserver: Microsoft und T-Online über ein Brevo-Relay zustellen","metatags":{"description":"Erfahrungsbericht: Stalwart 1.0 schickt Mail an Outlook.com, Hotmail, Live und T-Online über das Gratis-SMTP-Relay von Brevo. Anbieterwahl, Einrichtung, Fallstricke."},"dg-note-properties":{"title":"Stalwart Mailserver: Microsoft und T-Online über ein Brevo-Relay zustellen"}}
---

#stalwart #mail #smtp #relay #brevo #microsoft

Erfahrungsbericht: Ein kleiner selbst betriebener Stalwart-Mailserver auf einem VPS wird von Microsoft abgelehnt. Statt auf eine Freigabe zu warten, gehen Mails an Microsoft- und Telekom-Adressen jetzt über das Gratis-SMTP-Relay von Brevo. Alle anderen Ziele stellt Stalwart weiter direkt zu.

Stand November 2026 mit `stalwart:v1.0.3` und dem Gratis-Tarif von Brevo. Alle Namen sind Platzhalter (`example.com`, `<...>`). Die offizielle Dokumentation von [Stalwart](https://stalw.art/docs/mta/outbound/routing/) und Brevo hat Vorrang vor dieser Seite.

Der mit „Werbung“ markierte Link zu Brevo ist ein Affiliate-Link. Die Bewertung auf dieser Seite ist davon unabhängig und nennt auch die Schwächen.

## Das Wichtigste vorab

* Microsoft sperrt ganze Adressbereiche von VPS-Hostern (Fehler S3140). Eine saubere Konfiguration mit Reverse DNS, SPF, DKIM und DMARC hilft dann nicht, eine Freigabe dauert oder kommt nie.
* Stalwart kann pro Empfängerdomain ein anderes Ziel wählen. Ein Relay nur für die betroffenen Domains reicht, der Rest bleibt unverändert.
* Brevo ist im Gratis-Tarif mit 300 Mails pro Tag für diesen Zweck mehr als ausreichend.
* Port 587 mit `implicitTls` an funktioniert nicht. Entweder 587 ohne `implicitTls` oder 465 mit.
* Brevo deaktiviert SMTP-Schlüssel nach 90 Tagen ohne Versand. Wer selten an Microsoft-Adressen schreibt, braucht eine regelmäßige Testmail.
* Die erste Testmail landete trotz bestandenem SPF, DKIM und DMARC im Junk-Ordner von Outlook.com. Das Relay löst also das Ablehnen, nicht automatisch die Spam-Einstufung.

## Ausgangslage

Mails an eine `live.de`-Adresse kamen mit diesem Fehler zurück:

```
550 5.7.1 Unfortunately, messages from [<server-ip>] weren't sent. Please contact your Internet
service provider since part of their network is on our block list (S3140).
```

„Part of their network“ heißt: Gesperrt ist der Adressbereich des Hosters, nicht die eigene IP. Die IP stand auf keiner gängigen Blockliste (Spamhaus, Barracuda, SpamCop, SORBS, PSBL). Die Microsoft-Server für Consumer-Adressen (`eur.olc.protection.outlook.com`) haben keine IPv6-Adresse, ein Ausweichen auf IPv6 hilft also auch nicht.

Ein Antrag auf Aufhebung über [olcsupport.office.com](https://olcsupport.office.com/) ist gestellt. Bei Bereichssperren ist er aber oft erfolglos.

Andere große Anbieter (Gmail, GMX, web.de, Yahoo, iCloud) nahmen Mail vom selben Server problemlos an.

## Anbieterwahl

Verglichen wurden europäische Anbieter mit Gratis-Tarif und SMTP-Relay:

| Anbieter | Sitz | Gratis | Bemerkung |
|---|---|---|---|
| <a href="https://get.brevo.com/gqsuz2jqufr9" class="affiliate" rel="sponsored noopener" title="Affiliate-Link: Bei einer Anmeldung erhalte ich eine Provision. Für dich ändert sich nichts.">Brevo</a> | Frankreich | 300/Tag | großer Anbieter, Tracking standardmäßig an |
| [Mailjet](https://www.mailjet.com/pricing/?currency=eur) | Frankreich | 200/Tag, 6.000/Monat | Mailjet-Logo in der Mail im Gratis-Tarif |
| Scaleway TEM | Frankreich | 300/Monat | danach 0,25 € je 1.000 Mails, laut Anbieter nur für Transaktionsmail |
| [Lettermint](https://lettermint.co/pricing) | Niederlande | 300/Monat | klein und jung |
| [Sweego](https://www.sweego.io/pricing) | Frankreich | 100/Tag | klein und jung |

Belastbare Zahlen zur Zustellbarkeit bei Microsoft gibt es für keinen dieser Anbieter. Alle senden im Gratis-Tarif über IP-Adressen, die sich viele Kunden teilen, und auch solche Pools sperrt Microsoft gelegentlich. Die Wahl fiel auf <a href="https://get.brevo.com/gqsuz2jqufr9" class="affiliate" rel="sponsored noopener" title="Affiliate-Link: Bei einer Anmeldung erhalte ich eine Provision. Für dich ändert sich nichts.">Brevo</a> wegen des größten Kontingents und der Größe des Anbieters. Scaleway TEM bleibt die Reserve.

## Einrichtung

### Brevo

1. Domain authentifizieren: *Senders, Domains & Dedicated IPs → Domains → Domain hinzufügen*. Brevo nennt die DNS-Einträge:
   * TXT `brevo-code:...` zur Bestätigung
   * DKIM als CNAME: `brevo1._domainkey` und `brevo2._domainkey`
   * SPF um `include:spf.brevo.com` ergänzen
2. SMTP-Schlüssel erzeugen: *SMTP & API → SMTP*. Server `smtp-relay.brevo.com`, Login in der Form `<id>@smtp-brevo.com`, Passwort ist der Schlüssel.
3. Unter *Transactional → Settings* prüfen, ob Öffnungs- und Klick-Tracking an sind. Für persönliche Mail besser abschalten.

Die eigene DKIM-Signatur von Stalwart und der bestehende DMARC-Eintrag bleiben unverändert.

### Stalwart: Route

WebUI *Settings → MTA → Outbound → Routes*, neue Route vom Typ **Relay**:

| Feld | Wert |
|---|---|
| name | `brevo` |
| host | `smtp-relay.brevo.com` |
| port | `587` (dann `implicitTls` aus) oder `465` (dann `implicitTls` an) |
| protocol | `smtp` |
| authUsername | `<id>@smtp-brevo.com` |
| authSecret | Typ `Value` oder `EnvironmentVariable` |

Stalwart meldet sich nur an, wenn `authUsername` und `authSecret` beide gesetzt sind.

### Stalwart: Strategie

WebUI *Settings → MTA → Outbound → Strategy*, Feld `route`. Die Regel für lokale Domains bleibt an erster Stelle:

| Eintrag | if | then |
|---|---|---|
| match 0 | `is_local_domain(rcpt_domain)` | `'local'` |
| match 1 | siehe unten | `'brevo'` |
| else | | `'mx'` |

Bedingung für match 1 in einer Zeile:

```
rcpt_domain == 'outlook.com' || rcpt_domain == 'outlook.de' || rcpt_domain == 'hotmail.com' || rcpt_domain == 'hotmail.de' || rcpt_domain == 'live.com' || rcpt_domain == 'live.de' || rcpt_domain == 'msn.com' || rcpt_domain == 'windowslive.com' || rcpt_domain == 't-online.de' || rcpt_domain == 'magenta.de'
```

Weitere Länderdomains von Microsoft (`hotmail.fr`, `live.at` usw.) bei Bedarf ergänzen. Danach die Konfiguration neu laden.

## Fallstricke

### TLS-Fehler „InvalidContentType“

Mit Port 587 und `implicitTls` an steht im Log:

```
Implicit TLS error (delivery.implicit-tls-error) ... hostname = "smtp-relay.brevo.com",
reason = ... "TLS Error", reason = "received corrupt message of type InvalidContentType"
```

Port 587 erwartet erst Klartext und dann STARTTLS, Stalwart beginnt aber sofort mit TLS. Lösung: `implicitTls` aus, oder Port 465.

### Änderung greift nicht sofort

Nach dem Umstellen der Route lief der nächste Zustellversuch noch mit der alten Einstellung. Erst nach einem Reload der Konfiguration und einem erneuten Anstoßen der Nachricht in der Queue ging die Mail raus.

### Routenname muss stimmen

Gibt die Strategie einen Namen zurück, zu dem es keine Route gibt, meldet Stalwart `smtp.id-not-found` und stellt direkt zu. Die Mail wird dann wieder von Microsoft abgelehnt, ohne dass es auffällt.

### Kaputte DKIM-Signatur von Stalwart

Bei Microsoft kam an:

```
dkim=pass header.d=example.com (Brevo-Schlüssel brevo1)
dkim=fail (signature syntax error) header.d=none
dmarc=pass
```

Brevo signiert neu und gültig. Die ursprüngliche Ed25519-Signatur von Stalwart hat Brevo dabei aber verunstaltet: Aus `a=ed25519` wurde `aí25519`, offenbar wurde `=ed` als Quoted-Printable gelesen. Für DMARC ist das egal, weil die Brevo-Signatur zur Absenderdomain passt.

### Junk trotz bestandener Prüfungen

Die erste Testmail an ein Outlook.com-Postfach bekam `SCL 5` und landete im Junk-Ordner. Wahrscheinliche Gründe: neuer Absender ohne Vorgeschichte, geteilte Brevo-IPs, und Brevo fügt `List-Unsubscribe`, `X-CSA-Complaints` und eine Feedback-ID ein, was nach Newsletter aussieht. Eine reine Testmail mit zwei Zeilen Text hilft auch nicht. Empfänger, die die Mail als „Kein Junk“ markieren, verbessern das für ihr Postfach.

### Öffnungs-Tracking

Zeigt Brevo bei einer Mail das Ereignis „first opening“, wurde das eingebaute Zählbild geladen. Das spricht für eine gelesene Mail, ist aber kein Beweis: Apple Mail lädt Bilder vorab, Virenscanner auch, und wer Bilder blockiert, wird nie als „geöffnet“ gezählt. Außerdem zeigt das Ereignis, dass Brevo die eigenen Mails verändert. Wer das nicht will, schaltet das Tracking ab.

### Schlüssel läuft bei Inaktivität ab

Brevo deaktiviert einen SMTP-Schlüssel nach 90 Tagen ohne erfolgreichen Versand, auch wenn er als „läuft nie ab“ angelegt wurde ([Brevo-Hilfe](https://help.brevo.com/hc/en-us/articles/7959631848850)). Danach bleiben Mails an Microsoft-Adressen in der Queue hängen und kommen nach einigen Tagen als unzustellbar zurück.

Abhilfe: Ein Cronjob schickt einmal im Monat eine Testmail an ein eigenes Outlook.com-Postfach und prüft fünf Minuten später im Stalwart-Log, ob sie an Brevo übergeben wurde. Wenn nicht, geht eine Warnung an root. Die Warnung läuft nicht über Brevo und kommt deshalb auch bei abgelaufenem Schlüssel an.

```sh
#!/bin/sh
# Monthly test mail to an Outlook.com mailbox via the Brevo relay route in Stalwart.
set -eu

RCPT="<test>@outlook.com"
LOG="/path/to/stalwart/logs/stalwart.log"
WAIT=300

START=$(date -u +%Y-%m-%dT%H:%M:%SZ)

printf 'To: %s\nSubject: Brevo relay test %s\n\nMonthly test via the Brevo relay.\n' \
    "$RCPT" "$(date +%F)" | /usr/sbin/sendmail "$RCPT"

sleep "$WAIT"

if awk -v start="$START" -v rcpt="$RCPT" '
    $1 >= start && /delivery\.delivered/ && /smtp-relay\.brevo\.com/ && index($0, "to = \"" rcpt "\"") { found = 1 }
    END { exit !found }' "$LOG"; then
    exit 0
fi

{
    printf 'Subject: Brevo relay test FAILED\n\n'
    grep -F "$RCPT" "$LOG" | tail -5
} | /usr/sbin/sendmail root
```

```
# /etc/cron.d/brevo-relay-test
23 9 1 * * root /path/to/brevo-relay-test.sh
```

Ein eigenes Outlook.com-Postfach bekommt man, indem man im Microsoft-Konto unter *Anmeldung bei Microsoft verwalten* eine neue Adresse als Alias anlegt. Im Postfach sollte man den Absender als sicher markieren und die Testmails in einen eigenen Ordner sortieren, aber nicht automatisch löschen. Sonst fällt nicht auf, wenn sie ausbleiben.

## T-Online

T-Online hat hier noch nichts abgelehnt, steht aber vorsorglich mit auf der Liste. Die Telekom verlangt, dass der Hostname des Mailservers auf eine Website mit Impressum und Kontaktmöglichkeit führt. Für Cloud-Systeme empfiehlt sie ausdrücklich ein Relay, und auf Freigabelisten setzt sie niemanden ([postmaster.t-online.de](https://postmaster.t-online.de/), Abschnitte 4.1 und 4.11). Wer direkt zustellen will, braucht ein Impressum unter dem Hostnamen des Mailservers und schreibt dann an die Adresse aus der Fehlermeldung.

## Ergebnis

* Mails an `live.de` und `outlook.com` kommen seit der Umstellung an: die Testmail im eigenen Postfach, die `live.de`-Mail laut Brevo-Ereignis „first opening“. Stalwart braucht für die Übergabe an Brevo rund 20 bis 50 Millisekunden.
* SPF, DKIM und DMARC bestehen bei Microsoft.
* Die Spam-Einstufung bei Microsoft bleibt ein Thema und hängt vom Empfänger ab.
* Wird die Sperre bei Microsoft aufgehoben, reicht es, match 1 in der Strategie wieder zu entfernen.

**Siehe auch:** [[Software/Stalwart Migration 0.15 auf 1.0\|Stalwart Migration 0.15 auf 1.0]]
