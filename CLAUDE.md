# CLAUDE.md

## Project Overview

Serverless URL shortener using **Amazon API Gateway + DynamoDB** — no Lambda functions. All business logic lives in VTL (Velocity Template Language) mapping templates inside `api.yaml`. The project is a fork of an AWS sample that replaces Cognito auth with API Keys.

## Architecture

- **Backend**: API Gateway → DynamoDB (direct integration via VTL)
- **Frontend**: Vue.js 2 client (`client/`) — uses Axios, Vuex, Bulma, Vue Router
- **IaC**: AWS SAM (`template.yaml`) + OpenAPI spec (`api.yaml`)
- **Auth**: X-API-Key header required for write operations; GET `/{linkId}` is public
- **Owner**: Hard-coded as `"bfan"` in VTL templates (not dynamic)

## Key Files

| File | Role |
|------|------|
| `template.yaml` | SAM CloudFormation template — all AWS resources |
| `api.yaml` | OpenAPI 3.0.1 spec with VTL request/response mapping templates |
| `client/src/views/Dashboard.vue` | Main frontend UI |
| `client/src/store/index.js` | Vuex store (auth + links state) |
| `client/.env` | Frontend env vars (API endpoint, app name) |

## API Endpoints

| Method | Path | Auth | Purpose |
|--------|------|------|---------|
| GET | `/{linkId}` | None | Redirect to full URL (301) |
| GET | `/app` | API Key | List all links for owner |
| POST | `/app` | API Key | Create shortened URL |
| PUT | `/app/{linkId}` | API Key | Update URL |
| DELETE | `/app/{linkId}` | API Key | Delete URL |

## DynamoDB Table (`UrlShortenerLinks`)

- Partition key: `id`
- GSI: `OwnerIndex` on `owner` attribute
- TTL on `expiration` attribute (Unix timestamp)
- Attributes: `id`, `url`, `owner`, `timestamp`, `expiration`, `secret`, `hostname`

## Commands

### Frontend
```bash
cd client
npm run serve   # Dev server
npm run build   # Production build
npm run lint    # Lint
```

### Backend Deployment
```bash
sam deploy -g   # First-time (guided, interactive)
sam deploy      # Subsequent deployments (uses samconfig.toml)
```

## Key Conventions

- **No Lambda** — resist adding Lambda; all logic goes in VTL in `api.yaml`
- **VTL templates** — request mapping transforms HTTP → DynamoDB; response mapping transforms DynamoDB → HTTP
- **Cache**: GET `/{linkId}` has 5-min cache-control; PUT changes take 5+ min to propagate
- **URL validation** done via regex pattern in `api.yaml` schema (HTTP/HTTPS only)
- **Expiration default**: 31,536,000 seconds (1 year) calculated in VTL

## Infrastructure Resources

- `UrlShortenerAPI` — API Gateway REST API (edge endpoint, X-Ray tracing, CloudWatch logs)
- `UrlShortenerLinks` — DynamoDB table (PAY_PER_REQUEST billing)
- `DDBReadRole` / `DDBCrudRole` — IAM roles (least-privilege)
- API Key + Usage Plan: 5000 req/month quota, 100 RPS sustained, 200 RPS burst
- CloudWatch Alarms: 4xx, 5xx, p99 latency (API GW); 4xx, 5xx (DynamoDB)

## Testing

No test framework configured. Test manually:
- Frontend: `npm run serve` then use the UI
- Backend: `curl -H "x-api-key: <key>" <api-endpoint>/app`
- Reference payloads in `templates/` directory

## Branch Strategy

- `main`/`develop` — stable
- Current active branch: `feature/logging-v2`
