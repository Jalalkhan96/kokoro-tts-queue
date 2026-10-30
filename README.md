# kokoro-tts-queue

Kokoro TTS request queue — bots drop request JSON files, voices come back as .ogg results.

## What it is

A file-based queue for Kokoro text-to-speech jobs, used by chat bots (the samples here come from WhatsApp). There is no server and no database: the queue **is** this repo's two folders. A bot writes a request file, an external processor picks it up, generates the speech with Kokoro TTS, and writes back a result file with the same ID pointing at the hosted audio.

This repo contains only the queue data (requests and results). The bot and the TTS processor that service the queue live outside this repo.

## How it works

```
requests/<id>.json   →   (processor generates audio)   →   results/<id>.json
```

**Request** — one JSON file per job in `requests/`:

```json
{
  "id": "wa-72305774440506-mvkbor9j",
  "platform": "whatsapp",
  "to": "120363411920049474@g.us",
  "text": "Hi this is just a testing message...",
  "voice": "af_bella",
  "speed": 1,
  "status": "done",
  "created_at": "2026-10-23T02:00:16.903Z"
}
```

**Result** — the matching file in `results/`:

```json
{
  "id": "wa-72305774440506-mvkbor9j",
  "status": "done",
  "audio_url": "https://…/wa-72305774440506-mvkbor9j.ogg",
  "voice": "af_bella",
  "format": "ogg",
  "duration_sec": 4.3,
  "expires_at": "2026-10-25T02:14:44+00:00",
  "processed_at": "2026-10-23T02:00:16.903Z"
}
```

Field notes from the committed samples:

- `id` — `wa-<number>-<random suffix>`; the same ID names both the request and the result file
- `platform` / `to` — where the audio goes back to (here: a WhatsApp group JID)
- `voice` — a Kokoro voice name (samples use `af_bella` and `hm_omega`)
- `audio_url` — hosted `.ogg` file; `expires_at` shows results are time-limited, so consumers should fetch promptly

## Contents

```
requests/   7 request JSON files (2026-10-23 samples)
results/    7 matching result JSON files
README.md   This file
```

## Usage

To enqueue a job, a bot writes a new request JSON into `requests/` following the schema above, then polls `results/` for the file with the same ID and reads its `audio_url`.

## Tech

- Kokoro TTS (voice synthesis), JSON files as the queue transport, OGG audio output

## License

MIT — see [LICENSE](LICENSE).
