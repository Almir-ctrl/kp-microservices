# MusicGen Microservice

This microservice provides music generation using the MusicGen model.

What this microservice provides
- Model endpoint to generate short musical pieces from prompts / seed audio

Contract (quick)
- Inputs: prompt text, optional seed audio URL, generation params (length, style)
- Outputs: `job_id`, `status`, and generated audio `outputs[]` with URLs
- Error modes: missing prompt, model not loaded, OOM

Quickstart (developer)
1. Create a Python venv and install required packages (see `MUSICGEN_GUIDE.md`).
2. Start the service (example):

```powershell
python -m musicgen.server --port 7003
```

3. POST generation request to `/generate` with a JSON body containing `prompt` and optional params.

See `MUSICGEN_GUIDE.md` for environment and model-specific setup steps.
