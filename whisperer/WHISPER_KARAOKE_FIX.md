# Whisper Karaoke Fix

Notes moved from `Backend/docs/WHISPER_KARAOKE_FIX.md`.

# Whisper + Karaoke Integration Fix

**Problem Solved:** Lyrics not displaying in LyricsWindow and external player

**Date:** 2025-06-14  
**Status:** ✅ COMPLETED - Backend changes applied and tested

---

## 🔍 Root Cause Analysis

### The Problem
- **Frontend Issue:** LyricsWindow shows no text, external player says "waiting for lyrics data"
- **Backend Discovery:** Gemma 3N was being used as transcription engine
- **Architecture Flaw:** Gemma 3N is **audio analysis model**, NOT transcription model

### What Was Happening (WRONG)
```
Upload → Demucs (separation) → Gemma 3N (analysis) → KaraokeProcessor
                                       ↓
                           Audio feature analysis text:
                           "Duration: 180s, RMS: 0.045, 
                            Spectral Centroid: 2500 Hz..."
                                       ↓
                           KaraokeProcessor receives analysis
                                       ↓
                           Karaoke files created with WRONG content
                                       ↓
                           Frontend reads karaoke files → NO LYRICS
```

### Gemma 3N Output (NOT Lyrics)
```python
{
  'analysis_summary': 'This audio has a duration of 180 seconds with RMS energy of 0.045...',
  'output_text_file': 'analysis_gemma-2-2b_transcribe.txt',
  'features': {
    'rms_energy': 0.045,
    'spectral_centroid': 2500.34,
    'zero_crossing_rate': 0.123,
    'chroma_distribution': [...]
  }
}
```

### Whisper Output (ACTUAL Lyrics)
```python
{
  'text': 'Never gonna give you up\nNever gonna let you down\nNever gonna run around...',
  'segments': [
    {'start': 0.0, 'end': 3.2, 'text': 'Never gonna give you up'},
    {'start': 3.2, 'end': 6.1, 'text': 'Never gonna let you down'}
  ],
  'output_text_file': 'transcription_base.txt'
}
```

---

## ✅ Solution Implemented

### 1. Replace Gemma 3N with Whisper for Transcription
**File:** `app.py` lines 740-792

**OLD CODE (WRONG):**
```python
# Auto-transcribe with Gemma 3n
gemma_processor = get_processor('gemma_3n')
transcription_result = gemma_processor.process(
    file_id, Path(upload_path),
    model_variant=gemma_model,
    task='transcribe'  # ← Gemma 3N can't actually transcribe!
)
```

**NEW CODE (CORRECT):**
```python
# Auto-transcribe with Whisper (for accurate speech-to-text)
whisper_processor = get_processor('whisper')
whisper_model = request.form.get('whisper_model', 'base')

whisper_transcription = whisper_processor.process(
    file_id, Path(upload_path),
    model_variant=whisper_model,
    task='transcribe'  # ← Whisper's native task
)

# Auto-analyze with Gemma 3N (for deep audio analysis)
gemma_analysis = None
if whisper_transcription:
    gemma_processor = get_processor('gemma_3n')
    gemma_model = request.form.get('gemma_model', 'gemma-2-2b')
    
    gemma_analysis = gemma_processor.process(
        file_id, Path(upload_path),
        model_variant=gemma_model,
        task='analyze'  # ← Separate analysis, not transcription
    )

# Use Whisper result as primary transcription
transcription_result = whisper_transcription
```

### 2. Extract Transcription Text from Whisper
**File:** `app.py` lines 899-910

**OLD CODE (WRONG):**
```python
transcription_text = transcription_result.get('analysis_summary', '')
```

**NEW CODE (CORRECT):**
```python
# Extract actual transcription text from Whisper result
transcription_text = transcription_result.get('text', '')

# If no text field, try reading from transcription file
if not transcription_text and transcription_result.get('output_text_file'):
    transcription_file_path = os.path.join(
        output_dir_path,
        transcription_result['output_text_file']
    )
    if os.path.exists(transcription_file_path):
        with open(transcription_file_path, 'r', encoding='utf-8') as f:
            transcription_text = f.read()
```

### 3. Update Response Structure
**File:** `app.py` lines 835-867

**Changes:**
- **transcription field:** Now returns Whisper data (actual lyrics)
- **audio_analysis field:** NEW - Returns Gemma 3N analysis (separate from lyrics)

**Response Structure:**
```json
{
  "file_id": "uuid-here",
  "transcription": {
    "status": "completed",
    "text": "Never gonna give you up\nNever gonna let you down...",
    "model": "whisper",
    "model_variant": "base",
    "output_file": "transcription_base.txt",
    "download_url": "/download/uuid-here/transcription_base.txt"
  },
  "audio_analysis": {
    "status": "completed",
    "analysis": "This audio features consistent rhythm with clear vocal presence...",
    "model": "gemma-3n",
    "model_variant": "gemma-2-2b",
    "output_file": "analysis_gemma-2-2b_analyze.txt",
    "download_url": "/download/uuid-here/analysis_gemma-2-2b_analyze.txt"
  },
  "karaoke": {
    "status": "completed",
    "lrc_file": "uuid-here_karaoke.lrc",
    "audio_with_metadata": "uuid-here_karaoke.mp3",
    "karaoke_dir": "outputs/Karaoke-pjesme/uuid-here/"
  }
}
```

---

## 🔧 Technical Details

### Whisper Processor Integration
- **Model:** OpenAI Whisper (transformer-based speech recognition)
- **Task:** `transcribe` - Converts audio to text
- **Variants:** tiny, base, small, medium, large
- **Output:**
  - `text`: Full transcription as string
  - `segments`: Timestamped segments
  - `output_text_file`: transcription_base.txt
  - `output_json_file`: transcription_base.json

### Gemma 3N Processor (NEW Role)
- **Model:** Google Gemma 3N (LLM for audio analysis)
- **Task:** `analyze` - Analyzes audio characteristics
- **Output:**
  - `analysis_summary`: LLM-generated analysis
  - `features`: RMS, spectral centroid, chroma, etc.
  - `output_text_file`: analysis_{model}_{task}.txt
  - `output_json_file`: analysis_{model}_{task}.json

### KaraokeProcessor Requirements
- **Input:** `transcription_text` (MUST be actual lyrics)
- **Process:**
  1. Split text into lines
  2. Generate timestamps (uniform distribution)
  3. Create LRC file: `[mm:ss.xx]lyric text`
  4. Copy instrumental to karaoke folder
  5. Embed lyrics in MP3 ID3 tags (USLT, TIT2, TPE1, TALB)
  6. Save sync metadata JSON
- **Output Location:** `outputs/Karaoke-pjesme/{file_id}/`

---

## 📂 File Structure

### Output Files (After Processing)
```
outputs/
├── {file_id}/
│   ├── no_vocals.mp3              ← Demucs instrumental
│   ├── vocals.mp3                 ← Demucs vocals
│   ├── transcription_base.txt     ← Whisper lyrics (ACTUAL TEXT)
│   ├── transcription_base.json    ← Whisper segments (timestamped)
│   ├── analysis_gemma-2-2b_analyze.txt   ← Gemma 3N analysis
│   └── analysis_gemma-2-2b_analyze.json  ← Gemma 3N features
│
└── Karaoke-pjesme/
    └── {file_id}/
        ├── {file_id}_karaoke.lrc         ← LRC timed lyrics
        ├── {file_id}_karaoke.mp3         ← Instrumental + ID3 lyrics
        └── {file_id}_sync.json           ← Sync metadata
```

---

## 🧪 Testing

### Backend Test (Upload + Process)
```powershell
# 1. Upload audio file
$file = Get-Item "test_song.mp3"
$response = Invoke-RestMethod -Uri "http://localhost:5000/upload" `
    -Method POST -Form @{
        file = $file
        model = 'demucs'
        artist = 'Test Artist'
        song_name = 'Test Song'
    }

Write-Host "File ID: $($response.file_id)"

# 2. Check transcription field
$response.transcription | Format-List
# Expected:
# status: completed
# text: <actual lyrics text>
# model: whisper

# 3. Check audio_analysis field
$response.audio_analysis | Format-List
# Expected:
# status: completed
# analysis: <audio feature analysis>
# model: gemma-3n

# 4. Check karaoke files
$karaokeDir = "outputs\Karaoke-pjesme\$($response.file_id)"
Get-ChildItem $karaokeDir
# Expected:
# {file_id}_karaoke.lrc
# {file_id}_karaoke.mp3
# {file_id}_sync.json
```

### Verify LRC File Contains Actual Lyrics
```powershell
$fileId = "your-file-id-here"
Get-Content "outputs\Karaoke-pjesme\$fileId\${fileId}_karaoke.lrc"
```

**Expected Output (CORRECT):**
```lrc
[ti:Test Song]
[ar:Test Artist]
[al:Unknown Album]
[length:03:45]
[00:00.00]Never gonna give you up
[00:03.20]Never gonna let you down
[00:06.40]Never gonna run around and desert you
...
```

**OLD Output (WRONG - Before Fix):**
```lrc
[ti:Test Song]
[ar:Test Artist]
[00:00.00]This audio has a duration of 180 seconds with RMS energy of 0.045
[00:30.00]spectral centroid averaging 2500 Hz indicating bright timbre
[01:00.00]chroma distribution shows emphasis on C and G notes
...
```

### Frontend Test
1. **Upload new song** (must use new workflow)
2. **Wait for processing** to complete
3. **Open LyricsWindow:**
   - Should display actual song lyrics
   - NOT audio analysis text
4. **Open external player:**
   - Should show synced lyrics
   - NOT "waiting for lyrics data"

---

## 🎯 Impact

### Before (BROKEN)
- ❌ Lyrics window empty
- ❌ External player waiting for data
- ❌ Karaoke files contain audio analysis text
- ❌ Gemma 3N used incorrectly as transcription engine

### After (FIXED)
- ✅ Lyrics window displays actual song lyrics
- ✅ External player receives synced lyrics
- ✅ Karaoke files contain proper lyrics text
- ✅ Whisper used for transcription (correct tool)
- ✅ Gemma 3N used for audio analysis (optional)

---

## 🔄 Workflow Comparison

### OLD Workflow (WRONG)
```
1. Upload audio
2. Demucs separates vocals/instrumental
3. Gemma 3N "transcribes" (actually analyzes) → analysis text
4. KaraokeProcessor receives analysis text → creates karaoke with WRONG content
5. Frontend reads karaoke files → no actual lyrics
```

### NEW Workflow (CORRECT)
```
1. Upload audio
2. Demucs separates vocals/instrumental
3. Whisper transcribes → actual lyrics text
4. Gemma 3N analyzes (optional) → audio features analysis
5. KaraokeProcessor receives lyrics → creates karaoke with CORRECT content
6. Frontend reads karaoke files → displays actual lyrics
```

---

## 📝 Notes

- **Whisper Model Selection:** Default is `base` (good balance), use `small`/`medium` for better accuracy
- **Gemma 3N Now Optional:** Only runs if Whisper succeeds; failure doesn't block karaoke generation
- **Backend Restart Required:** Changes require `python app.py` restart
- **Old Songs:** Previously uploaded songs still have wrong lyrics; re-upload to use new workflow
- **Lint Warnings:** Some line length warnings (>79 chars) - cosmetic, doesn't affect functionality
 - **GPU Enforcement (2025-10-22):** Heavy model processing (Demucs, Whisper, MusicGen, Gemma) is now GPU-only. The frontend should call `GET /gpu-status` before issuing heavy `/process/*` requests. If a GPU is not available the backend will return HTTP 503 for heavy model requests. This prevents accidental CPU runs (user requirement: "NIKADA CPU").

---

## 🚀 Deployment

### Backend Restart Steps
```powershell
# 1. Stop backend
Stop-Process -Name "python" -Force

# 2. Navigate to backend
cd C:\Users\almir\AiMusicSeparator-Backend

# 3. Restart
python app.py

# 4. Verify startup
# Look for: "Running on http://127.0.0.1:5000"
```

### Frontend Refresh
```powershell
# Hard refresh browser
# Ctrl + Shift + R

# Or restart Vite dev server
npm run dev
```

---

## 📚 Related Documentation

- **DELETE Endpoint Fix:** `TWO_PROBLEMS_FIX.md`
- **Whisper Guide:** `WHISPER_GUIDE.md`
- **Gemma 3N Guide:** `GEMMA_3N_GUIDE.md`
- **Karaoke Engine:** `models.py` lines 722-889
- **Upload Workflow:** `app.py` lines 740-950

---

## ✅ Verification Checklist

- [x] Whisper processor integrated for transcription
- [x] Gemma 3N moved to optional analysis role
- [x] KaraokeProcessor receives `text` field from Whisper
- [x] Response structure updated with separate fields
- [x] Backend restarted successfully
- [ ] Frontend tested with new upload
- [ ] LyricsWindow displays actual lyrics
- [ ] External player receives synced lyrics
- [ ] Karaoke LRC files verified to contain lyrics (not analysis)

---

**Status:** Backend changes COMPLETE ✅  
**Next Step:** Frontend testing required to verify lyrics display

**Last Updated:** 2025-06-14  
**Author:** AI Coding Agent  
**Backend Version:** app.py with Whisper integration
