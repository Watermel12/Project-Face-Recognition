# Face Re-Identification System

Integrated Real-time Facial Recognition System **Desktop GUI**, **REST API**, **PostgreSQL** and **React Dashboard**.

![Python](https://img.shields.io/badge/Python-3.10+-blue)
![FastAPI](https://img.shields.io/badge/FastAPI-0.110+-green)
![React](https://img.shields.io/badge/React-18-61DAFB)
![Docker](https://img.shields.io/badge/Docker-required-2496ED)

---

## System Architecture

```
┌──────────────┐   WebSocket    ┌──────────────────┐
│  GUI (PyQt6) │ ─────────────► │  API (FastAPI)    │
│  Camera Feed │ ◄───────────── │  SCRFD + ArcFace  │
└──────────────┘   bbox/names   │  FAISS Search     │
                                └────────┬─────────┘
                                         │ asyncpg
                                ┌────────▼─────────┐
                                │   PostgreSQL DB   │
                                │  (Docker)         │
                                └────────┬─────────┘
                                         │ REST API
                                ┌────────▼─────────┐
                                │  React Dashboard  │
                                │  localhost:5173    │
                                └──────────────────┘
```

---

## System requirements

| Component | Version |
|---|---|
| Python | 3.10+ |
| Node.js | 18+ |
| Docker Desktop | newest |
| GPU (tuỳ chọn) | CUDA 11.8+ |

---

## Installation

### 1. Clone repository

```bash
git clone https://github.com/Watermel12/Project-Face-Recognition.git
cd Project-Face-Recognition
```

### 2. Download model weights
Download and place them in the **weights/** folder:

| Model | Link | Size |
|---|---|---|
| SCRFD 10G (detection) | [det_10g.onnx](https://github.com/yakhyo/face-reidentification/releases/download/v0.0.1/det_10g.onnx) | 16.1 MB |
| SCRFD 500M (lightweight) | [det_500m.onnx](https://github.com/yakhyo/face-reidentification/releases/download/v0.0.1/det_500m.onnx) | 2.4 MB |
| ArcFace MobileFace | [w600k_mbf.onnx](https://github.com/yakhyo/face-reidentification/releases/download/v0.0.1/w600k_mbf.onnx) | 13 MB |
| ArcFace ResNet-50 | [w600k_r50.onnx](https://github.com/yakhyo/face-reidentification/releases/download/v0.0.1/w600k_r50.onnx) | 166 MB |

```bash
# Linux/Mac — auto download
sh download.sh
```

### 3. Add face images for recognition

Place face images in the `assets/faces/`. **File name = person's name.**.

```
assets/faces/
├── NguyenVanA.jpg
├── TranThiB.jpg
└── ...
```

> One image per person is sufficient. Use a clear, well-lit frontal photo.

### 4. Start PostgreSQL (Docker)

```bash
docker compose up -d
```

Verify the DB is ready:
```bash
docker compose ps
```

### 5. Install Python dependencies

```bash
pip install -r requirements.txt
```

> **CPU only (default):** The `requirements.txt` uses `onnxruntime` by default - works on any machine without a GPU.
>
> **GPU (CUDA):** If your machine has an NVIDIA GPU, replace `onnxruntime` with `onnxruntime-gpu` in `requirements.txt`, then reinstall:
> ```bash
> pip install -r requirements.txt
> ```
> Then verify GPU is detected:
> ```bash
> python -c "import onnxruntime as ort; print(ort.get_available_providers())"
> ```
> You should see `CUDAExecutionProvider` in the list.
>
> **CUDA + cuDNN setup (Windows):**
> 1. Install [CUDA Toolkit 12.x or 13.x](https://developer.nvidia.com/cuda-downloads) matching your driver version (check with `nvidia-smi`)
> 2. Install [cuDNN 9.x](https://developer.nvidia.com/cudnn-downloads) for your CUDA version
> 3. Copy all `.dll` files from the cuDNN `bin/` folder into your CUDA `bin/` folder:
>    ```
>    C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\vXX.X\bin\
>    ```
> 4. Install the correct onnxruntime-gpu version:
>    ```bash
>    pip install onnxruntime-gpu==1.24.4
>    ```

### 6. Run the API backend

```bash
python api.py
```

API starts at `http://localhost:8000`. Swagger docs: `http://localhost:8000/docs`

### 7. Run the desktop GUI

```bash
python gui.py
```

- Click **▶ Start** to enable the camera and begin recognition
- Click **⚙ Settings** to change models, camera source, or threshold

### 8. Run the React Dashboard (optional)

```bash
cd web
npm install
npm run dev
```

Open in browser: `http://localhost:5173`

---

## Project Structure

```
face-reidentification/
├── api.py                 # FastAPI backend (REST + WebSocket)
├── gui.py                 # Desktop GUI (PyQt6)
├── db.py                  # PostgreSQL helpers (asyncpg)
├── main.py                # CLI entry point (no GUI)
│
├── models/                # SCRFD, ArcFace model wrappers
├── database/              # FAISS database implementation
├── utils/                 # Logging, helpers
│
├── weights/               # Model weights (.onnx) — not tracked in git
├── assets/
│   ├── faces/             # Face images for recognition
│   └── captures/          # Auto-saved cropped images — not tracked in git
│
├── web/                   # React dashboard (Vite)
│   ├── src/
│   │   ├── pages/         # Dashboard, Attendance, Unknowns, Settings
│   │   ├── api.js         # API service layer
│   │   └── App.jsx
│   └── package.json
│
├── init.sql               # PostgreSQL schema
├── docker-compose.yml     # PostgreSQL container
└── requirements.txt
```

---

## PostgreSQL Connection Info

| Parameter| Default Value |
|---|---|
| Host | `localhost` |
| Port | `5432` |
| Database | `faceid_db` |
| Username | `faceid_user` |
| Password | `faceid_pass` |

Override connection details via environment variable:
```bash
DATABASE_URL=postgresql://user:pass@host:5432/dbname python api.py
```

---

## API Endpoints

| Method | Endpoint | Description|
|---|---|---|
| `GET` | `/api/settings` | Get current configuration |
| `POST` | `/api/settings` | Update configuration |
| `POST` | `/api/infer/start` | Start inference |
| `POST` | `/api/infer/stop` | Stop inference |
| `GET` | `/api/attendance` | Attendance history |
| `GET` | `/api/unknowns` | Unknown person log |
| `GET` | `/api/stats` | Overall statistics |
| `POST` | `/api/attendance/log` | Log an attendance entry (multipart) |
| `POST` | `/api/unknown/log` | Log an unknown person (multipart) |
| `WS` | `/ws/infer` | Real-time recognition WebSocket |

---

## Troubleshooting

**Camera won't open:**
- Check the camera source in Settings (0 = default webcam, or use an RTSP URL)

**ONNX Runtime / CUDA errors::**
- Use `onnxruntime (CPU)` if no GPU is available or CUDA isn't set up correctly
- Check logs at `app.log`
- If logs show no errors but the model still runs on CPU only, verify which runtime is being imported:
```bash
python -c "import onnxruntime as ort; print(ort.__file__); print(ort.get_available_providers())"
```
- If the path points to `AppData\Roaming\Python\...` instead of the conda env, the environment is mixing user-site packages
- The most reliable fix is to create a new env, enable `PYTHONNOUSERSITE=1`, then reinstall dependencies using `python -m pip`

**Cannot connect to PostgreSQL:**
- Make sure Docker is running: `docker compose up -d`
- Verify: `docker compose ps`

**Web dashboard not showing images:**
- Ensure the Vite dev server is running (`npm run dev` inside the `web/` directory)
- Images are served via proxy `/captures` → FastAPI

---

## References

- [SCRFD: Efficient Face Detection](https://github.com/deepinsight/insightface/tree/master/detection/scrfd)
- [ArcFace: Deep Face Recognition](https://github.com/deepinsight/insightface/tree/master/recognition/arcface_torch)
- [YOLOFace training guide](https://drive.google.com/drive/folders/1Df3xxfUsWDbMfqwTgOE7q2CeXakW4V8D?usp=sharing)
- [FAISS: Facebook AI Similarity Search](https://github.com/facebookresearch/faiss)
