# AI Video Generation Workflow

An open-source workflow for generating short finance explainer videos with **script + slides + voice + subtitles + render**.

This project focuses on reliability for batch production (for example, 3 themed videos with consistent style), instead of relying purely on text-to-video models with unstable output quality.

## Demo Videos

GitHub may not play large MP4 files inline in all cases, so we provide preview frames + direct links.

PE Demo Preview:

![PE Demo Preview](./showcase/pe-case-demo-first-frame.jpg)

[Open PE demo video](./showcase/pe-case-demo.mp4)

VC Demo Preview:

![VC Demo Preview](./showcase/vc-case-demo-first-frame.jpg)

[Open VC demo video](./showcase/vc-case-demo.mp4)

## Why This Project

- Stable, reproducible pipeline for short-form finance content.
- Modular reruns: regenerate only the failed stage (script/image/voice/render).
- Strong sync control across voice, subtitles, and image transitions.
- Works well with NotebookLM slide workflow (PPT -> slide images -> final video).

## Core Workflow

1. Generate script by structured segments (`hook`, `definition`, `example`, etc.).
2. Prepare NotebookLM input to generate PPT slides.
3. Import slide images per video (`video-01`, `video-02`, `video-03`).
4. Generate voice (Edge / ElevenLabs / Gemini fallback).
5. Build subtitles from real audio durations.
6. Render final MP4 with synchronized timeline.

Architecture diagram and details: [docs/ARCHITECTURE.md](./docs/ARCHITECTURE.md)

## Tech Stack

- Node.js + TypeScript (`tsx`)
- FFmpeg (render and audio concat)
- Optional Python (`edge-tts`) for Chinese natural voice
- Gemini / ElevenLabs APIs (configurable)

## Prerequisites

- Node.js >= 20
- Python >= 3.10
- FFmpeg

FFmpeg install examples:

```bash
# macOS
brew install ffmpeg

# Ubuntu / Debian
sudo apt update && sudo apt install -y ffmpeg
```

Windows: install FFmpeg and add it to `PATH`, or set `FFMPEG_PATH` in `.env`.

## Quick Start

### 1. Install dependencies

```bash
npm install
python3 -m pip install -r requirements.txt
cp .env.example .env
```

### 2. Configure `.env`

At minimum, configure one text/image provider and one voice path.

Required keys by module:

| Module | Minimal Required Keys |
|---|---|
| Script generation | `GOOGLE_API_KEY` or `GEMINI_API_KEY` |
| Image generation | `GOOGLE_API_KEY` |
| Voice generation (Edge) | `VOICE_PROVIDER=edge` + `edge-tts` installed |
| Voice generation (ElevenLabs) | `VOICE_PROVIDER=elevenlabs` + `ELEVENLABS_API_KEY` + `ELEVENLABS_VOICE_ID` |
| Rendering | `FFMPEG_PATH` (optional if `ffmpeg` is already in `PATH`) |

### 3. Generate plans

```bash
npm run build:all
```

### 4. Generate scripts

```bash
npm run script:gen
```

### 5. NotebookLM route (recommended for slide quality)

```bash
npm run slides:prepare
```

Then use generated markdown files in `build/video-xx/notebooklm-input.md` to generate PPT in NotebookLM, export each page as images, and place them under:

- `external-slides/video-01/`
- `external-slides/video-02/`
- `external-slides/video-03/`

Import slides:

```bash
npm run slides:import
```

### 6. Generate voice

```bash
npm run voice:gen
```

### 7. Render video

```bash
npm run video:render
```

## Commands

- `npm run build:all` -> plan + QA checks
- `npm run script:gen` -> generate scripts
- `npm run image:gen` -> generate images from prompts
- `npm run slides:prepare` -> prepare NotebookLM prompts
- `npm run slides:import` -> import exported slide images
- `npm run voice:gen` -> generate TTS audio
- `npm run video:render` -> render final MP4
- `npm run run:all` -> full pipeline

## Troubleshooting

- `ffmpeg not found`:
  - install FFmpeg and verify `ffmpeg -version`
  - or set `FFMPEG_PATH=/absolute/path/to/ffmpeg` in `.env`
- `edge-tts` not found:
  - run `python3 -m pip install edge-tts`
- API 429 / quota errors:
  - reduce concurrency and retry later
  - switch provider/model in `.env`
- Subtitle or timing mismatch after script/voice changes:
  - rerun `npm run voice:gen` then `npm run video:render`

## Repository Layout

```text
src/
  lib/
  scripts/
config/
content/topics/
prompts/
examples/
  scripts/
  notebooklm-inputs/
showcase/
docs/
```

## What Is Included in This Open-Source Copy

Included:
- Pipeline source code
- Config templates
- Prompt templates
- Example scripts and NotebookLM inputs
- Documentation and contribution templates

Not included:
- Private API keys
- Private/uncleared source PDFs
- Large local build artifacts

## Compliance & Usage Disclaimer

- MIT license applies to this repository's **source code only**.
- Generated assets (text/audio/image/video) may be subject to third-party provider terms (Google, ElevenLabs, NotebookLM, etc.).
- `edge-tts` and connected speech services may have usage restrictions (including commercial usage constraints). You are responsible for verifying the latest Terms of Service before production/commercial use.
- You are responsible for rights clearance of all source materials and generated media.
- FFmpeg is invoked as an external CLI dependency in this project.

## Roadmap

- Add Docker / docker-compose for one-command reproducible setup.
- Add optional local model path (for lower API cost).
- Add more TTS backends (for example ChatTTS / CosyVoice adapters).
- Add BGM auto-mix presets and loudness normalization.
- Add explicit artifact checkpoint strategy for long workflows.

## Pre-publish Check

```bash
./scripts/publish-check.sh
```

## License

MIT
