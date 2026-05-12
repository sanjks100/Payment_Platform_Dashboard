# BNY Payment Platform Dashboard — Project Documentation

> **Version:** 1.0  
> **Research Date:** March 2026  
> **Status:** Phase 1 — Intelligence Dashboard

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Architecture Overview](#2-architecture-overview)
3. [Technology Stack](#3-technology-stack)
4. [Features & Capabilities](#4-features--capabilities)
5. [Data Pipeline](#5-data-pipeline)
6. [Data Models](#6-data-models)
7. [AI Agent System](#7-ai-agent-system)
8. [Frontend Dashboard](#8-frontend-dashboard)
9. [API Reference](#9-api-reference)
10. [Workflow Diagrams](#10-workflow-diagrams)
11. [Configuration & Environment](#11-configuration--environment)
12. [Launch & Operations](#12-launch--operations)
13. [Known Limitations & Future Work](#13-known-limitations--future-work)

---

## 1. Project Overview

The **BNY Payment Platform Dashboard** is a single-page intelligence dashboard designed to research, document, and interact with the BNY Mellon Treasury Payments API. It combines a web scraping pipeline, a structured PostgreSQL database, and a Claude/Gemini-powered AI agent into one unified interface.

### Goals
- Aggregate and normalize BNY Mellon's public API documentation into a queryable database
- Provide a research dashboard covering platform capabilities, API definitions, competitive landscape, market data, and ISO 20022 standards
- Offer a conversational AI agent that can generate, validate, and explain BNY payment API requests in real time

### Scope (Phase 1)
| Section | Coverage |
|---|---|
| Platform Overview | 100% |
| API Platform | 85% |
| Competitive Analysis | 70% |
| Market Data | 90% |
| ISO 20022 | In Progress |

---

## 2. Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                     User's Browser                              │
│              web_page_file/index.html (SPA)                     │
└────────────────────────────┬────────────────────────────────────┘
                             │ HTTP fetch (localhost:8000)
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                  FastAPI Backend  (main.py)                     │
│                  Web_Scraper/  · port 8000                      │
│                                                                 │
│   ┌─────────────────┐        ┌─────────────────── ─────────┐    │
│   │  PaymentAgent   │◄──────►│  LLMClient                  │    │
│   │  agent.py       │        │  Claude / Gemini            │    │
│   └────────┬────────┘        └──────────────── ────────────┘    │
│            │ tool calls                                         │
│   ┌────────┴──────────────────────────────────── ──────┐        │
│   │              Agent Tools                           │        │
│   │  generate.py · validate.py · explain.py            │        │
│   │  code_samples.py                                   │        │
│   └────────┬────────────────────────────────── ────────┘        │
│            │ schema lookup (db.py)                              │
└────────────┼────────────────────────────────────────────────────┘
             ▼
┌─────────────────────────────────────────────────────────────────┐
│              PostgreSQL Database                                │
│              api_definitions table                              │
│              api_definitions_quarantine table                   │
└────────────────────────────┬────────────────────────────────────┘
                             ▲
                             │ written by pipeline
┌────────────────────────────┴────────────────────────────────────┐
│                   Data Ingestion Pipeline                       │
│   bny_crawler.py → page_segmenter.py → extractor.py             │
│   → database_writer.py → api_definitions                        │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Technology Stack

### Backend
| Component | Technology | Version / Notes |
|---|---|---|
| Web Framework | **FastAPI** | Async REST API, auto-generates OpenAPI docs |
| ASGI Server | **Uvicorn** | `--reload` mode for development |
| AI Provider (default) | **Anthropic Claude** | `claude-haiku-4-5-20251001` |
| AI Provider (alternate) | **Google Gemini** | `models/gemini-2.0-flash-lite` |
| Database | **PostgreSQL** | Accessed via `psycopg2` |
| ORM / Validation | **Pydantic v2** | Schema validation on ingestion |
| Environment Config | **python-dotenv** | `.env` file at project root |

### Web Scraping Pipeline
| Component | Technology | Notes |
|---|---|---|
| Headless Browser | **Selenium + ChromeDriver** | Handles BNY's JavaScript SPA |
| Driver Management | **webdriver-manager** | Auto-downloads ChromeDriver |
| HTML Parsing | **BeautifulSoup4** | Parses rendered DOM |
| PDF Parsing | **pdfplumber** | Extracts content from PDF docs |
| HTTP Client | **requests** | Fallback for static pages |

### Frontend
| Component | Technology | Notes |
|---|---|---|
| Dashboard | **Vanilla HTML/CSS/JS** | Single file `index.html`, no build step |
| Fonts | **Google Fonts** | Inter typeface |
| Charts | Inline SVG / CSS | No external chart library |
| API Communication | **fetch API** | Calls `http://localhost:8000` |

### DevOps / Launch
| Component | Technology | Notes |
|---|---|---|
| Launch Script | **PowerShell** (`start.ps1`) | Manages port, starts backend, opens browser |
| Shortcut | **Windows Batch** (`Launch_Dashboard.bat`) | Double-click launcher |

---

## 4. Features & Capabilities

### 4.1 Intelligence Dashboard (Sections 1–6)

The dashboard is a multi-section research tool. Each section is navigated via the top navigation bar.

| # | Section | Description |
|---|---|---|
| 1 | **Overview** | Platform summary, BNY stock ticker, research date, coverage metrics |
| 2 | **Platform & Capabilities** | BNY payment platform product breakdown and capability matrix |
| 3 | **API Platform** | API definitions, endpoints, methods, field rules pulled from the database |
| 4 | **ISO 20022** | ISO 20022 message standards mapping relevant to BNY payment types |
| 5 | **Market Data** | Market context and competitive positioning data |
| 6 | **Competitive** | Competitor landscape analysis for payment platform features |
| 7 | **AI Agent** | Interactive chat agent (see Section 4.2) |

### 4.2 AI Payment Agent (Section 7)

The AI Agent is the interactive core of the dashboard. It connects the frontend directly to the backend LLM via the `/chat` endpoint.

#### Quick Action Buttons
Pre-built prompts that fire with a single click:

| Button | Prompt Sent to Agent | Intent |
|---|---|---|
| ⚡ Generate ACH | `"Generate a valid ACH payment request"` | `generate` |
| ⚡ Generate IAT | `"Generate a valid IAT payment request"` | `generate` |
| ⚡ Generate WIRE | `"Generate a valid WIRE payment request"` | `generate` |
| ⚡ Generate RTP | `"Generate a valid RTP payment request"` | `generate` |
| ⚡ Generate SWIFT | `"Generate a valid SWIFT payment request"` | `generate` |
| Python | `"Generate a Python code sample for an ACH payment"` | `code_sample` |
| Java | `"Generate a Java code sample for an ACH payment"` | `code_sample` |
| Node.js | `"Generate a Node.js code sample for an ACH payment"` | `code_sample` |
| C# | `"Generate a C# code sample for an ACH payment"` | `code_sample` |
| cURL | `"Generate a cURL code sample for an ACH payment"` | `code_sample` |

#### Chat Window vs Output Panel

| | Chat Window (Left) | Output Panel (Right) |
|---|---|---|
| Content | Full conversational reply — explanations, field descriptions, context | Raw extracted code block or JSON only |
| Always populates | ✅ Yes | Only when reply contains a ` ``` ` fenced code block |
| Panel title | — | `generate` → `"ACH Request JSON"`, `code_sample` → `"Python Code Sample"`, `validate` → `"Validation Result"` |
| Copy button | ❌ | ✅ One-click copy to clipboard |

#### Agent Capabilities
- **Generate** — Produces a fully populated payment initiation JSON for ACH, IAT, WIRE, RTP, or SWIFT using live database schema + Claude
- **Validate** — Accepts a pasted JSON payload and checks it against the stored schema; reports every issue with plain-English fix instructions
- **Explain** — Explains any field, validation error, or payment concept in developer-friendly language
- **Code Samples** — Generates working API call code in Python, Java, Node.js, C#, or cURL with inline comments and error handling
- **Conversational Fallback** — Any message that doesn't match a known intent goes directly to Claude for a free-form answer

#### Session Management
- A **New Session** button clears conversation history and resets the chat
- Session state is held in-process on the backend (`PaymentAgent.history`)

---

## 5. Data Pipeline

The data pipeline is a four-stage ETL process that populates the PostgreSQL database from BNY Mellon's public API documentation website.

### Stage 1 — Crawl (`bny_crawler.py`)
- Uses **Selenium with headless Chrome** to render BNY Mellon's JavaScript SPA at `https://marketplace.bnymellon.com/treasury/api-library/`
- Extracts all API page links from the live DOM
- Fetches each page with a 10-second render wait to allow JavaScript to settle
- Also supports **PDF extraction** via `pdfplumber` for documentation provided as PDFs
- Raw page content is saved as JSON files to `Web_Scraper/bny_raw_pages/`

### Stage 2 — Segment (`bny_segmenter/page_segmenter.py`)
- Reads raw JSON files and splits each page into logical sections:
  - API Details (endpoint, method, description)
  - Headers
  - Request Schema
  - Response Schema
  - JSON Examples
  - Field Rules
  - Code Samples
  - Error Codes
- Output written to `Web_Scraper/bny_segmented_pages/`

### Stage 3 — Extract (`extractor.py`)
- `PageExtractor` reads all segmented JSON files
- `ApiDetailsParser` — extracts endpoint path using regex pattern `{base-url}(/path...)`; infers HTTP method from filename keywords (e.g. "initiate" → POST, "delete" → DELETE, "track" → GET)
- `HeadersParser` — identifies header names using a known-headers whitelist and noise-word filter
- `FieldRulesParser`, `JsonExamplesParser`, `CodeSamplesParser`, `ErrorCodesParser` — each parse their respective sections
- Returns validated records and a list of skipped records (those missing an endpoint)

### Stage 4 — Write (`database_writer.py` / `pipeline.py`)
- `DataNormalizer` — standardizes field names, sanitizes types, validates JSON examples
- `ApiDefinitionParams` (Pydantic model) — validates the full record structure before any DB write
- **Content hashing** (SHA-256 on normalized payload) — skips unchanged records on re-runs
- **Versioning** — if content has changed for the same `(api_name, endpoint, method)`, a new version row is inserted
- **Quarantine** — records that fail Pydantic validation are written to `api_definitions_quarantine` instead of the main table
- A `pipeline_report.json` is generated after each run summarising: inserted, skipped (no changes), skipped (no endpoint), and failed records

---

## 6. Data Models

### 6.1 `api_definitions` Table

The primary table storing all scraped and normalized BNY API definitions.

| Column | Type | Description |
|---|---|---|
| `id` | `UUID` | Primary key, auto-generated |
| `api_name` | `TEXT` | Human-readable API name (e.g. `"Initiate Ach Credit Transfer"`) |
| `endpoint` | `TEXT` | API path (e.g. `/payments/v1/payments`) |
| `method` | `TEXT` | HTTP method (`POST`, `GET`, `DELETE`, `PUT`) |
| `version` | `INT` | Increments when content changes on re-run |
| `headers` | `JSONB` | Required/optional request headers |
| `request_schema` | `JSONB` | Field definitions for the request body |
| `response_schema` | `JSONB` | Field definitions for the response body |
| `json_examples` | `JSONB` | Array of example request/response payloads |
| `field_rules` | `JSONB` | Validation rules, enums, constraints per field |
| `code_samples` | `JSONB` | Pre-scraped code examples |
| `error_codes` | `JSONB` | Known error codes and descriptions |
| `source_url` | `TEXT` | Original BNY documentation URL |
| `content_hash` | `TEXT` | SHA-256 of normalized payload for change detection |
| `created_at` | `TIMESTAMPTZ` | Insertion timestamp |

**Unique Constraint:** `(api_name, endpoint, method, version)`

### 6.2 `api_definitions_quarantine` Table

Holds records that failed Pydantic validation during ingestion.

| Column | Type | Description |
|---|---|---|
| `id` | `UUID` | Primary key |
| `raw_payload` | `JSONB` | The raw record that failed validation |
| `validation_errors` | `JSONB` | List of Pydantic validation errors |
| `created_at` | `TIMESTAMPTZ` | Insertion timestamp |

### 6.3 Payment Type → Schema Mapping

The `get_api_schema()` function in `db.py` maps each payment type to the correct initiation endpoint using a priority-ordered exact name lookup:

| Payment Type | Primary Schema | Fallback Schema |
|---|---|---|
| `ACH` | `Initiate Ach Credit Transfer` | `Initiate Ach Debit Transfer` |
| `IAT` | `Initiate Ach Credit Transfer` | `Initiate Ach Debit Transfer` |
| `WIRE` | `Initiate Credit Transfer` | `Initiate Debit Transfer` |
| `RTP` | `Initiate Realtime Payment` | — |
| `SWIFT` | `Initiate Credit Transfer` | `Initiate Debit Transfer` |

> **Design Decision:** WIRE and SWIFT share the `Initiate Credit Transfer` schema because BNY's database does not have distinct "wire" or "swift" named entries. The system prompt in `prompts.py` instructs the LLM to differentiate them by payment rail context (e.g. SWIFT uses `/payments/v1/gpi/payments`).

### 6.4 Pydantic Validation Model (`ApiDefinitionParams`)

```python
class ApiDefinitionParams(BaseModel):
    id:              Optional[str]
    api_name:        str                        # required
    endpoint:        str                        # required
    method:          str                        # required
    headers:         Dict[str, Any]             # default: {}
    request_schema:  Dict[str, Any]             # default: {}
    response_schema: Dict[str, Any]             # default: {}
    json_examples:   List[Dict[str, Any]]       # default: []
    field_rules:     Dict[str, Any]             # default: {}
    code_samples:    List[Dict[str, Any]]       # default: []
    error_codes:     Dict[str, Any]             # default: {}
    source_url:      Optional[str]
```

---

## 7. AI Agent System

### 7.1 Intent Detection

The agent uses keyword matching to route each user message to the correct tool. No LLM call is made for routing — it is purely deterministic.

| Intent | Keywords | Tool Called |
|---|---|---|
| `generate` | generate, create, build, make, give me, show me a, sample, example request | `generate_payment_request()` |
| `validate` | validate, check, verify, is this valid, review my, look at this json | `validate_payment_json()` |
| `explain` | explain, what does, why is, what is, tell me about, help me understand | `explain_error()` |
| `code_sample` | code, sample code, python, java, node, c#, csharp, curl, snippet, how do i call | `generate_code_sample()` |
| `conversational` | *(none of the above match)* | Direct `LLMClient.chat()` |

**Payment Type Detection:** Scans the message for `ach`, `iat`, `wire`, `rtp`, `swift`. Defaults to `ACH` if none found.

**Language Detection (for code samples):** Scans for `python`, `java`, `node`, `csharp`, `c#`, `curl`. Defaults to `python`.

### 7.2 LLM Client (`llm_client.py`)

A unified client that supports both providers behind the same interface:

```
LLMClient
├── .generate(prompt)     → single-turn, used by all tools
└── .chat(message)        → stateful multi-turn, used by conversational fallback
```

**Provider switching:** Set `AGENT_PROVIDER=claude` or `AGENT_PROVIDER=gemini` in `.env`.

| Setting | Claude | Gemini |
|---|---|---|
| Default Model | `claude-haiku-4-5-20251001` | `models/gemini-2.0-flash-lite` |
| Override Env Var | `CLAUDE_MODEL` | `GEMINI_MODEL` |
| API Key Env Var | `CLAUDE_API_KEY` | `GEMINI_API_KEY` |

### 7.3 Tool Functions

| File | Function | What It Does |
|---|---|---|
| `tools/generate.py` | `generate_payment_request()` | Fetches schema from DB → formats `GENERATE_PROMPT` → calls `llm.generate()` |
| `tools/validate.py` | `validate_payment_json()` | Fetches schema → formats `VALIDATE_PROMPT` with user JSON → calls `llm.generate()` → heuristically determines `valid: true/false` |
| `tools/explain.py` | `explain_error()` | Formats `EXPLAIN_PROMPT` → calls `llm.generate()` |
| `tools/code_samples.py` | `generate_code_sample()` | Fetches schema for endpoint/headers → formats `CODE_SAMPLE_PROMPT` → calls `llm.generate()` |

### 7.4 System Prompt Design

The system prompt (`SYSTEM_PROMPT` in `prompts.py`) establishes the agent's persona and rules:
- Identity: Expert BNY Payments API Assistant
- Always wraps JSON in ` ```json ``` ` fences (so the frontend can extract it to the Output panel)
- Always wraps code in the correct language fence
- Validation must report every issue, not stop at the first
- Payment type routing rules are included so the LLM knows which endpoint and fields apply per rail

---

## 8. Frontend Dashboard

### 8.1 Structure

The entire frontend is a single HTML file: `web_page_file/index.html`.

- **No build step, no npm, no framework** — pure HTML, CSS, and JavaScript
- All sections are rendered in the DOM simultaneously and shown/hidden via `navigateTo(section)` JavaScript
- The file is opened directly in the browser using `file://` protocol

### 8.2 Navigation

```
Top Nav Bar
├── 1. Overview
├── 2. Platform & Capabilities
├── 3. API Platform
├── 4. ISO 20022
├── 5. Market Data
├── 6. Competitive
└── 7. ⚡ API Agent          ← highlighted in gold
```

### 8.3 Agent UI Components

| Element | ID | Purpose |
|---|---|---|
| Chat message container | `agentMessages` | Displays conversation history |
| Text input | `agentInput` | User types message here |
| Send button | `agentSendBtn` | Triggers `agentSend()` |
| Status dot | `agentStatusDot` | Green = idle, pulsing = loading, red = error |
| Output code panel | `agentCodeContent` | Shows extracted JSON or code block |
| Output panel title | `agentCodeTitle` | Dynamically set based on intent |
| Copy button | `agentCopyBtn` | Copies output panel to clipboard |
| New Session button | — | Calls `/reset` endpoint, clears chat |

### 8.4 Key JavaScript Functions

| Function | Description |
|---|---|
| `agentQuickAction(text)` | Fills input and calls `agentSend()` |
| `agentSend()` | POSTs to `/chat`, handles response, routes to chat + output panel |
| `agentExtractCodeBlock(text)` | Regex extracts first ` ``` ` block from reply |
| `agentFormatReply(text)` | Converts markdown (bold, inline code, fences) to HTML |
| `agentSetCode(content, title)` | Updates the output panel |
| `agentReset()` | Calls `/reset`, clears chat UI and output panel |

---

## 9. API Reference

The backend exposes the following REST endpoints at `http://localhost:8000`.

### `GET /`
Health check.
```json
{ "status": "ok", "service": "BNY Payment API Agent" }
```

### `POST /chat`
Main conversational endpoint. Auto-detects intent.
```json
// Request
{ "message": "Generate a valid ACH payment request" }

// Response
{
  "intent": "generate",
  "payment_type": "ACH",
  "endpoint": "/payments/v1/payments",
  "method": "POST",
  "reply": "..."
}
```

### `POST /reset`
Clears the in-process conversation history for the current session.

### `POST /generate`
Direct generate endpoint (bypasses intent detection).
```json
// Request
{ "payment_type": "WIRE" }
```

### `POST /validate`
Direct validate endpoint.
```json
// Request
{ "payment_type": "ACH", "payload": { ... } }
```

### `POST /explain`
Direct explain endpoint.
```json
// Request
{ "field": "standardEntryClassCode", "error": "missing required field", "payment_type": "ACH" }
```

### `POST /code`
Direct code sample endpoint.
```json
// Request
{ "payment_type": "RTP", "language": "python", "payload": { ... } }
```

> Full interactive API docs available at `http://localhost:8000/docs` (FastAPI Swagger UI) when the backend is running.

---

## 10. Workflow Diagrams

### 10.1 Full System Startup Flow

```
Double-click Launch_Dashboard.bat
        │
        ▼
start.ps1 executes
        │
        ├─► Check port 8000
        │       ├── In use? → Kill old process
        │       └── Free? → Continue
        │
        ├─► Start backend in new PowerShell window
        │       └── uvicorn main:app --host 0.0.0.0 --port 8000 --reload
        │
        ├─► Poll http://localhost:8000/ every 2s (up to 30s)
        │       └── 200 OK received → Backend ready
        │
        └─► Open web_page_file/index.html in default browser
```

### 10.2 Data Ingestion Pipeline Flow

```
bny_crawler.py
    │
    ├── Selenium headless Chrome → BNY Marketplace SPA
    ├── Wait 10s for JS render
    ├── Extract all /treasury/api-library/* links
    ├── Fetch each page → save raw JSON
    └── Output: bny_raw_pages/*.json
        │
        ▼
page_segmenter.py
    │
    ├── Read each raw JSON
    ├── Split into sections: api_details, headers, request_schema,
    │   response_schema, json_examples, field_rules, code_samples, error_codes
    └── Output: bny_segmented_pages/*.json
        │
        ▼
extractor.py (PageExtractor)
    │
    ├── ApiDetailsParser   → endpoint path + HTTP method
    ├── HeadersParser      → header name/required/description
    ├── FieldRulesParser   → field constraints and types
    ├── JsonExamplesParser → validated JSON example payloads
    ├── CodeSamplesParser  → pre-scraped code snippets
    └── ErrorCodesParser   → error code catalog
        │
        ├── Valid records   ──────────────────────────────┐
        └── Skipped records (no endpoint) → pipeline_report.json │
                                                                  ▼
database_writer.py (DatabaseWriter)
    │
    ├── DataNormalizer.normalize_raw_data()
    ├── ApiDefinitionParams (Pydantic) → validation
    │       ├── FAIL → api_definitions_quarantine
    │       └── PASS → continue
    ├── SHA-256 content hash → compare to existing
    │       ├── SAME → skip (no changes)
    │       └── DIFFERENT / NEW → insert new version row
    └── api_definitions table ← inserted record
```

### 10.3 Chat Agent Request Flow

```
User types message / clicks Quick Action button
        │
        ▼
agentSend() [frontend]
    │
    └── POST /chat  { "message": "Generate a valid ACH payment request" }
            │
            ▼
    PaymentAgent.chat() [agent.py]
        │
        ├── detect_payment_type()  → "ACH"
        ├── Keyword scan for intent
        │
        ├── Intent = GENERATE
        │       │
        │       └── generate_payment_request("ACH", llm_client)
        │               │
        │               ├── get_api_schema("ACH")  [db.py]
        │               │       └── SELECT * FROM api_definitions
        │               │           WHERE api_name = 'Initiate Ach Credit Transfer'
        │               │
        │               ├── Format GENERATE_PROMPT with schema context
        │               └── llm_client.generate(prompt)
        │                       └── Anthropic Claude API call
        │
        ├── Intent = VALIDATE
        │       └── validate_payment_json() → schema lookup + VALIDATE_PROMPT
        │
        ├── Intent = EXPLAIN
        │       └── explain_error() → EXPLAIN_PROMPT
        │
        ├── Intent = CODE_SAMPLE
        │       └── generate_code_sample() → schema lookup + CODE_SAMPLE_PROMPT
        │
        └── Intent = CONVERSATIONAL
                └── llm_client.chat(message)  ← stateful multi-turn
        │
        ▼
    JSON response  { intent, payment_type, reply, ... }
        │
        ▼
agentSend() [frontend] receives response
    │
    ├── agentAppendMessage('assistant', agentFormatReply(reply))
    │       └── Renders in Chat Window (left panel)
    │
    └── agentExtractCodeBlock(reply)
            ├── Code block found?
            │       └── agentSetCode(block, title) → Output Panel (right panel)
            └── No code block → Output Panel unchanged
```

### 10.4 Schema Resolution Decision Tree

```
get_api_schema(payment_type) called
        │
        ▼
Does payment_type have an exact name map?
    │
    ├── YES → Try each candidate in priority order
    │           │
    │           ├── Query: SELECT * FROM api_definitions WHERE api_name = <candidate>
    │           │
    │           ├── Row found? → Return schema ✅
    │           └── Not found? → Try next candidate
    │
    └── NO → Fallback LIKE search
                WHERE LOWER(api_name) LIKE '%<payment_type>%'
                AND method = 'POST'
                AND api_name LIKE 'initiat%'
                │
                ├── Row found? → Return schema ✅
                └── Not found? → Return None
                                    │
                                    └── LLM uses built-in BNY knowledge
```

---

## 11. Configuration & Environment

### `.env` File (project root)

```ini
# AI Provider: 'claude' (default) or 'gemini'
AGENT_PROVIDER=claude

# Claude (Anthropic)
CLAUDE_API_KEY=sk-ant-...
# CLAUDE_MODEL=claude-haiku-4-5-20251001   # optional override

# Gemini (Google)
GEMINI_API_KEY=AIza...
# GEMINI_MODEL=models/gemini-2.0-flash-lite  # optional override

# PostgreSQL
DB_NAME=api_db
DB_USER=postgres
DB_PASSWORD=...
DB_HOST=127.0.0.1
DB_PORT=5432
```

### Switching AI Providers

Change `AGENT_PROVIDER` in `.env` and restart the backend. No code changes required.

| Provider | Best For |
|---|---|
| Claude (Haiku) | Speed, cost efficiency, strong instruction following |
| Gemini Flash Lite | Google ecosystem integration, alternative provider |

---

## 12. Launch & Operations

### Starting the Dashboard

**Option A — Double-click (recommended):**
```
Launch_Dashboard.bat
```

**Option B — PowerShell:**
```powershell
.\start.ps1
```

**Option C — Manual:**
```powershell
# Terminal 1 — Backend
cd Web_Scraper
python -m uvicorn main:app --host 0.0.0.0 --port 8000 --reload

# Terminal 2 — Open dashboard
Start-Process "web_page_file\index.html"
```

### `start.ps1` Behaviour
1. Checks if port 8000 is occupied → kills old process if needed
2. Launches backend in a **new visible PowerShell window**
3. Polls `http://localhost:8000/` every 2 seconds (timeout: 30s)
4. Opens `web_page_file/index.html` in the default browser once backend is confirmed healthy

### Running the Data Pipeline
```powershell
cd Web_Scraper

# Step 1: Crawl BNY website (requires internet + Chrome)
python bny_crawler.py

# Step 2: Segment raw pages
python bny_segmenter/page_segmenter.py

# Step 3 + 4: Extract and write to database
python pipeline.py
```
Results are logged to console and summarised in `pipeline_report.json`.

### Database Initialisation
```powershell
# Run once to create tables and indexes
psql -U postgres -d api_db -f Web_Scraper/init_db.sql
```

---

## 13. Known Limitations & Future Work

### Current Limitations

| Area | Limitation |
|---|---|
| **Request schemas** | Many `request_schema` entries in the DB contain only a stub `metadata` field — the scraper did not fully parse the nested field tables from BNY's SPA. The LLM falls back to its own BNY knowledge to fill gaps. |
| **WIRE / SWIFT** | No dedicated database entries exist with "wire" or "swift" in the api_name. Both share the `Initiate Credit Transfer` schema. The LLM's system prompt differentiates them by payment rail context. |
| **Session state** | Conversation history is held in-process (single `PaymentAgent` instance per server process). Multi-user concurrent sessions would share state. |
| **Authentication** | The backend API has no authentication. It is designed for local use only. |
| **Frontend file protocol** | The dashboard opens via `file://` protocol. Some browsers restrict fetch calls from `file://` to `localhost`. Chrome allows it; others may require a local HTTP server. |
| **IAT specificity** | IAT resolves to the same schema as ACH Credit Transfer. IAT-specific fields (foreign address, IBAN) are handled by the LLM prompt only, not enforced by schema data. |

### Recommended Future Improvements

1. **Re-run the pipeline** with an improved segmenter that fully parses BNY's field tables to populate complete `request_schema` entries
2. **Add WIRE and SWIFT specific entries** to the database either via additional scraping or manual entry
3. **Add per-session isolation** by keying the `PaymentAgent` to a session ID passed from the frontend
4. **Serve the frontend** via a FastAPI `StaticFiles` mount to eliminate `file://` protocol constraints
5. **Add Phase 2 sections** to the dashboard: deeper ISO 20022 field mapping, payment status tracking, and webhook event documentation
6. **CI pipeline** to re-run the scraper on a schedule and detect API changes via the content hash versioning sys