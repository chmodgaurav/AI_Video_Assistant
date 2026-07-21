# AI Video Assistant

AI Video Assistant is a Python application that extracts meeting intelligence from YouTube or local audio/video files. It downloads or converts audio, transcribes speech, summarizes content, extracts action items, decisions, questions, and enables RAG-powered Q&A over transcript context.

## Features

- Download and convert YouTube audio to WAV
- Chunk audio for reliable transcription
- Local Whisper transcription for English audio
- Sarvam transcription for Hinglish audio
- Meeting summarization and title generation
- Extraction of action items, key decisions, and open questions
- Retrieval-augmented generation (RAG) chat over meeting transcript
- Optional Streamlit UI for friendly web-based interaction

## Requirements

- Python 3.10+
- `ffmpeg` installed on the system

Install Python dependencies:

```bash
pip install -r requirements.txt
```

## Environment Variables

Create a `.env` file in the project root with the following keys:

```env
MISTRAL_API_KEY=your_mistral_api_key
SARVAM_API_KEY=your_sarvam_api_key
WHISPER_MODEL=small
```

- `MISTRAL_API_KEY` is required for summarization, extraction, title generation, and RAG chat.
- `SARVAM_API_KEY` is required only when using `hinglish` transcription.
- `WHISPER_MODEL` defaults to `small` if not set.

## Usage

### 1. CLI mode

Run the main pipeline from `main.py`:

```bash
python main.py
```

The CLI prompts for a YouTube URL or a local file path and a language option.

Supported language values:

- `english` — local Whisper transcription
- `hinglish` — Sarvam transcription with Hinglish support

### 2. Streamlit UI

Start the app with:

```bash
streamlit run streamlit_app.py
```

Open the local Streamlit URL shown in the terminal to use the web interface.

### 3. Test script

Run `test.py` to execute the pipeline against a sample YouTube URL:

```bash
python test.py
```

## Project Structure

- `main.py` — CLI entry point and pipeline orchestration
- `streamlit_app.py` — Streamlit user interface
- `core/` — main AI pipeline components
  - `audio_processor.py` — download/convert audio and chunk WAV files
  - `transcriber.py` — Whisper and Sarvam transcription logic
  - `summarizer.py` — transcript summarization and title generation
  - `extractor.py` — action items, decisions, and question extraction
  - `rag_engine.py` — retrieval-augmented generation chain and question answering
  - `vector_store.py` — embeddings and Chroma vector store persistence
- `utils/` — supporting utilities
  - `audio_processor.py` — audio download/conversion and chunking functions

## Notes

- `yt-dlp` is used for downloading audio from YouTube links.
- `whisper` requires a local model and may use significant disk space.
- `pydub` and `ffmpeg` are required for audio format conversion and chunking.
- The RAG chat uses Chroma and Hugging Face embeddings for retrieval.

## License

Use and modify this project freely.
