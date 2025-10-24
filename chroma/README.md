# Chroma Microservice

This microservice provides chroma and pitch analysis using enhanced chroma and librosa analyzers.

What this microservice provides
- Chroma feature extraction and pitch analysis endpoints

Contract (quick)
- Inputs: audio file path or URL
- Outputs: JSON with chroma matrix, timestamps, and confidence metrics
- Error modes: unsupported sample rate, corrupted audio

Quickstart (developer)
1. From this folder create a Python venv and install dependencies (librosa, numpy, etc.).
2. Run the analyzer (example):

```powershell
python enhanced_chroma_analyzer.py --port 7002
```

3. POST audio or URL to `/analyze` and inspect the JSON response.

See `ENHANCED_CHROMA_ANALYSIS.md` for extended usage and tuning options.
