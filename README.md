# SignMaster: ASL Alphabet Recognition

A web app for learning the American Sign Language (ASL) alphabet. You sign a letter in front of your webcam, and a machine-learning model checks whether you got it right.

Hand landmarks are extracted with [MediaPipe](https://developers.google.com/mediapipe), turned into geometric features (relative joint positions, fingertip distances, finger angles), and classified into letters A–Z by a scikit-learn model.

## Features

- **Live sign recognition**: the browser sends webcam frames to a Flask backend, which returns the predicted letter, its confidence, and the top 3 guesses.
- **Two learning modes**:
  - *Random*: work through the whole alphabet in random order.
  - *Practice*: pick any letter from the sidebar and practice it, with a reference image.
- **Signing and multiple-choice questions**, with hints and answer reveal.
- **XP and levels**: progress is saved in your browser (`localStorage`).
- **Desktop demo** (`cam.py`): an OpenCV window that shows predictions live, no browser needed.

## Project structure

```
.
├── server.py                            # Flask backend: POST /predict (port 5000)
├── cam.py                               # Standalone OpenCV webcam demo
├── asl_model_improved_continued.joblib  # Trained classifier (letters A–Z)
├── login.html                           # Login page (cosmetic, no real auth)
├── index.html                           # Main learning app
├── script.js                            # Frontend logic (quiz, webcam capture, XP)
├── style.css                            # Styles
├── images/                              # Reference images A.jpg–Z.jpg and logo
├── final hand/                          # Placeholder for Arduino servo code (empty for now)
├── requirements.txt                     # Pinned Python dependencies
└── HOW_TO_RUN.txt                       # Detailed run and troubleshooting notes
```

## Requirements

- **Python 3.11** (or older). The code uses MediaPipe's `mp.solutions` API, which only exists in older MediaPipe releases (0.10.9 here), and those don't support Python 3.13+.
- A webcam.
- A modern browser (Chrome, Firefox, or Edge).

## Setup

Clone the repo and create a virtual environment with Python 3.11.

**Linux / macOS** (with [uv](https://docs.astral.sh/uv/), which downloads Python 3.11 for you):

```bash
uv venv --python 3.11 .venv
uv pip install --python .venv/bin/python -r requirements.txt
```

**Windows** (with Python 3.11 from python.org):

```bat
py -3.11 -m venv .venv
.venv\Scripts\pip install -r requirements.txt
```

## Running

1. **Start the backend** (leave this terminal open):

   ```bash
   .venv/bin/python server.py        # Windows: .venv\Scripts\python server.py
   ```

   Wait for `Running on http://127.0.0.1:5000`.

2. **Serve the frontend** from a second terminal:

   ```bash
   python3 -m http.server 8000
   ```

3. Open **http://localhost:8000/login.html**, enter any username and password, and allow camera access.

### Desktop demo (optional)

```bash
.venv/bin/python cam.py
```

Press `q` to quit.

## API

`POST /predict`

Request:

```json
{ "image": "data:image/jpeg;base64,..." }
```

Response:

```json
{
  "success": true,
  "pred": "A",
  "confidence": 0.93,
  "top": [["A", 0.93], ["S", 0.04], ["E", 0.01]]
}
```

If no hand is found in the frame, the response has `"error": "no_hand_detected"` and an empty `pred`.

## Troubleshooting

| Problem | Fix |
| --- | --- |
| `module 'mediapipe' has no attribute 'solutions'` | You're using the system Python. Run with `.venv/bin/python`. |
| `InconsistentVersionWarning ... StandardScaler` | The model was saved with scikit-learn 1.7.2. Reinstall from `requirements.txt`. |
| `Address already in use` | Another copy is running on port 5000 or 8000. Stop it first. |
| No predictions in the website | Make sure `server.py` is running. The frontend calls `http://127.0.0.1:5000/predict`. |
| Camera won't open in `cam.py` | Close any browser tab that is already using the webcam. |

More detail is in [HOW_TO_RUN.txt](HOW_TO_RUN.txt).

## Tech stack

- **Backend**: Python, Flask, Flask-CORS
- **Vision / ML**: MediaPipe Hands, OpenCV, NumPy, scikit-learn, joblib
- **Frontend**: HTML, CSS, vanilla JavaScript, Font Awesome
