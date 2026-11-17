---
{"dg-publish":true,"dg-permalink":"ki/paperless-ai-raspberry-pi","permalink":"/ki/paperless-ai-raspberry-pi/","title":"paperless-ai mit Ollama auf dem Raspberry Pi 5","metatags":{"description":"Erfahrungsbericht: paperless-ngx mit paperless-ai und einem lokalen Sprachmodell über Ollama auf einem Raspberry Pi 5 mit 8 GB. Modellwahl, Tempo, Einstellungen und Fallstricke."},"dg-note-properties":{"title":"paperless-ai mit Ollama auf dem Raspberry Pi 5"}}
---

#ki #llm #ollama #paperless #raspberrypi #docker

Erfahrungsbericht: paperless-ngx verschlagwortet Dokumente mit Hilfe von paperless-ai, und das Sprachmodell läuft lokal über Ollama auf demselben Raspberry Pi 5 mit 8 GB. Kein Dokument verlässt das eigene Netz.

Stand November 2026 mit paperless-ngx 3.5.0, paperless-ai 4.1.0 und Ollama auf Raspberry Pi OS (Debian 12, Kernel 6.12), alles in Docker Compose. Hostnamen und Pfade sind Platzhalter (`<pi>`, `<...>`). Einstellungsnamen können sich in neueren Versionen ändern.

**English version:** [[En/AI/paperless-ai with Ollama on a Raspberry Pi 5\|paperless-ai with Ollama on a Raspberry Pi 5]]

## Das Wichtigste vorab

* Es funktioniert, aber langsam: Ein Dokument braucht mit einem 4B-Modell rund eine Minute. Für die laufende Post reicht das, für einen Altbestand von Tausenden Dokumenten nicht.
* Das „Nachdenken“ des Modells muss aus sein. Sonst läuft fast jede Anfrage in den Timeout.
* Raspberry Pi OS schaltet die Speicherkontrolle für Container ab. Ein `mem_limit` in Docker Compose wirkt deshalb erst nach einer Änderung an den Startparametern.
* `qwen3.5:4b` ist auf dem Pi der beste Kompromiss. Kleinere Modelle sind schneller, erfinden aber massenhaft Tags.
* paperless-ngx verändert die Originaldateien nie. Kleine Dateien entstehen beim Scannen, nicht in paperless.

## Aufbau

| Dienst | Aufgabe |
|---|---|
| paperless-ngx (Webserver, PostgreSQL, Redis) | Dokumente einlesen, Texterkennung, Ablage |
| paperless-ai | Holt neue Dokumente über die API, fragt das Sprachmodell, schreibt Titel, Tags, Korrespondent, Typ und Datum zurück |
| Ollama | Führt das Sprachmodell aus, auf dem Pi rein über die CPU |
| Open WebUI (optional) | Chat-Oberfläche für dieselben Modelle |

paperless-ai verarbeitet nur Dokumente mit einem Auslöser-Tag (hier `neu`) und markiert sie danach mit `ai-processed`. Es prüft alle zehn Minuten auf neue Dokumente.

## Modellwahl

Getestet mit 15 echten Dokumenten. Verglichen wurde exakt mit den von Hand gepflegten Werten, mit einem eigenen Prompt und höchstens vier Tags.

| | qwen3.5:4b | qwen2.5:3b |
|---|---|---|
| Tags getroffen | 33 von 74 | 12 von 77 |
| Tags frei erfunden | 1 | 40 |
| Korrespondent | 14 von 14 | 9 von 15 |
| Dokumenttyp | 8 von 14 | 2 von 14 |
| Datum | 10 von 14 | 10 von 15 |
| Zeit je Dokument | ca. 90 s | ca. 50 s |

Das kleinere Modell spart etwas Zeit, legt aber so viele neue Tags an, dass die Tag-Liste schnell unbrauchbar wird. Größere Modelle als 4B passen kaum neben die übrigen Dienste in 8 GB und wären noch langsamer.

## Tempo auf dem Pi 5

* Ausgabe: etwa **2,2 Tokens pro Sekunde** mit `qwen3.5:4b` auf der CPU.
* Eine typische Anfrage von paperless-ai hat 2.000 bis 2.400 Tokens.
* Mit abgeschaltetem Nachdenken dauerte ein Dokument im Betrieb **55 Sekunden**.
* Ollama arbeitet Anfragen nacheinander ab. Chattet jemand parallel in Open WebUI, wartet paperless-ai.

## Einstellungen in paperless-ai

paperless-ai 4.x speichert seine Einstellungen in `/app/data/config.json` im Container. Die wichtigen Werte für den Pi:

```json
"ollama": {
  "url": "http://<pi>:11434",
  "model": "qwen3.5:4b",
  "contextSize": 8192,
  "think": "off"
},
"timeoutSeconds": 900
```

* **`think: "off"`:** Mit `auto` denkt `qwen3.5` vor der Antwort ausführlich nach. Bei 2,2 Tokens pro Sekunde lief jede Anfrage in das Standard-Zeitlimit von 300 Sekunden. Erlaubt sind `"auto"` und `"off"`. Ein JSON-`false` verwirft paperless-ai mit einer Warnung im Log.
* **`timeoutSeconds: 900`:** Luft für lange Dokumente.
* **`contextSize: 8192`:** reicht für die Anfragen und spart Speicher. 16k brachten keinen Vorteil.

Weitere Einstellungen, die sich lohnen:

* **Auslöser-Tag entfernen:** Der Tag `neu` verschwindet nach der Verarbeitung. So sieht man in paperless-ngx sofort, was noch offen ist.
* **Nur vorhandene Tags, Korrespondenten und Typen erlauben:** Sonst legt das Modell bei jedem Dokument Neues an.
* **Passende Dokumenttypen vorgeben:** Fehlt ein Typ „Kontoauszug“, ordnet das Modell einen Kontoauszug zum Beispiel als „Rechnung“ ein.

## Fallstrick: config.json wird still ignoriert

Der paperless-ai-Container läuft mit `cap_drop: ALL`. Auch root darf darin keine fremden Dateien lesen oder Besitzer ändern. Gehört `config.json` nach einer Bearbeitung einem anderen Benutzer, meldet paperless-ai nur `EACCES` im Log und lädt **still `config.json.bak`**. Die neuen Werte gelten dann nicht.

Die Datei muss `root:root` mit Rechten `600` gehören. Am sichersten bearbeitet man sie über das Datenvolume mit einem Hilfscontainer:

```bash
docker run --rm -u 0 --entrypoint sed \
  -v <paperless-ai-volume>:/app/data clusterzx/paperless-ai \
  -i 's/"think": "auto"/"think": "off"/' /app/data/config.json
docker restart paperless-ai
docker logs --since 1m paperless-ai | grep -E "config|WARN|ERROR"
```

Danach im Log prüfen, dass weder `EACCES` noch „Ignored invalid configuration values“ auftaucht.

## Fallstrick: Speicherlimit wirkt nicht

Ollama wächst während der Arbeit. In einem Test stieg `llama-server` von 4,3 auf 6,45 GB, bis der Kernel ihn wegen Speichermangels beendete. Auf einem Pi, auf dem auch andere Dienste laufen, kann es genauso gut die anderen treffen.

Das naheliegende Speicherlimit in Docker Compose:

```yaml
services:
  ollama:
    image: ollama/ollama:<version>
    ports: ["11434:11434"]
    volumes: [ollama:/root/.ollama]
    restart: unless-stopped
    mem_limit: 5g
    memswap_limit: 5g
    oom_score_adj: 1000
volumes:
  ollama: {}
```

Auf Raspberry Pi OS wird `mem_limit` aber **ignoriert**. Der Kernel startet mit `cgroup_disable=memory`. Zu erkennen ist das an `docker info` mit „WARNING: No memory limit support“ und an `docker stats`, das für alle Container 0 B anzeigt.

Abhilfe: in `/boot/firmware/cmdline.txt` am Ende der einzigen Zeile anhängen und neu starten:

```
cgroup_enable=memory cgroup_memory=1
```

Die Datei muss eine einzige Zeile bleiben. Vorher eine Kopie anlegen, damit man sie notfalls über die SD-Karte zurückspielen kann. Danach den Container neu erstellen:

```bash
docker compose up -d --no-deps --pull never --force-recreate ollama
docker inspect ollama --format '{{.HostConfig.Memory}}'
```

`--pull never` verhindert, dass dabei ungewollt ein neues Image geladen wird, falls in der Compose-Datei `pull_policy: always` steht. Mit 8k Kontext belegt `qwen3.5:4b` rund 4,2 von 5 GiB. Wird es mehr, trifft der Kernel nur noch den Ollama-Container, und paperless-ai versucht es später erneut. `oom_score_adj: 1000` sorgt zusätzlich dafür, dass Ollama bei Speichermangel als Erstes beendet wird.

## Open WebUI zähmen

Läuft Open WebUI auf denselben Modellen, erzeugt es im Hintergrund zusätzliche Anfragen: Chat-Titel, Tags, Folgefragen und Suchanfragen. Auf dem Pi blockiert jede davon das Modell für paperless-ai. In den Admin-Einstellungen unter „Interface“ lassen sich diese Aufgaben abschalten.

Außerdem gehört Port 11434 nicht ins Internet. Ollama hat keine Anmeldung. Wer kann, hängt paperless-ai und Ollama an dasselbe Docker-Netz und veröffentlicht den Port gar nicht.

## Dateigröße in paperless-ngx

paperless-ngx speichert das Original unverändert und legt bei Bedarf zusätzlich eine Archivfassung (PDF/A mit Textebene) an. Verkleinern lässt sich nur die Archivfassung:

```
PAPERLESS_OCR_USER_ARGS={"optimize": 3}
```

Das nutzt `jbig2enc` und `pngquant`, die im offiziellen Image enthalten sind. In paperless-ngx 3.x hat `PAPERLESS_OCR_SKIP_ARCHIVE_FILE` keine Wirkung mehr. Der Nachfolger `PAPERLESS_ARCHIVE_FILE_GENERATION=auto` (Standard) legt für PDFs, die schon Text enthalten, keine Archivfassung an.

Kleine Originale entstehen nur beim Scannen. Grobe Erfahrungswerte je A4-Seite:

| Einstellung | Größe | Texterkennung |
|---|---|---|
| 150 dpi Graustufen | ca. 100–200 KB | normaler Text gut, kleine Schrift schwach |
| 200 dpi Graustufen | ca. 150–300 KB | gut |
| 300 dpi Graustufen | ca. 400–700 KB | sehr gut |
| 300 dpi Schwarzweiß | ca. 30–80 KB | sehr gut bei sauberem Text, schlecht bei blassem Druck |

Für normale Post ist 300 dpi Schwarzweiß meist die beste Wahl, für blassen Druck, Stempel oder farbige Formulare 200 dpi Graustufen. Die Qualität der Texterkennung bestimmt auch, was paperless-ai erkennen kann: Falsch gelesene Ziffern ergeben falsche Daten und Beträge.

## Checkliste

1. `cgroup_enable=memory cgroup_memory=1` in `cmdline.txt`, Neustart, `docker info` ohne Warnung.
2. Ollama mit `mem_limit`, `memswap_limit` und `oom_score_adj`, neu erstellt mit `--pull never --force-recreate`.
3. Modell `qwen3.5:4b` laden, Port 11434 nicht nach außen öffnen.
4. paperless-ai: `think` auf `"off"`, Timeout 900 s, Kontext 8192, Auslöser-Tag festlegen.
5. Nach jeder Änderung an `config.json` das Log auf `EACCES` und „Ignored invalid configuration values“ prüfen.
6. Tags, Korrespondenten und Dokumenttypen vorab in paperless-ngx anlegen und paperless-ai auf vorhandene beschränken.
7. In Open WebUI die Hintergrundaufgaben abschalten.
8. Scanner auf 300 dpi Schwarzweiß oder 200 dpi Graustufen stellen.
9. Die ersten Dokumente von Hand kontrollieren, vor allem Datum und Dokumenttyp.
