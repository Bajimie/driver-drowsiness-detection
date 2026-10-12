# Real-Time Driver Drowsiness Detection

**Computer vision • Python • MediaPipe • OpenCV • OpenAI text-to-speech**

A webcam-based prototype that monitors facial landmarks to detect sustained eye closure and head drop, classifies alertness into five states, and plays pre-generated voice warnings. It also logs frame-level measurements, state transitions, screenshots and session summaries for analysis.

> **Research prototype only:** Not validated for road use or suitable as a substitute for safe driving, breaks or certified driver-monitoring systems.

## How it works

1. OpenCV captures webcam frames (requested resolution: 640 × 480).
2. MediaPipe FaceLandmarker estimates facial landmarks.
3. Eye Aspect Ratio (EAR) is computed from six landmarks per eye; a geometric head-pitch proxy is computed from nose, chin and forehead landmarks. Both are smoothed using 10-frame rolling windows.
4. A threshold-based state machine emits `ALERT`, `WARNING_EYE`, `WARNING_HEAD`, `WARNING_COMBINED`, or `CRITICAL`.
5. Cached MP3 warnings are played locally through pygame. Frame/event CSV logs, session JSON, screenshots and diagnostic charts are written to a timestamped session folder.

## Prototype thresholds

| Setting | Value |
|---|---:|
| EAR threshold | 0.22 |
| Head pitch threshold | 15° |
| Warning persistence | 20 frames |
| Critical persistence | 40 frames |
| Smoothing history | 10 frames |

Thresholds are heuristic and were not validated across a diverse population. Actual delay depends on measured frame rate.

## Setup

Use Python 3.11 with webcam and speaker access. Install dependencies in a virtual environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Create a **new** OpenAI API key, then set it in the terminal (never paste it into source code):

```bash
export OPENAI_API_KEY="your-new-key"
python src/drowsiness_detect_tts_logged.py
```

The script downloads the MediaPipe `face_landmarker.task` model on first run. On startup it creates the voice alert MP3 files using OpenAI TTS (`tts-1`, `nova`) if they are not already cached; this requires API access and may incur charges. If the key is not set, the program can only play previously cached MP3 alerts. Press **Q** to exit.

Run from the repository root so the downloaded model, cache and session folders use the expected relative paths.

## Session outputs

Each run creates `session_YYYYMMDD_HHMMSS/` containing `frame_log.csv`, `event_log.csv`, `session_summary.json`, screenshots of state changes, and plots of EAR/head pitch over time. These outputs and cached audio are excluded from Git.

## Research results and limitations

The accompanying research paper reports a **single controlled 40-second session**, 1,184 frames at approximately 29.6 FPS, with warning and critical transition latencies near 611 ms and 1,301 ms respectively. These are limited prototype observations, **not** validated diagnostic accuracy or proof of reliability in real vehicles.

Known limitations include single-participant calibration, sensitivity to lighting/occlusion/head orientation, fixed thresholds, simplified head-pitch estimation, and no external benchmark or diverse-user validation. The state machine is rule-based; MediaPipe provides the learned facial landmark detector. OpenAI TTS generates voice recordings but does not classify drowsiness.

## Security and privacy

Never commit an API key or `.env` file. The camera runs locally; OpenAI receives the predefined alert text when audio is generated, not webcam frames in this implementation. Session screenshots may contain identifiable faces, so they are excluded from Git by default.

## Project files

- `src/drowsiness_detect_tts_logged.py` — application
- `requirements.txt` — Python dependencies
- `reports/research_paper.docx` — research write-up
- `.env.example` — example configuration (do not place real credentials in it)

## Portfolio notes

This repository version removes a hardcoded credential and corrects voice-alert selection on state transitions. The project has been syntax-checked, but webcam and API integration have **not** been re-tested in this packaging environment.
