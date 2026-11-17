---
{"dg-publish":true,"dg-permalink":"en/ai/paperless-ai-raspberry-pi","permalink":"/en/ai/paperless-ai-raspberry-pi/","title":"paperless-ai with Ollama on a Raspberry Pi 5","metatags":{"description":"Field report: paperless-ngx with paperless-ai and a local language model via Ollama on a Raspberry Pi 5 with 8 GB. Model choice, speed, settings and pitfalls."},"dg-note-properties":{"title":"paperless-ai with Ollama on a Raspberry Pi 5"}}
---

#ai #llm #ollama #paperless #raspberrypi #docker

Field report: paperless-ngx tags documents with the help of paperless-ai, and the language model runs locally through Ollama on the same Raspberry Pi 5 with 8 GB. No document leaves the local network.

As of November 2026 with paperless-ngx 3.5.0, paperless-ai 4.1.0 and Ollama on Raspberry Pi OS (Debian 12, kernel 6.12), all in Docker Compose. Host names and paths are placeholders (`<pi>`, `<...>`). Setting names may change in later versions.

**Deutsche Version:** [[KI/paperless-ai mit Ollama auf dem Raspberry Pi 5\|paperless-ai mit Ollama auf dem Raspberry Pi 5]]

## Key points

* It works, but slowly: one document takes about a minute with a 4B model. Fine for incoming mail, not for a backlog of thousands of documents.
* The model's "thinking" must be off. Otherwise almost every request runs into the timeout.
* Raspberry Pi OS disables memory control for containers. A `mem_limit` in Docker Compose only takes effect after changing the boot parameters.
* `qwen3.5:4b` is the best compromise on the Pi. Smaller models are faster but invent tags by the dozen.
* paperless-ngx never changes the original files. Small files come from the scanner, not from paperless.

## Setup

| Service | Job |
|---|---|
| paperless-ngx (web server, PostgreSQL, Redis) | Ingest documents, OCR, storage |
| paperless-ai | Fetches new documents through the API, asks the language model, writes back title, tags, correspondent, type and date |
| Ollama | Runs the language model, CPU only on the Pi |
| Open WebUI (optional) | Chat interface for the same models |

paperless-ai only processes documents with a trigger tag (here `neu`, German for "new") and marks them with `ai-processed` afterwards. It checks for new documents every ten minutes.

## Choosing a model

Tested with 15 real documents, compared exactly against hand-maintained values, using a custom prompt and at most four tags.

| | qwen3.5:4b | qwen2.5:3b |
|---|---|---|
| Tags matched | 33 of 74 | 12 of 77 |
| Tags invented | 1 | 40 |
| Correspondent | 14 of 14 | 9 of 15 |
| Document type | 8 of 14 | 2 of 14 |
| Date | 10 of 14 | 10 of 15 |
| Time per document | approx. 90 s | approx. 50 s |

The smaller model saves some time but creates so many new tags that the tag list quickly becomes useless. Models larger than 4B barely fit into 8 GB next to the other services and would be even slower.

## Speed on the Pi 5

* Output: about **2.2 tokens per second** with `qwen3.5:4b` on the CPU.
* A typical paperless-ai request has 2,000 to 2,400 tokens.
* With thinking switched off, one document took **55 seconds** in production.
* Ollama handles requests one after another. If someone chats in Open WebUI at the same time, paperless-ai waits.

## paperless-ai settings

paperless-ai 4.x stores its settings in `/app/data/config.json` inside the container. The important values for the Pi:

```json
"ollama": {
  "url": "http://<pi>:11434",
  "model": "qwen3.5:4b",
  "contextSize": 8192,
  "think": "off"
},
"timeoutSeconds": 900
```

* **`think: "off"`:** With `auto`, `qwen3.5` reasons at length before answering. At 2.2 tokens per second every request hit the default timeout of 300 seconds. Allowed values are `"auto"` and `"off"`; a JSON `false` is dropped with a warning in the log.
* **`timeoutSeconds: 900`:** headroom for long documents.
* **`contextSize: 8192`:** enough for the requests and saves memory. 16k brought no benefit.

Other settings worth changing:

* **Remove the trigger tag:** the `neu` tag disappears after processing, so paperless-ngx shows at a glance what is still pending.
* **Allow only existing tags, correspondents and types:** otherwise the model creates new ones with every document.
* **Provide suitable document types:** without a "bank statement" type the model files a bank statement as an "invoice", for example.

## Pitfall: config.json silently ignored

The paperless-ai container runs with `cap_drop: ALL`. Even root cannot read other users' files or change owners inside it. If `config.json` belongs to another user after an edit, paperless-ai only logs `EACCES` and **silently loads `config.json.bak`**. The new values then do not apply.

The file must belong to `root:root` with mode `600`. The safest way to edit it is through the data volume with a helper container:

```bash
docker run --rm -u 0 --entrypoint sed \
  -v <paperless-ai-volume>:/app/data clusterzx/paperless-ai \
  -i 's/"think": "auto"/"think": "off"/' /app/data/config.json
docker restart paperless-ai
docker logs --since 1m paperless-ai | grep -E "config|WARN|ERROR"
```

Then check that the log shows neither `EACCES` nor "Ignored invalid configuration values".

## Pitfall: the memory limit has no effect

Ollama grows while working. In one test `llama-server` went from 4.3 to 6.45 GB until the kernel killed it for lack of memory. On a Pi that runs other services too, it could just as well hit those.

The obvious memory limit in Docker Compose:

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

On Raspberry Pi OS, however, `mem_limit` is **ignored**. The kernel boots with `cgroup_disable=memory`. You can tell from `docker info` showing "WARNING: No memory limit support" and from `docker stats` showing 0 B for every container.

Fix: append to the end of the single line in `/boot/firmware/cmdline.txt` and reboot:

```
cgroup_enable=memory cgroup_memory=1
```

The file must stay a single line. Make a copy first so you can restore it from the SD card if needed. Then recreate the container:

```bash
docker compose up -d --no-deps --pull never --force-recreate ollama
docker inspect ollama --format '{{.HostConfig.Memory}}'
```

`--pull never` prevents an unwanted new image from being pulled if the compose file says `pull_policy: always`. With an 8k context `qwen3.5:4b` uses about 4.2 of 5 GiB. If it grows beyond that, the kernel only hits the Ollama container and paperless-ai retries later. `oom_score_adj: 1000` additionally makes Ollama the first process to go when memory runs out.

## Taming Open WebUI

If Open WebUI uses the same models, it creates extra background requests: chat titles, tags, follow-up questions and search queries. On the Pi each of these blocks the model for paperless-ai. They can be switched off in the admin settings under "Interface".

Port 11434 also does not belong on the internet. Ollama has no authentication. If possible, attach paperless-ai and Ollama to the same Docker network and do not publish the port at all.

## File size in paperless-ngx

paperless-ngx keeps the original unchanged and, where needed, adds an archive version (PDF/A with a text layer). Only the archive version can be made smaller:

```
PAPERLESS_OCR_USER_ARGS={"optimize": 3}
```

This uses `jbig2enc` and `pngquant`, which ship with the official image. In paperless-ngx 3.x `PAPERLESS_OCR_SKIP_ARCHIVE_FILE` no longer has any effect. Its successor `PAPERLESS_ARCHIVE_FILE_GENERATION=auto` (the default) creates no archive version for PDFs that already contain text.

Small originals only come from the scanner. Rough values per A4 page:

| Setting | Size | OCR |
|---|---|---|
| 150 dpi grayscale | approx. 100–200 KB | normal text fine, small print weak |
| 200 dpi grayscale | approx. 150–300 KB | good |
| 300 dpi grayscale | approx. 400–700 KB | very good |
| 300 dpi black and white | approx. 30–80 KB | very good on clean text, poor on faint print |

For normal mail 300 dpi black and white is usually the best choice; for faint print, stamps or colored forms use 200 dpi grayscale. OCR quality also limits what paperless-ai can recognize: misread digits lead to wrong dates and amounts.

## Checklist

1. `cgroup_enable=memory cgroup_memory=1` in `cmdline.txt`, reboot, `docker info` without warnings.
2. Ollama with `mem_limit`, `memswap_limit` and `oom_score_adj`, recreated with `--pull never --force-recreate`.
3. Pull `qwen3.5:4b`, do not expose port 11434.
4. paperless-ai: `think` set to `"off"`, timeout 900 s, context 8192, trigger tag defined.
5. After every change to `config.json`, check the log for `EACCES` and "Ignored invalid configuration values".
6. Create tags, correspondents and document types in paperless-ngx beforehand and restrict paperless-ai to existing ones.
7. Switch off the background tasks in Open WebUI.
8. Set the scanner to 300 dpi black and white or 200 dpi grayscale.
9. Check the first documents by hand, especially date and document type.
