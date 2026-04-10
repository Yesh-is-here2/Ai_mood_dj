# AI Mood DJ 🎧
### Real-Time Emotion Detection → Music Recommendation

<div align="center">

![Python](https://img.shields.io/badge/Python-3.11+-blue?style=for-the-badge&logo=python&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-1.x-red?style=for-the-badge&logo=streamlit&logoColor=white)
![DeepFace](https://img.shields.io/badge/DeepFace-0.0.93-orange?style=for-the-badge)
![OpenCV](https://img.shields.io/badge/OpenCV-4.x-green?style=for-the-badge&logo=opencv&logoColor=white)
![Spotify](https://img.shields.io/badge/Spotify-API-1DB954?style=for-the-badge&logo=spotify&logoColor=white)

**A multimodal AI application that detects your facial emotion in real-time and recommends music that matches your mood**

[Demo](#demo) · [Architecture](#architecture) · [Setup](#setup)

</div>

---

## What is AI Mood DJ?

AI Mood DJ captures your face via webcam, runs deep learning facial emotion analysis using DeepFace, and queries the Spotify API to play music that matches your emotional state — all in real-time through a browser-based interface.

**Detected emotions:** 😄 Happy · 😢 Sad · 😠 Angry · 😲 Surprise · 😨 Fear · 😐 Neutral

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      AI MOOD DJ SYSTEM                          │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                  Streamlit Web App                       │   │
│  │                                                          │   │
│  │  [Start Camera Button]                                   │   │
│  │        │                                                 │   │
│  │        ▼                                                 │   │
│  │  ┌─────────────────────┐                                 │   │
│  │  │   WebRTC Stream     │  ← streamlit-webrtc             │   │
│  │  │                     │                                 │   │
│  │  │  VideoProcessor     │                                 │   │
│  │  │    .recv(frame)     │  ← called per frame             │   │
│  │  │    self.frame = img │  ← stores latest frame          │   │
│  │  └─────────────────────┘                                 │   │
│  │        │                                                 │   │
│  │  [Capture Image Button]                                  │   │
│  │        │                                                 │   │
│  │        ▼                                                 │   │
│  │  st.session_state["last_frame"] = frame                  │   │
│  │  st.rerun()  ← triggers re-execution                     │   │
│  │        │                                                 │   │
│  └────────┼─────────────────────────────────────────────────┘   │
│           │                                                     │
│           ▼                                                     │
│  ┌────────────────────┐                                         │
│  │  detect_emotion()  │                                         │
│  │                    │                                         │
│  │  DeepFace.analyze( │                                         │
│  │    frame,          │                                         │
│  │    actions=        │                                         │
│  │    ["emotion"],    │                                         │
│  │    enforce_        │                                         │
│  │    detection=False │                                         │
│  │  )                 │                                         │
│  │                    │                                         │
│  │  → dominant_emotion│                                         │
│  └────────┬───────────┘                                         │
│           │ "happy" / "sad" / "angry" / ...                     │
│           ▼                                                     │
│  ┌────────────────────┐                                         │
│  │  generate_music()  │                                         │
│  │                    │                                         │
│  │  Spotify REST API  │                                         │
│  │  GET /playlists/   │                                         │
│  │  {emotion_id}/     │                                         │
│  │  tracks            │                                         │
│  │                    │                                         │
│  │  random_track()    │                                         │
│  │  → (uri, art_url)  │                                         │
│  └────────┬───────────┘                                         │
│           │                                                     │
│           ▼                                                     │
│  ┌────────────────────────────────────────┐                     │
│  │           UI Display                   │                     │
│  │  • Captured image                      │                     │
│  │  • Emotion + emoji  (e.g. Happy 😄)   │                     │
│  │  • Album art                           │                     │
│  │  • [Open in Spotify] button            │                     │
│  └────────────────────────────────────────┘                     │
└─────────────────────────────────────────────────────────────────┘
```

---

## DeepFace Emotion Detection Pipeline

```
Webcam Frame (BGR numpy array)
          │
          ▼
┌─────────────────────────┐
│   Face Detection        │
│   MTCNN / RetinaFace    │
│   → Locate face region  │
└──────────┬──────────────┘
           │ Cropped face
           ▼
┌─────────────────────────┐
│   Preprocessing         │
│   Resize → 48×48 px     │
│   Normalize pixel vals  │
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│   Emotion CNN           │
│   (FER2013-trained)     │
│                         │
│   Output probabilities: │
│   angry:   0.02         │
│   disgust: 0.01         │
│   fear:    0.03         │
│   happy:   0.87  ←max   │
│   sad:     0.04         │
│   surprise:0.02         │
│   neutral: 0.01         │
└──────────┬──────────────┘
           │ argmax
           ▼
    dominant_emotion = "happy"
```

---

## Streamlit Session State Flow

```
Script run #1 (Start Camera clicked)
  session_state["camera_active"] = True
  session_state["last_frame"]    = None
         │
         ▼
WebRTC stream active → VideoProcessor.recv() runs per frame
         │
         ▼
Script run #2 (Capture Image clicked)
  session_state["last_frame"] = frame
  session_state["camera_active"] = False
  st.rerun() → forces script re-execution
         │
         ▼
Script run #3 (after rerun)
  last_frame is not None → process emotion
  detect_emotion(frame) → "happy"
  generate_music("happy") → (uri, art_url)
  Display results
```

---

## Tech Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **UI Framework** | Streamlit | Browser-based Python web app |
| **Video Stream** | streamlit-webrtc + WebRTC | Live webcam access |
| **Frame Processing** | PyAV (av library) | Video frame format conversion |
| **Image Processing** | OpenCV (cv2) | BGR frame handling |
| **Emotion Detection** | DeepFace 0.0.93 | Deep learning facial analysis |
| **Music API** | Spotify Web API | Track recommendations |
| **HTTP Client** | Requests | Spotify REST API calls |

---

## Emotion → Music Mapping

| Emotion | Playlist | Vibe |
|---------|----------|------|
| 😄 Happy | Upbeat Pop Playlist | High energy, positive |
| 😢 Sad | Calm/Chill Playlist | Soothing, slow |
| 😠 Angry | (extend) | Heavy, intense |
| 😲 Surprise | (extend) | Dynamic |
| 😨 Fear | (extend) | Ambient |
| 😐 Neutral | (extend) | Balanced |

---

## Setup

### Prerequisites
- Python 3.11+
- Spotify Developer Account → [Create App](https://developer.spotify.com/dashboard)

### Installation

```bash
git clone https://github.com/Yesh-is-here2/Ai_mood_dj
cd Ai_mood_dj
pip install -r requirements.txt
```

### Configure Spotify API

1. Go to [Spotify Developer Dashboard](https://developer.spotify.com/dashboard)
2. Create a new app
3. Get your Bearer token
4. Add to `music_generator.py`:
```python
api_key = os.getenv("SPOTIFY_API_KEY")  # use env variable
```

### Run

```bash
streamlit run app.py
# → http://localhost:8501
```

---

## Project Structure

```
Ai_mood_dj/
├── app.py               # Streamlit UI + WebRTC video capture
├── emotion_detection.py # DeepFace emotion analysis
├── music_generator.py   # Spotify API music recommendation
└── requirements.txt
```

---

## Key Design Decisions

**Why DeepFace over training custom CNN?**
Training a facial emotion CNN requires 35,000+ labeled images (FER2013 dataset), GPU access, and weeks of work. DeepFace provides pre-trained state-of-the-art models out of the box with simple API — ideal for rapid prototyping and production demos.

**Why enforce_detection=False?**
Prevents crashes when face is partially visible, at an angle, or in poor lighting. Returns best-effort emotion analysis instead of throwing an exception.

**Why streamlit-webrtc over st.camera_input?**
st.camera_input takes a single static photo. streamlit-webrtc provides a live video stream with real-time preview, creating a more engaging and realistic user experience.

**Why Streamlit session_state?**
Streamlit re-runs the entire Python script on every user interaction. session_state persists data (captured frame, detected emotion, music result) across reruns — essential for multi-step workflows.

---

## Future Improvements

- [ ] Extend emotion-to-playlist mapping for all 6 emotions
- [ ] Add Spotify OAuth for personal library integration  
- [ ] Real-time emotion tracking (continuous, not single capture)
- [ ] Store emotion history with timestamps
- [ ] Multi-face detection support
- [ ] Custom playlist creation based on session emotion journey

---

## Author

**Yeshwanth Akula**
M.S. Computer Science — Saint Louis University (May 2026)
Focus: AI Engineering, Computer Vision, Multimodal AI

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?style=flat&logo=linkedin)](https://linkedin.com/in/yeshwanth-akula-0339a925b)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-black?style=flat&logo=github)](https://github.com/Yesh-is-here2)
