# SmartAttend 🎓

**AI-Powered Attendance Management System** built with Streamlit, face recognition, and voice recognition — designed to make classroom attendance fast, contactless, and hard to fake.

![Streamlit](https://img.shields.io/badge/Streamlit-FF4B4B?style=flat&logo=streamlit&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.x-blue?style=flat&logo=python&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-3ECF8E?style=flat&logo=supabase&logoColor=white)

## Overview

SmartAttend replaces manual roll-calls and easily-proxied attendance sheets with two biometric pipelines:

- **Face recognition** — a class photo is run through a `dlib`-based face detector and encoder, and an SVM classifier matches detected faces against enrolled students.
- **Voice recognition** — a `resemblyzer` voice-encoder pipeline can identify students from short audio clips, including bulk audio containing multiple speakers (via `librosa` silence-based segmentation).

The app has two portals — **Teacher** and **Student** — backed by a Supabase (Postgres) database.

## Features

**Teacher Portal**
- Register/login with hashed passwords (`bcrypt`)
- Create subjects and share a join code / QR code for students to enroll
- Add student face photos to build the training set
- Take attendance via class photo (face recognition) or audio clip (voice recognition)
- View attendance logs and per-subject stats (total students, total classes)

**Student Portal**
- Register/login and enroll in subjects using a join code (including auto-enroll via a shareable link)
- Enroll face and/or voice biometrics
- View enrolled subjects and personal attendance history
- Unenroll from a subject

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend / App framework | [Streamlit](https://streamlit.io/) |
| Face recognition | `dlib`, `face_recognition_models`, `scikit-learn` (SVM classifier) |
| Voice recognition | `resemblyzer`, `librosa` |
| Database & Auth | [Supabase](https://supabase.com/) (Postgres), `bcrypt` for password hashing |
| QR codes | `segno` |
| Data handling | `numpy`, `pandas`, `pillow` |

## Project Structure

```
Smart-Attend-main/
├── app.py                     # Entry point — routes between home/teacher/student screens
├── requirements.txt
└── src/
    ├── components/            # Reusable UI components and dialogs
    │   ├── dialog_add_photo.py
    │   ├── dialog_attendance_results.py
    │   ├── dialog_auto_enroll.py
    │   ├── dialog_create_subject.py
    │   ├── dialog_enroll.py
    │   ├── dialog_share_subject.py
    │   ├── dialog_voice_attendance.py
    │   ├── footer.py
    │   ├── header.py
    │   └── subject_card.py
    ├── database/
    │   ├── config.py           # Supabase client setup
    │   └── db.py               # All database queries (teachers, students, subjects, attendance)
    ├── pipelines/
    │   ├── face_pipeline.py    # Face embedding extraction + SVM classifier + prediction
    │   └── voice_pipeline.py   # Voice embedding extraction + speaker identification
    ├── screens/
    │   ├── home_screen.py
    │   ├── student_screen.py
    │   └── teacher_screen.py
    └── ui/
        └── base_layout.py      # Shared page styling
```

## Getting Started

### Prerequisites
- Python 3.9+
- A [Supabase](https://supabase.com/) project (URL + API key)
- `cmake` and build tools available on your system (required to build `dlib`)

### Installation

1. Clone the repository
   ```bash
   git clone <repo-url>
   cd Smart-Attend-main
   ```

2. Create and activate a virtual environment
   ```bash
   python -m venv venv
   source venv/bin/activate      # Windows: venv\Scripts\activate
   ```

3. Install dependencies
   ```bash
   pip install -r requirements.txt
   ```

4. Configure Supabase secrets. Create `.streamlit/secrets.toml`:
   ```toml
   SUPABASE_URL = "your-supabase-project-url"
   SUPABASE_KEY = "your-supabase-api-key"
   ```

5. Set up the database schema in Supabase with tables for `teachers`, `students`, `subjects`, `subject_students`, and `attendance_logs` (matching the fields referenced in `src/database/db.py`).

### Run the app

```bash
streamlit run app.py
```

The app will open at `http://localhost:8501`.

## How It Works

1. **Enrollment** — a student's face photo and/or voice sample is captured, converted into an embedding (128-d face vector or voice vector), and stored against their profile.
2. **Training** — an SVM classifier is trained on-the-fly from all enrolled students' face embeddings (cached via `st.cache_resource`).
3. **Attendance** — a teacher uploads a class photo or audio recording; the pipeline detects faces/speakers, matches them against the trained model within a similarity threshold, and logs attendance for matched students.

## Notes

- Face matching uses a Euclidean distance threshold (`0.6`) and voice matching uses cosine similarity (`0.65`) — both tunable in `src/pipelines/`.
- Models are cached with `st.cache_resource` and can be retrained by clearing the cache (`train_classifier()`).

## License

No license specified.
