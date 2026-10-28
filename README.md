# FestPass — Docker & Container Security Assessment (CA-2)

| Item | Value |
|------|-------|
| Name | _<your name>_ |
| REGNO | 202401100200027 |
| PORT | 8027 |
| Docker Hub username | ai0man |
| Docker Hub repository | https://hub.docker.com/r/ai0man/fest-backend |

## Stack

- `db`: `postgres:17-alpine`, data in the named volume `db-data`, `pg_isready` healthcheck
- `backend`: Flask API (`ai0man/fest-backend`), starts only after `db` is healthy
- `frontend`: `nginx:1.30-alpine` on `http://localhost:8027`, proxies `/api/` to the backend

Networks: `web` (frontend + backend) and `data` (backend + db), so the frontend cannot reach the database.

```sh
cp .env.example .env   # then set a real POSTGRES_PASSWORD
docker compose up -d
```

## Vulnerability scan (Trivy 0.75.0)

| Image | Base | CRITICAL | HIGH | Image size |
|-------|------|---------:|-----:|-----------:|
| `fest-backend:0.1` | `python:3.9` | 234 | 2249 | 1.6 GB (412 MB compressed) |
| `fest-backend:0.2` | `python:3.14.10-alpine` | 0 | 0 | 94.2 MB (23.3 MB compressed) |

Fixes in 0.2:
- Base switched from `python:3.9` (Debian, end-of-life Python) to the pinned `python:3.14.10-alpine`.
- Libraries upgraded: Flask 2.2.2 → 3.1.3, Werkzeug 2.2.2 → 3.1.9, psycopg2-binary 2.9.10 → 2.9.13.
- `pip` and `setuptools` removed from the final image: the app does not use them at runtime, and they vendor
  vulnerable copies of urllib3, msgpack and setuptools.
- Runs as the non-root user `app` (uid 10001).

`trivy image --severity CRITICAL --ignore-unfixed --exit-code 1 fest-backend:0.2` exits with code 0.

Full reports: [`reports/trivy-before.txt`](reports/trivy-before.txt), [`reports/trivy-after.txt`](reports/trivy-after.txt).

## Release on Docker Hub

Tags `ai0man/fest-backend:1.0.0` and `ai0man/fest-backend:1.0` (built with `APP_VERSION=1.0.0`).

**1.0.0 digest:** `sha256:3765959447de1dd3b711cb04e0af9781e11818974364d5d33ccf6e972ce0ddb3`

`compose.yaml` runs the backend from `ai0man/fest-backend:1.0.0` pulled from Docker Hub instead of building it.

## AI tools used

- Claude Code (Claude Opus 5.5): wrote the Dockerfiles, `compose.yaml` and README, ran and explained the Trivy results,
  and suggested removing pip/setuptools to clear the vendored library findings.
