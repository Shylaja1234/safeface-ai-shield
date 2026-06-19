# SafeFace AI - Backend

Node.js + Express + MongoDB REST API for the SafeFace AI deepfake detection platform.
Calls a Python FastAPI microservice for the actual analysis.

## Stack

- Node.js / Express
- MongoDB (Mongoose)
- Multer (file uploads)
- JWT auth + bcrypt
- helmet, cors, express-rate-limit, express-validator
- Axios -> Python FastAPI service

## Install

```bash
cd backend
cp .env.example .env   # edit values
npm install
npm run dev
```

API starts on `http://localhost:5000`.

## Environment

| var | description |
|---|---|
| `PORT` | API port (default 5000) |
| `MONGODB_URI` | Mongo connection string |
| `JWT_SECRET` | Long random string |
| `JWT_EXPIRES_IN` | e.g. `7d` |
| `AI_SERVICE_URL` | Python FastAPI base URL (default `http://localhost:8000`) |
| `CORS_ORIGIN` | Comma-separated allowed origins, e.g. `http://localhost:5173` |
| `MAX_FILE_SIZE_MB` | Upload limit (default 100) |

## API

All `/api/*` routes are rate-limited.

### Auth
- `POST /api/auth/register` - `{ name, email, password }`
- `POST /api/auth/login` - `{ email, password }` -> `{ token, user }`
- `GET  /api/auth/me` (Bearer token)

### Detection
- `POST /api/detection/upload` (Bearer token, `multipart/form-data`, field `file`)

Accepts: `jpg`, `jpeg`, `png`, `webp`, `mp4`, `mov` (up to `MAX_FILE_SIZE_MB`).

Response:
```json
{
  "scanId": "...",
  "status": "SAFE",
  "riskScore": 18,
  "confidence": 94,
  "analysis": {
    "ganArtifacts": false,
    "faceConsistency": true,
    "lightingConsistency": true,
    "metadataIntegrity": true
  },
  "fileName": "photo.jpg",
  "fileType": "image/jpeg",
  "fileSize": 123456,
  "createdAt": "2026-06-16T12:34:56.000Z"
}
```

### Scans
- `GET /api/scans` - list current user's scans
- `GET /api/scans/:id` - single scan

### Dashboard
- `GET /api/dashboard/stats` - aggregated counts + recent scans

## Architecture

```
React frontend
     |
     v
Node/Express API  (auth, validation, persistence)
     |
     v
Python FastAPI    (deepfake detection - swap model freely)
     |
     v
   Result
     |
     v
  MongoDB
```

The Python service contract is stable, so you can swap the placeholder
detector for DeepFace / FaceForensics++ / InsightFace / a custom PyTorch
model without touching this Node API or the React frontend.

## Security

- Helmet security headers
- CORS allowlist via `CORS_ORIGIN`
- Per-IP rate limiting (300 req / 15 min on `/api`)
- JWT auth required for uploads, scans, dashboard
- bcrypt password hashing (cost 12)
- Multer MIME-type + size validation
- `express-validator` body validation
