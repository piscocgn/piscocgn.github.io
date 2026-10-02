---
{"dg-publish":true,"permalink":"/notizen/ki/chat-gpt-ueber-agent-os/","dg-note-properties":{}}
---

| **framerslab/agentos**     |
| -------------------------- |
| **earthwalker17/agent-os** |
| **use-agent-os/agent-os**  |

Fas ist der Unterschied zwischen [https://github.com/framerslab/agentos](https://github.com/framerslab/agentos) und [https://github.com/earthwalker17/agent-os?utm_source=chatgpt.com](https://github.com/earthwalker17/agent-os?utm_source=chatgpt.com) und [https://github.com/use-agent-os/agent-os](https://github.com/use-agent-os/agent-os)

Ja – jetzt lässt sich das sauber auseinanderhalten. **Die drei Projekte sind eigenständige Projekte und nicht drei Varianten desselben AgentOS.** Der gemeinsame Name „AgentOS“ ist hier tatsächlich ziemlich verwirrend.

| Projekt                    | Was ist es?                       | Schwerpunkt                                                           | Technik                      |
| -------------------------- | --------------------------------- | --------------------------------------------------------------------- | ---------------------------- |
| **framerslab/agentos**     | KI-Agent-Framework als Bibliothek | Memory, Tools, Multi-Agenten                                          | **TypeScript / Node.js**     |
| **earthwalker17/agent-os** | komplette Coding-Agent-Umgebung   | Softwareprojekte planen → programmieren → testen → deployen           | **Python + FastAPI + React** |
| **use-agent-os/agent-os**  | kompletter persönlicher Agent     | Chat, Tools, Memory, Subagents, Scheduling, Provider-Routing, Trading | **Python**, Web/Desktop/CLI  |

### 1. FramersLab AgentOS

[framerslab/agentos auf GitHub](https://github.com/framerslab/agentos?utm_source=chatgpt.com)

Das ist am ehesten ein **Framework für Entwickler**, aus dem man einen eigenen Agenten baut.

Das Besondere ist beispielsweise:

- persistentes „cognitive memory“
- mehrere Agenten
- 11 LLM-Provider
- Agenten können **zur Laufzeit eigene Tools erzeugen**
- Tool-Ausführung in einer Sandbox
- Sessions und langfristiger Kontext
- optionale „Personality“ über HEXACO

Man benutzt es beispielsweise als Node/TypeScript-Bibliothek:

```
deine Anwendung
      │
      ▼
  AgentOS Framework
      │
 ┌────┼─────┐
 ▼    ▼     ▼
LLM  Memory Tools
```

Es ist **nicht primär ein fertiges Programm**, das du installierst und dann wie Hermes benutzt. [GitHub](https://github.com/framerslab/agentos)

---

### 2. Earthwalker Agent OS

[earthwalker17/agent-os auf GitHub](https://github.com/earthwalker17/agent-os?utm_source=chatgpt.com)

Das ist konzeptionell etwas völlig anderes.

Hier geht es speziell darum:

> **„Ich möchte ein KI-Modell meine Softwareprojekte bauen lassen, aber mit einer kontrollierten Umgebung darum herum.“**

Die zentrale Idee ist tatsächlich:

```
             LLM
              │
              ▼
        ┌───────────┐
        │ Agent OS  │
        │  Harness  │
        └─────┬─────┘
              │
      ┌───────┼────────┐
      ▼       ▼        ▼
    Plan    Code     Verify
                       │
                       ▼
                    Deploy
```

Es gibt bewusst zwei Agenten:

- **Main Agent** = plant, verwaltet Memory und entscheidet
- **Coding Agent** = darf tatsächlich Dateien ändern und Shell-Kommandos ausführen

Dazu kommen Sandbox, Git, Tests, Browser-Checks, Recovery usw. [GitHub](https://github.com/earthwalker17/agent-os/blob/main/ARCHITECTURE.md?utm_source=chatgpt.com)

Das ist daher eher ein **„Claude Code/Codex/OpenCode mit Sicherheits- und Projektmanagement-Schicht“** als ein allgemeiner persönlicher Assistent.

---

### 3. `use-agent-os/agent-os`

[use-agent-os/agent-os auf GitHub](https://github.com/use-agent-os/agent-os?utm_source=chatgpt.com)

**Das ist dasjenige, das zu deiner vorherigen Hermes-Frage passt.**

Es ist ein fertiger **lokaler General-Purpose-Agent**:

```
                 AgentOS
                    │
      ┌─────────────┼─────────────┐
      ▼             ▼             ▼
    Web UI         CLI          Telegram
      │             │             │
      └─────────────┼─────────────┘
                    ▼
              Agent / Tools
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
      Memory     Sandbox       Web
                    │
             Pilot Router
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
    OpenRouter    Ollama       OpenAI
```

Es kann unter anderem:

- lokal laufen
- Memory speichern
- Tools verwenden
- Subagents starten
- Webzugriff verwenden
- geplante Jobs ausführen
- Telegram/Discord/Slack anbinden
- verschiedene LLM-Provider benutzen
- lokale Modelle über **Ollama** verwenden
- einen lokalen Router benutzen, der Aufgaben auf unterschiedliche Modelle verteilt. [GitHub](https://github.com/use-agent-os/agent-os)

Und für deine ursprüngliche Frage besonders interessant:

**Es kann Daten aus Hermes übernehmen.**

Das Repository enthält ausdrücklich:

```
agentos migrate hermes
```

und auch eine Migration von OpenClaw. Das ist ein ziemlich deutlicher Hinweis darauf, dass dieses AgentOS als **Nachfolger/Alternative für bestehende Hermes/OpenClaw-Installationen** gedacht ist. [GitHub](https://github.com/use-agent-os/agent-os?utm_source=chatgpt.com)

---

## Der entscheidende Unterschied

Wenn ich die drei auf **deine Anwendung** herunterbreche:

### Du willst selbst einen Agenten programmieren

→ **FramersLab AgentOS**

```
TypeScript
    ↓
@framers/agentos
    ↓
dein eigener Agent
```

### Du willst hauptsächlich Software entwickeln lassen

→ **earthwalker17/agent-os**

```
Chat
 ↓
Plan
 ↓
Coding Agent
 ↓
Test
 ↓
Review
 ↓
Git / Deploy
```

### Du willst einen fertigen persönlichen lokalen Agenten

→ **use-agent-os/agent-os**

```
Telegram / Web / CLI
          ↓
       AgentOS
          ↓
   Memory + Tools
          ↓
  Ollama / OpenRouter /
  Claude / GPT / usw.
```

---

### Und wo steht Hermes dabei?

Das ist jetzt der interessante Teil:

```
                 ┌── FramersLab AgentOS
                 │      Framework
                 │
LLM ─────────────┼── Earthwalker Agent OS
                 │      Coding-Harness
                 │
                 ├── use-agent-os AgentOS
                 │      General-purpose Agent
                 │
                 └── Hermes
                        General-purpose Agent
```

**Hermes und `use-agent-os/agent-os` sind also tatsächlich die beiden, die du sinnvoll direkt vergleichen kannst.**

Das `use-agent-os`-Projekt bezeichnet sich selbst als von **OpenClaw und Hermes Agent beeinflusst** und bietet sogar eine direkte Hermes-Migration an. [GitHub](https://github.com/use-agent-os/agent-os?utm_source=chatgpt.com)

Und **Earthwalker Agent OS ist eine ganz andere Idee**: dort steht das Coding von Softwareprojekten im Mittelpunkt. [GitHub](https://github.com/earthwalker17/agent-os/blob/main/BLUEPRINT.md?utm_source=chatgpt.com)

**Für deinen konkreten Fall – M3 Mac, 24 GB RAM, Ollama/Qwen und Programmierhilfe – würde ich deshalb vor allem `Hermes` vs. `use-agent-os/agent-os` vs. `earthwalker17/agent-os` vergleichen.** Das wäre deutlich aussagekräftiger als die drei unter dem gemeinsamen Namen „AgentOS“ zu betrachten.