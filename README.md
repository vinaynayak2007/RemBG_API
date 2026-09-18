# 🎨 RemBG_API — Background Removal Backend

The **backend service** behind my background-removal tooling: a thin, focused Flask wrapper around [rembg](https://github.com/danielgatis/rembg) that exposes background removal over HTTP.

> 🔗 Full-stack version with Docker, deployment config and a complete walkthrough lives in **[background-remover](https://github.com/vinaynayak2007/background-remover)**.

---

## ✨ What it does

- Accepts an image over HTTP and returns a PNG with a transparent background
- Strips the background using the U²-Net model (via `rembg` + ONNX Runtime)
- Stateless — nothing is stored server-side
- CORS-enabled so any frontend can call it

---

## 🛠️ Stack

<p align="left">
  <img src="https://img.shields.io/badge/Python-3.11-3776AB?style=flat-square&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/Flask-3.0-000000?style=flat-square&logo=flask&logoColor=white" />
  <img src="https://img.shields.io/badge/rembg-2.0-FF6F00?style=flat-square" />
  <img src="https://img.shields.io/badge/ONNX_Runtime-1.16-005CED?style=flat-square&logo=onnx&logoColor=white" />
</p>

---

## 🚀 Setup

```bash
git clone https://github.com/vinaynayak2007/RemBG_API.git
cd RemBG_API

python -m venv .venv && source .venv/bin/activate
pip install flask flask-cors rembg pillow onnxruntime numpy gunicorn

PORT=10000 python app.py
```

> **Python version note:** use **3.11**. `onnxruntime` wheels are unreliable on newer interpreters, and that's the single most common build failure with `rembg`.

---

## 📡 Endpoints

| Method | Route | Purpose |
| :--- | :--- | :--- |
| `GET` | `/` | Health check — returns `{"status":"ok"}` |
| `POST` | `/remove` | Remove background from a base64 image |

### `POST /remove`

```bash
curl -X POST http://localhost:10000/remove \
  -H "Content-Type: application/json" \
  -d "{\"image\": \"$(base64 -w0 input.jpg)\"}" \
  | python -c "import sys,json,base64;open('out.png','wb').write(base64.b64decode(json.load(sys.stdin)['image']))"
```

---

## ☁️ Production

```bash
gunicorn --bind 0.0.0.0:$PORT --workers 1 --timeout 120 app:app
```

`--workers 1` is deliberate: each worker loads its own copy of the ONNX model into memory. Scale with `--threads`, not workers, unless you have RAM to spare.

---

## 📄 License

MIT

---

<p align="center"><sub>Part of <a href="https://vinunayak.pages.dev">Vinay N's</a> toolkit</sub></p>
