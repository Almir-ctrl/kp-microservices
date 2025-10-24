# Advanced Model Selection Feature

## Overview

The Advanced Model Selection feature allows users to choose different AI models and their variations for audio processing tasks in Frontend. This gives users fine-grained control over the trade-off between processing speed and output quality.

## Supported Models

### 1. Demucs (Music Source Separation)

Demucs is used for two tasks:
- **Vocal Separation**: Extract vocals and create an instrumental version
- **Stem Separation**: Separate audio into 4 tracks (vocals, drums, bass, other)

#### Available Variations:

| Variation | Name | Quality | Speed | Use Case |
|-----------|------|---------|-------|----------|
| `demucs` | Conv Demucs (Fast) | Medium | Fast | Quick tests |
| `htdemucs` | HT Demucs (Default) | High | Slow | Most use cases |
| `htdemucs_ft` | HT Demucs FT (Fine-tuned) | Highest | Slow | Best quality |
| `htdemucs_6s` | HT Demucs 6s | High | Slow | Balanced window |
| `tasnet` | TasNet (Lightweight) | Medium | Fastest | Resource-limited |
| `tasnet_extra` | TasNet Extra (Enhanced) | Med-High | Fast | Better TasNet |

### 2. Whisper (Speech Recognition & Transcription)

Whisper is used for:
- **Song Transcription**: Automatically transcribe lyrics with timestamps

#### Available Variations:

| Variation | Name | Quality | Speed | Memory | Parameters | Languages |
|-----------|------|---------|-------|--------|------------|-----------|
| `tiny` | Tiny | Low | Fastest | Low | ~39M | Multilingual |
| `tiny.en` | Tiny English | Low | Fastest | Low | ~39M | English only |
| `base` | Base (Default) | Medium | Fast | Low | ~74M | Multilingual |
| `base.en` | Base English | Medium | Fast | Low | ~74M | English only |
| `small` | Small | High | Medium | Medium | ~244M | Multilingual |
| `small.en` | Small English | High | Medium | Medium | ~244M | English only |
| `medium` | Medium | Very High | Slow | Medium | ~769M | Multilingual |
| `medium.en` | Medium English | Very High | Slow | Medium | ~769M | English only |
| `large` | Large (Highest) | Highest | Slowest | High | ~1550M | Multilingual |

**Note:** `.en` variants are optimized for English-language audio only and run faster than multilingual variants.

## Frontend Implementation

### Components

#### 1. **AdvancedToolsWindow.tsx** (Updated)
The main window component that displays the advanced tools interface.

**Changes:**
- Added state management for model and variation selection
- Integrated `ModelSelectorModal` component
- Passes model and variation to `processSongWithAdvancedTool`

**Key Functions:**
```typescript
handleVocalIsolation()        // Opens modal for vocal separation
handleStemSeparation()        // Opens modal for stem separation  
handleTranscription()         // Opens modal for transcription
handleVariationSelected()     // Called when user confirms a variation
```

#### 2. **ModelSelectorModal.tsx** (New)
A modal dialog that displays available model variations for selection.

**Features:**
- Fetches available models from `/models` endpoint
- Displays model variations with detailed specs
- Shows quality, speed, and memory metrics
- Radio button selection interface
- Confirm/Cancel actions

**Props:**
```typescript
interface ModelSelectorModalProps {
  isOpen: boolean;              // Whether modal is visible
  modelName: string;            // 'demucs' or 'whisper'
  onSelectVariation: (variation: string) => void;
  onCancel: () => void;
  isLoading?: boolean;          // Show loading state
}
```

### Workflow

1. User clicks a tool button (Vocal Separation, Stem Separation, or Transcribe)
2. `ModelSelectorModal` opens showing available variations
3. User selects a variation and clicks Confirm
4. Modal passes selected variation to `processSongWithAdvancedTool`
5. Processing begins with the specified model and variation

## Backend Implementation

### New API Endpoints

#### 1. **GET /models**
Returns all available models and their variations.

**Response:**
```json
{
  "models": {
    "demucs": {
      "name": "Demucs - Stem Separation",
      "description": "...",
      "variations": {
        "htdemucs": {
          "name": "HT Demucs (Default)",
          "description": "...",
          "quality": "high",
          "speed": "slow"
        },
        // ... other variations
      },
      "default_variation": "htdemucs"
    },
    "whisper": {
      // ... similar structure
    }
  }
}
```

#### 2. **GET /models/<model_name>**
Returns details for a specific model.

**Parameters:**
- `model_name`: 'demucs' or 'whisper'

**Response:**
```json
{
  "model": "demucs",
  "details": {
    "name": "Demucs - Stem Separation",
    "description": "...",
    "variations": { ... },
    "default_variation": "htdemucs"
  }
}
```

#### 3. **POST /process/<model>/<file_id>** (Updated)
Starts processing with optional model variation.

**Request Body (Optional):**
```json
{
  "variation": "htdemucs_ft"  // Specify model variation
}
```

**Response:**
```json
{
  "message": "Processing started with demucs",
  "file_id": "uuid-string",
  "status": "queued",
  "model": "demucs",
  "variation": "htdemucs_ft"
}
```

### Updated Processing Functions

#### 1. **process_audio_async()**
```python
def process_audio_async(
    file_path: str,
    file_id: str,
    model: str,
    variation: str = None
):
    """Process audio file asynchronously"""
    # Uses default variation if not specified
    if variation is None:
        variation = AVAILABLE_MODELS[model]['default_variation']
    # ... routes to appropriate processor
```

#### 2. **process_with_demucs()**
```python
def process_with_demucs(
    file_path: str,
    file_id: str,
    variation: str = 'htdemucs'
) -> bool:
    """Process audio file with Demucs for stem separation"""
    # Uses specified Demucs variation
    model = demucs.api.load_model(variation)
    # ...
```

#### 3. **process_with_whisper()**
```python
def process_with_whisper(
    file_path: str,
    file_id: str,
    variation: str = 'base'
) -> bool:
    """Process audio file with Whisper for transcription"""
    # Uses specified Whisper model
    model = whisper.load_model(variation)
    # ...
```

### Model Configuration

The `AVAILABLE_MODELS` dictionary in `main.py` defines all available models and variations:

```python
AVAILABLE_MODELS = {
    'demucs': {
        'name': 'Demucs - Stem Separation',
        'description': '...',
        'variations': {
            'htdemucs': { ... },
            'htdemucs_ft': { ... },
            'demucs': { ... }
        },
        'default_variation': 'htdemucs'
    },
    'whisper': {
        'name': 'Whisper - Speech Recognition & Transcription',
        'description': '...',
        'variations': {
            'tiny': { ... },
            'base': { ... },
            'small': { ... },
            'medium': { ... },
            'large': { ... }
        },
        'default_variation': 'base'
    }
}
```

## Usage Examples

### Frontend Example

```typescript
// Selecting a variation and starting processing
const handleVocalIsolation = () => {
  if (currentSong) {
    setPendingTool('vocals');
    setSelectedModel('demucs');
    setModelSelectorOpen(true);
  }
};

// When user confirms selection:
const handleVariationSelected = (variation: string) => {
  if (currentSong && pendingTool) {
    processSongWithAdvancedTool(
      currentSong.id,
      pendingTool,
      'demucs',
      variation  // e.g., 'htdemucs_ft'
    );
  }
};
```

### Backend Example

```python
# User selects 'large' Whisper model for transcription
# Request: POST /process/whisper/file-id
# Body: { "variation": "large" }

# The backend:
# 1. Validates variation: validate_model_and_variation('whisper', 'large')
# 2. Calls process_audio_async with variation parameter
# 3. process_with_whisper loads the large model:
#    model = whisper.load_model('large')
# 4. Processes and returns results
```

## UI/UX Flow

```
User clicks "Transcribe Lyrics"
         ↓
Model Selector Modal Opens
         ↓
User sees available Whisper variations:
  - Tiny (Fastest)
  - Base (Default)
  - Small (Good Balance)
  - Medium (High Quality)
  - Large (Highest Quality)
         ↓
User selects "Large (Highest Quality)"
         ↓
User clicks "Confirm"
         ↓
Modal closes, processing begins
         ↓
Progress notification shown
         ↓
Results returned when complete
```

## Performance Considerations

### Demucs Models

- **htdemucs**: 2-5 minutes for 3-minute song (GPU), 15-30 minutes (CPU)
- **htdemucs_ft**: 2-5 minutes for 3-minute song (GPU), 15-30 minutes (CPU)
- **demucs**: 1-2 minutes for 3-minute song (GPU), 5-10 minutes (CPU)

### Whisper Models

- **tiny**: 30-60 seconds for 3-minute audio
- **base**: 1-2 minutes for 3-minute audio
- **small**: 3-5 minutes for 3-minute audio
- **medium**: 8-12 minutes for 3-minute audio
- **large**: 15-25 minutes for 3-minute audio

*Times vary based on CPU/GPU availability and audio complexity*

## Error Handling

The system includes comprehensive error handling:

1. **Model Not Found**: Returns 404 if model/variation doesn't exist
2. **Invalid Variation**: Returns 400 with descriptive message
3. **Server Connection**: Displays user-friendly error messages
4. **Processing Errors**: Caught and reported with context

## Future Enhancements

Potential improvements for future versions:

- [ ] Model download management UI
- [ ] Processing time estimates
- [ ] Batch processing with different models
- [ ] Model performance metrics dashboard
- [ ] Custom model support
- [ ] GPU/CPU preference selection
- [ ] Preset configurations for common use cases
- [ ] Model comparison tool

## Testing

### Manual Testing Steps

1. **Test Model Fetch**:
   - Open Advanced Tools window
   - Open any model selector modal
   - Verify models load correctly

2. **Test Variation Selection**:
   - Select different variations
   - Verify UI updates correctly
   - Check that selection is remembered

3. **Test Processing**:
   - Select a variation and confirm
   - Monitor processing progress
   - Verify correct model/variation is used in backend

4. **Test Edge Cases**:
   - Network timeout during model fetch
   - Invalid model/variation combinations
   - Rapid modal open/close cycles

## Documentation

- **Backend README**: Updated with new endpoints
- **Frontend Components**: Inline JSDoc comments
- **Type Definitions**: Comprehensive TypeScript interfaces

## References

- [Demucs GitHub](https://github.com/facebookresearch/demucs)
- [Whisper GitHub](https://github.com/openai/whisper)
- [Demucs Models](https://github.com/facebookresearch/demucs#pretrained-models)
- [Whisper Model Card](https://huggingface.co/openai/whisper-base)
