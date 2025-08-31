# Qur’an Institute (Qi) — Full Product Repository

This README describes the full, production-ready Qur’an Institute platform (mobile-first, offline-capable, scholar-governed learning platform for Qur’an, Tarbiyah & Sharia). It’s not an MVP doc — it’s the canonical handbook for engineering, ops, governance and contributors for the full system.

#  1. Project summary (one line)
Qi: a global, academy-grade, offline-first mobile learning platform delivering Qur’an, Tarbiyah and Sharia education — scholar-vetted, localized, and integrated into daily life.

#  2. Mission & non-negotiable principles
Mission: Make reliable, source-linked Islamic learning available to any Muslim, anywhere, regardless of connectivity or schedule.


Non-negotiables: Scholar governance, offline-first, low-bandwidth-first, pedagogically sound (mastery + SRS), auditable provenance, privacy-by-default.

#3. High-level system overview (components)
Mobile (primary): Expo React Native app (JavaScript; no TypeScript). PWA fallback for web.


Backend API: Node.js (Express or Fastify) microservices — auth, content, recitation, sync, realtime.


Content CMS: Admin console for scholars, editors, and curriculum team.


Media & Storage: S3-compatible object store + CDN for global delivery.


DBs: PostgreSQL (primary relational), Redis (cache / job queue), Vector DB (semantic search / embeddings: Milvus/Pinecone), ElasticSearch (text search).


Realtime: WebRTC (Jitsi/mediasoup) for live halaqas.


ML infra: Tiny Tajweed advisory, forced-aligner, batch embedding pipeline.


Offline tooling: Pack generator (pack manifest JSON), pack installer (client-side), SD/USB image generator for no-internet seeds.


Monitoring / Observability: Prometheus + Grafana (metrics), Sentry (errors), ELK/Graylog (logs).



4. Key product features (production scope)
Structured curriculum (Qur’an, Tarbiyah, Fiqh) with levels and prerequisites.


Micro-lessons (1–15 min) with transcript + multilingual translations.


Spaced Repetition System (SRS) for memorization and doctrinal facts.


Live cohort courses + recorded halaqas.


Mentor review workflow (audio recitation queue + timestamped comments).


Sharia Board approval workflow + content provenance metadata.


Offline course packs (manifested, resumable downloads, peer-seedable).


Adaptive recommendation engine with madhhab preference filter.


Fatwa request & management system with provenance & publication controls.


Audit logs and content versioning.


GDPR-like privacy controls and encrypted at-rest storage.



5. Tech stack (recommended)
Frontend: Expo (React Native, JS), PWA (React).


Backend: Node.js (Express/Fastify), PostgreSQL, Redis.


Media: S3 (or MinIO for on-prem), CloudFront or equivalent CDN.


ML: Python microservices (for forced-aligner/embeddings). Vector DB: Milvus or Pinecone.


CI/CD: GitHub Actions or GitLab CI. IaC: Terraform.


Auth: OAuth2 + JWT (refresh tokens), device-bound offline tokens.


Dev tooling: Docker for local dev, Docker Compose for integration.



6. Repo layout (canonical)
/ (repo root)
  /frontend               # Expo app (React Native) + PWA
    /components
    /screens
    /services             # api clients, sync engine
    app.json
    package.json
  /backend
    /services
      /api                # express apps (auth, content, recitation)
      /media-processor
      /sync-worker
    /migrations
    /scripts
    package.json
  /cms                    # admin web console for scholars & editors
  /infra                  # terraform modules & kustomize manifests
  /ops
    /packs                # pack generator scripts
    /sd-image             # SD/USB image generator
  /docs
    sharia-board-mou.md
    curriculum-guidelines.md
    privacy-policy.md
    sla.md
  README.md


7. Quickstart (developer: local dev) — copy & run
Prerequisites: Node 18+, npm, Expo CLI, Docker, PostgreSQL (local or Docker), Redis (local or Docker).
Backend (local)
# from repo root
cd backend
cp .env.example .env      # edit values as needed
npm install
npm run dev               # runs nodemon server
# or with docker compose if provided:
docker compose -f docker-compose.dev.yml up --build

.env (example keys — never commit .env)
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/qidb
PORT=4000
S3_ENDPOINT=
S3_BUCKET=
JWT_SECRET=changeme
NODE_ENV=development

Frontend (Expo)
cd frontend
npm install
npx expo start -c
# scan QR with Expo Go or open in simulator

CMS (admin console)
cd cms
npm install
npm run dev


8. API & contract highlights (public surface)
POST /auth/login — returns JWT + refresh token.


GET /courses — list approved courses (filters: language, level, madhhab).


GET /lessons/:id — lesson detail (text, media refs, SRS tokens).


POST /recitations — multipart upload (auth required).


POST /sync/offline — delta-sync from client to server.


POST /scholar/review — submit content for board approval (scholar role required).


Full OpenAPI/Swagger specification lives in /backend/docs/openapi.yml.

9. Offline packs & sync (behavioral contract)
Pack manifest { id, version, files: [{url, checksum, size, type}], sizeBytes } exposed at /packs/:packId/manifest.


Clients download prioritized assets: text → audio → video. Downloads resume and verify checksums.


Client has local SQLite store for SRS, progress, recitation queue; delta-sync pushes operations to /sync/offline.


Conflict resolution: server content authoritative; user progress merged by timestamp. Recitations are append-only.



10. Content governance & Sharia Board
Every doctrinal module must include provenance metadata: {sourceRefs[], authorId, scholarApprovals: [{scholarId, timestamp, notes}]}.


Publishing workflow: Draft → Scholar Review → Board Approved → Public. All edits logged.


Sharia Board MoU and review SLA live in /docs/sharia-board-mou.md.



11. Security, privacy & compliance
Auth: JWT + refresh tokens. Device-bound offline tokens with expiry (e.g., 30 days).


Encryption: AES-256 at rest for user files; TLS everywhere.


Audio uploads: max file size 10MB by default; content scanning + human review queue.


Personal data minimization: store minimal PII; optional pseudonymous accounts.


Exportable learning record for users (right to portability).


Data retention policy and opt-in/opt-out flows defined in /docs/privacy-policy.md.



12. Observability & SLOs
Errors: Sentry (capture exceptions).


Metrics: Prometheus metrics (request latencies, queue sizes, upload throughput).


Logs: structured JSON logs shipped to ELK/Cloud logging.


SLO examples: recitation upload success rate ≥ 99% (24h rolling), Mentor review SLA median ≤ 48 hours (pilot/scale targets vary).



13. Testing & QA
Unit tests: Jest (backend and frontend). No TypeScript.


Integration: Docker Compose integration tests for sync and upload flows.


E2E: Detox for mobile or playwright for PWA.


Accessibility & localization checks: include Arabic layout & RTL testing on CI.



14. Deployment & infra (production notes)
Use IaC (Terraform) — modules in /infra.


Recommended cloud: AWS (EKS or ECS), S3 + CloudFront, RDS (Postgres), ElastiCache (Redis).


Use managed vector DB or Milvus on k8s for embeddings.


CI: GitHub Actions — build, test, security-scan, deploy. Use separate envs for staging and prod.


Blue/green for app releases; database migration via Flyway or node-pg-migrate.



15. Operational runbooks (short)
Recitation backlog increase: scale workers, increase upload throughput, notify Mentor Manager.


Sharia dispute: freeze content, notify Board, create emergency patch.


Data breach: follow incident response plan (contacts in /ops/incident-response.md).



16. Roadmap (next 6–12 months)
Multi-madhhab support & per-region editorial flows.


Tajweed ML advisory + forced-aligner.


Public verifiable certificates (blockchain optional).


Local seed distribution program (SD cards + community hubs).


Institutional partnerships & licensing portal.



17. KPIs & success metrics (organizational)
Activation: % users completing lesson 1 within 7d. Target ≥ 40%.


Retention: 7-day retention for engaged cohort ≥ 30%.


Mentor SLA: median review time ≤ 48 hours.


Offline packs installed per partner mosque: track installs.


NPS ≥ 30 at 6 months.



18. Contribution guide (engineers & scholars)
Engineers: branch from develop, branch naming: feat/<short>, fix/<short>. PRs require 2 approvals and passing CI. Add tests for new logic.


Scholars & content editors: use /cms to prepare drafts; submit via Scholar Review workflow. Use docs/curriculum-guidelines.md for lesson format.



19. Developer contacts & governance
Product Owner: [REPLACE]


Tech Lead: [REPLACE]


Sharia Board Chair: [REPLACE]
 (Replace placeholders in production README with emails/contacts before onboarding.)



20. License & legal
Default: MIT (or choose organization-specific license). Place LICENSE file at repo root. Legal review required for content licensing and third-party tafsir translations.



21. How to get help / next steps
Fork & clone this repo.


Run npm run dev in backend, run npx expo start in frontend.


Seed curriculum content using markdown files under /content and run pack generator script: node ops/packs/generate.js --course courseA.


Invite Sharia Board members and run pilot with one partner mosque.



