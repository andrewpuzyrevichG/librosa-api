# Librosa Async API

**Librosa Async API** is a FastAPI-based server for audio analysis.  
It allows uploading audio files (MP3/WAV) and returns detailed audio features, beat detection, downbeats, onset events, and visualizations.  
Optionally, if `madmom` is installed, the API can also detect downbeats for more precise rhythm analysis.

---

## Features

- **Audio Metadata Extraction**: Extracts song title, artist, and genre using `mutagen`.
- **Beat & Onset Detection**: Uses `librosa` for tempo, beat times, onset events, and RMS (loudness) analysis.
- **Downbeat Detection (optional)**: Uses `madmom` to detect downbeats for more precise rhythm mapping.
- **Harmonic/Percussive Separation**: Separates harmonic and percussive components of audio and visualizes them.
- **Plot Generation**: Produces base64-encoded PNG plots of:
  - Onset strength and beats
  - PLP (Percussive-Like Peaks) with beats
  - Harmonic vs Percussive components with downbeats
- **JSON Response**: Returns tempo, duration, beat times, onset times, RMS, downbeats, events, and metadata.

---

## Installation

### Prerequisites

- Python 3.11+
- Linux server (Ubuntu/Debian recommended)
- `ffmpeg` (required by `librosa` to process MP3 files)
- Optional: `madmom` for downbeat detection

```bash
sudo apt update
sudo apt install python3 python3-venv python3-pip ffmpeg -y

## Clone and Setup

git clone <your-repo-url>
cd <your-repo-folder>
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt

**Optional:** Install madmom for downbeat detection:
pip install madmom


##Configuration

- MAX_FILE_SIZE: Maximum allowed audio file size (default: 20 MB)
- DEFAULT_SR: Sampling rate for processing audio (default: 22050 Hz)
- You can adjust these values in main.py to optimize memory and performance for large audio files.

##Running the Server

Run the FastAPI server:

uvicorn main:app --host 0.0.0.0 --port 8000

- The API will be available at http://<server-ip>:8000.
- Interactive API documentation: http://<server-ip>:8000/docs.

##Usage

Endpoint: /analyze
Method: POST
Form Data:

audio: Audio file (MP3 or WAV)

Example with cURL:

curl -X POST "http://localhost:8000/analyze" \
  -F "audio=@/path/to/audio.mp3"


Example JSON Response:

{
  "tempo": 120.5,
  "beat_times": [0.0, 0.5, 1.0, ...],
  "onset_times": [0.1, 0.4, 0.9, ...],
  "rms": [0.02, 0.03, ...],
  "duration": 180.0,
  "downbeats": [0.0, 2.0, ...],
  "image_base64": "iVBORw0KGgoAAAANSUhEUgAA...",
  "events": [
    {"timeshow_id":160,"event_id":"M1","event_label":"measure","time_stamp":0.0,"event_color":"#F3F6EC","Value":1},
    ...
  ],
  "song_label": "Song Title",
  "artist": "Artist Name",
  "genre": "Genre"
}

##Deployment on Linux
###1. Use a Virtual Environment

Ensure your project runs in a Python virtual environment to avoid dependency conflicts.

python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

###2. Run with Uvicorn

For development:

uvicorn main:app --host 0.0.0.0 --port 8000 --reload


####For production:

uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4


--workers 4 allows handling multiple requests concurrently.

###3. Use Systemd to Auto-Start

Create a systemd service file /etc/systemd/system/librosa-api.service:

[Unit]
Description=Librosa Async API
After=network.target

[Service]
User=youruser
WorkingDirectory=/path/to/repo
ExecStart=/path/to/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4
Restart=always

[Install]
WantedBy=multi-user.target


###Enable and start the service:

sudo systemctl daemon-reload
sudo systemctl enable librosa-api
sudo systemctl start librosa-api
sudo systemctl status librosa-api