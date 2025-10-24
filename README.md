# Microservices Overview

This directory contains per-model microservices for the AiMusicSeparator project. Each subfolder is a standalone service for a specific model, with its own environment, dependencies, and documentation.

Why this layout
- Keeps model code, docs, and environment notes next to each microservice.
- Makes it easy to build, test, and deploy a single model service independently.

Structure

Each microservice lives in a subfolder named after the model. The typical contents are:
- `README.md` — quickstart, contract, and troubleshooting notes (this file added/standardized).
- model code and wrappers (example: `demucs_wrapper.py`).
- model-specific documentation (guides, migration notes, dependencies).

Moved documentation (by microservice)

- demucs/
	- MULTI_MODEL_GUIDE.md
	- demucs_wrapper.py

- chroma/
	- ENHANCED_CHROMA_ANALYSIS.md
	- enhanced_chroma_analyzer.py

- musicgen/
	- MUSICGEN_GUIDE.md

- gemma3n/
	- GEMMA_3N_GUIDE.md
	- GEMMA_3N_MIGRATION.md
	- GEMMA_DEPENDENCIES.md
	- GEMMA_SETUP_GUIDE.md
	- GEMMA_LYRICS_SYNC_COMPLETE.md

- whisperer/
	- WHISPER_GUIDE.md
	- WHISPER_KARAOKE_FIX.md

Quick validation steps (developer)

1. Open this repo in your editor and confirm each `Microservices/*/README.md` was updated.
2. Search for `Backend/docs/` references and confirm pointer files were left intentionally (archival).
3. Run a lightweight repo search to ensure there are no dangling links to removed filenames.

Example search (PowerShell):

```powershell
Select-String -Path . -Pattern 'Backend\\docs\\' -SimpleMatch -List
Select-String -Path . -Pattern 'Microservices\\' -SimpleMatch -List
```

How to add a new microservice

1. Create `Microservices/<name>/`.
2. Add `README.md` using the existing files as templates (quickstart, contract, run/test snippets).
3. Add model code and environment notes.
4. Update the top-level `Microservices/README.md` list above.

See each subfolder's README for usage and API details.
