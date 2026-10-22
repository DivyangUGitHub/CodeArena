# CodeArena: Dynamic AI Coding-Agent Evaluation Platform

[![Acceptance Status](https://img.shields.io/badge/Status-Production--Ready-success.svg)](#)
[![Python](https://img.shields.io/badge/Python-3.11%20%7C%203.14-blue.svg)](#)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.141+-009688.svg)](#)
[![Next.js](https://img.shields.io/badge/Next.js-16.4+-black.svg)](#)
[![Research Paper](https://img.shields.io/badge/Research-SWE--DIVE%20(NeurIPS%202026)-purple.svg)](#)

**CodeArena** is a dynamic, multi-turn evaluation platform designed to benchmark AI coding agents under realistic developer interaction patterns. 

Built on the research foundation of **SWE-DIVE** (*"SWE-DIVE: Multi-Turn Coding Agent Evaluation with Dynamic User Simulation"*, Brehme et al., NeurIPS 2026), CodeArena shifts evaluation away from static, single-turn prompts to interactive, dynamic multi-turn development sessions.

---

## 1. Research Foundation: SWE-DIVE

Traditional benchmarks (such as SWE-bench and Terminal-Bench) evaluate agents using detailed, single-shot GitHub issues. However, real-world software development with coding agents is iterative and collaborative.

CodeArena faithfully implements the paper's core architecture:
1. **Grounded Benchmark Tasks**: Software repositories containing real issues, isolated execution workspaces, and unit test suites.
2. **Dynamic User Simulator**: An LLM-powered actor grounded in the benchmark problem statement and reference solution.
3. **Three Human Personas**:
   - **Expert Nitpicker**: Maintains a stable overall objective while delivering precise, code-level feedback; inspects git diffs to detect misplaced code, leftover variables, and slicing errors.
   - **Vague Requester**: Gives broad, underspecified instructions; avoids low-level implementation hints; tests agent requirement inference (often triggering large-scale rewrites and regressions).
   - **Mind Changer**: Fundamentally alters the preferred architectural strategy midway through the interaction (e.g. pivoting from recursion to tree flattening) while maintaining the root objective.
   - **Baseline (Control)**: Delivers the full issue prompt in a single turn without iterative feedback.
4. **Two-Stage Conversation Validator**:
   - *Stage 1 (Trajectory)*: Validates Turn 1 subtask plans for coverage, logical progression, and security.
   - *Stage 2 (Turn-Level)*: Enforces persona style rules, prevents out-of-scope drifts, and checks understandability.
   - *Chain-of-Verification Feedback Loop*: Rejects non-compliant simulator prompts and drives bounded regeneration.
5. **Unified Coding-Agent Adapter Layer**: Supports deterministic mocks, live LLM autonomous agents, OpenHands, OpenCode, and Pi.
6. **Task Completion & Regression Evaluator**: Computes target bug resolution (`FAIL_TO_PASS`), side-effect regressions (`PASS_TO_FAIL`), file churn, unrelated file bloat, and token costs.

---

## 2. Platform Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       CodeArena Web Application (Next.js 16)                │
│  - Dashboard  - Wizard  - Live Monitor  - Transcripts  - Diffs  - Reports   │
└──────────────────────────────────────┬──────────────────────────────────────┘
                                       │ REST / SSE
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         CodeArena Core API (FastAPI)                        │
│  - Auth & RBAC  - Secret Redactor  - SSRF Guard  - Path Traversal Guard     │
└──────────────┬───────────────────────┬───────────────────────────┬──────────┘
               │                       │                           │
               ▼                       ▼                           ▼
      ┌────────────────┐     ┌───────────────────┐       ┌──────────────────┐
      │ User Simulator │<--->│ Conv. Validator   │       │ Workspace Engine │
      │ (3 Personas)   │     │ (2-Stage CoV)     │       │ (Git & Sandbox)  │
      └────────────────┘     └───────────────────┘       └─────────┬────────┘
                                                                   │
                                                                   ▼
                                                         ┌──────────────────┐
                                                         │   Coding Agent   │
                                                         │ (Mock, LLM, ...) │
                                                         └──────────────────┘
```

---

## 3. Technology Stack

- **Frontend**: Next.js 16 (App Router, Turbopack), React 19, TypeScript, Tailwind CSS, Lucide React icons.
- **Backend**: Python 3.11+, FastAPI 0.141+, Pydantic 2.13+, SQLAlchemy 2.0+ (relational ORM with async capabilities).
- **Database**: SQLite (zero-config local dev) / PostgreSQL (production).
- **Background Orchestration**: Asynchronous event runner with Server-Sent Events (SSE) telemetry.
- **Security**: PBKDF2-HMAC-SHA256 password hashing, PyJWT tokens, secret scrubbing regex, SSRF protection, path traversal confinement.
- **Testing**: pytest test harness covering unit, integration, and end-to-end evaluation.

---

## 4. Getting Started (Windows PowerShell)

### Prerequisites
- Python 3.11+ (or `py` launcher)
- Node.js 20+ and npm

### 1. Clone & Set Up Environment
```powershell
# Navigate to project root
cd C:\Users\DivyangUpreti\.gemini\antigravity\scratch\codearena

# Copy environment variables
Copy-Item .env.example .env
```

### 2. Start the Backend API Server
```powershell
# In terminal 1:
py -m uvicorn backend.app.main:app --host 127.0.0.1 --port 8000 --reload
```
The FastAPI interactive documentation will be live at: `http://127.0.0.1:8000/docs`

### 3. Start the Next.js Frontend
```powershell
# In terminal 2:
cd frontend
npm run dev
```
The CodeArena web application will be live at: `http://localhost:3000`

---

## 5. Running Automated Tests

Run the full automated test suite (unit tests, security tests, and end-to-end experiment simulation):
```powershell
py -m pytest backend/tests -v
```

### Test Suite Verification Summary:
- `backend/tests/test_simulator.py`: Verified prompt schemas, 3 persona generation paths, and baseline.
- `backend/tests/test_validator.py`: Verified Stage 1 subtask checks, Stage 2 prompt validation, persona style enforcement, and prompt injection detection.
- `backend/tests/test_security.py`: Verified PBKDF2 hashing, JWT generation, regex secret scrubbing, path traversal rejection, and SSRF loopback blocking.
- `backend/tests/test_evaluator.py`: Verified `FAIL_TO_PASS` detection, `PASS_TO_FAIL` regression classification, and golden patch file bloat tracking.
- `backend/tests/test_e2e.py`: Verified complete multi-turn run on Astropy-12907, database persistence, and JSON/CSV/HTML export generation.

---

## 6. Running a Research Experiment from CLI

You can also run experiments directly via the CLI reproduction script:
```powershell
# Run Expert Nitpicker on Astropy-12907
py scripts/run_experiment.py --persona expert_nitpicker --task astropy-12907

# Run all 4 personas for comparative analysis
py scripts/run_experiment.py --run-all-personas --task astropy-12907
```

Exported research reports are automatically written to `experiments/reports/` in JSON, CSV, and standalone HTML formats.

---

## 7. Security Architecture & Threat Model

For the complete threat model, trust boundaries, and verification checklist, see [`docs/SECURITY.md`](docs/SECURITY.md).

Key verified safeguards:
- **Environment Scrubbing**: Sensitive API keys and connection strings are stripped from workspace execution environments.
- **Path Traversal Guard**: Prevents symlinks or relative paths from escaping the isolated sandbox directory.
- **SSRF Blocker**: Rejects localhost, 127.0.0.1, private RFC1918 subnets, and cloud metadata IPs (`169.254.169.254`).
- **Prompt Injection Defense**: Deterministic heuristic rules reject override commands before reaching agent contexts.
- **Automatic Secret Redaction**: All logged text is filtered through regex scrubbers replacing credentials with `[REDACTED]`.

---

## 8. Research Paper vs Engineering Extensions

| Component | In Research Paper | CodeArena Production Extension |
| :--- | :--- | :--- |
| Multi-turn Simulator & 3 Personas | Yes (Section 3.1) | Production prompt templates, bounded retries, and fallback engine |
| 2-Stage Conversation Validator | Yes (Section 3.2) | Pydantic validation schemas, injection detection, and reason feedback |
| Astropy-12907 Case Study | Yes (Section 4.1) | Full unit-testable reproduction fixture with automated pytest verifier |
| Harbor Container Execution | Yes (Section 4) | Dual-mode execution: Docker containerization + Isolated process sandbox |
| Web Application & UI | No (Paper is research only) | Full 14-page Next.js web application with live SSE monitoring |
| Data Export & Reports | No | Standalone HTML, CSV, and JSON export engines |
| User Auth & Database Persistence | No | SQLAlchemy 2.0 relational schema, PBKDF2 auth, and audit logging |
#   C o d e A r e n a  
 