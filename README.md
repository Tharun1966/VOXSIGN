# VoxSign — inclusive sign communication (demo)

College-project–friendly full stack that demonstrates **four interaction paths**:

| Flow | What happens |
|------|----------------|
| **Sign → Text** | Webcam + MediaPipe Hands → landmarks → Flask classifier → text |
| **Sign → Voice** | Same recognized text → browser speech **or** server MP3 (gTTS) |
| **Text → Sign** | Typed text → word Lottie animations, or **letter tiles** if no word asset |
| **Voice → Sign** | Microphone → text (browser Web Speech API and/or Flask STT) → same sign pipeline |

> **Demo honesty:** Hand poses for **a–z** are **synthetic training data** for the RandomForest model, not certified ASL fingerspelling. **16** vocabulary glosses use word-level Lottie clips in `frontend/public/animations/` (see `WORD_TO_ANIMATION` in `backend/services/text_to_sign_service.py`). Extend with real recordings for serious research.

---

## Folder structure

```
VOXSIGN/
  frontend/           # React + Vite (3 panels: sign / text / voice)
  backend/              # Flask API + services (STT, TTS, text-to-sign)
    services/           # Modular: text_to_sign, stt, tts
    api_routes.py       # Route registration
    models/             # gesture_model.joblib (train via ml-model/)
  ml-model/             # train_gesture_model.py
  animations/           # Source Lottie JSON (copy into frontend/public/animations)
  audio/                # Optional project audio assets (TTS is in-memory or temp)
  future/               # Roadmap notes (NLP, mobile, i18n)
```

---

## Prerequisites

- **Node.js** 18+
- **Python** 3.10+
- **Internet** for Google Speech Recognition, gTTS, and (optional) browser speech APIs
- **ffmpeg** on `PATH` (recommended) — used by **pydub** to decode WebM from the “Record → server STT” button. Without it, use **Listen (browser)** for voice input.

---

## Setup (local)

### 1) Backend (terminal A)

```powershell
cd backend
python -m venv .venv
.\.venv\Scripts\activate
pip install -r requirements.txt
python app.py
```

API base: `http://127.0.0.1:5000`

Health check:

```powershell
curl http://127.0.0.1:5000/api/health
```

### 2) Retrain gestures (optional but recommended after pulling code)

```powershell
.\.venv\Scripts\python.exe ..\ml-model\train_gesture_model.py
```

Writes `backend/models/gesture_model.joblib` (words + **a–z** classes).

### 3) Frontend (terminal B)

```powershell
cd frontend
npm install
npm run dev
```

Open the printed URL (usually `http://localhost:5173`). Vite proxies `/api/*` to Flask.

---

## API overview

| Method | Path | Purpose |
|--------|------|---------|
| `GET` | `/api/health` | Status + `classifier`: `ml` or `rules` |
| `POST` | `/api/predict-sign` | Body `{ "landmarks": [[x,y,z]×21] }` → sign recognition |
| `POST` | `/api/predict` | Same as `predict-sign` (alias) |
| `POST` | `/api/text-to-sign` | Body `{ "text": "hello xyz" }` → `{ "segments": [...] }` |
| `POST` | `/api/speech-to-text` | Multipart `audio` file + optional `format` (e.g. `webm`) |
| `POST` | `/api/text-to-speech` | Body `{ "text": "..." }` → MP3 bytes (gTTS) |
| `GET` | `/api/session` | Cookie session stats |

---

## UI panels

1. **Sign detection** — Start camera, show landmarks, recognition + **Speak (browser)** / **Speak (server gTTS)**.
2. **Text input** — Type a message → **Show sign animations** (words + letter fallback).
3. **Voice input** — **Listen (browser)** or **Record → server STT** → edit transcript → **Show sign animations**.

---

## Dataset ideas (for a real project)

1. **Record MediaPipe landmarks** (CSV/JSON) per class: static words + fingerspelling **from a fluent signer**, not synthetic poses.
2. **300–500+ samples per class** minimum for robust sklearn or small neural nets; include lighting and background variety.
3. **Label** with gloss text (`hello`, `thanks`, …) and store **train/val/test** splits.
4. **Export** the same 63-d feature vector as `backend/landmark_features.py` so training matches inference.
5. **Ethics:** consent, anonymize video if publishing, cite sign language variant (ASL, BSL, ISL, …).

---

## Troubleshooting

| Issue | What to try |
|-------|-------------|
| Webcam blocked | Browser site permissions → allow camera |
| `/api/...` fails | Ensure Flask is running on port 5000 |
| Server STT fails on WebM | Install **ffmpeg**; or use **Listen (browser)** |
| gTTS errors | Check internet; use **Speak (browser)** |
| Gesture unstable | Improve lighting; retrain with more realistic data |

---

## Future scope

See `future/FUTURE_SCOPE.md` (sentence NLP, mobile, multi-language).
