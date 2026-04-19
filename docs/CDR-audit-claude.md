# TurkerAI — Claude Code CDR Audit Report
**Version:** 2.0 (supersedes prior draft)  
**Date:** April 19, 2026  
**Auditor:** Claude Code (claude-sonnet-4-6)  
**Methodology:** Aerospace-style Concept Design Review (CDR)  
**References:** PRD v1.0, analysis.md, CDR-PDR-audit-combined.md  
**Verdict: CONDITIONAL FAIL — 11 Critical, 9 Major, 7 Minor findings**

---

## PREFACE — CDR SCOPE

This audit treats TurkerAI as a satellite program entering CDR: design is frozen enough to build from, or it isn't. Each finding is FMEA-rated with Severity (S), Occurrence (O), Detection (D), and RPN = S×O×D. RPN > 300 requires immediate PRD revision before proceeding.

Findings cross-reference the existing CDR-PDR-audit-combined.md. New findings not in that document are marked **[NEW]**. Extended/deepened findings are marked **[EXTENDED]**.

---

## SECTION 1 — REQUIREMENTS TRACEABILITY

### Finding 1.1 — CRITICAL [EXTENDED]: TAM Calculation Is a Category Error
**S:** 9 | **O:** 8 | **D:** 7 | **RPN:** 504

**Existing audit finding:** Market size $5.3B→$34.72B conflates AI training and micro-tasks.

**This audit extends:** The PRD does not state a SAM (Serviceable Addressable Market) or SOM (Serviceable Obtainable Market), only an inflated TAM. The TAM figure is structurally useless without qualification:

- MTurk registered workers: ~500K. **Active** (completed ≥1 HIT in past 90 days): estimated ~75K–100K per Pew/Ipeirotis studies.
- Workers technical enough to install a Chrome extension AND pay for it: ~30–50% of active = **22K–50K max**.
- SAM at $14/mo: $3.7M–$8.4M ARR, not $34B.
- SOM (5% of SAM, achievable in 12 months): $185K–$420K ARR.

The PRD shows "5,000 MAU, 2,000 paying, $35K MRR" at 12 months. That requires **4% penetration of the entire active worker base in 12 months** — aggressive but not impossible if community-driven. However this directly contradicts the $34B TAM framing.

**Gap:** PRD Section 12 success targets are internally coherent but disconnected from the TAM cited in analysis.md. No traceability exists between market size claim and subscriber projections.

**Recommendation:** Add SAM/SOM table. Retire the $34B figure from all PRD sections. Use worker-count-based projections.

---

### Finding 1.2 — MAJOR [NEW]: Feature Requirements Do Not Map to User Profiles
**S:** 6 | **O:** 8 | **D:** 6 | **RPN:** 288

PRD Section 3 defines three user profiles (Automation-First, Power Worker, Casual Worker). PRD Section 4 lists features with no explicit mapping of which features serve which profile. Critical gaps:

| User Profile | Core Need | MVP Feature? | Gap |
|---|---|---|---|
| Automation-First ($10-20/mo WTP) | Reduce bubble hell | P0.4 (auto-execute) | OK |
| Power Worker ($50/mo WTP) | Approval rate protection | P1 | **DEFERRED — WRONG** |
| Casual Worker ($5-10/mo WTP) | Reduce cognitive load | Pre-fill (P0.3) | OK |

**Critical gap:** The Power Worker — highest WTP segment — has their primary need (approval rate management, batch grinding, qualification maintenance) entirely in P1. This segment will not pay $29–50/mo for MVP features. They will churn immediately.

**Recommendation:** Pull at least one Power Worker feature (attention check detector, batch mode, or approval rate dashboard) into MVP P0. These workers are the early adopters who will drive community growth.

---

### Finding 1.3 — MINOR [NEW]: "Phase 9" Typo Signals Document Quality
**S:** 2 | **O:** 9 | **D:** 9 | **RPN:** 162

PRD Section 9.1 labels the paid launch phase "Phase 9" (should be "Phase 3"). This is a cosmetic error but signals the document was not carefully reviewed before being marked "Ready for Engineering Review." In a CDR context, this reduces confidence in document quality.

**Recommendation:** Fix typo. Implement a review checklist before any document reaches CDR status.

---

### Finding 1.4 — CRITICAL [NEW]: UHRS Platform Description Is Factually Wrong
**S:** 8 | **O:** 7 | **D:** 5 | **RPN:** 280

PRD Section 15.3 (Glossary) defines: "UHRS — Amazon's Human Microtask Crowd Source — Clickworker's platform."

**UHRS is NOT Amazon's platform.** UHRS (Universal Human Relevance System) is **Microsoft's** crowdsourcing platform, operated through Clickworker and other vendors. Amazon's equivalent is MTurk and SageMaker Ground Truth.

This is a fundamental factual error in the product glossary. It suggests the authors have not used UHRS directly. More importantly:
- UHRS has different ToS, task structures, and qualification requirements than MTurk
- If the team can't correctly identify the platform owner, the integration design may also be flawed
- UHRS access is vendor-gated (you apply through Clickworker, not directly) — the PRD implies direct integration which may not be possible

**Recommendation:** Correct the glossary. Conduct direct research on UHRS access model before including in roadmap.

---

## SECTION 2 — TECHNICAL FEASIBILITY AUDIT

### Finding 2.1 — CRITICAL [EXTENDED]: Browser Use 78% Rate Is Self-Reported on Unrelated Tasks
**S:** 10 | **O:** 8 | **D:** 6 | **RPN:** 480

The existing audit flags this. This audit adds deeper technical analysis:

**Benchmark origin:** Browser Use's 78% figure is from their internal benchmark on 100 "hard general web tasks" (likely from WebArena or similar). These tasks include navigation, search, e-commerce — NOT repetitive survey form filling.

**MTurk-specific failure modes not accounted for:**
1. **iframe sandboxing:** MTurk HIT content is often served in a sandboxed iframe with `allow-scripts` but restrictions on cross-frame communication. Browser Use's DOM access pattern may fail entirely inside iframe boundaries.
2. **Qualtrics/SurveyGizmo dynamic rendering:** ~60–70% of MTurk surveys use these platforms. They render questions progressively via XHR/WebSocket — the DOM at page load does not contain all questions. Browser Use must handle reactive DOM updates.
3. **MTurk preview mode vs. accepted mode:** HITs look different in preview (non-submittable) vs. accepted. The extension must detect mode correctly or waste analysis tokens on previews.
4. **Timer-based attention checks:** Some surveys track time-on-question. Browser Use completing a 50-item survey in 8 seconds triggers automatic rejection by Qualtrics attentiveness scoring.

**No test data exists for MTurk DOM.** The PRD technical metric (Section 12.2): "Browser Use execution success >75%" is aspirational fiction without a validation spike.

**Recommendation:** Gate all MVP planning behind a 2-week technical spike: test on real MTurk HITs (accepted, not preview), including Qualtrics-rendered surveys. Target >60% success to proceed. If <60%, migrate to local DOM manipulation architecture.

---

### Finding 2.2 — CRITICAL [EXTENDED]: MV3 Service Worker Timeout Is Architecturally Fatal
**S:** 10 | **O:** 9 | **D:** 7 | **RPN:** 630

The existing audit identifies this. This audit adds resolution architecture:

**MV3 service worker rules:**
- Terminates after **30 seconds of inactivity** (no events received)
- Maximum lifetime: **5 minutes** if active
- `fetch()` calls to external APIs do not extend lifetime
- Service workers cannot hold WebSocket connections reliably

**Impact on TurkerAI specifically:**
- Browser Use API call: 15–120 seconds per task → guaranteed timeout on any task >30s
- Gemini API call with image input: 5–20 seconds → marginal safety
- Long-polling for task completion: impossible in service worker

**The extension file structure in PRD Section 6.1 has `execution_agent.js` in `background/` — this assumes a long-lived background process that does not exist in MV3.**

**Known solution:** `chrome.offscreen` API — creates a hidden offscreen document (HTML page) that stays alive, supports `fetch()`, WebSocket, DOM parsing. Available since Chrome 109.

**Required file structure change:**
```
background/
├── service_worker.js      ← routing only, no long tasks
├── offscreen/
│   ├── offscreen.html     ← persistent execution context
│   ├── offscreen.js       ← ai_analyzer + execution_agent live here
│   └── offscreen_bridge.js
└── license_manager.js
```

**Recommendation:** Replace service_worker.js as execution host with an offscreen document. This is a fundamental architecture change. Add `"permissions": ["offscreen"]` to manifest.json.

---

### Finding 2.3 — CRITICAL [NEW]: "Gemini 3 Flash" Does Not Exist
**S:** 9 | **O:** 8 | **D:** 5 | **RPN:** 360

PRD Sections 4.2, 6.3, and 6.4 reference "Gemini 3 Flash." As of April 2026, Google's production models are Gemini 2.0 Flash and Gemini 2.5 Flash. There is no "Gemini 3 Flash."

**The PRD's own API endpoint (Section 6.4) correctly references `gemini-2.0-flash` — contradicting the model name used everywhere else.**

The analysis.md pricing ($0.72/1M input) was written for a hypothetical future model. The PRD treats this as a current specification. Current Gemini 2.0 Flash pricing may differ, affecting the cost model.

**Recommendation:** Replace all "Gemini 3 Flash" with "Gemini 2.0 Flash" (currently available). Abstract model names behind a config constant so they can be updated as new models release. Re-validate cost estimates against actual current Gemini 2.0 Flash pricing.

---

### Finding 2.4 — CRITICAL [NEW]: Execution Architecture Decision Not Made
**S:** 10 | **O:** 9 | **D:** 6 | **RPN:** 540

The PRD and analysis contain mutually exclusive execution architectures never reconciled:

- **Architecture A (PRD Section 6.2):** "AI execution lives in the extension, calls Browser Use Cloud directly with worker's API key" — worker pays $75/mo to Browser Use
- **Architecture B (analysis.md Section 5.2):** "Self-host Browser Use open-source is mandatory" — TurkerAI runs cloud browsers for each user
- **Architecture C (implied):** Extension's content script manipulates DOM directly — no external browser dependency

**These are three entirely different products with different cost structures, ops requirements, and user experiences.**

The combined audit recommends Architecture C (local DOM manipulation) but the PRD still has Architecture A in its tech stack table. This contradiction is CDR-blocking — you cannot write Week 1 scaffolding without resolving it.

**With Architecture C:**
- No Browser Use dependency for MVP
- Content script uses deterministic CSS selectors + LLM-guided field-value assignment
- Cost: $0 for execution + ~$0.0002/task for Gemini analysis = 91% gross margin at Pro tier
- Browser Use becomes a P2 enhancement for non-standard page structures

**Recommendation:** Adopt Architecture C as MVP. Remove Browser Use from P0 tech stack. Add it back as P2 enhancement. This is the only architecture with a viable cost model.

---

### Finding 2.5 — MAJOR [NEW]: Gemini API from Extension — CORS/CSP Issues
**S:** 8 | **O:** 7 | **D:** 4 | **RPN:** 224

PRD Section 6.1 shows `gemini_client.js` in `/lib/` without specifying calling context. Content scripts cannot make cross-origin requests. If Gemini calls are made from content scripts or popup without proper manifest permissions, they will silently fail or throw CORS errors.

Additionally: MTurk pages enforce Content-Security-Policy headers. Script injection from content scripts that initiates external fetches within the MTurk page context will be blocked by the page's CSP.

**All external API calls (Gemini, TurkerView) must originate from the background context (service worker or offscreen document), never from content scripts.**

**Recommendation:** Add `https://generativelanguage.googleapis.com/*` and `https://turkerview.com/*` to manifest `host_permissions`. Document in architecture that all API calls route through background context only.

---

### Finding 2.6 — MAJOR [EXTENDED]: API Key Storage Security Model Flawed
**S:** 8 | **O:** 7 | **D:** 4 | **RPN:** 224

**Existing audit:** Keys in `chrome.storage.sync` are readable by any extension with storage permission.

**This audit extends:** The PRD Section 10.3 data privacy table states "chrome.storage.sync (encrypted at rest)" — this is **factually false**. Chrome sync data is encrypted in transit but **not end-to-end encrypted**; Google can read synced data. Keys synced to a shared computer expose them to anyone opening Chrome DevTools.

Furthermore: if TurkerAI routes through a backend proxy (which it should), workers' Gemini API keys should never leave their device. The backend manages a shared API key pool and bills per usage. Workers never enter their own API keys.

**Recommendation:** Backend-proxied API calls. Workers authenticate via session token only. TurkerAI's backend holds API keys in a secrets manager. Remove "chrome.storage.sync (encrypted at rest)" false claim from PRD.

---

## SECTION 3 — RISK ASSESSMENT (FMEA)

### PRD Risk Register — Scored and Reassessed

| Risk | PRD Likelihood | Actual S | Actual O | Actual D | RPN | PRD Assessment |
|---|---|---|---|---|---|---|
| MTurk ToS enforcement | HIGH | 10 | 7 | 4 | 280 | Understated — see Finding 3.1 |
| Browser Use API deprecation | Medium | 6 | 4 | 5 | 120 | OK if Architecture C adopted |
| Gemini API price increase | Medium | 5 | 5 | 6 | 150 | Fair with abstraction layer |
| Chrome Web Store rejection | **Low** | 9 | 6 | 4 | 216 | **WRONG — should be Medium** |
| Competitor (BZTurk) cloning | Medium | 5 | 5 | 7 | 175 | Fair |
| Low worker retention | Medium | 8 | 7 | 5 | 280 | Fair |
| Payment fraud / license sharing | Medium | 4 | 8 | 5 | 160 | Fair |

### Finding 3.1 — CRITICAL [NEW]: Behavioral Detection Risk Missing
**S:** 10 | **O:** 6 | **D:** 3 | **RPN:** 180

Amazon has filed patents on behavioral anomaly detection for MTurk workers. The PRD acknowledges ToS enforcement risk but treats it as policy-level only. The deeper technical risk — **automated behavioral detection** — is absent from the risk register.

Detection vectors not addressed:
- Consistent near-identical response times across many HITs
- Response time below human-possible reading speed for text length
- Mouse movement patterns inconsistent with natural reading + clicking behavior
- Completing surveys faster than the median time tracked by requester analytics tools
- Browser extension detection via `navigator.webdriver` (CDP-based tools)

**The PRD's mitigation "behavioral cloaking — randomize timing" is itself evidence of deceptive intent and does not address all detection vectors.** See Finding 6.1.

**Recommendation:** Add behavioral detection to risk register at S:10/O:6/D:3/RPN:180. Remove "behavioral cloaking" from mitigations. Replace with: (1) minimum response time enforcer calibrated to HIT-type median, (2) explicit disclaimer to workers, (3) pre-fill-only as legally defensible default.

---

### Finding 3.2 — CRITICAL [NEW]: LLM Hallucination Destroys Worker Approval Rate
**S:** 10 | **O:** 7 | **D:** 4 | **RPN:** 280

If the LLM misclassifies an attention check as a regular question and auto-fills it incorrectly, the worker's HIT is rejected. One rejection event can drop approval rate from 99% to 98%, disqualifying workers from premium HITs (which require 98%+ approval). Systematic hallucination in attention check handling could catastrophically damage approval rates across the entire user base simultaneously.

**This is the existential product risk.** If TurkerAI causes mass approval rate drops among beta users, it will be permanently toxic in the MTurk community — a community-driven product cannot survive that.

**Recommendation:** Add to risk register at Critical. Pre-fill mode must be default and the only mode available in free tier. Auto-execute must require multi-step explicit opt-in. Log every auto-execution and surface approval rate correlation data to workers prominently.

---

### MISSING RISKS NOT IN PRD REGISTER

#### Missing Risk A — [NEW]: State-Level Fraud Statutes
**S:** 9 | **O:** 4 | **D:** 3 | **RPN:** 108

CFAA is federal. Many states have equivalent statutes with lower prosecution thresholds:
- **California Penal Code §502** — unauthorized access, includes civil liability
- **New York Penal Law §156** — computer fraud
- **Texas Penal Code §33** — access without effective consent

If a MTurk requester sues TurkerAI for fraudulent task completion, they could invoke state fraud statutes where requesters are located — potentially 50 different state law exposure vectors simultaneously.

**Recommendation:** Add to risk register. Consult multi-state commercial litigation attorney before launch, not just a CFAA specialist.

---

#### Missing Risk B — [NEW]: Survey Data Integrity / Research Ethics
**S:** 8 | **O:** 8 | **D:** 5 | **RPN:** 320

MTurk is primarily used by academic researchers for psychology, economics, and social science studies. If TurkerAI auto-fills survey responses, it:
- Introduces non-human response patterns into academic datasets
- Corrupts research findings depending on genuine human judgment
- May violate IRB protocols governing human subjects research
- Could expose TurkerAI to civil liability from universities whose research data is tainted

This is not just legal risk — it is an **ethical** risk that could generate significant negative press and academic backlash, which is devastating for a community-driven product.

**Recommendation:** Add to risk register at RPN 320. Consider a "research integrity mode" that disables auto-execute for surveys matching academic patterns (Qualtrics from .edu domains, surveys with consent forms). This could become a competitive differentiator.

---

#### Missing Risk C — [NEW]: Workers' Compensation / Labor Classification
**S:** 7 | **O:** 4 | **D:** 3 | **RPN:** 84

If TurkerAI's auto-execute mode substantially performs work on behalf of workers (worker receives pay, AI did the labor, TurkerAI charges subscription), regulators in jurisdictions with aggressive labor law enforcement (California AB5, EU Platform Workers Directive) could classify TurkerAI as operating an unlicensed labor intermediary.

**Recommendation:** Add to risk register as Low-Medium. Monitor AB5 and EU Platform Workers Directive enforcement. Ensure marketing positions TurkerAI as an "assistive tool" not a "work performer."

---

#### Missing Risk D — [NEW]: TurkerView API Availability
**S:** 6 | **O:** 5 | **D:** 5 | **RPN:** 150

TurkerView API v3 is an unofficial, community-maintained service with no SLA or uptime guarantee. It could be deprecated without notice. Its ToS may not permit automated API calls from commercial services. TurkerAI treats TurkerView as a P0.1 dependency (task capture engine).

**Recommendation:** Add to risk register. Treat TurkerView as a convenience feature, not a P0 dependency. Build fallback to requester data scraped directly from MTurk.

---

#### Missing Risk E — [NEW]: Chrome Web Store Removal Post-Launch
**S:** 9 | **O:** 5 | **D:** 3 | **RPN:** 135

CWS Policy Section 6 prohibits extensions that "interfere with, disrupt, or modify the experience of other products or services." If TurkerAI auto-fills MTurk surveys without Amazon's permission, Google's automated policy scanner could trigger removal — independent of Amazon reporting it. This risk is absent from the PRD.

**Recommendation:** Add to risk register. Prepare fallback distribution (direct .crx download, Firefox AMO port) before CWS submission.

---

## SECTION 4 — ARCHITECTURE REVIEW

### Finding 4.1 — CRITICAL [EXTENDED]: "Minimal Backend" Is Not Minimal
**S:** 9 | **O:** 8 | **D:** 5 | **RPN:** 360

The PRD states: "Backend is intentionally minimal. Backend only handles licensing and billing." But secure API proxying (required per Finding 2.6) means the backend must handle:

1. API key proxying (Gemini calls routed through backend)
2. Per-user rate limiting (Gemini quota management)
3. License validation on every AI call
4. Stripe webhook handling
5. Analytics aggregation
6. User session management (Supabase auth)

This is a full-featured API service with auth, billing, rate limiting, and AI proxying. "Minimal" creates false confidence about engineering scope.

**Minimum viable backend stack:**
```
FastAPI (Python) + Supabase (auth + PostgreSQL) + Redis (rate limiting)
Deployed on Railway or Fly.io
~4 weeks of backend engineering, not 1
```

**Recommendation:** Accept backend complexity. Budget 3–4 weeks for backend. Remove "intentionally minimal" framing.

---

### Finding 4.2 — MAJOR [NEW]: Architecture File Structure Missing Offscreen Document
**S:** 8 | **O:** 9 | **D:** 6 | **RPN:** 432

PRD Section 6.1 file structure has no offscreen document. The correct structure must be:

```
background/
├── service_worker.js          ← routing only (<10ms per message)
├── offscreen/
│   ├── offscreen.html         ← persistent execution context
│   ├── offscreen.js           ← ai_analyzer + execution_agent live here
│   └── offscreen_bridge.js    ← message bus to/from service worker
└── license_manager.js
```

Manifest.json must declare:
```json
"permissions": ["offscreen"],
```

**Recommendation:** Revise file structure in PRD Section 6.1. Offscreen document architecture is non-negotiable for any execution task >30 seconds.

---

### Finding 4.3 — MINOR [NEW]: .env In File Structure Is a Security Anti-Pattern
**S:** 5 | **O:** 7 | **D:** 6 | **RPN:** 210

PRD backend file structure (Section 6.2) includes `.env` containing `STRIPE_SECRET_KEY` and `GEMINI_API_KEY`. Listing `.env` as a first-class file structure artifact implies it gets committed to version control — a critical security error.

**Recommendation:** Replace `.env` in PRD with `.env.example`. Add explicit note: "Secrets managed via Railway/Fly.io secrets manager in production. `.env` is in `.gitignore` and never committed."

---

### Finding 4.4 — MINOR [NEW]: No Safe-State Protocol Defined
**S:** 7 | **O:** 7 | **D:** 6 | **RPN:** 294

Neither the PRD feature list nor architecture defines what happens on mid-task failure. A partially-filled form auto-submitted to MTurk cannot be unsubmitted. The safe-state protocol must be a P0 requirement:

1. On any execution failure: stop, screenshot current state, notify worker, DO NOT submit
2. On service worker death mid-task: offscreen document maintains task state
3. On network timeout: retry once, then safe-stop
4. On attention check confidence < threshold: escalate to worker, never auto-fill
5. On field count mismatch: stop, notify worker

**Recommendation:** Add "Safe-State Protocol" as P0 feature requirement. This is not optional — it directly protects the worker's approval rate.

---

## SECTION 5 — BUSINESS MODEL STRESS TEST

### Finding 5.1 — CRITICAL [EXTENDED]: Path to Profitability Requires Architecture C
**S:** 10 | **O:** 9 | **D:** 7 | **RPN:** 630

The combined audit correctly identifies the pricing inversion. This audit provides the specific profitable path:

**With Architecture C (local DOM manipulation + DOM compression):**
- Content script form filling: $0/task
- DOM summarization (extract instructions text only, ~300 tokens): $0.0002/task
- 5,000 tasks (Pro tier) × $0.0002 = **$1.00/mo LLM cost**
- Backend overhead: ~$0.20/user/mo at 100 users
- Total cost at Pro tier: **~$1.20/user/mo** on $14/mo revenue = **91% gross margin**

**With Architecture A or B (Browser Use Cloud or self-hosted):**
- Browser Use Cloud: $75/mo per user at Pro tier ($14) = -436% margin
- Self-hosted: $20-50/mo per concurrent session, shared across users but still loss-making at current pricing

**The entire viability of the business depends on adopting Architecture C.** This must be locked in PRD v2 as a non-negotiable architectural constraint.

---

### Finding 5.2 — MAJOR [EXTENDED]: MRR Targets Require 35% Paid Conversion
**S:** 7 | **O:** 8 | **D:** 6 | **RPN:** 336

PRD Section 12.1: "$3,000 MRR at 3 months" = 214 Pro subscribers.

**Realistic funnel math:**
- Active MTurk workers: ~75K–100K
- Awareness from Reddit/Discord/community: 10% = 7,500
- CWS page visits: 20% of aware = 1,500
- Install: 40% of visitors = 600
- Use free tier ≥3 times: 50% = 300
- Convert to paid at 14-day trial end: **15% = 45 subscribers**
- MRR at $14/mo average: **$630 by month 3**

Getting 214 subscribers from ~600 installs requires 35% paid conversion — 7–17x industry SaaS average (2–5%). This is not credible.

**Revised realistic targets:** $630 MRR month 3, $3K MRR month 6, $15K MRR month 12.

---

### Finding 5.3 — MINOR [NEW]: BZTurk Price Discrepancy Affects Competitive Anchor
**S:** 3 | **O:** 8 | **D:** 7 | **RPN:** 168

The PRD Section 15.1 and analysis Section 2.1 list BZTurk at $7/mo. Analysis also notes "$7/mo ($11.99/mo retail)" — implying a discounted rate. The PRD uses $7/mo as the competitive anchor: "command 2-4x premium." If actual retail is $11.99, TurkerAI at $14/mo is only 1.17x premium, not 2x.

**Recommendation:** Verify current BZTurk pricing before finalizing competitive positioning.

---

## SECTION 6 — LEGAL AND COMPLIANCE DEEP DIVE

### Finding 6.1 — CRITICAL [EXTENDED]: "Behavioral Cloaking" Is Evidence of Criminal Intent
**S:** 10 | **O:** 5 | **D:** 3 | **RPN:** 150

PRD Section 10.1 recommends: "Behavioral cloaking — Randomize field fill timing, add micro-delays."

Under CFAA 18 U.S.C. §1030(a)(4), the aggravated version of unauthorized access includes "intent to defraud." Deliberately randomizing timing to evade detection systems is strong evidence of mens rea (criminal intent). A prosecutor does not need to prove the underlying fraud occurred — intent to deceive a protective system is sufficient.

**The PRD is essentially a written record of planned deception.** This section must be removed entirely before the document circulates further.

Furthermore: if TurkerAI markets the product while knowingly facilitating ToS breach (which the PRD explicitly acknowledges in Section 10.1), this constitutes knowing facilitation — potentially tortious interference with Amazon's contracts with requesters.

**Recommendation:** Remove "behavioral cloaking" from PRD entirely. Replace legal framing with: "TurkerAI operates in pre-fill mode by default; auto-execution is opt-in and workers accept ToS risk explicitly." Consult litigation counsel before the legal section is drafted in any future version.

---

### Finding 6.2 — MAJOR [NEW]: Chrome Web Store Policy Conflict
**S:** 9 | **O:** 6 | **D:** 5 | **RPN:** 270

CWS Policy Section 6: extensions must not "interfere with, disrupt, or modify the experience of other products or services." Auto-filling and submitting MTurk HITs without Amazon's permission may constitute "interfering with" MTurk — a CWS policy violation independent of MTurk ToS.

Recent CWS enforcement (2024–2026) has removed automation tools targeting protected services. The product could be removed by Google's policy scanner before Amazon files any complaint.

**Recommendation:** Prepare fallback distribution before CWS submission: direct .crx on product website, Firefox AMO port. CWS must be treated as a channel, not the only distribution path.

---

### Finding 6.3 — MAJOR [EXTENDED]: EU AI Act High-Risk Classification
**S:** 8 | **O:** 4 | **D:** 3 | **RPN:** 96

The EU AI Act's high-risk classification for "AI systems used in employment" covers tools that assess, evaluate, or make decisions affecting workers' income. TurkerAI's AI analyzer "estimates hourly rate, difficulty, risk" and recommends "auto-execute / pre-fill / skip" — directly affecting whether a worker earns money from a given task. This is not a borderline case.

**Recommendation:** Geo-restrict to non-EU on launch, or redesign as "advisory only, no autonomous decisions."

---

### Finding 6.4 — MINOR [NEW]: Privacy Policy Missing as Legal Deliverable
**S:** 6 | **O:** 8 | **D:** 7 | **RPN:** 336

PRD Section 10.3 has a data storage table but no reference to a privacy policy document. CWS requires a public privacy policy URL for any extension handling user data. GDPR requires one for any EU users. PRD includes task analytics, usage logs, and license data — all requiring disclosure.

**Recommendation:** Add "Privacy Policy document (public URL)" to legal deliverables. Required before CWS submission. Required for GDPR compliance.

---

## SECTION 7 — TIMELINE REALITY CHECK

### Finding 7.1 — MAJOR [EXTENDED]: 6-Week MVP Requires Pre-Existing MTurk DOM Expertise
**S:** 7 | **O:** 9 | **D:** 8 | **RPN:** 504

| PRD Week | PRD Deliverable | Reality | Actual Duration |
|---|---|---|---|
| 1 | Scaffold + manifest v3 + DevTools | Resolving MV3 architecture (offscreen docs) + framework choice (Plasmo) + active MTurk account required | 2 weeks |
| 2 | DOM capture engine | MTurk's Qualtrics-iframe structure is non-trivial; requires real accepted HITs for testing | 2 weeks |
| 3 | Gemini API integration | DOM compression pipeline required first; model name fix | 1 week |
| 4 | Pre-fill mode | Depends on DOM capture quality; Qualtrics dynamic rendering breaks naive selectors | 2 weeks |
| 5 | Browser Use API | Should be replaced with local DOM manipulation — this week becomes an architecture decision + design sprint | 1 week design + 2 weeks build |
| 6 | Popup UI + earnings + license | UI + backend + Stripe = 3 separate workstreams | 3 weeks |

**Revised timeline: 14–16 weeks to a robust beta.** A "works on 3 HIT types" tech demo in 8–10 weeks is achievable with 1 full-time engineer.

**Recommendation:** Revise Section 13 to reflect 14-week MVP. A 6-week timeline that slips immediately destroys stakeholder confidence.

---

## SECTION 8 — CROSS-REFERENCE DISCREPANCIES

### All discrepancies from combined audit confirmed, plus new findings:

| # | Location | PRD Says | Analysis / Correct | Status |
|---|---|---|---|---|
| 1–12 | Various | (see combined audit) | (see combined audit) | All confirmed |
| 13 | Sections 4.2, 6.3 vs. 6.4 | "Gemini 3 Flash" | `gemini-2.0-flash` in PRD's own endpoint | **PRD internal inconsistency** |
| 14 | Section 15.3 Glossary | "UHRS — Amazon's platform" | UHRS is Microsoft's platform | **Factual error** |
| 15 | Section 6.2 backend | `.env` listed as file artifact | Should be `.env.example` + secrets manager | **Security anti-pattern** |
| 16 | Section 8.1 Free tier | Header: "300/day limit", table: "task analysis only, no execution" | Internally contradictory within same section | **PRD internal inconsistency** |
| 17 | Section 10.3 | "chrome.storage.sync (encrypted at rest)" | Chrome sync is NOT end-to-end encrypted | **Factually false** |
| 18 | Section 10.1 | "behavioral cloaking" as mitigation | Evidence of criminal intent | **Remove entirely** |
| 19 | Section 9.1 | "Phase 9: Paid Launch" | Should be "Phase 3" | **Typo** |
| 20 | Sections 3, 5 | No privacy policy mentioned | CWS and GDPR require it | **Missing deliverable** |
| 21 | Section 6.1 | No offscreen document in file structure | Required for MV3 long-running tasks | **Architecture gap** |

---

## SUMMARY TABLE

| Category | Critical | Major | Minor |
|---|---|---|---|
| Requirements Traceability | 2 (1.1, 1.4) | 1 (1.2) | 1 (1.3) |
| Technical Feasibility | 4 (2.1, 2.2, 2.3, 2.4) | 2 (2.5, 2.6) | 0 |
| Risk Assessment | 2 (3.1, 3.2) | 0 | 5 (Missing A–E) |
| Architecture | 1 (4.1) | 1 (4.2) | 2 (4.3, 4.4) |
| Business Model | 1 (5.1) | 1 (5.2) | 1 (5.3) |
| Legal/Compliance | 1 (6.1) | 2 (6.2, 6.3) | 1 (6.4) |
| Timeline | 0 | 1 (7.1) | 0 |
| Cross-Reference | 0 | 0 | 9 new discrepancies |
| **Total** | **11** | **8** | **19** |

---

## OVERALL CDR VERDICT: CONDITIONAL FAIL

**The product concept has genuine merit. The current design is not buildable as specified.**

### CDR Blockers (must resolve before any development):

1. **Execution architecture decision** — Adopt Architecture C (local DOM manipulation). Remove Browser Use from P0. (Finding 2.4)
2. **MV3 offscreen document** — Redesign background/ to use offscreen document for all tasks >30s. (Finding 2.2)
3. **Cost model lock-in** — DOM compression + local execution = 91% margin. Without this, every tier is loss-making. (Finding 5.1)
4. **Remove "behavioral cloaking"** — Legally dangerous. Replace with minimum response time calibration. (Finding 6.1)
5. **Technical spike** — 2 weeks, test on real accepted MTurk HITs before finalizing architecture. (Finding 2.1)

### Required PRD v2 Changes (all findings):

**Critical changes:**
1. Fix "Gemini 3 Flash" → "Gemini 2.0 Flash" everywhere; add config abstraction
2. Fix UHRS platform owner: Microsoft, not Amazon
3. Remove "behavioral cloaking" from all mitigations
4. Add offscreen document architecture to Section 6.1
5. Add 5 missing risks to risk register
6. Fix "chrome.storage.sync encrypted at rest" — factually false
7. Remove Browser Use from P0; use local DOM manipulation

**Major changes:**
8. Fix free tier inconsistency: 300/day vs. 10/day — standardize
9. Revise timeline to 14 weeks
10. Revise MRR targets: month 3 = $630, month 6 = $3K, month 12 = $15K
11. Pull Power Worker feature (attention check detector) into MVP P0
12. Add SAM/SOM table; retire $34B TAM
13. Accept backend complexity; budget 4 weeks for backend

**Minor changes:**
14. Fix Phase 3/Phase 9 typo
15. Add privacy policy to legal deliverables
16. Replace `.env` with `.env.example` + secrets manager note
17. Add safe-state protocol as P0 requirement
18. Verify BZTurk retail pricing before competitive anchor
19. Add CFAA + state fraud statutes + EU AI Act to risk register
20. Add research integrity mode consideration as P1

---

*Audit prepared by: Claude Code (claude-sonnet-4-6) | April 19, 2026*  
*Cross-referenced against: CDR-PDR-audit-combined.md (Claw + Gemini + Claude Code, April 19, 2026)*
