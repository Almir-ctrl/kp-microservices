# Gemma 3N Microservice

This microservice provides audio analysis and transcription using the Gemma 3N model.

What this microservice provides
- Speech/transcription endpoints, word-level timing for karaoke sync, and metadata outputs

Contract (quick)
- Inputs: audio file URL or `file_id`
- Outputs: transcription JSON with segments, timestamps, and confidence; optional `lrc`/karaoke files
- Error modes: missing audio, language mismatch, model resource errors

Quickstart (developer)
1. Create a Python venv and install dependencies described in `GEMMA_DEPENDENCIES.md`.
2. Start the service (example):

```powershell
python gemma_service.py --port 7004
```

3. POST to `/transcribe` with `{ "file_url": "https://..." }` and check callback or `GET /status/{job_id}`.

See `GEMMA_SETUP_GUIDE.md` and `GEMMA_3N_MIGRATION.md` for setup, migrations and tuning.
