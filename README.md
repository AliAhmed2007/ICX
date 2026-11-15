# ICX Platform — Industrial Circular Exchange

> **A high-integrity B2B engineering exchange for surplus, critical-path capital equipment (valves, mechanical seals, pumps) across heavy process industries (Oil & Gas, Petrochemicals, Fertilizers, Power Generation).**

[![Node Version](https://img.shields.io/badge/Node.js-%3E%3D24.0.0-339933?logo=node.js)](https://nodejs.org/)
[![pnpm Version](https://img.shields.io/badge/pnpm-%3E%3D9.12.0-F69220?logo=pnpm)](https://pnpm.io/)
[![Python Version](https://img.shields.io/badge/Python-%3E%3D3.12-3776AB?logo=python)](https://python.org/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16%20%2B%20RLS-4169E1?logo=postgresql)](https://www.postgresql.org/)
[![Docker](https://img.shields.io/badge/Docker-Compose%20v2-2496ED?logo=docker)](https://www.docker.com/)

---

## Table of Contents
1. [Core Mission & Safety Mandate](#1-core-mission--safety-mandate)
2. [Monorepo Architecture & Directory Topology](#2-monorepo-architecture--directory-topology)
3. [Unified 5-Step Manual Spare Part Insertion Workflow](#3-unified-5-step-manual-spare-part-insertion-workflow)
4. [Prerequisites & System Requirements](#4-prerequisites--system-requirements)
5. [Quick Start & Local Environment Setup](#5-quick-start--local-environment-setup)
6. [Docker Infrastructure & Network Topology](#6-docker-infrastructure--network-topology)
7. [Daily Developer Workflows & Commands](#7-daily-developer-workflows--commands)
8. [Non-Negotiable System Laws](#8-non-negotiable-system-laws)
9. [Canonical Knowledge Base & Developer Skills](#9-canonical-knowledge-base--developer-skills)
10. [Troubleshooting & FAQ](#10-troubleshooting--faq)

---

## 1. Core Mission & Safety Mandate

Operating equipment in toxic ($\text{H}_2\text{S}$), high-pressure (Class 2500 / 6,000+ PSI), and cryogenic ($-196^\circ\text{C}$) environments means **an incorrect part can cause catastrophic plant failure, lethal chemical releases, or severe environmental disasters**.

### Rule 0: Strict Abstention ("Missing Is Always Better Than a Guess")
- **Never guess, infer, or hallucinate missing mechanical attributes.**
- If an attribute cannot be mathematically verified from a physical nameplate, catalog grammar, or EN 10204 3.1 Mill Test Certificate, the system records `"UNKNOWN"`.
- Missing mandatory attributes automatically route to `REVIEW_REQUIRED`, never to an approximate or fuzzy match.

---

## 2. Monorepo Architecture & Directory Topology

The monorepo is managed with **pnpm workspaces** and **Turborepo**:

```
icx-platform/
├── .ICX_context/          # Canonical Declarative Knowledge Base (00_README.md + 01..09 Specs)
├── .skills/               # 9 Procedural Task Runbooks for Engineers & AI Agents (SKILL_01..09)
├── AGENTS.md              # Universal Directives for AI Coding Assistants
├── CLAUDE.md / GEMINI.md  # Tool-specific agent configurations
├── apps/
│   ├── api/               # NestJS 12 Gateway + BullMQ Worker Daemon + SSE Streamer (Port 4000)
│   └── web/               # Next.js 16 (App Router) + React 19 + shadcn/ui Dynamic Forms (Port 3000)
├── packages/
│   ├── dna-registry/      # 31 Class YAMLs + 8 Shared Master YAMLs + Code Generator
│   ├── contracts/         # TypeScript DTOs, Zod Schemas, UI Manifests, Artifact Policies
│   ├── engine-rules/      # Deterministic Matching Engine, Auto-Inference, DCS 0-100, Crosswalks
│   ├── db/                # PostgreSQL 16 + Drizzle ORM Schema, Migrations, RLS Policies
│   ├── security/          # Cryptographic primitives (HMAC masking, Argon2id, tokens)
│   └── config/            # Shared ESLint, Prettier, and TypeScript configurations
├── services/
│   └── extractor/         # Python 3.12+ FastAPI, OpenCV CLAHE, PaddleOCR, MPN Decoders (Port 8000)
└── infra/
    ├── compose/           # Multi-container Docker Compose definitions & secrets
    ├── postgres/          # Custom PostgreSQL 16 image + 00-init-roles.sql
    └── proxy/             # Nginx 1.27 edge reverse proxy configuration
```

### Monorepo Boundaries & Key Patterns

| Directory | Technology | Role / Responsibility | Key Invariant |
| :--- | :--- | :--- | :--- |
| **`packages/dna-registry`** | YAML + Zod + TS AST | Equipment taxonomy source of truth (**31 governed classes** + **8 shared master registries**) & code generator. | **Zero Runtime YAML Parsing.** Edit YAMLs in `registry/` and run `pnpm --filter @icx/dna-registry run generate`. |
| **`packages/contracts`** | TypeScript + Zod | Master discriminated unions (`TechnicalDnaSchema`, `ExactTechnicalDnaSchema`), `UiManifestRegistry`, `TaxonomyRootManifest`, `DATA_SOURCES`, and `getArtifactPolicyForClass()`. | Pure compile-time & runtime validation contracts. Zero database access. |
| **`packages/engine-rules`**| TypeScript + Vitest | Production deterministic matching engine (`evaluator.ts`), `runDeterministicAutoInference()`, `calculateClassDcs()`, and international crosswalks. | Strict **5-State Match Model**: `EXACT`, `ENGINEERING_EQUIVALENT`, `REVIEW_REQUIRED`, `DISQUALIFIED`, `QUARANTINED`. |
| **`packages/db`** | Drizzle ORM + PostgreSQL 16 | Relational models (`tables.ts`), migrations, RLS security policies. | Runtime queries run as `icx_app` (`NOBYPASSRLS`). Migrations run as `icx_owner`. |
| **`apps/api`** | NestJS 12 + BullMQ | HTTP API gateway, `LocationScopeGuard`, S3 presigned uploads, and Redis SSE streaming. | Enforces PostgreSQL RLS via `RlsDatabaseService.withTenant()`. |
| **`apps/web`** | Next.js 16 + React 19 + shadcn/ui | Unified 5-Step Seller Intake Wizard (Modes A, B, C), dynamic form engine, and RFQ portal. | Renders forms dynamically from `UiManifestRegistry` (`select`, `combobox`, `input_number`, `switch`). |
| **`services/extractor`** | Python 3.12+ / FastAPI / PaddleOCR | 2-Pass AI extraction (OpenCV CLAHE + OCR bounding boxes + Pydantic v2 `<ClassId>ExtractModel`) & MPN decoders. | **Air-gapped.** Zero database access. Enforces Rule 0 (`ConfigDict(extra='forbid')`). |

---

## 3. Unified 5-Step Manual Spare Part Insertion Workflow

The platform governs **31 industrial equipment classes** (**16 Fluid Valve classes** `FLUD-VALV-*` and **15 Mechanical Seal & Dry Gas Seal classes** `MECH-SEAL-*`). Single spare part insertion follows a unified 5-step architecture (see [`.ICX_context/09_MANUAL_INSERTION_WORKFLOW_SPEC.md`](./.ICX_context/09_MANUAL_INSERTION_WORKFLOW_SPEC.md); *Bulk Excel/SAP `.xlsx` multi-part ingestion is deferred to future sprints*):

1. **Step 0 — Warehouse / Plant Scope Selection:**
   - Seller selects `location_id` from their `assignedLocationIds` (enforced by `LocationScopeGuard` on `POST /api/v1/inventory/drafts`).
2. **Step 1 — Identification & Intake (3 Convergent Modes on One Screen):**
   - **Mode A (Smart Photo & Document Dropzone — Default):** Browser computes SHA-256 hashes, uploads Zone A photos & Zone B PDFs directly to S3 via presigned `PUT` URLs (`POST /upload-urls`), triggers BullMQ extraction (`POST /extract`), and streams live SSE events (`GET /events`).
   - **Mode B (Instant OEM Part / Figure Number Decoder — `< 50 ms`):** Seller selects from `CONTROLLED_OEM_MANUFACTURERS` (`VELAN`, `KITZ`, `EMERSON_FISHER`, `EMERSON_CROSBY`, `JOHN_CRANE`, `EAGLEBURGMANN`, `FLOWSERVE`, `CAMERON_WKM`, `PALL_CORPORATION`, `PARKER_PRADIFA`) and enters a regex-validated `oem_part_number` (`^[A-Z0-9\-\._/]{3,50}$`, `POST /decode-mpn`), pre-filling 60–75% of Layers 1–3 synchronously.
   - **Mode C (Guided 3-Click Manual Class Selection — Fallback):** Driven by `TaxonomyRootManifest` and `UiManifestRegistry` (`L2` Family $\rightarrow$ `L3` Category $\rightarrow$ `L4` Governed Class Card with bilingual labels and `included` / `excluded` scope bullets).
3. **Steps 2–5 — Split-Screen Interactive 4-Layer Form & Publication:**
   - **Deterministic Auto-Inference (`runDeterministicAutoInference`):** Automatically derives mathematically locked secondary fields (`NPS` $\leftrightarrow$ `DN`, `body_material` $\rightarrow$ ASME B16.34 material group/temps, `ASME_B16_5` for flanged $\text{NPS} \le 24$, API 600 Stellite seat inserts, SAE AS568 O-ring dimensions, and ISO 2230 elastomer `shelf_life_expiry_date`).
   - **Low-Friction Attachment Policy (`getArtifactPolicyForClass`):** Requires only **2 to 3 physical smartphone photos (Zone A)** to publish (`SEARCHABLE_LISTING` / `RFQ_READY_LISTING`), while treating PDF certificates (`EN_10204_3_1_MTC`, `OEM_GA_DRAWING_PDF`, `TEST_CERTIFICATE`) as **Optional Boosters (Zone B)** that unlock `EXACT_MATCH_ELIGIBLE` once verified by an ICX `STEWARD`.
   - **Data Completeness Score (`DCS 0–100`) & 5 Completeness Stages (`calculateClassDcs`):** Gates every item into `QUARANTINED`, `DRAFT_LISTING` ($\text{DCS} < 50$ or Zone A incomplete), `SEARCHABLE_LISTING` ($\text{DCS} \ge 50$ + Zone A photos), `RFQ_READY_LISTING` ($\text{DCS} \ge 75$), or `EXACT_MATCH_ELIGIBLE` ($\text{DCS} \ge 90$ + `<ClassId>ExactSchema` + Steward-verified PDFs).

---

## 4. Prerequisites & System Requirements

Before setting up the repository, ensure your development machine has:

- **Node.js**: `>= 24.0.0` (LTS recommended)
- **pnpm**: `>= 9.12.0` (Install via `corepack enable` or `npm install -g pnpm@latest`)
- **Python**: `>= 3.12`
- **uv**: Fast Python package manager ([Astral uv](https://docs.astral.sh/uv/))
- **Docker & Docker Compose**: v2+ with Docker Desktop running
- **Git**

---

## 5. Quick Start & Local Environment Setup

Follow these exact steps to set up the repository from scratch:

### Step 1: Clone the Repository
```bash
git clone https://github.com/your-org/icx-platform.git
cd icx-platform
```

### Step 2: Configure Environment Variables
Copy the master environment template to `.env` and `.env.local`:
```bash
# On Linux / macOS
cp .env.example .env
cp .env.example .env.local

# On Windows (PowerShell)
Copy-Item .env.example .env
Copy-Item .env.example .env.local
```

### Step 3: Install Node Dependencies
```bash
pnpm install
```

### Step 4: Sync Python AI Extractor Dependencies
```bash
uv sync --project services/extractor
```

### Step 5: Start Supporting Infrastructure via Docker
Spin up PostgreSQL 16, Redis, and MinIO S3 Object Storage:
```bash
docker compose -f infra/compose/compose.yaml -f infra/compose/compose.dev.yaml up -d postgres redis storage
```
*Wait ~5 seconds for Postgres and MinIO healthchecks to pass.*

### Step 6: Generate Code, Apply Migrations & Seed Database
Compile the equipment taxonomy YAMLs into TypeScript & Python contracts, then run migrations & seed demo data:
```bash
# 1. Compile Taxonomy YAMLs -> TypeScript/Python DTOs & UI manifests
pnpm --filter @icx/dna-registry run generate

# 2. Run Database Migrations & Seed All 11 Tables
pnpm run db:setup

# 3. Launch Drizzle Studio to inspect tables, columns, and records
pnpm run db:studio
```
pnpm --filter @icx/db run db:migrate
```

### Step 7: Verify Everything Runs 100%
Execute the four mandatory validation test gates:
```bash
# Gate 1: Taxonomy Validation Vectors (124/124 across all 31 governed classes)
pnpm --filter @icx/dna-registry run test:vectors

# Gate 2: Deterministic Matching Engine, Real-World User Simulation & Insertion Workflow (30/30 tests)
pnpm --filter @icx/engine-rules run test

# Gate 3: Database Row-Level Security Integrity Test
pnpm --filter @icx/db test:rls

# Gate 4: Python MPN Grammar Decoder Unit Tests
uv run --project services/extractor python -m unittest services/extractor/test_mpn_decoders.py
```

### Step 8: Start the Local Development Stack
You can start all services in parallel, or start them individually:

```bash
# Option A: Start all services via Turborepo (API, Web, Worker)
pnpm run dev

# Option B: Run services in separate terminal tabs for detailed logging
# Terminal 1: NestJS API
pnpm --filter @icx/api run dev:api

# Terminal 2: BullMQ Queue Worker
pnpm --filter @icx/api run dev:worker

# Terminal 3: Next.js Frontend
pnpm --filter @icx/web run dev

# Terminal 4: Python AI Extractor
uv run --project services/extractor uvicorn src.main:app --reload --port 8000
```

---

## 6. Docker Infrastructure & Network Topology

The local and staging environment runs an isolated multi-container network topology:

```
[ Web Browser / Client ]
           │
           ▼ (Ports 80 / 443)
┌─────────────────────────────────────────────────────────────┐
│ proxy (Nginx 1.27 Alpine)                                   │
└──────────┬───────────────────────────┬──────────────────────┘
           │                           │
           ▼ (:3000)                   ▼ (:4000)
┌─────────────────────┐     ┌─────────────────────────────────┐
│ web (Next.js 16)    │     │ api (NestJS 12 Gateway)         │
└─────────────────────┘     └──────────┬──────────────┬───────┘
                                       │              │
                     ┌─────────────────┘              │
                     ▼                                ▼ (:8000)
┌─────────────────────────────────┐        ┌──────────────────┐
│ worker (BullMQ Worker Daemon)   │───────>│ extractor        │
└──────────┬──────────────┬───────┘ (HTTP) │ (Python FastAPI) │
           │              │                └──────────────────┘
           │              │                   (NO DB ACCESS)
           ▼              ▼
┌───────────────────┐  ┌───────────────────┐
│ postgres (PG 16)  │  │ redis (Redis 7)   │
│ Role: icx_app     │  │ (Queue & Cache)   │
└───────────────────┘  └───────────────────┘
```

### Service Endpoints & Web GUIs

| Service | Host URL | Description / Credentials |
| :--- | :--- | :--- |
| **Next.js Web UI** | `http://localhost:3000` | Dynamic intake forms, search catalog, and RFQ deal room. |
| **NestJS API Gateway** | `http://localhost:4000` | Core REST API and authentication endpoints. |
| **AI Extractor (FastAPI)** | `http://localhost:8000/docs` | Swagger interactive OpenAPI documentation for OCR & MPN decoders. |
| **Drizzle Studio (Database GUI)** | `https://local.drizzle.studio` | Visual interactive schema & data explorer (launch via `pnpm run db:studio`). |
| **pgAdmin (PostgreSQL GUI)**| `http://localhost:5050` | Enterprise DB console. User: `admin@icx.test`, Pass: `icx_local_admin_2026`. |
| **MinIO S3 Storage Console**| `http://localhost:9001` | S3 bucket management. User: `icx_storage_admin`, Pass: in `.env`. S3 API on `9000`. |

### The Dual-Role Database Architecture

ICX enforces a strict separation between schema migration and runtime query execution:
1. **`icx_owner` (Migration Role):** Has `SUPERUSER`/`CREATEROLE` privileges. Used solely by `drizzle-kit` or the ephemeral `migrate` container to apply DDL schema changes. **Never used by runtime services.**
2. **`icx_app` (Application Role):** Unprivileged role (`NOSUPERUSER`, `NOCREATEROLE`, `NOBYPASSRLS`). Used by NestJS and BullMQ workers. Bound strictly by PostgreSQL Row-Level Security policies.

---

## 7. Daily Developer Workflows & Commands

### Working with Equipment Taxonomy & Codegen
When adding or updating a governed equipment class (e.g., ball valves, gate valves, mechanical seals):
1. Navigate to [`packages/dna-registry/registry/class_files/`](file:///d:/Work/ICX/icx-platform/packages/dna-registry/registry/class_files/) or [`packages/dna-registry/registry/shared/`](file:///d:/Work/ICX/icx-platform/packages/dna-registry/registry/shared/).
2. Edit or add the `.yaml` specification.
3. Recompile the taxonomy into TypeScript Zod contracts, shadcn UI manifests, engine rules, and Python Pydantic v2 models:
   ```bash
   pnpm --filter @icx/dna-registry run generate
   ```
4. Verify all 124 validation vectors and 30 engine/insertion tests:
   ```bash
   pnpm --filter @icx/dna-registry run test:vectors
   pnpm --filter @icx/engine-rules run test
   ```

### Working with the Database Schema
1. Edit schema definitions in [`packages/db/src/schema/tables.ts`](file:///d:/Work/ICX/icx-platform/packages/db/src/schema/tables.ts).
2. Generate migration SQL files:
   ```bash
   pnpm --filter @icx/db run db:generate
   ```
3. Apply migrations to the local database:
   ```bash
   pnpm --filter @icx/db run db:migrate
   ```
4. Run security tests:
   ```bash
   pnpm run test:rls
   ```

### Working with the AI Extractor & MPN Decoders
1. Decoder implementations live in [`services/extractor/src/grammars/mpn_decoders/`](file:///d:/Work/ICX/icx-platform/services/extractor/src/grammars/mpn_decoders/).
2. Run test suites:
   ```bash
   uv run --project services/extractor python -m unittest services/extractor/test_mpn_decoders.py
   ```

### Monorepo Build Commands
```bash
# Build all packages and applications
pnpm run build

# Run linting across the repository
pnpm run lint

# Format all files (TypeScript, Python, Markdown, YAML)
pnpm run format
```

---

## 8. Non-Negotiable System Laws

Every engineer and AI agent operating in this codebase must adhere to the **Core System Invariants**:

1. **Law 1: Rule 0 — Strict Abstention ("Missing Is Always Better Than a Guess"):** Never hallucinate or interpolate missing engineering data. Missing mandatory attributes are recorded as `"UNKNOWN"` and route to `REVIEW_REQUIRED`.
2. **Law 2: One Governed Class = One Archetype:** Cross-class auto-matching is strictly forbidden (`DISQ-L0-01`). A gate valve can never auto-match a ball valve.
3. **Law 3: Flange & Mating Geometry Compatibility Is Absolute:** Components that cannot physically bolt together leak-free under design pressure are immediately `DISQUALIFIED` (`DISQ-L1-01` to `DISQ-L1-03`).
4. **Law 4: Progressive Clarification Pipeline:** Layer 1 (Boundary/Mating) $\rightarrow$ Layer 2 (Body MOC) $\rightarrow$ Layer 3 (Trim/Sealing) $\rightarrow$ Layer 4 (Actuation & Compliance).
5. **Law 5: 5-State Match Evaluation Model:** Every match evaluates strictly to `EXACT`, `ENGINEERING_EQUIVALENT`, `REVIEW_REQUIRED`, `DISQUALIFIED`, or `QUARANTINED`.
6. **Law 6: Dual-Table Isolation, PostgreSQL RLS & `LocationScopeGuard`:** Private items live in `inventory_item` behind tenant RLS (`icx_app` with `NOBYPASSRLS`) and warehouse scope checks (`assignedLocationIds`). Public listings live in `market_listing` and are 100% tenant-blind.
7. **Law 7: Insertion Pipeline Guardrails:** Zero runtime YAML parsing (compile-time codegen only), Datasheet $\neq$ Physical Proof of Stock (2–3 Zone A physical smartphone photos required to publish; Zone B PDFs are optional boosters), and Controlled OEM Part Numbers (`^[A-Z0-9\-\._/]{3,50}$`, `sufficient_alone: false`).

---

## 9. Canonical Knowledge Base & Developer Skills

For in-depth domain specifications, architectural diagrams, and procedural guides, consult:

### Declarative Knowledge Base ([`.ICX_context/00_README.md`](./.ICX_context/00_README.md))
- [`01_PROJECT_VISION_AND_DOMAIN.md`](./.ICX_context/01_PROJECT_VISION_AND_DOMAIN.md) — Business context, 31 governed classes, and process industry standards (ASME, API, NACE, ISO).
- [`02_SYSTEM_ARCHITECTURE_AND_CONTAINERS.md`](./.ICX_context/02_SYSTEM_ARCHITECTURE_AND_CONTAINERS.md) — Container topology, BullMQ + SSE queues, and network segmentation.
- [`03_NON_NEGOTIABLE_ENGINEERING_LAWS.md`](./.ICX_context/03_NON_NEGOTIABLE_ENGINEERING_LAWS.md) — The 7 engineering & insertion laws, 5 match states, and 5 completeness stages.
- [`04_DATABASE_ARCHITECTURE_AND_PATTERNS.md`](./.ICX_context/04_DATABASE_ARCHITECTURE_AND_PATTERNS.md) — Multi-tenancy, PostgreSQL 16 RLS, and Drizzle ORM `withTenant` patterns.
- [`05_TAXONOMY_AND_CODEGEN_SYSTEM.md`](./.ICX_context/05_TAXONOMY_AND_CODEGEN_SYSTEM.md) — 8 shared registries, 31 class YAMLs, and `scripts/generate.ts` codegen pipeline.
- [`06_AI_EXTRACTION_AND_MPN_DECODERS.md`](./.ICX_context/06_AI_EXTRACTION_AND_MPN_DECODERS.md) — 2-Stage OpenCV CLAHE + PaddleOCR + Pydantic v2 pipeline and MPN regex grammars.
- [`07_DATA_SOURCES_AND_PROVENANCE.md`](./.ICX_context/07_DATA_SOURCES_AND_PROVENANCE.md) — Controlled Data Sources (`SRC-01` to `SRC-18`), Zone A vs. Zone B artifact policy, and ISO 2230 shelf-life rules.
- [`08_UI_FORM_ENGINE_AND_SHADCN_INTEGRATION.md`](./.ICX_context/08_UI_FORM_ENGINE_AND_SHADCN_INTEGRATION.md) — Dynamic `UiFieldMetadata` alignment with shadcn/ui (`select`, `combobox`, `input_number`, `switch`).
- [`09_MANUAL_INSERTION_WORKFLOW_SPEC.md`](./.ICX_context/09_MANUAL_INSERTION_WORKFLOW_SPEC.md) — End-to-End Manual Single Spare Part Insertion Workflow (Modes A/B/C, `DnaAutoInferenceService`, DCS 0–100, REST + SSE APIs).

### Procedural Task Runbooks ([`.skills/README.md`](./.skills/README.md))
- [**Skill 01**](./.skills/SKILL_01_TAXONOMY_CLASS_AUTHORING.md): Authoring & compiling governed equipment classes.
- [**Skill 02**](./.skills/SKILL_02_MPN_DECODER_DEVELOPMENT.md): Building manufacturer part number grammars & Mode B synchronous decoding.
- [**Skill 03**](./.skills/SKILL_03_BACKEND_RLS_DEVELOPMENT.md): Writing NestJS RLS transactional & inventory insertion endpoints.
- [**Skill 04**](./.skills/SKILL_04_FRONTEND_UI_INTEGRATION.md): Building the Unified 5-Step Seller Intake Wizard & consuming UI manifests.
- [**Skill 05**](./.skills/SKILL_05_TEST_VECTORS_AND_VERIFICATION.md): Executing the 124 validation vectors & 30 real-world user simulation/insertion tests.
- [**Skill 06**](./.skills/SKILL_06_DOCUMENT_OCR_PIPELINE.md): 2-Stage Document OCR & spatial bounding-box extraction pipeline.
- [**Skill 07**](./.skills/SKILL_07_DETERMINISTIC_CROSSWALK_MAPPING.md): Deterministic auto-inference & global enterprise crosswalks (UNSPSC, ECLASS, ISO 14224, CFIHOS, HS).
- [**Skill 08**](./.skills/SKILL_08_DATA_COMPLETENESS_SCORING.md): 8-Weight DCS (`0–100`) calculation & 5-Stage completeness gating.
- [**Skill 09**](./.skills/SKILL_09_ASYNC_QUEUE_WORKER.md): Developing BullMQ queue workers & Redis Pub/Sub SSE streams.

---

## 10. Troubleshooting & FAQ

### 1. `FATAL ERROR: Zone Allocation failed - process out of memory` during API build
**Solution:** The Master Discriminated Union across all **31 governed classes** creates a large TypeScript AST. The build script in `apps/api/package.json` is configured with `node --max-old-space-size=4096 ./node_modules/typescript/bin/tsc`. Ensure your machine has at least 4 GB free RAM during compilation.

### 2. PostgreSQL throws `invalid input syntax for type uuid: ""`
**Cause:** Attempting to cast an empty string or unset session variable to UUID: `current_setting('app.current_tenant_id', true)::uuid`.  
**Solution:** Always use the `nullif` safeguard: `nullif(current_setting('app.current_tenant_id', true), '')::uuid`.

### 3. PostgreSQL throws `cannot insert multiple commands into a prepared statement`
**Cause:** Using `SET LOCAL app.current_tenant_id = $1` with parameter binding. PostgreSQL utility statements reject prepared parameters.  
**Solution:** Use the parameter-safe function: `SELECT set_config('app.current_tenant_id', $1, true)`.

### 4. pgAdmin or Drizzle Studio cannot connect to PostgreSQL
- Ensure the containers are running: `docker compose -f infra/compose/compose.yaml -f infra/compose/compose.dev.yaml ps`
- When connecting via pgAdmin, set the server to `postgres` (internal Docker hostname) or `127.0.0.1:5432` if connecting from the host machine.
- User: `icx_owner` (password: `icx_local_owner_secret_2026`) or `icx_app` (password: `icx_local_app_secret_2026`).

---

## License & Intellectual Property
Proprietary & Confidential. All rights reserved by ICX (Industrial Circular Exchange). Unauthorized copying, transfer, or distribution of this software and associated engineering grammars is strictly prohibited.

