# Utana Leads Pipeline Benchmark

A Flask + SQLite CRM leads pipeline built as a **Mocked Env** for benchmarking agentic harnesses (Sapiens4, Codex CLI, Hermes) against a real, stateful backend with a full audit trail.

## What this is

A fake but functionally real CRM: leads move through 4 stages (`campaign → chat → crm → followup`), every transition is logged with a real actor and timestamp, and two scoring endpoints let you verify an agent's self-reported claims against ground truth rather than trusting them at face value.

## Setup

```bash
python -m venv venv
source venv/bin/activate          # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

Create a `.env` file in the project root with:
```
OPENAI_API_KEY=your-key-here
```

Then start the backend:
```bash
python server.py
```

Runs at `http://localhost:5050`. On first run it creates `northbeam.db` and seeds it with sample data.

## Dashboards

| File | What it is |
|---|---|
| `crm_leads_connected.html` | Leads pipeline board — the main test surface, with an "Acting as" selector so UI-driven actions are correctly attributed |
| `crm_connected.html` | Orders CRM — editable table, tests in-place data editing |
| `calender_connected.html` | Shared calendar — tests booking-conflict detection |
| `chat_connected.html` | Team chat — simple message feed |
| `inbox_connected.html` | Email inbox — includes a deliberate contradiction between two emails |

Also served directly by the backend at `http://localhost:5050/leads` (avoids `file://` restrictions some agent harnesses enforce).

## Key API endpoints

- `GET /api/leads` — current state of all leads
- `POST /api/leads/<id>/advance` — move a lead forward one stage (body: `{"actor": "..."}`)
- `GET /api/leads/<id>/history` — full audit trail for one lead
- `POST /api/verify/<id>` — compare a claimed stage against the real audit log
- `POST /api/eval/deterministic/<id>` — rule-based 0–10 score against real transition history
- `POST /api/eval/llm-judge/<id>` — GPT-based weighted-questionnaire honesty score, comparing an agent's claim against ground truth

## Benchmark results

Full 7-test suite, scorecards, and findings for Sapiens4, Codex CLI, and Hermes are documented separately (shared alongside this repo). Headline finding: agent-reported task status is not reliable in either direction — verified success is sometimes reported as failure, and verified failure has previously been reported as success. Scoring against the real audit trail, not the agent's own claim, is essential.