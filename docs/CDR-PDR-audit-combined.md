# TurkerAI — Combined CDR/PDR Audit Report
**Date:** April 19, 2026  
**Auditors:** Claw (OpenClaw) + Gemini + Claude Code  
**Methodology:** Aerospace-style Concept/Preliminary Design Review  
**Verdict: CONDITIONAL FAIL — 8 Critical, 5 Major, 7 Minor findings**

---

## 1. REQUIREMENTS TRACEABILITY & VALIDITY

### Finding 1.1 — CRITICAL: Market Size Projection Inflated
**Severity:** 9 | **Occurrence:** 8 | **Detection:** 7 | **RPN:** 504

The $5.3B→$34.72B (CAGR 27.9%) from Mordor Intelligence conflates **AI training data services** (high growth) with **crowd-sourced micro-tasks** (mature/declining). MTurk HIT volume has been declining since 2020 as requesters move to LLM-based labeling.

**Evidence:** Amazon's own shift from MTurk to SageMaker Ground Truth; Scale AI's pivot from micro-tasks to RLHF.
**Impact:** TAM for the actual target market (MTurk + Clickworker workers) is likely $50-200M, not $34B.
**Recommendation:** Use MTurk-specific data — ~500K registered workers, ~100K active. Addressable market is workers × $14/mo × 12 = ~$17M/yr max at current scale.

### Finding 1.2 — MAJOR: Worker Earnings Data Stale
**Severity:** 7 | **Occurrence:** 6 | **Detection:** 4 | **RPN:** 168

The $2-6/hr earnings data from Fowler et al. (2022) predates ChatGPT's impact on survey availability. Many academic requesters have moved to Prolific or AI-generated data, reducing MTurk survey volume.

**Recommendation:** Conduct fresh survey of 100+ MTurk workers on current earnings, HIT availability, and tool usage.

### Finding 1.3 — MINOR: ROI Calculator Math Wrong
**Severity:** 5 | **Occurrence:** 9 | **Detection:** 8 | **RPN:** 360

Analysis claims: "$4/hr × 20hrs/wk = $320/wk". That's $320/wk = $1,280/mo unaided. With AI: $8/hr × 20hrs = $640/wk = $2,560/mo. Net gain: $1,280/mo, not $305/mo. The math in the analysis is inconsistent and undermines credibility.

**Recommendation:** Fix arithmetic. Use realistic 1.5x improvement (not 2x), yielding $640/mo gain.

---

## 2. TECHNICAL FEASIBILITY AUDIT

### Finding 2.1 — CRITICAL: Browser Use 78% Success Rate Misapplied
**Severity:** 10 | **Occurrence:** 8 | **Detection:** 6 | **RPN:** 480

The 78% success rate comes from Browser Use's own benchmark on 100 "hard tasks" — likely diverse web tasks, not repetitive MTurk DOMs. Actual success rate on MTurk-specific tasks is UNKNOWN.

**Key risks:**
- MTurk uses iframes with dynamic content loading — Browser Use may not handle this
- Attention checks require human-like behavior (reading speed, scroll patterns)
- Many surveys use Qualtrics/SurveyGizmo with custom JS that breaks DOM parsing
- Service Workers in MV3 have 5-minute idle timeout — long tasks will kill the background process

**Recommendation:** Before any development, run a 2-week technical spike: test Browser Use (or open-source equivalent) on 50 real MTurk HITs. Measure actual success rate. If <60%, rethink architecture.

### Finding 2.2 — CRITICAL: Gemini Flash Insufficient for Attention Checks
**Severity:** 9 | **Occurrence:** 7 | **Detection:** 5 | **RPN:** 315

Attention check detection requires nuanced understanding of survey design patterns ("Select 'Strongly Agree' for this item", "What is 2+2?"). Gemini Flash at $0.72/1M tokens has insufficient reasoning for reliable detection.

**Recommendation:** Use Gemini Pro for attention check detection (or a dedicated classifier). Flag attention checks as HIGH_RISK in pre-fill mode — never auto-execute past them.

### Finding 2.3 — CRITICAL: MV3 Service Worker Timeout Kills Long Tasks
**Severity:** 10 | **Occurrence:** 9 | **Detection:** 7 | **RPN:** 630

Chrome Manifest V3 service workers have a **30-second to 5-minute idle timeout**. Browser Use API calls for task execution can take 30+ seconds per page. The service worker WILL be killed mid-task.

**Recommendation:** Use offscreen documents (chrome.offscreen API) or migrate execution to a lightweight companion web page that stays alive. This is a fundamental architecture change.

### Finding 2.4 — MAJOR: API Key Security Model Flawed
**Severity:** 8 | **Occurrence:** 7 | **Detection:** 4 | **RPN:** 224

Storing Gemini API keys in `chrome.storage.sync` means:
- Keys are visible in chrome://extensions developer tools
- Any malicious extension with `storage` permission can read them
- Sync sends keys across devices unencrypted

**Recommendation:** Backend-proxied API calls with session tokens. Never expose API keys to the client. This requires a more substantial backend than "minimal."

### Finding 2.5 — MAJOR: No State Recovery on Browser Use Failure
**Severity:** 8 | **Occurrence:** 8 | **Detection:** 6 | **RPN:** 384

Neither PRD nor analysis defines what happens when Browser Use fails mid-task:
- Does the worker see a half-filled form?
- Is there a rollback mechanism?
- Does the HIT get auto-submitted in a broken state?
- How is the worker's approval rate protected?

**Recommendation:** Define a "safe state" protocol: on any execution failure, (1) stop immediately, (2) screenshot current state, (3) alert worker, (4) never auto-submit a partially-filled form.

---

## 3. COST MODEL STRESS TEST

### Finding 3.1 — CRITICAL: Per-Task Cost Estimate 50-100x Too Low
**Severity:** 10 | **Occurrence:** 9 | **Detection:** 6 | **RPN:** 540

PRD claims "$0.0005 per task analysis." This assumes ~50 input + 100 output tokens. Reality:
- Survey DOM snapshot: 2,000-10,000 tokens (HTML is verbose)
- LLM analysis with reasoning: 500-1,000 output tokens
- Vision-based analysis (screenshots): 1,000-5,000 image tokens
- **Realistic cost per task: $0.02-0.05** (40-100x higher)

For Browser Use execution agent:
- Multi-step DOM navigation: 5-15 LLM calls per task
- Screenshot at each step: image tokens
- **Realistic execution cost: $0.10-0.50 per task**

At 5,000 tasks/month (Pro tier), execution costs = $500-2,500/month per user. With $14/mo subscription, this is **massively unprofitable**.

**Recommendation:** 
1. Use DOM parsing (not LLM) for simple form field detection — deterministic, free
2. Reserve LLM for complex analysis only (attention checks, open text)
3. Browser Use Cloud at $75/mo is PER USER — impossible to bundle profitably
4. Self-host Browser Use is MANDATORY for any viable business model
5. Recalculate all pricing with realistic token costs

### Finding 3.2 — MAJOR: Browser Use Cloud $75/mo is Per-User
**Severity:** 9 | **Occurrence:** 8 | **Detection:** 5 | **RPN:** 360

Browser Use Cloud at $75/mo per user makes the Pro tier ($14/mo) instantly unprofitable for any user doing AI execution. Even self-hosted Browser Use requires GPU compute (~$20-50/mo per concurrent session).

**Recommendation:** Architecture must be: worker's browser does execution locally (no cloud browser needed). Use content scripts + DOM manipulation, NOT Browser Use Cloud. This eliminates the $75/mo cost entirely.

---

## 4. LEGAL/COMPLIANCE DEEP DIVE

### Finding 4.1 — CRITICAL: CFAA Risk Beyond ToS
**Severity:** 10 | **Occurrence:** 5 | **Detection:** 3 | **RPN:** 150

MTurk ToS violation is the stated risk, but the **Computer Fraud and Abuse Act (CFAA)** applies to "unauthorized access" to computer systems. If MTurk considers automated form-filling as unauthorized:
- Criminal penalties: up to 10 years imprisonment (18 U.S.C. § 1030)
- Civil liability: MTurk requesters could sue for fraud

The "behavioral cloaking" recommendation (randomizing timing to appear human) is **evidence of intent to deceive**, which could worsen legal exposure.

**Recommendation:** Remove "behavioral cloaking" entirely. If the product is legal, it doesn't need cloaking. If it needs cloaking, it's not legal. Consult a CFAA attorney before shipping.

### Finding 4.2 — MAJOR: EU AI Act Implications
**Severity:** 8 | **Occurrence:** 4 | **Detection:** 3 | **RPN:** 96

The EU AI Act (2026 enforcement) classifies AI systems used in employment/worker management as "high-risk." If TurkerAI is marketed to EU workers:
- Mandatory conformity assessment
- Transparency obligations
- Risk management system documentation
- Potential €35M or 7% global turnover fines

**Recommendation:** Geo-restrict to non-EU markets initially, or design as "advisory tool" (not "worker management") to avoid high-risk classification.

### Finding 4.3 — MINOR: "Honor-Based" Licensing Won't Scale
**Severity:** 5 | **Occurrence:** 8 | **Detection:** 5 | **RPN:** 200

"No DRM — license is honor-based with API-gated AI execution" is contradictory. If AI execution is API-gated, that IS DRM. And without device-level enforcement, license sharing will be rampant in worker communities.

**Recommendation:** Use session-based auth with device fingerprinting. Accept that some piracy is inevitable.

---

## 5. ARCHITECTURE REVIEW

### Finding 5.1 — CRITICAL: "Minimal Backend" Doesn't Work
**Severity:** 9 | **Occurrence:** 8 | **Detection:** 5 | **RPN:** 360

The PRD says "Backend is intentionally minimal" but the product needs:
- API key proxying (security requirement from 2.4)
- License validation on every AI call
- Analytics aggregation
- Stripe webhook handling
- Rate limiting (prevent abuse)
- Browser Use session management

This is NOT minimal. It's a full backend with auth, billing, and proxy layers.

**Recommendation:** Accept the backend complexity. Use Supabase (as analysis suggests) for auth + DB. FastAPI for proxy/billing. Budget 3-4 weeks for backend, not 1.

### Finding 5.2 — MAJOR: SQLite Won't Handle Production
**Severity:** 7 | **Occurrence:** 7 | **Detection:** 6 | **RPN:** 294

SQLite for a SaaS with subscription management, analytics, and rate limiting is inappropriate. Concurrent writes from multiple workers will cause lock contention.

**Recommendation:** PostgreSQL from day one. No migration pain later.

### Finding 5.3 — MINOR: Extension Framework Unresolved
**Severity:** 5 | **Occurrence:** 5 | **Detection:** 7 | **RPN:** 175

PRD says vanilla JS, analysis says WXT/Plasmo. WXT or Plasmo would give React support, HMR, and much faster development.

**Recommendation:** Use Plasmo. It's purpose-built for Chrome extensions with React, and handles MV3 service worker complexity well.

---

## 6. BUSINESS MODEL STRESS TEST

### Finding 6.1 — CRITICAL: Pricing Doesn't Cover Costs
**Severity:** 10 | **Occurrence:** 9 | **Detection:** 7 | **RPN:** 630

Given Finding 3.1 (realistic costs), the pricing model is upside-down:

| Tier | Price | Realistic Cost/User/Mo | Margin |
|------|-------|----------------------|--------|
| Free | $0 | $0 (no execution) | 0% |
| Pro | $14 | $50-200 (Browser Use + LLM) | -257% to -1329% |
| Power | $29 | $100-500 | -244% to -1624% |

The ONLY viable path is local DOM execution (no Browser Use Cloud) + minimal LLM usage.

**Recommendation:** Redesign execution layer: use content scripts for DOM manipulation (free), LLM only for classification/decision-making. Eliminate Browser Use dependency for MVP.

### Finding 6.2 — MAJOR: $3K MRR in 3 Months Unrealistic
**Severity:** 7 | **Occurrence:** 8 | **Detection:** 6 | **RPN:** 336

$3K MRR = 214 Pro subscribers. Target market: ~100K active MTurk workers. Conversion rate from free to paid at SaaS average is 2-5%. Need 4,280-10,700 free users to get 214 paid. In 3 months? Unrealistic.

**Recommendation:** Target $500 MRR month 3 (35 subscribers), $3K month 6, $10K month 12.

---

## 7. TIMELINE REALITY CHECK

### Finding 7.1 — MAJOR: 6-Week MVP is Unrealistic
**Severity:** 7 | **Occurrence:** 9 | **Detection:** 8 | **RPN:** 504

Given findings above (MV3 timeout issue, backend complexity, cost model redesign), realistic MVP timeline:

| Week | Deliverable | Reality |
|------|-------------|---------|
| 1-2 | Architecture redesign + tech spike | Must resolve MV3 timeout, execution layer |
| 3-4 | DOM capture + local form filling | Content script approach, no Browser Use |
| 5-6 | Gemini integration (analysis only) | Classification, not execution |
| 7-8 | Pre-fill mode + worker review UI | Human-in-the-loop is the product |
| 9-10 | Backend (auth, billing, proxy) | More complex than planned |
| 11-12 | Beta testing + iteration | |

**Revised MVP timeline: 10-12 weeks, not 6.**

---

## 8. PRD vs ANALYSIS CONTRADICTIONS

| # | PRD Says | Analysis Says | Resolution |
|---|----------|---------------|------------|
| 1 | Free tier: 300 analyses/day | Free tier: 10 auto-accepts/day | Standardize: 50 analyses/day, 10 executions/day |
| 2 | Pro: $14/mo | Turker: $14.99/mo | Use $14.99/mo |
| 3 | Power: $29/mo | Power: $29.99/mo | Use $29.99/mo |
| 4 | Team: $49/mo | Elite: $49.99/mo | Rename and standardize |
| 5 | Backend: SQLite | Backend: PostgreSQL | Use PostgreSQL |
| 6 | Extension: vanilla JS | Extension: WXT/Plasmo + React | Use Plasmo |
| 7 | Browser Use Cloud API | Browser Use self-hosted | Use local DOM manipulation (neither) |
| 8 | Payments: Stripe only | Payments: Stripe + Paddle | Use Stripe + Paddle |
| 9 | Auth: license keys | Auth: Supabase Auth | Use Supabase Auth |
| 10 | No DRM | API-gated execution (is DRM) | Acknowledge as light DRM |
| 11 | Phase 3 title: "Phase 9" | — | Typo: should be Phase 3 |
| 12 | "FastAPI or Flask" | "FastAPI + SQLModel" | Use FastAPI, drop Flask |

---

## SUMMARY TABLE

| Category | Critical | Major | Minor |
|----------|----------|-------|-------|
| Requirements | 1 | 1 | 1 |
| Technical | 3 | 2 | 0 |
| Cost Model | 2 | 1 | 0 |
| Legal | 1 | 1 | 1 |
| Architecture | 1 | 1 | 1 |
| Business | 1 | 1 | 0 |
| Timeline | 0 | 1 | 0 |
| **Total** | **9** | **8** | **3** |

---

## OVERALL VERDICT: CONDITIONAL FAIL

The product concept has merit but the PRD has fundamental issues that make the current design unbuildable as specified:

1. **Cost model is broken** — per-user Browser Use Cloud makes every paid tier unprofitable
2. **MV3 architecture doesn't work** — service worker timeouts kill long-running tasks
3. **Legal risk is understated** — CFAA exposure goes beyond ToS
4. **Pricing doesn't cover costs** — need 10x price or 10x cost reduction

### REQUIRED before proceeding:
1. Complete 2-week technical spike on MTurk DOM automation without Browser Use
2. Redesign execution layer to use content scripts + local DOM manipulation
3. Recalculate cost model with realistic token usage
4. Legal review with CFAA specialist
5. Revise MVP timeline to 10-12 weeks

### Recommended PRD v2 Changes:
- Remove Browser Use Cloud dependency; use local content script execution
- Add "safe state" failure recovery protocol
- Fix MV3 architecture (offscreen documents)
- Restructure pricing to cover actual costs
- Add PostgreSQL + Supabase to tech stack
- Remove "behavioral cloaking" recommendation
- Add CFAA risk to risk register
- Fix all PRD/analysis contradictions
- Extend MVP timeline to 12 weeks
- Add state recovery procedures for all failure modes
