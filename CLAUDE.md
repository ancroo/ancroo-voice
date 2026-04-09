# ancroo-voice — Desktop STT Client

**Language:** Python 3.11 (CustomTkinter GUI)
**License:** MIT
**Distribution:** PyInstaller binary (Linux AppImage, Windows .exe)
**Platform:** Linux + Windows

## Key Files

| File | Purpose |
|------|---------|
| `main.py` | Entry point (config loading, GUI init) |
| `ancroo_voice/core.py` | Core orchestrator (recording, hotkeys, STT) |
| `ancroo_voice/audio_recorder.py` | Audio capture via sounddevice (16kHz WAV) |
| `ancroo_voice/hotkey_manager.py` | Global hotkey detection via pynput |
| `ancroo_voice/text_inserter.py` | Platform-specific text insertion (xdotool/pynput) |
| `ancroo_voice/constants.py` | Audio config, language mappings, key mappings |
| `ancroo_voice/gui/main_window.py` | Main GUI window (device, language, hotkey, controls) |
| `ancroo_voice/gui/config_manager.py` | Load/save GUI settings (JSON) |
| `ancroo_voice/gui/device_manager.py` | Audio device enumeration (PulseAudio/sounddevice) |
| `ancroo_voice/gui/dialogs.py` | About dialog |
| `providers/ancroo_backend.py` | HTTP client for backend STT endpoint |

## Architecture

1. User presses hotkey (or on-screen button on Wayland)
2. `AudioRecorder` captures audio via sounddevice (16kHz, 16-bit mono WAV)
3. `AncrooBackendProvider` sends WAV to backend `POST /api/v1/transcribe`
4. Transcribed text returned, `TextInserter` types it at cursor position

**Thread model:** Main GUI thread + background recording/processing threads.

## Cross-Repo Interfaces

**Calls ancroo-backend:**
- `POST /api/v1/transcribe` — Audio transcription (WAV + language)
- `GET /health` — Health check
- Auth: Optional Bearer token (`ANCROO_BACKEND_API_KEY`)

**No direct dependency on:** ancroo-web, ancroo-runner, ancroo-stack

## Configuration

`.env` (Linux) or `ancroo-voice.ini` (Windows):
```
ANCROO_BACKEND_ENDPOINT=http://localhost:8900/api/v1/transcribe
ANCROO_BACKEND_API_KEY=optional_token
ANCROO_BACKEND_VERIFY_SSL=false
```

GUI settings saved as `ancroo-voice_config.json`.

## Platform Notes

| | Linux | Windows |
|---|-------|---------|
| Hotkeys | pynput (X11 only) | pynput (native) |
| Text insertion | xdotool | pynput keyboard |
| Audio devices | PulseAudio/PipeWire + sounddevice | sounddevice/PortAudio |
| Wayland | No hotkeys, use on-screen button | N/A |
| Build | AppImage | .exe (PyInstaller) |

## Development

```bash
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
python main.py
```

## CI/CD

`.github/workflows/build-releases.yml` — Triggered on `v*` tags, builds Linux AppImage + Windows .exe, creates GitHub Release.
