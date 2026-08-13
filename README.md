# AI Video Assistant

A Python pipeline that turns a YouTube link or local audio/video file into meeting intelligence: transcript, title, summary, action items, key decisions, open questions, and a RAG chat interface over the transcript.

## Features

- Download and convert YouTube audio to WAV via `yt-dlp`
- Chunk audio into memory-safe pieces before transcription
- Local Whisper transcription for English audio
- Sarvam speech-to-text-translate API for Hinglish audio (30s API limit handled by slicing into 25s pieces)
- Automatic title generation and meeting summarization
- Extraction of action items, key decisions, and open questions
- Retrieval-augmented chat over the transcript (Chroma + Hugging Face sentence-transformer embeddings)
- CLI and Streamlit interfaces

## Tech Stack

- **Transcription:** `openai-whisper` (local, English) / Sarvam API (Hinglish)
- **LLM:** Mistral (`mistral-small-latest`) via `langchain-mistralai`, used for titling, summarization, extraction, and RAG answers
- **RAG:** `langchain`, Chroma vector store, `sentence-transformers` (`all-MiniLM-L6-v2`) via `langchain-huggingface`
- **Audio:** `yt-dlp`, `pydub` + `ffmpeg`
- **UI:** Streamlit (`streamlit_app.py`), plain CLI (`main.py`)

## Pipeline

```
YouTube URL / local file
        │
        ▼
utils/audio_processor.process_input()   → download/convert → chunked WAV files
        │
        ▼
core/transcriber.transcribe_all()       → Whisper (English) or Sarvam (Hinglish)
        │
        ├─→ core/summarizer.generate_title() / summarize()   (Mistral, chunked at 3000 chars)
        ├─→ core/extractor: action items / key decisions / open questions   (Mistral)
        └─→ core/rag_engine.build_rag_chain()
                 │
                 ▼
        core/vector_store.py — chunk transcript (500/50), embed with
        all-MiniLM-L6-v2, persist to a local Chroma collection ("meeting_transcript")
                 │
                 ▼
        ask_question() — retrieve top-k chunks, answer via Mistral, grounded in transcript
```

## Installation

```bash
git clone https://github.com/chmodgaurav/AI_Video_Assistant.git
cd AI_Video_Assistant
pip install -r requirements.txt
```

**System requirement:** `ffmpeg` must be installed and on `PATH` (required by `pydub` for audio conversion/chunking).

## Configuration

Create a `.env` file in the project root:

```env
MISTRAL_API_KEY=your_mistral_api_key
SARVAM_API_KEY=your_sarvam_api_key
WHISPER_MODEL=small
```

- `MISTRAL_API_KEY` — required for summarization, extraction, title generation, and RAG chat
- `SARVAM_API_KEY` — required only when transcribing with `hinglish`
- `WHISPER_MODEL` — Whisper model size (`tiny`/`base`/`small`/`medium`/`large`); defaults to `small`

## Usage

### CLI

```bash
python main.py
```

Prompts for a YouTube URL or local file path, then a language (`english` or `hinglish`), runs the full pipeline, prints title/summary/action items/decisions/questions, then drops into an interactive RAG chat loop over the transcript (type `exit` to quit).

### Streamlit UI

```bash
streamlit run streamlit_app.py
```

Open the local URL printed in the terminal to upload a file or paste a link through the browser.

### Smoke test

```bash
python test.py
```

Runs the full pipeline end-to-end against a hardcoded sample YouTube URL.

## Project Structure

| Path | Purpose |
|---|---|
| `main.py` | CLI entry point and pipeline orchestration |
| `streamlit_app.py` | Streamlit web interface |
| `test.py` | Runs the pipeline against a sample YouTube URL |
| `core/transcriber.py` | Whisper and Sarvam transcription logic |
| `core/summarizer.py` | Transcript summarization and title generation (Mistral) |
| `core/extractor.py` | Action item, decision, and question extraction (Mistral) |
| `core/rag_engine.py` | RAG chain construction and question answering |
| `core/vector_store.py` | Chunking, embeddings, and Chroma persistence |
| `utils/audio_processor.py` | YouTube download, format conversion, and chunking helpers |
| `render.yaml` | Render.com deployment config |

## Deployment

`render.yaml` defines a Render.com web service that installs `requirements.txt` and starts Streamlit. Its `startCommand` currently points at `app.py`, which does not exist in this repo — update it to `streamlit_app.py` before deploying to Render.

## Notes

- `openai-whisper` downloads and caches a model locally on first use; larger `WHISPER_MODEL` values need more disk space and RAM.
- The Chroma vector store used for RAG is rebuilt per run (`vector_db/`, collection `meeting_transcript`) — it isn't a persistent cross-session knowledge base.
- Sarvam's API has a hard 30-second-per-request limit, so `transcriber.py` re-slices each audio chunk into ~25s pieces for that path.

## Future Improvements

- Persist/reuse Chroma collections across runs instead of rebuilding per transcript
- Fix or remove the stale `app.py` reference in `render.yaml`
- Add export options (PDF/Markdown) — `reportlab` and `fpdf2` are already in `requirements.txt` but unused in the current pipeline

## Screenshots

<img width="1920" height="1080" alt="Screenshot From 2026-08-13 12-12-12" src="https://github.com/user-attachments/assets/532c7630-2044-46f4-a29b-599cb9e94f0e" />


## Contributing

Issues and pull requests are welcome.

## License

Use and modify this project freely.
