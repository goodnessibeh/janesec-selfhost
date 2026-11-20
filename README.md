# JaneSec — self-hosting

Run your own **Jane**, the autonomous penetration tester, on your own hardware.

Jane ships as a **single public image** — Postgres (with pgvector), the API, the
Next.js war room and nginx in one container, behind one port (`8080`). This repo
holds everything you need to run it: the Compose file, an env template, and the
setup below. The application source stays private; only the image is public.

> Full documentation: **https://janesecai.xyz/docs**

## Quickstart

Grab the Compose file and bring it up:

```sh
curl -O https://raw.githubusercontent.com/goodnessibeh/janesec-selfhost/main/docker-compose.yml
JWT_SIGNING_KEY=$(openssl rand -hex 32) docker compose up -d

# open the war room
open http://localhost:8080
```

Or run the image directly — no file needed:

```sh
docker run -d --name janesec -p 8080:8080 \
  -e JWT_SIGNING_KEY=$(openssl rand -hex 32) \
  -v janesec-db:/var/lib/postgresql/data \
  -v janesec-state:/var/lib/janesec \
  ghcr.io/goodnessibeh/janesec:latest
```

A real Postgres database comes up inside the container and persists on the
`janesec-db` volume; app state — evidence, the encrypted vault, ingested trees —
persists on `janesec-state`. Everything serves on `:8080`.

## Configuration

Copy `.env.example` to `.env` and set what you need. The essentials:

| Variable | Required | What it does |
| --- | --- | --- |
| `JWT_SIGNING_KEY` | **Yes** | Signs session JWTs — no insecure fallback. `openssl rand -hex 32`. |
| `CREDENTIALS_KEY` | No | Fernet key for secrets at rest. Empty = auto-generated & persisted (single node). |
| `JANESEC_DATABASE_URL` | No | Point at an external Postgres instead of the bundled one. |
| `PORT` | No | Host port the war room is published on (default `8080`). |

> Your **LLM provider, model and API keys** are **not** environment variables —
> set them in the app's Settings screen after you sign in, before starting a
> mission (secrets are write-only and never read back).

The full environment-variable reference — feature flags, scanners, integrations
(Slack / Jira / HackerOne), Cloudflare Access — is in the
[docs](https://janesecai.xyz/docs).

## Drive it

- **Browser** — the war room at `http://localhost:8080`.
- **CLI, VS Code, and Claude over MCP** — see the [docs](https://janesecai.xyz/docs).

## Updating

```sh
docker compose pull && docker compose up -d
```

## Notes

- Jane is for **authorized testing only**.
- Running the bundled Kali toolbox needs Docker access (socket or DinD) on the host.
