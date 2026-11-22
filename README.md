# AgriVoice AI

GitHub-ready development prototype for a Marathi/Hindi voice-first agricultural assistant.

## Features
- FastAPI backend and Swagger docs
- PDF/TXT/MD ingestion with source metadata and approval gate
- Local TF-IDF retrieval
- Optional Gemini grounded generation
- Confidence-based refusal when evidence is missing
- Optional Faster-Whisper speech recognition and Edge TTS
- Open-Meteo weather lookup
- Mandi price CSV adapter (not a live Agmarknet integration)
- Basic web chat interface

> This is a prototype, not a certified agronomy service. Verify advice with ICAR/KVK/agricultural experts. Never rely on it alone for pesticide dosage or other high-risk decisions.

## Windows 11 setup
```powershell
py -3.11 -m venv venv
.\venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
Copy-Item .env.example .env
uvicorn backend.main:app --reload
```
If activation is blocked:
```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\venv\Scripts\Activate.ps1
```

Open http://127.0.0.1:8000 and http://127.0.0.1:8000/docs.

## Included sample knowledge document
The repository includes one Marathi educational summary for cotton pink bollworm, based on the ICAR Kharif Agro-Advisories for Farmers 2025 (page 146). It is included to demonstrate document approval and retrieval. Confirm current local recommendations with KVK/agricultural experts. It intentionally does not provide pesticide dosages.

Rebuild the index after cloning or changing documents:
```powershell
python scripts/ingest_documents.py
```

## Add knowledge documents
Put approved PDF/TXT/MD files in `data/raw/`. Add a sidecar named like `guide.pdf.metadata.json`:
```json
{
  "source": "Publisher advisory title",
  "publisher": "Verified publisher",
  "published_date": "2026-01-15",
  "region": "Maharashtra",
  "crop": "cotton",
  "language": "mr",
  "license": "Check publisher terms",
  "approved": true
}
```
Only mark `approved` true after verifying the document and redistribution/use terms. Build the index:
```powershell
python scripts/ingest_documents.py
```
The app indexes only approved documents.

## Optional integrations
```powershell
pip install -r requirements-optional.txt
```
Set `GEMINI_API_KEY` for Gemini. Set `ENABLE_WHISPER=true` for Faster-Whisper and `ENABLE_TTS=true` for Edge TTS in `.env`. Whisper may download a model on first use.

## API routes
- `GET /health`
- `GET /api/v1/status`
- `POST /api/v1/chat`
- `POST /api/v1/documents/upload`
- `POST /api/v1/voice/transcribe`
- `POST /api/v1/voice/speak`
- `GET /api/v1/weather?latitude=...&longitude=...`
- `GET /api/v1/mandi/prices?commodity=...&market=...&state=...`

Mandi prices are read from `data/mandi_prices.csv`; this is a CSV adapter, not a live Agmarknet/e-NAM connection. Populate it only from an authorized current source and preserve date/source details.

## GitHub publishing
```powershell
git init
git add .
git commit -m "Initial AgriVoice AI prototype"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/AgriVoice_AI.git
git push -u origin main
```
Never commit `.env`, private farmer recordings, phone numbers, or documents you are not allowed to redistribute.

## Production work still required
WhatsApp Business integration, live authorized mandi provider, IndicTrans2/Bhashini integration, robust hybrid retrieval/reranking, PostgreSQL/pgvector, admin authentication and audit logs, consent/retention/deletion workflows, expert review, full evaluation dashboard, deployment and field pilot. See `docs/PROJECT_PLAN.md`.
