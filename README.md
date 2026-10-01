# TranscribeScanned — POC

Transcribes scanned handwritten PDF notes into a structured `.docx` using a vision LLM via **Vercel AI Gateway**.

Target job: [Freelancer: Transcribe Scanned Notes to Word](https://www.freelancer.com/projects/content-writing/Transcribe-Scanned-Notes-Word).

## How it works

```
PDF → page images (pdf2image) → Claude vision (via AI Gateway, OpenAI-compatible)
    → markdown transcript → .docx with Heading 1/2/3 (python-docx)
```

- Fidelity rules: literal transcription, line breaks preserved, `[illegible]` for unclear words, `[figure: …]` for diagrams.
- No custom fonts: Word defaults + built-in heading styles (as the client requested).

## Setup

System deps: `poppler-utils` (for `pdf2image`).

```bash
sudo apt install poppler-utils          # Debian/Ubuntu
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp env.example .env    # fill AI_GATEWAY_API_KEY
```

## Run (sandbox PDF: MIT 6.622 lecture 01, CC BY-NC-SA)

```bash
curl -L -o samples/mit_lec01.pdf \
  https://ocw.mit.edu/courses/6-622-power-electronics-spring-2023/mit6_622_s23_lec01_hand.pdf

# Transcribe only the first 2 pages first, to validate quality cheaply:
python transcribe.py samples/mit_lec01.pdf samples/mit_lec01.docx --pages 1-2
```

Outputs: `samples/mit_lec01.docx` + `samples/mit_lec01.md` (raw markdown for debugging).

## Model

Default: `anthropic/claude-opus-4-7` (string routed through Vercel AI Gateway).
Change via `MODEL` env var — e.g. `anthropic/claude-sonnet-5-5` for cheaper runs.

## Known limitations (POC)

- Markdown→docx mapper only handles headings and bullets. Tables/nested lists/inline formatting ignored.
- One LLM call per page — no retry/backoff.
- No parallelism — pages processed sequentially.
