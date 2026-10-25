# DOPL — Digital Ownership Protection Layer

DOPL helps creators and course owners detect when their paid content (courses, PDFs, videos, notes) is being leaked or resold on public Telegram channels, and gives them a paper trail to act on it.

## The Problem

Creators selling courses or gated content have no visibility into piracy. Their material shows up in random Telegram groups within days of launch, and by the time someone reports it manually, hundreds of people already have it for free. There is no lightweight way to monitor for this continuously.

## What DOPL Does

1. **Connect your platform:** link the LMS/host where your content lives (`hostType` + `endpointUrl`).
2. **Set keywords to monitor:** course names, product titles, or any term likely to appear if your content leaks.
3. **Add scan targets:** public Telegram channels/groups (`t.me/s/{channel}` links) to watch.
4. **Automated scanning:** a background job scrapes those public channels every 30 minutes and checks new messages against your keywords.
5. **Get alerted:** a match creates a `ScanResult` and an `Alert` in your dashboard, with a preview of the offending message and a link to it.
6. **Track takedowns:** log and update the status of takedown requests you file against confirmed leaks, so nothing falls through the cracks.

## Scope and Boundaries (by design)

- Only **public** Telegram channel preview pages (`t.me/s/...`) are scraped.
- Private groups, invite links (`joinchat`, `t.me/+...`), and any form of login/infiltration are explicitly out of scope and are skipped by the scanner.
- Scans are sequential, with a 2-second delay between channels to respect rate limits.
- Duplicate matches (same keyword + same channel) are not re-recorded.

## Tech Stack

**Frontend**
- React 19 + Vite
- Tailwind CSS 4
- Three.js (background visuals)

**Backend**
- Node.js + Express 5
- PostgreSQL via Prisma ORM
- JWT-based auth (`jsonwebtoken`, `bcryptjs`)
- `node-cron` for the scheduled scan job
- `axios` + `cheerio` for scraping public Telegram pages

**Deployment**
- Backend: Render (see `render.yaml`)
- Frontend: Vercel

## Project Structure

```
DOPL/
├── prisma/
│   ├── schema.prisma          # DB models: User, PlatformIntegration,
│   │                          # MonitoredKeyword, ScanTarget, ScanResult,
│   │                          # TakedownRequest, Alert
│   └── migrations/
├── src/
│   ├── components/            # React pages (Landing, Signin, Signup,
│   │                          # CommandCenter, Platform Integration,
│   │                          # Telegram Monitoring, Success)
│   ├── context/               # React context (auth state)
│   ├── middleware/            # Express auth middleware
│   ├── routes/                # Express route handlers (see API below)
│   ├── scanner/
│   │   └── telegramScanner.js # Core scraping + matching engine
│   ├── utils/                 # Token generation, helpers
│   ├── db.js                  # Prisma client
│   ├── index.js               # Express app entry point
│   └── main.jsx               # React app entry point
├── public/
├── render.yaml
└── vite.config.js
```

## Data Model

| Model | Purpose |
|---|---|
| `User` | Creator account (email, hashed password, plan) |
| `PlatformIntegration` | The LMS/host a creator's content lives on |
| `MonitoredKeyword` | Terms to watch for across scanned channels |
| `ScanTarget` | Public Telegram channels/groups to scan |
| `ScanResult` | A keyword match found during a scan |
| `TakedownRequest` | Status tracking for action taken on a match |
| `Alert` | In-app notification tied to a scan result |

## API Overview

All protected routes require a Bearer JWT from `/api/auth/login` or `/api/auth/signup`.

| Method | Route | Description |
|---|---|---|
| `POST` | `/api/auth/signup` | Create an account |
| `POST` | `/api/auth/login` | Log in |
| `GET` | `/api/auth/me` | Get current user (protected) |
| `POST` / `GET` | `/api/platform/connect` | Connect or fetch platform integration |
| `POST` / `GET` | `/api/keywords` | Add / list monitored keywords |
| `DELETE` | `/api/keywords/:id` | Remove a keyword |
| `POST` / `GET` | `/api/scan-targets` | Add / list Telegram scan targets |
| `DELETE` | `/api/scan-targets/:id` | Remove a scan target |
| `POST` / `GET` | `/api/scan-results` | Log / list scan results |
| `DELETE` | `/api/scan-results/:id` | Remove a scan result |
| `POST` / `GET` | `/api/takedowns` | File / list takedown requests |
| `PATCH` | `/api/takedowns/:id` | Update takedown status |
| `POST` / `GET` | `/api/alerts` | Create / list alerts |
| `PATCH` | `/api/alerts/:id` | Mark an alert as read |
| `POST` | `/api/scan/run-now` | Trigger a scan cycle manually |

## Getting Started

### Prerequisites

- Node.js 18+
- A PostgreSQL database (e.g. Supabase, Neon, or local Postgres)

### Environment Variables

| Variable | Description |
|---|---|
| `DATABASE_URL` | PostgreSQL connection string |
| `DIRECT_URL` | Direct database connection URL (used by Prisma) |
| `JWT_SECRET` | Secret used to sign JWTs |

### Deployment

- The backend is configured for Render via `render.yaml` (Node web service). `DATABASE_URL`, `DIRECT_URL`, and `JWT_SECRET` are set as environment secrets.
- The frontend is deployed on Vercel.

## Roadmap

- [ ] Support for scanning other public platforms beyond Telegram
- [ ] Automated takedown request generation/submission
- [ ] Email/push notifications for alerts, not just in-app
- [ ] Plan-based limits on keywords/scan targets
