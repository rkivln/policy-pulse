# 🏛️ PolicyPulse

> **Tracks government policy changes that affect your business — automatically**

PolicyPulse is a RegTech / SME Tools SaaS that monitors government portals, scrapes regulatory updates, summarises them using LLMs, and delivers plain-language alerts to businesses — helping SMEs avoid costly compliance fines.

---

## 📋 Table of Contents

- [Problem Statement](#-problem-statement)
- [Solution Overview](#-solution-overview)
- [Tech Stack](#-tech-stack)
- [System Architecture](#-system-architecture)
- [Project Structure](#-project-structure)
- [Implementation Plan](#-implementation-plan)
  - [Phase 1 — Foundation & Scraping Engine](#phase-1--foundation--scraping-engine)
  - [Phase 2 — LLM Summarisation Pipeline](#phase-2--llm-summarisation-pipeline)
  - [Phase 3 — Alert System](#phase-3--alert-system)
  - [Phase 4 — React Dashboard](#phase-4--react-dashboard)
  - [Phase 5 — Monetisation & SaaS Layer](#phase-5--monetisation--saas-layer)
  - [Phase 6 — Deployment & DevOps](#phase-6--deployment--devops)
- [API Reference](#-api-reference)
- [Database Schema](#-database-schema)
- [Environment Variables](#-environment-variables)
- [Monetisation Model](#-monetisation-model)
- [Roadmap](#-roadmap)
- [Getting Started](#-getting-started)

---

## 🔴 Problem Statement

SMEs (Small & Medium Enterprises) miss critical regulatory updates from government portals, resulting in:

- Unexpected **compliance fines**
- Wasted legal consultation fees
- Operational disruptions due to outdated practices
- No dedicated team or budget to track policy changes

---

## 🟢 Solution Overview

PolicyPulse automates the full compliance monitoring lifecycle:

1. User sets their **industry vertical** and **location**
2. The system continuously **scrapes government portals** relevant to that profile
3. An **LLM pipeline** summarises raw policy documents into plain-language digests
4. The **alert system** pushes updates via email, in-app notifications, or webhooks
5. A **React dashboard** displays all changes with severity levels, deadlines, and action items

---

## 🔧 Tech Stack

| Layer | Technology |
|---|---|
| **Scraping Engine** | Python, Scrapy / Playwright, BeautifulSoup |
| **Task Queue** | Celery + Redis |
| **LLM Summariser** | OpenAI GPT-4o / Claude API (Anthropic) |
| **Backend API** | FastAPI (Python) |
| **Database** | PostgreSQL (primary), Redis (cache/queue) |
| **Alert System** | SendGrid (email), Twilio (SMS), WebSockets (in-app) |
| **Frontend** | React 18, Next.js 14, Tailwind CSS, shadcn/ui |
| **Auth** | NextAuth.js / Supabase Auth |
| **Payments** | Stripe |
| **Deployment** | Docker, AWS (ECS + RDS + ElastiCache) |
| **CI/CD** | GitHub Actions |

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                         │
│              React / Next.js Dashboard (Vercel)             │
└────────────────────────┬────────────────────────────────────┘
                         │ REST / WebSocket
┌────────────────────────▼────────────────────────────────────┐
│                      API GATEWAY                            │
│                 FastAPI Backend (AWS ECS)                   │
│      Auth │ User Profiles │ Policy Feed │ Reports           │
└──────┬────────────┬──────────────────┬──────────────────────┘
       │            │                  │
┌──────▼──────┐ ┌───▼────────┐ ┌──────▼──────────────┐
│  PostgreSQL │ │   Redis    │ │   Celery Workers    │
│  (RDS)      │ │  (Cache +  │ │  (Scrape + LLM)     │
│             │ │   Queue)   │ │                     │
└─────────────┘ └────────────┘ └──────────┬──────────┘
                                           │
              ┌───────────────────────────▼──────────────────┐
              │              SCRAPING ENGINE                 │
              │   Playwright (JS-heavy sites) + Scrapy       │
              │   Gov portals: MCA, DPIIT, RBI, GST, MOL..  │
              └───────────────────────────┬──────────────────┘
                                          │ Raw HTML / PDFs
              ┌───────────────────────────▼──────────────────┐
              │          LLM SUMMARISATION PIPELINE          │
              │    Extract → Chunk → Embed → Summarise       │
              │    GPT-4o / Claude API                       │
              └───────────────────────────┬──────────────────┘
                                          │ Structured Policy Objects
              ┌───────────────────────────▼──────────────────┐
              │               ALERT SYSTEM                   │
              │   Email (SendGrid) │ SMS (Twilio) │ In-App   │
              └──────────────────────────────────────────────┘
```

---

## 📁 Project Structure

```
policypulse/
├── backend/                        # FastAPI backend
│   ├── app/
│   │   ├── api/
│   │   │   ├── routes/
│   │   │   │   ├── auth.py
│   │   │   │   ├── policies.py
│   │   │   │   ├── alerts.py
│   │   │   │   ├── reports.py
│   │   │   │   └── users.py
│   │   │   └── deps.py             # Auth dependencies
│   │   ├── core/
│   │   │   ├── config.py           # Settings / env vars
│   │   │   ├── security.py         # JWT helpers
│   │   │   └── database.py         # SQLAlchemy setup
│   │   ├── models/                 # ORM models
│   │   │   ├── user.py
│   │   │   ├── policy.py
│   │   │   ├── alert.py
│   │   │   └── subscription.py
│   │   ├── schemas/                # Pydantic schemas
│   │   ├── services/
│   │   │   ├── scraper_service.py
│   │   │   ├── llm_service.py
│   │   │   └── alert_service.py
│   │   └── main.py
│   ├── celery_worker/
│   │   ├── tasks/
│   │   │   ├── scrape_tasks.py
│   │   │   ├── summarise_tasks.py
│   │   │   └── alert_tasks.py
│   │   └── celery_app.py
│   ├── alembic/                    # DB migrations
│   ├── tests/
│   ├── Dockerfile
│   └── requirements.txt
│
├── scraper/                        # Scraping engine (standalone)
│   ├── spiders/
│   │   ├── mca_spider.py           # Ministry of Corporate Affairs
│   │   ├── gst_spider.py           # GST Council
│   │   ├── rbi_spider.py           # Reserve Bank of India
│   │   ├── dpiit_spider.py         # DPIIT / Startup India
│   │   ├── mol_spider.py           # Ministry of Labour
│   │   └── base_spider.py          # Base class
│   ├── extractors/
│   │   ├── pdf_extractor.py        # pdfplumber / PyMuPDF
│   │   └── html_extractor.py
│   ├── browser/
│   │   └── playwright_runner.py    # For JS-heavy portals
│   ├── pipelines.py                # Scrapy item pipelines
│   └── settings.py
│
├── llm_pipeline/                   # LLM summarisation module
│   ├── chunker.py                  # Text splitting
│   ├── embedder.py                 # Vector embeddings
│   ├── summariser.py               # LLM API calls
│   ├── classifier.py               # Industry/severity tagging
│   └── prompts/
│       ├── summary_prompt.txt
│       └── classification_prompt.txt
│
├── frontend/                       # Next.js dashboard
│   ├── app/
│   │   ├── (auth)/
│   │   │   ├── login/page.tsx
│   │   │   └── register/page.tsx
│   │   ├── (dashboard)/
│   │   │   ├── dashboard/page.tsx
│   │   │   ├── policies/page.tsx
│   │   │   ├── alerts/page.tsx
│   │   │   ├── reports/page.tsx
│   │   │   └── settings/page.tsx
│   │   └── layout.tsx
│   ├── components/
│   │   ├── PolicyCard.tsx
│   │   ├── AlertBadge.tsx
│   │   ├── IndustryFilter.tsx
│   │   ├── ComplianceCalendar.tsx
│   │   └── ReportExporter.tsx
│   ├── lib/
│   │   ├── api.ts
│   │   └── auth.ts
│   └── package.json
│
├── infra/                          # AWS / Docker config
│   ├── docker-compose.yml
│   ├── docker-compose.prod.yml
│   └── aws/
│       ├── ecs-task-definition.json
│       └── rds-config.json
│
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── deploy.yml
│
└── README.md
```

---

## 🗓️ Implementation Plan

### Phase 1 — Foundation & Scraping Engine
**Duration: 2–3 weeks**

#### 1.1 Project Setup
- [ ] Initialise monorepo with `backend/`, `scraper/`, `frontend/` directories
- [ ] Set up PostgreSQL + Redis with Docker Compose
- [ ] Configure FastAPI with SQLAlchemy, Alembic, and Pydantic
- [ ] Implement JWT-based authentication (register, login, refresh)
- [ ] Create user model with `industry`, `location`, `plan_tier` fields

#### 1.2 Scraping Engine
- [ ] Build `base_spider.py` with retry logic, rate limiting, and polite crawl delays
- [ ] Implement spiders for priority government portals:
  - `mca_spider.py` → MCA21 portal (company law, ROC filings)
  - `gst_spider.py` → GST Council notifications
  - `rbi_spider.py` → RBI circulars and press releases
  - `mol_spider.py` → Labour law amendments
- [ ] Build `pdf_extractor.py` using `pdfplumber` for gazette notifications
- [ ] Build `playwright_runner.py` for JavaScript-rendered portals
- [ ] Store raw scraped content in `raw_policies` table with deduplication (URL hash)

#### 1.3 Celery Task Queue
- [ ] Set up Celery with Redis as broker
- [ ] Create `scrape_tasks.py` with scheduled periodic scraping (every 6 hours via `celery beat`)
- [ ] Add task monitoring via Flower dashboard

---

### Phase 2 — LLM Summarisation Pipeline
**Duration: 2 weeks**

#### 2.1 Text Processing
- [ ] Build `chunker.py` to split long policy documents into ~2000 token chunks
- [ ] Build `embedder.py` using OpenAI `text-embedding-3-small` or sentence-transformers
- [ ] Store embeddings in PostgreSQL with `pgvector` extension for semantic search

#### 2.2 Summarisation
- [ ] Design `summary_prompt.txt` to extract:
  - **What changed** (plain language, ≤3 sentences)
  - **Who is affected** (industry, company size, region)
  - **Deadline / effective date**
  - **Action required** (yes/no + description)
  - **Severity** (low / medium / high / critical)
- [ ] Build `summariser.py` calling GPT-4o or Claude API
- [ ] Build `classifier.py` to tag policies by industry vertical (FMCG, IT/Software, Manufacturing, Finance, etc.)
- [ ] Store structured results in `policies` table

#### 2.3 Quality Control
- [ ] Add confidence scoring to flag low-quality summaries for human review
- [ ] Implement fallback chain: if primary LLM fails → retry with secondary model

---

### Phase 3 — Alert System
**Duration: 1–2 weeks**

#### 3.1 Alert Trigger Logic
- [ ] Build `alert_service.py` that matches new policies to user profiles (industry + location)
- [ ] Support alert preferences per user: immediate / daily digest / weekly digest
- [ ] Implement severity threshold filtering (e.g., only alert on `high` or `critical`)

#### 3.2 Delivery Channels
- [ ] **Email**: SendGrid integration with HTML email templates (policy summary + CTA)
- [ ] **In-App**: WebSocket push via FastAPI WebSocket endpoint
- [ ] **Webhook**: POST to user-defined URL (for Zapier / Slack integrations) — Pro tier
- [ ] **SMS**: Twilio for critical-severity alerts — Enterprise tier

#### 3.3 Alert Management
- [ ] Mark alerts as read/unread
- [ ] Snooze alerts
- [ ] Track alert open rates for analytics

---

### Phase 4 — React Dashboard
**Duration: 3 weeks**

#### 4.1 Core Pages

**Dashboard Home (`/dashboard`)**
- Policy feed (latest updates, severity badges)
- Quick stats: policies this week, pending actions, upcoming deadlines
- Industry filter chips

**Policy Explorer (`/policies`)**
- Searchable, filterable list of all policies
- Filters: industry, date range, severity, source portal, action required
- Detail modal with full summary, source link, and "export as PDF" option

**Alerts (`/alerts`)**
- Notification center with read/unread state
- Alert preference settings (channels, frequency, severity threshold)

**Compliance Calendar (`/reports`)**
- Calendar view of upcoming compliance deadlines
- Exportable compliance report as PDF (watermarked for Free tier)

**Settings (`/settings`)**
- Industry and location profile setup
- Subscription management (Stripe portal embed)
- Team member invites (Business tier)

#### 4.2 UI Components
- [ ] `PolicyCard.tsx` — card with severity colour coding, deadline badge, and action tag
- [ ] `AlertBadge.tsx` — severity indicator (🔴 Critical / 🟠 High / 🟡 Medium / 🟢 Low)
- [ ] `IndustryFilter.tsx` — multi-select chip filter for industry verticals
- [ ] `ComplianceCalendar.tsx` — deadline calendar using `react-big-calendar`
- [ ] `ReportExporter.tsx` — triggers PDF generation via backend API

#### 4.3 Auth Flow
- [ ] Login / Register pages with form validation
- [ ] Onboarding wizard: select industries → select locations → choose alert preferences
- [ ] Protected routes via NextAuth.js middleware

---

### Phase 5 — Monetisation & SaaS Layer
**Duration: 1–2 weeks**

#### 5.1 Subscription Tiers

| Feature | Free | Pro (₹999/mo) | Business (₹3999/mo) |
|---|---|---|---|
| Industries tracked | 1 | 5 | Unlimited |
| Alert frequency | Weekly | Daily | Real-time |
| Alert channels | Email | Email + In-App | Email + In-App + SMS + Webhook |
| Compliance reports | Watermarked PDF | Full PDF | Full PDF + Excel |
| Team members | 1 | 1 | Up to 10 |
| API access | ❌ | ❌ | ✅ |
| Custom portal requests | ❌ | ❌ | ✅ |

#### 5.2 Stripe Integration
- [ ] Set up Stripe products and price IDs for each tier
- [ ] Build subscription checkout flow (Stripe Checkout)
- [ ] Implement webhook handler for `invoice.paid`, `customer.subscription.deleted`
- [ ] Gate features in backend with plan-tier checks on protected endpoints
- [ ] Embed Stripe Customer Portal for self-serve plan management

#### 5.3 Compliance Report Export
- [ ] PDF generation using `WeasyPrint` or `pdfkit` (Python backend)
- [ ] Reports include: policy list by industry, deadlines, action items, generated date
- [ ] Watermark injection for Free tier reports

---

### Phase 6 — Deployment & DevOps
**Duration: 1 week**

#### 6.1 Docker Setup
- [ ] `Dockerfile` for FastAPI backend
- [ ] `Dockerfile` for Celery worker
- [ ] `docker-compose.yml` for local dev (Postgres + Redis + Backend + Worker + Frontend)
- [ ] `docker-compose.prod.yml` for production overrides

#### 6.2 AWS Infrastructure
- [ ] **ECS Fargate** — Run backend API and Celery worker containers
- [ ] **RDS PostgreSQL** — Managed DB with automated backups
- [ ] **ElastiCache Redis** — Managed Redis for Celery broker + caching
- [ ] **S3** — Store raw scraped PDFs and generated compliance reports
- [ ] **CloudWatch** — Logs and alerting for scraper failures

#### 6.3 Frontend Deployment
- [ ] Deploy Next.js frontend on **Vercel** (auto-deploys from `main` branch)
- [ ] Configure environment variables for API URL and Auth secret

#### 6.4 CI/CD (GitHub Actions)
- [ ] `ci.yml` — Run tests on every PR (pytest for backend, jest for frontend)
- [ ] `deploy.yml` — On merge to `main`, build Docker images → push to ECR → update ECS service

---

## 📡 API Reference

### Authentication
```
POST   /api/auth/register         Register new user
POST   /api/auth/login            Get access + refresh tokens
POST   /api/auth/refresh          Refresh access token
```

### User Profile
```
GET    /api/users/me              Get current user profile
PATCH  /api/users/me              Update industry/location preferences
```

### Policies
```
GET    /api/policies              List policies (filterable by industry, severity, date)
GET    /api/policies/:id          Get single policy detail
GET    /api/policies/search?q=    Semantic search across policies
```

### Alerts
```
GET    /api/alerts                List user's alerts (paginated)
PATCH  /api/alerts/:id/read       Mark alert as read
PATCH  /api/alerts/:id/snooze     Snooze alert
GET    /api/alerts/preferences    Get alert preferences
PUT    /api/alerts/preferences    Update alert preferences
```

### Reports
```
GET    /api/reports               List generated compliance reports
POST   /api/reports/generate      Trigger new compliance report generation
GET    /api/reports/:id/download  Download report as PDF
```

---

## 🗄️ Database Schema

```sql
-- Users
CREATE TABLE users (
  id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email        VARCHAR(255) UNIQUE NOT NULL,
  password     VARCHAR(255) NOT NULL,
  name         VARCHAR(255),
  plan_tier    VARCHAR(20) DEFAULT 'free',   -- free | pro | business
  stripe_id    VARCHAR(255),
  created_at   TIMESTAMP DEFAULT now()
);

-- User industry/location preferences
CREATE TABLE user_preferences (
  id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id      UUID REFERENCES users(id) ON DELETE CASCADE,
  industries   TEXT[],                        -- ['IT/Software', 'FMCG']
  locations    TEXT[],                        -- ['India', 'Maharashtra']
  alert_freq   VARCHAR(20) DEFAULT 'daily',  -- realtime | daily | weekly
  min_severity VARCHAR(20) DEFAULT 'medium'  -- low | medium | high | critical
);

-- Raw scraped policy documents
CREATE TABLE raw_policies (
  id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  source_url   VARCHAR(1024) UNIQUE NOT NULL,
  portal_name  VARCHAR(255),
  raw_text     TEXT,
  raw_pdf_s3   VARCHAR(512),
  scraped_at   TIMESTAMP DEFAULT now(),
  processed    BOOLEAN DEFAULT FALSE
);

-- Processed, summarised policies
CREATE TABLE policies (
  id             UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  raw_policy_id  UUID REFERENCES raw_policies(id),
  title          VARCHAR(512),
  summary        TEXT,
  what_changed   TEXT,
  who_affected   TEXT,
  action_required BOOLEAN DEFAULT FALSE,
  action_desc    TEXT,
  effective_date DATE,
  deadline       DATE,
  severity       VARCHAR(20),               -- low | medium | high | critical
  industries     TEXT[],
  locations      TEXT[],
  source_url     VARCHAR(1024),
  published_at   TIMESTAMP,
  created_at     TIMESTAMP DEFAULT now()
);

-- Alerts
CREATE TABLE alerts (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID REFERENCES users(id) ON DELETE CASCADE,
  policy_id   UUID REFERENCES policies(id),
  channel     VARCHAR(20),                  -- email | sms | inapp | webhook
  is_read     BOOLEAN DEFAULT FALSE,
  sent_at     TIMESTAMP,
  created_at  TIMESTAMP DEFAULT now()
);
```

---

## 🔐 Environment Variables

```env
# Backend
DATABASE_URL=postgresql://user:pass@localhost:5432/policypulse
REDIS_URL=redis://localhost:6379/0
SECRET_KEY=your-jwt-secret-key
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# LLM
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...

# Email
SENDGRID_API_KEY=SG....
FROM_EMAIL=noreply@policypulse.in

# SMS
TWILIO_ACCOUNT_SID=AC...
TWILIO_AUTH_TOKEN=...
TWILIO_PHONE=+1...

# Payments
STRIPE_SECRET_KEY=sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...

# AWS
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
AWS_S3_BUCKET=policypulse-docs
AWS_REGION=ap-south-1

# Frontend (Next.js)
NEXT_PUBLIC_API_URL=https://api.policypulse.in
NEXTAUTH_SECRET=...
NEXTAUTH_URL=https://app.policypulse.in
```

---

## 💰 Monetisation Model

### Revenue Streams
1. **Subscription tiers** — Primary revenue (SaaS MRR)
2. **Compliance report exports** — Included in paid tiers; optional add-on for Free users
3. **API access** — Business tier; enables integration with internal compliance tools
4. **Custom portal onboarding** — One-time fee for adding niche government portals on request (Enterprise)
5. **White-label licensing** — License the platform to legal/accounting firms

### Target Customer Segments
- SME founders and compliance officers
- CA / Legal firms managing multiple SME clients
- HR teams monitoring labour law changes
- Finance teams tracking GST/tax amendments

---

## 🗺️ Roadmap

| Milestone | Target |
|---|---|
| Phase 1 — Scraper MVP (3 portals, raw data) | Week 3 |
| Phase 2 — LLM pipeline live | Week 5 |
| Phase 3 — Alert emails working | Week 6 |
| Phase 4 — Dashboard v1 (feed + alerts) | Week 9 |
| Phase 5 — Stripe payments live | Week 10 |
| Phase 6 — AWS production deploy | Week 11 |
| Beta launch (invite-only) | Week 12 |
| Public launch | Week 16 |

**Post-launch additions:**
- WhatsApp alert channel via Meta Business API
- Mobile app (React Native)
- AI chatbot: "Ask PolicyPulse" for compliance Q&A
- Peer benchmarking: "How are similar businesses handling this?"

---

## 🚀 Getting Started

### Prerequisites
- Python 3.11+
- Node.js 20+
- Docker & Docker Compose
- PostgreSQL 15+ (or use Docker)
- Redis 7+ (or use Docker)

### Local Development Setup

```bash
# 1. Clone the repo
git clone https://github.com/yourusername/policypulse.git
cd policypulse

# 2. Start infrastructure (Postgres + Redis)
docker-compose up -d postgres redis

# 3. Backend setup
cd backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env          # Fill in your API keys

# 4. Run DB migrations
alembic upgrade head

# 5. Start FastAPI server
uvicorn app.main:app --reload --port 8000

# 6. Start Celery worker (new terminal)
celery -A celery_worker.celery_app worker --loglevel=info

# 7. Start Celery beat scheduler (new terminal)
celery -A celery_worker.celery_app beat --loglevel=info

# 8. Frontend setup (new terminal)
cd ../frontend
npm install
cp .env.local.example .env.local   # Fill in API URL
npm run dev
```

The app will be available at:
- **Frontend**: http://localhost:3000
- **Backend API**: http://localhost:8000
- **API Docs**: http://localhost:8000/docs
- **Celery Monitor**: http://localhost:5555 (Flower)

---

## 📄 License

MIT License — see `LICENSE` for details.

---

*Built with ❤️ as part of the RegTech / SME Tools space. PolicyPulse — because compliance shouldn't be a surprise.*
