# AI Video Assistant

A Python application that extracts meeting intelligence from YouTube links or local audio/video files: transcription, summarization, action items, key decisions, open questions, and retrieval-augmented Q&A over the transcript.

## Features

- Download and convert YouTube audio to WAV (`yt-dlp`)
- Chunk audio for reliable, memory-safe transcription
- Local Whisper transcription for English audio
- Sarvam transcription for Hinglish audio
- Meeting summarization and automatic title generation
- Extraction of action items, key decisions, and open questions
- Retrieval-augmented generation (RAG) chat over the meeting transcript
- Optional Streamlit UI

## Pipeline

```
YouTube URL / local file
        │
        ▼
audio_processor.process_input()   → download/convert → chunked WAV files
        │
        ▼
transcriber.transcribe_all()      → Whisper (English) or Sarvam (Hinglish)
        │
        ├─→ summarizer.generate_title() / summarize()
        ├─→ extractor.extract_action_items() / extract_key_decisions() / extract_questions()
        └─→ rag_engine.build_rag_chain()  → vector_store.py (Chroma + HF embeddings)
                        │
                        ▼
                 ask_question() — chat over the transcript
```

## Project Structure

| Path | Purpose |
|---|---|
| `main.py` | CLI entry point and pipeline orchestration |
| `streamlit_app.py` | Streamlit web interface |
| `test.py` | Runs the pipeline against a sample YouTube URL |
| `core/audio_processor.py` | Download/convert audio, chunk WAV files |
| `core/transcriber.py` | Whisper and Sarvam transcription logic |
| `core/summarizer.py` | Transcript summarization and title generation |
| `core/extractor.py` | Action item, decision, and question extraction |
| `core/rag_engine.py` | RAG chain construction and question answering |
| `core/vector_store.py` | Embeddings and Chroma vector store persistence |
| `utils/audio_processor.py` | Shared audio download/conversion/chunking helpers |

## Requirements

- Python 3.10+
- `ffmpeg` installed on the system (required by `pydub` for format conversion/chunking)

## Installation

```bash
git clone https://github.com/chmodgaurav/AI_Video_Assistant.git
cd AI_Video_Assistant
pip install -r requirements.txt
```

## Configuration

Create a `.env` file in the project root:

```env
MISTRAL_API_KEY=your_mistral_api_key
SARVAM_API_KEY=your_sarvam_api_key
WHISPER_MODEL=small
```

- `MISTRAL_API_KEY` — required for summarization, extraction, title generation, and RAG chat
- `SARVAM_API_KEY` — required only when using `hinglish` transcription
- `WHISPER_MODEL` — defaults to `small` if unset

## Usage

### CLI

```bash
python main.py
```

Prompts for a YouTube URL or local file path, then a language option:

- `english` — local Whisper transcription
- `hinglish` — Sarvam transcription

### Streamlit UI

```bash
streamlit run streamlit_app.py
```

Open the local URL shown in the terminal to upload a file or paste a link through the browser.

### Sample run

```bash
python test.py
```

Runs the full pipeline end-to-end against a sample YouTube URL for a quick smoke test.

## Notes

- `yt-dlp` handles YouTube audio extraction.
- `openai-whisper` downloads and caches a local model on first use — this can take significant disk space depending on `WHISPER_MODEL`.
- `pydub` + `ffmpeg` handle audio format conversion and chunking.
- The RAG chat uses Chroma with Hugging Face embeddings for transcript retrieval.

## License

Use and modify this project freely.
