# Demucs Microservice

This microservice provides audio source separation using the Demucs model.

What this microservice provides
- Source separation endpoints (vocals / instrumental)
- Optional variations (fast / high-quality) exposed by the wrapper

Contract (quick)
- Inputs: audio file URL or `file_id` (presigned GET URL accepted)
- Outputs: JSON with `job_id`, `status`, and `outputs[]` (paths/URLs)
- Error modes: invalid file, missing model weights, OOM

Quickstart (developer)
1. Create a Python venv and install requirements for this service (see `requirements.txt` in this folder if present).
2. Run the wrapper as a dev server (example):

```powershell
python demucs_wrapper.py --port 7001
```

3. Call the job endpoint (example): send POST /jobs with `{ "file_url": "https://...", "variation": "default", "callback_url": "http://orchestrator/notify" }`.

Tests & validation
- There is a small smoke test in `tests/` (if present) — run `pytest tests -q` inside this folder.

See `MULTI_MODEL_GUIDE.md` for full environment and model-weight instructions.
