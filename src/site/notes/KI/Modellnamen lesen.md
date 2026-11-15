---
{"dg-publish":true,"dg-permalink":"ki/modellnamen","permalink":"/ki/modellnamen/","title":"KI-Modellnamen lesen: Hugging Face und Ollama","metatags":{"description":"Was Namen wie Qwen3.6-35B-A3B-GGUF:UD-Q4_K_XL bedeuten: Familie, Größe, Mixture of Experts, Variante, Format und Quantisierung. Mit Faustregeln für Speicher und Tempo und einer Prüfliste."},"dg-note-properties":{"title":"KI-Modellnamen lesen: Hugging Face und Ollama"}}
---

#ki #llm #ollama #huggingface #lokal

Ein Name wie `Qwen3.6-35B-A3B-GGUF:UD-Q4_K_XL` sagt fast alles, was man vor dem Download wissen muss: Hersteller, Größe, Bauart, Zweck, Dateiformat und Kompression. Diese Seite zeigt, wie man die Teile liest und was sie für Speicher, Tempo und Einsatz bedeuten.

Stand November 2026. Faustregeln und Bit-Werte sind gerundete Richtwerte. Die Messungen stammen von einem MacBook Air M3 mit 24 GB und Ollama 0.40.

**English version:** [[En/AI/Reading model names\|Reading model names]]

## Die Bausteine

Die Reihenfolge schwankt je nach Hersteller, die Bausteine sind fast immer dieselben.

| Baustein | Beispiel | Bedeutung |
|---|---|---|
| Uploader | `unsloth/`, `Qwen/` | Vor dem Schrägstrich: Hersteller oder Umverpacker |
| Familie | `Qwen`, `gemma`, `Llama`, `gpt-oss` | Modellreihe des Herstellers |
| Generation | `3.6`, `4`, `3.3` | Höher ist neuer |
| Größe | `27B`, `360M` | Parameter in Milliarden (B) oder Millionen (M) |
| Aktive Parameter | `A3B` | Mixture of Experts: rechnet je Wort nur mit einem Teil |
| Variante | `Instruct`, `it`, `Thinking`, `Coder`, `VL` | Wofür das Modell trainiert ist |
| Stand | `2507`, `0528` | Juli 2025 (JJMM) oder 28. Mai (MMTT) |
| Format | `GGUF`, `MLX`, `AWQ`, `FP8` | Bestimmt, welche Software es lesen kann |
| Quantisierung | `Q4_K_M`, `Q8_0`, `4bit` | Wie stark die Gewichte komprimiert sind |
| Kontext | `64k`, `128k` | Bei Ollama-Namen meist selbst vergeben |
| Ollama-Tag | `:latest`, `:27b` | Hinter dem Doppelpunkt: welche Variante |
| Cloud | `-cloud` | Läuft nicht lokal, für private Daten tabu |

## Drei Namen zerlegt

**`unsloth/Qwen3.6-35B-A3B-GGUF:UD-Q4_K_XL`**

* `unsloth/`: bekannter Umverpacker, das Original stammt von Alibaba
* `Qwen` `3.6`: Familie Qwen, Generation 3.6
* `35B-A3B`: Mixture of Experts mit 35 Mrd. Parametern, davon 3 Mrd. aktiv
* `GGUF`: Format für Ollama, llama.cpp und LM Studio
* `UD-Q4_K_XL`: rund 5 Bit je Gewicht, wichtige Schichten genauer („Unsloth Dynamic“)
* Ergebnis: rund 24 GB Datei, für einen Mac mit 24 GB zu groß

**`Qwen/Qwen3-235B-A22B-Thinking-2507`**

* `Qwen/`: Upload des Herstellers selbst
* `235B-A22B`: sehr großes MoE-Modell, 22 Mrd. aktiv
* `Thinking`: denkt vor der Antwort ausführlich nach
* `2507`: Versionsstand Juli 2025
* Keine Formatangabe: Originalgewichte für Server-GPUs

**`deepseek-ai/DeepSeek-R1-Distill-Qwen-7B`**

* `DeepSeek-R1`: Reasoning-Serie von DeepSeek
* `Distill-Qwen-7B`: eigentlich ein Qwen 7B, das von R1 gelernt hat. Der Name nach `Distill` ist das Grundmodell.

## Größe und Bauart

Die Zahl vor dem B ist der wichtigste Wert im Namen. Sie bestimmt Speicherbedarf, Tempo und grob auch die Fähigkeiten.

| Schreibweise | Bedeutung | Praktisch |
|---|---|---|
| `27B` | Dichtes Modell mit 27 Mrd. Parametern | Jedes erzeugte Wort liest alle Gewichte. Speicher und Tempo hängen direkt an der Zahl. |
| `35B-A3B` | Mixture of Experts: 35 Mrd. gesamt, 3 Mrd. aktiv | Braucht Speicher für 35B, rechnet aber fast so schnell wie ein 3B-Modell. Auf Macs oft die beste Wahl, wenn es passt. |
| `8x7B`, `17B-16E` | Ältere MoE-Schreibweise: 8 Experten zu 7B, oder 16 Experten | Gesamtgröße kleiner als das Produkt, Mixtral 8x7B hat rund 47B. |
| `e4b` | „Effective“ 4B, eine Gemma-Eigenheit | Rechnet wie 4B, die Datei ist durch Zusatzteile größer. |
| `360M` | Millionen statt Milliarden | Winzlinge für Handy, Autovervollständigung oder Klassifizierung. |

## Varianten

| Kürzel | Bedeutung | Einsatz |
|---|---|---|
| `Instruct`, `it`, `Chat` | Auf Anweisungen und Gespräche trainiert | Das, was man fast immer will. `it` ist Googles Kürzel. |
| `Base`, `pt` | Rohes Grundmodell ohne Chat-Training | Nur als Ausgangspunkt für eigenes Feintuning. |
| `Thinking`, `Reasoning`, `R1` | Denkt vor der Antwort ausführlich nach | Genauer bei Logik und Rechnen, dafür langsamer. |
| `Coder`, `Code` | Auf Programmieren spezialisiert | Code schreiben und prüfen. |
| `VL`, `Vision`, `Omni` | Versteht Bilder, Omni auch Audio | Belege, Screenshots, Fotos. |
| `Embedding` | Wandelt Text in Zahlenvektoren | Suche über eigene Dokumente (RAG). Kein Chatmodell. |
| `Guard`, `safeguard` | Prüft Inhalte auf Richtlinien | Filter vor oder nach einem Chatmodell. |
| `Distill` | Kleines Modell, das von einem großen gelernt hat | Der Name dahinter ist das Grundmodell. |
| `Flash`, `Mini`, `Small`, `Pro`, `Max` | Stufe innerhalb einer Serie | Nur innerhalb der Familie vergleichbar. |
| `abliterated`, `uncensored` | Community-Version ohne Schutzfilter | Herkunft oft unklar, für ernsthafte Arbeit meiden. |

## Formate

| Format | Läuft mit | Hinweis |
|---|---|---|
| `safetensors` (ohne Zusatz) | Transformers, vLLM, Server-GPUs | Originalgewichte in voller Präzision. Für den Laptop meist zu groß. |
| `GGUF` | Ollama, llama.cpp, LM Studio | Standard für lokale Nutzung. Ein Repo enthält oft viele Quantisierungsstufen. |
| `MLX`, `-mlx` | Apple Silicon (LM Studio, mlx-lm, neuere Ollama-Versionen) | Für Macs gebaut. Ob es schneller ist als GGUF, hängt vom Modell ab. |
| `AWQ`, `GPTQ`, `EXL2/3` | Nvidia-GPUs | Auf dem Mac nicht nutzbar. |
| `FP8`, `NVFP4`, `MXFP4` | Neuere Nvidia-Rechenzentrumskarten | NVFP4 braucht Blackwell-GPUs. MXFP4 ist das Originalformat von gpt-oss. |

## Quantisierung

Die Gewichte werden von 16 Bit auf weniger Bit gerundet. Das spart Speicher und macht das Modell schneller, kostet aber etwas Qualität. Die Bit-Werte sind Durchschnitte inklusive Verwaltungsdaten.

| Stufe | Bit je Gewicht | GB je Mrd. Parameter | Qualität |
|---|---|---|---|
| `BF16`, `F16` | 16 | ≈ 2,0 | Original, Referenz |
| `Q8_0`, `8bit` | ≈ 8,5 | ≈ 1,1 | Praktisch verlustfrei |
| `Q6_K` | ≈ 6,6 | ≈ 0,8 | Sehr gut |
| `Q5_K_M` | ≈ 5,7 | ≈ 0,7 | Gut, kaum Unterschied zum Original |
| `Q4_K_M`, `4bit` | ≈ 4,5–4,9 | ≈ 0,6 | **Der übliche Kompromiss**, Ollamas Standard |
| `IQ4_XS` | ≈ 4,3 | ≈ 0,55 | Fast wie Q4_K_M, etwas kleiner |
| `Q3_K_M`, `IQ3_XXS` | ≈ 3,1–3,9 | ≈ 0,45 | Merklicher Verlust, Notlösung |
| `Q2_K`, `IQ2_XXS` | ≈ 2,1–3,0 | ≈ 0,35 | Stark beschädigt, nur zum Ausprobieren |

* `K` steht für das k-quant-Verfahren, `S`/`M`/`L`/`XL` für den Anteil höher aufgelöster Schichten.
* `I` vorne (`IQ3`) oder `i1`, `imatrix`: mit Wichtigkeitsmatrix quantisiert, bei niedrigen Stufen besser.
* `UD`: „Unsloth Dynamic“, wichtige Schichten bleiben genauer. Etwas größer als die gleichnamige Normalstufe.
* `QAT`: Der Hersteller hat das Modell auf die Kompression vorbereitet. Verliert bei 4 Bit am wenigsten.

## Woher es kommt

**Ollama:** `name:tag`. Ohne Schrägstrich stammt das Modell aus der kuratierten Ollama-Bibliothek, mit `konto/name` von einem Community-Konto.

* `:latest` ist die Standardvariante, nicht unbedingt die neueste oder größte.
* Tags wählen Größe und Kompression: `:27b`, `:27b-q8_0`, `:26b-mlx`.
* `-cloud` läuft auf Ollamas Servern. Für private Daten ausschließen.
* `hf.co/konto/repo:Q4_K_M` lädt eine GGUF-Datei direkt von Hugging Face.
* Ein Kontext im Namen wie `-64k` ist meist selbst angelegt. Ollama startet sonst mit 4k Kontext. Eigene Varianten entstehen per Modelfile mit `PARAMETER num_ctx`.

**Hugging Face:** `konto/repo`. Der Teil vor dem Schrägstrich sagt, wer die Datei hochgeladen hat.

* Hersteller: `Qwen`, `google`, `meta-llama`, `openai`, `mistralai`, `deepseek-ai`, `microsoft`, `nvidia`.
* Bekannte Umverpacker: `unsloth`, `bartowski`, `lmstudio-community`, `mlx-community`. Sie liefern GGUF- und MLX-Fassungen der Herstellermodelle.
* Repo-Endungen wie `-GGUF`, `-MLX`, `-AWQ`, `-FP8` nennen das Format. Die Stufe steht dann im Dateinamen.
* Lizenz, Grundmodell und empfohlene Einstellungen stehen auf der Modellkarte, nicht im Namen.

## Faustregeln für Speicher und Tempo

**Passt es in den Speicher?**

```
GB ≈ Parameter (Mrd.) × Bit ÷ 8 × 1,1
```

Dazu kommen 1 bis 3 GB für den Kontext und bei Bildmodellen ein Bild-Encoder. Ein Mac gibt der Grafikeinheit von sich aus nur einen Teil des Arbeitsspeichers. Als Richtwert gelten etwa 70 %, bei 24 GB also rund 16 bis 17 GB für Modell und Kontext.

**Wie schnell schreibt es?**

```
Tokens/s ≈ Speicherbandbreite ÷ (aktive Parameter × Bit ÷ 8) × 0,75
```

Jedes Wort liest alle aktiven Gewichte einmal. Deshalb entscheidet die Bandbreite, nicht die Rechenleistung. Bei MoE-Modellen liegt der Faktor eher bei 0,4. Ein MacBook Air M3 hat 100 GB/s, ein M4 Pro 273 GB/s, ein M4 Max bis 546 GB/s. Ab etwa 8 Tokens pro Sekunde liest sich eine Antwort flüssig.

**Gegenprobe** auf einem MacBook Air M3 mit 24 GB, Ollama, kurze Eingabe:

| Modell | Im Speicher | Gemessen | Laut Faustregel |
|---|---|---|---|
| `qwen3:4b` | 2,9 GB | 23 Tokens/s | ≈ 31 |
| `qwen3.5:4b` | 3,1 GB | 21 Tokens/s | ≈ 31 |
| `qwen3.5` 9B (Q4_K_M) | 6,9 GB | 14–16 Tokens/s | ≈ 14 |
| `gemma4:12b` | 7,7 GB | 11 Tokens/s | ≈ 10 |
| `gpt-oss:20b` (MoE, 3,6B aktiv) | 12 GB | 19 Tokens/s | ≈ 21 |

Kleine Modelle bleiben unter der Schätzung, weil fester Aufwand je Wort stärker ins Gewicht fällt. Das Einlesen langer Eingaben ist ein eigener Engpass: Rund 13.000 Tokens brauchten auf demselben Rechner zweieinhalb bis drei Minuten.

## Prüfliste vor dem Einsatz

1. **Herkunft:** Kommt die Datei vom Hersteller oder von einem bekannten Umverpacker? Unbekannte Konten und „uncensored“-Varianten meiden.
2. **Lizenz:** auf der Modellkarte prüfen. Apache 2.0 und MIT sind frei nutzbar. Llama, Gemma und einige Mistral-Modelle haben eigene Bedingungen. „Research only“ oder „non-commercial“ schließt kommerzielle Arbeit aus.
3. **Datenschutz:** Läuft es wirklich lokal? Kein `-cloud`-Tag, keine API-Schlüssel externer Anbieter, Websuche im Werkzeug abgeschaltet.
4. **Hardware:** Dateigröße plus Kontext unter etwa 70 % des Arbeitsspeichers (Mac) oder des Grafikspeichers (PC).
5. **Passende Variante:** Instruct für Chat, Coder für Code, Embedding für Suche. Für Agenten muss das Modell Werkzeuge aufrufen können. Ollama zeigt das als Fähigkeit „tools“.
6. **Aktualität:** Generation und Datum vergleichen. Ein neues 9B schlägt oft ein altes 30B.
7. **Selbst testen:** Herstellerbenchmarks sind Werbung. Drei bis fünf echte Aufgaben aus dem eigenen Alltag reichen, um die Kandidaten zu trennen.
