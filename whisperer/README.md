# Whisperer Microservice

This microservice provides speech transcription using the Whisper model.

What this microservice provides
- Speech-to-text endpoints and optional timestamped segments for karaoke

Contract (quick)
- Inputs: audio file URL or `file_id`
- Outputs: `transcription_base.txt` / JSON with segments, `job_id`, and `outputs[]`
- Error modes: unsupported codec, missing model files

Quickstart (developer)
1. Create a Python venv and install the Whisper-compatible dependencies (see `WHISPER_GUIDE.md`).
2. Start the service (example):

```powershell
python whisper_service.py --port 7005
```

3. POST audio to `/transcribe` and either poll `GET /status/{job_id}` or provide a `callback_url`.

See `WHISPER_GUIDE.md` and `WHISPER_KARAOKE_FIX.md` for karaoke-specific steps and troubleshooting.
