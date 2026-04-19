# TurkerAI — Product Requirements Document (PRD)

**Version:** 1.0  
**Date:** April 19, 2026  
**Status:** Draft — For Implementation  
**Product:** TurkerAI Browser Extension SaaS

---

## 1. Overview

### 1.1 Product Definition

**TurkerAI** is a Chrome Extension subscription SaaS that helps microtask workers on Amazon Mechanical Turk (MTurk) and Clickworker automate the *execution* of repetitive tasks — particularly surveys with Likert scales, data entry, and transcription — using AI browser agents.

**Core differentiation vs. existing tools:** All current tools (BZTurk, Queuebicle, TurkerHub) focus on *finding* and *accepting* tasks. TurkerAI is the first to automate actual *task execution* (filling forms, answering questions, submitting).

### 1.2 Problem Statement

| Worker Reality | Data |
|----------------|------|
| Average MTurk earnings | **$2–6/hour** |
| Workers earning above federal minimum wage ($7.25/hr) | **Only 4%** |
| Top frustration: "bubble hell" (endless radio buttons) | **37.84%** of workers |
| Repetitive survey questions | **24.16%** |
| Tool-assisted experienced workers earn | **$8–12/hour** |

The gap between unaided and tool-assisted earnings is massive — but even existing tools don't close it. TurkerAI closes it.

### 1.3 Solution

An AI agent that:
1. **Captures** the task page (DOM, instructions, requester info)
2. **Analyzes** it via LLM — estimates hourly rate, difficulty, risk
3. **Decides** action based on worker-configured thresholds (auto-execute / pre-fill / skip)
4. **Executes** filling fields, handling attention checks, submitting
5. **Logs** results for earnings analytics and approval rate tracking

---

## 2. Product Vision

> *"Every click a worker saves is money in their pocket. TurkerAI saves thousands."*

TurkerAI positions itself as the **first AI-native work tool** for the microtask economy — not a hit-catcher, but an AI coworker that does the tedious work so humans can focus on higher-value tasks.

**Personality:** Empowering, precise, respectful of worker agency. The AI is an assistant, not a replacement. Workers always have final say.

---

## 3. User Profiles

### 3.1 Primary User — The Automation-First Worker

- **Age:** 20–45
- **Platform:** MTurk (primary), Clickworker (secondary)
- **Behavior:** Works 20–40 hrs/week on microtasks; uses multiple tools; seeks efficiency
- **Pain:** Spends 40%+ of time on tedious "bubble hell" surveys
- **Willingness to pay:** $10–20/month if ROI is clear (realistic: 2–3x earnings improvement)
- **Tech literacy:** Medium — comfortable with browser extensions, can configure settings

### 3.2 Secondary User — The Power Worker

- **Age:** 25–55
- **Behavior:** MTurk as primary income; uses scripts, VPNs, multiple accounts
- **Pain:** Approval rate management, qualification maintenance, batch grinding
- **Willingness to pay:** Up to $50/month for edge
- **Tech literacy:** High — wants advanced config, API access, custom scripts

### 3.3 Tertiary User — The Casual Worker

- **Behavior:** MTurk as side income; few hours/week
- **Pain:** Low pay, confusing UI, rejections
- **Willingness to pay:** $5–10/month
- **Value:** Automated pre-fill reduces cognitive load

---

## 4. Feature Requirements

### 4.1 MVP Features (P0)

#### P0.1 — Task Capture Engine
- Inject content script on MTurk/Clickworker task pages
- Extract: page DOM, task instructions text, form fields, requester name/ID, reward, estimated time, TurkerView reputation score
- Send structured payload to background script for LLM analysis

#### P0.2 — AI Task Analyzer (LLM)
- Use **Gemini 3 Flash** (lowest cost: $0.72/1M input, $4.20/1M output)
- Prompt: classify task type, estimate hourly rate, assess automation risk, emit recommendation (auto-execute / pre-fill / skip)
- Cost target: <$0.001 per task analysis

#### P0.3 — Human-in-the-Loop Decision Engine
- Worker configures: minimum hourly rate threshold, minimum requester reputation, automation level preference (full auto / pre-fill / manual only)
- Three modes:
  - **Auto-Execute:** Agent fills and submits if thresholds pass
  - **Pre-Fill:** Agent fills fields, worker reviews and submits
  - **Manual:** Pass-through, no AI involvement

#### P0.4 — Browser AI Execution Agent
- Integration with **Browser Use Cloud API** (78% success rate on hard tasks, SaaS API at $75/mo)
- Fallback: OpenAI Computer Use API (CUA, ~60-70% success)
- Task: navigate page elements, fill inputs, handle attention checks, click submit
- Screenshot capture before/after for worker review

#### P0.5 — Earnings Dashboard
- Per-session earnings, hourly rate approximation
- Approval rate tracker
- Task type breakdown (survey vs. data entry vs. transcription)
- Export to CSV

#### P0.6 — Subscription & License Manager
- Freemium model: 10 free AI tasks/day, then subscription
- Tiers: Free, Pro ($14/mo), Power ($29/mo), Team ($49/mo)
- License key validation via backend API (simple Node/Python backend)
- No DRM — license is honor-based with API-gated AI execution

---

### 4.2 P1 Features (Post-MVP)

| Feature | Description | Priority Rationale |
|---------|-------------|-------------------|
| **TurkerView Integration** | Auto-fetch requester reputation from TurkerView API | Reduces risk of bad requesters |
| **Attention Check Detector** | LLM flags suspicious attention checks before execution | Protects approval rate |
| **Batch Auto-Accept** | Queue multiple HITs, AI analyzes and executes sequentially | Power user demand |
| **Clickworker Support** | Extend pipeline to Clickworker UHRS platform | Market expansion |
| **Whisper Transcription** | Audio tasks auto-transcribed via Whisper API before submission | High-ROI task type |
| **Open-Text AI Writer** | Generate plausible open-text responses for survey comments | Medium risk — see ToS |
| **Mobile Companion App** | Simple status dashboard on phone | Retention feature |

---

### 4.3 P2 Features (Future)

- Custom LLM model fine-tuned on MTurk task patterns
- Multi-account management with session isolation
- VPN/proxy integration for IP management
- HIT finder with AI task quality scoring
- Community presets (worker-shared automation configs)

---

## 5. Automation Risk Matrix

| Task Type | Auto-Execute | Pre-Fill | Manual Only | Notes |
|-----------|:------------:|:---------:|:-----------:|-------|
| Likert scale surveys | ✅ | ✅ | ✅ | Best automation target |
| Single-sentence open text | ⚠️ | ✅ | ✅ | AI plausible, risk medium |
| Long-form written responses | ❌ | ⚠️ | ✅ | ToS risk if auto-submitted |
| Data entry / transcription | ✅ | ✅ | ✅ | High accuracy expected |
| Audio transcription | ✅ | ✅ | ✅ | Whisper handles |
| Image categorization | ⚠️ | ✅ | ✅ | VLM-dependent |
| Attention checks | ⚠️ | ⚠️ | ✅ | Must appear human |
| Captcha | ❌ | ❌ | ❌ | Never automate |

---

## 6. Technical Architecture

### 6.1 Extension Architecture

```
turkerai-extension/
├── manifest.json              # Chrome Extension manifest v3
├── background/
│   ├── service_worker.js      # Long-lived background process
│   ├── ai_analyzer.js         # LLM task analysis logic
│   ├── execution_agent.js     # Browser AI agent (Browser Use API)
│   └── license_manager.js     # Subscription validation
├── content/
│   ├── inject.js               # DOM capture + field injection
│   ├── page_monitor.js        # Detects new task pages
│   └── dom_parser.js          # Extracts structured task data
├── popup/
│   ├── popup.html
│   ├── popup.js               # Worker dashboard UI
│   └── popup.css
├── options/
│   ├── options.html           # Settings page
│   ├── options.js
│   └── options.css
├── lib/
│   ├── browser_use_sdk.js     # Browser Use Cloud client
│   ├── gemini_client.js       # Google Gemini API client
│   └── storage.js             # chrome.storage wrapper
├── assets/
│   ├── icon16.png, icon48.png, icon128.png
│   └── logo.svg
└── locales/
    └── en/messages.json
```

### 6.2 Backend Architecture (Minimal)

```
turkerai-backend/
├── app.py                     # FastAPI or Flask
├── routes/
│   ├── license.py             # POST /validate_license
│   ├── analytics.py           # POST /track_event (earnings, usage)
│   └── subscription.py       # Stripe webhook handler
├── models/
│   ├── license.py
│   └── subscription.py
└── .env                       # STRIPE_SECRET_KEY, GEMINI_API_KEY
```

**Note:** Backend is intentionally minimal. AI execution lives in the extension (calls Browser Use Cloud directly with the worker's API key). Backend only handles licensing and billing.

### 6.3 Tech Stack

| Layer | Technology | Rationale |
|-------|------------|-----------|
| Extension shell | Chrome Extension Manifest v3 | Required for Chrome Web Store |
| Background process | Service Worker (ES Modules) | Modern Chrome extension standard |
| AI Analysis | Gemini 3 Flash (Google AI Studio) | Lowest cost, good reasoning |
| AI Execution | Browser Use Cloud API | 78% success, best-in-class |
| Storage | chrome.storage.sync + chrome.storage.local | Extension-native, no backend needed |
| Backend (licensing) | FastAPI + SQLite | Minimal, deploy anywhere |
| Payments | Stripe (future) | Standard for SaaS |

### 6.4 Key API Integrations

#### Gemini API (Task Analysis)
- Endpoint: `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent`
- Auth: API key stored per-user in `chrome.storage.sync`
- Cost: ~$0.0005 per task analysis

#### Browser Use Cloud API (Task Execution)
- Endpoint: See `https://www.browseruse.com/docs`
- Auth: Worker provides their own Browser Use API key (or tiered keys from TurkerAI)
- Fallback: OpenAI CUA API

#### TurkerView API (Requester Reputation)
- Endpoint: `https://turkerview.com/api/v3`
- Auth: Worker's TurkerView API key stored locally
- Use: Fetch requester rejection rate, pay review, speed

---

## 7. UI/UX Requirements

### 7.1 Popup Window (Primary Interface)

**Size:** 400×600px (standard Chrome extension popup)

**Sections:**
1. **Status Bar** — Connection status, current MTurk page, AI enabled/disabled toggle
2. **Task Analyzer** — Shows current task analysis: type, estimated rate, recommendation badge
3. **Quick Actions** — Three buttons: "Auto-Execute", "Pre-Fill", "Skip"
4. **Earnings Widget** — Today's earnings, hourly rate estimate, tasks completed
5. **Subscription Status** — Plan tier, remaining AI tasks (free tier)

### 7.2 Options Page (Settings)

**Sections:**
1. **Account** — License key input, subscription management link
2. **AI Configuration** — Gemini API key input, model selection (Flash/Pro), cost limits
3. **Execution Preferences** — Mode (Auto/Pre-Fill/Manual), minimum hourly rate ($), minimum requester rating
4. **Integrations** — TurkerView API key, Browser Use API key
5. **Appearance** — Dark/light mode
6. **Advanced** — Custom LLM endpoint, proxy settings, log export

### 7.3 In-Page Overlay (Task Page)

When active on an MTurk task page:
- Floating pill button (bottom-right) showing AI status
- Click expands to: task analysis summary, auto/pre-fill/skip controls
- Semi-transparent overlay highlighting fields AI will fill

---

## 8. Subscription & Pricing

### 8.1 Tiers

| Tier | Price | AI Tasks/Month | Features |
|------|-------|----------------|----------|
| **Free** | $0 | 300/day limit | Task analysis only, no execution |
| **Pro** | **$14/mo** | 5,000 | Auto-execute + Pre-Fill, earnings dashboard |
| **Power** | **$29/mo** | 25,000 | All Pro + batch mode, TurkerView integration, Whisper |
| **Team** | **$49/mo** | Unlimited | All Power + multi-account, priority support, API access |

### 8.2 Conversion Strategy

- **Free tier is generous** (300 AI task analyses/day) to drive adoption
- **14-day Pro trial** on first install (no credit card)
- **ROI calculator in popup** — "You've earned $X extra this week with TurkerAI"
- **Referral program** — 1 month free per referral, both get it

---

## 9. Go-to-Market Plan

### 9.1 Launch Strategy

**Phase 1: Closed Beta (Weeks 1–4)**
- Target: 50 power MTurk workers from TurkerView/TurkerHub forums
- Invite-only via Google Form
- Feedback loop: Discord server, weekly check-ins
- Goal: Validate task types, refine prompts, identify ToS risk patterns

**Phase 2: Public Beta (Weeks 5–8)**
- Chrome Web Store listing (unpacked .crx for beta)
- Reddit r/mturk, r/beermoney, Twitter/X automation communities
- Free tier fully functional, Pro trial extended
- Goal: 500–1,000 active users

**Phase 3: Paid Launch (Weeks 9–12)**
- Stripe integration live
- Tiered pricing activated
- Chrome Web Store public listing
- Launch blog post + YouTube demo
- Goal: 200 paying subscribers, $3,000 MRR

### 9.2 Community & Content

- **TurkerAI Blog** — Guides: "How to earn $15/hr on MTurk", case studies
- **YouTube Channel** — Setup tutorials, automation demos
- **Discord Community** — Worker tips, feature requests, beta access
- **Reddit** — r/mturk AMAs, productivity threads

---

## 10. Legal & Platform Risk

### 10.1 MTurk Terms of Service

**⚠️ CRITICAL RISK — MTurk ToS explicitly prohibits automated task completion.**

Relevant ToS sections:
- *"Workers may not use bots, scripts, or other automated tools to complete HITs."*
- Violation consequences: account suspension, earnings forfeiture

**Risk Mitigation:**
1. **Human-in-the-loop as default** — Auto-execute is opt-in, not default
2. **Behavioral cloaking** — Randomize field fill timing, add micro-delays
3. **Worker discretion** — Clear warnings in UI: "Using auto-execute may violate MTurk ToS"
4. **Legal exposure framing** — Tool is positioned as "AI assistant", not "auto-submitter"
5. **Multi-platform** — Clickworker UHRS has more permissive ToS; prioritize there
6. **ToS monitoring** — Weekly automated checks on MTurk ToS pages

### 10.2 Chrome Web Store Compliance

- ** deceptive practices** — Must not misrepresent what the extension does
- **Data collection** — Privacy policy required; minimize data collected
- **Payment** — Chrome Web Store payment system preferred; TurkerAI uses Stripe (allowed if disclosed)

### 10.3 Data Privacy

| Data | Stored Where | Retention |
|------|-------------|-----------|
| License info | Backend DB (SQLite) | Duration of subscription |
| AI API keys | chrome.storage.sync (encrypted at rest) | Until worker removes |
| Task analytics (earnings) | chrome.storage.local | Worker-controlled |
| Extension usage logs | Anonymous, backend aggregate | 90 days, no PII |

---

## 11. Risk Register

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| MTurk ToS enforcement | **HIGH** | Critical | Multi-platform (Clickworker), human-in-loop, legal framing |
| Browser Use API deprecation | Medium | High | OpenAI CUA fallback; open-source self-host option |
| Gemini API price increase | Medium | Medium | Abstraction layer; allow custom LLM endpoint |
| Chrome Web Store rejection | Low | High | Full ToS compliance review before submission |
| Competitor (BZTurk) cloning | Medium | Medium | Speed to market, community lock-in, feature velocity |
| Low worker retention | Medium | High | ROI tracking, community, regular feature releases |
| Payment fraud (license sharing) | Medium | Medium | API-gated execution, device fingerprinting, rate limiting |

---

## 12. Success Metrics

### 12.1 Product Metrics

| Metric | Target (3 months) | Target (12 months) |
|--------|-------------------|---------------------|
| Active users (MAU) | 500 | 5,000 |
| Paying subscribers | 200 | 2,000 |
| MRR | $3,000 | $35,000 |
| Churn rate (monthly) | <8% | <5% |
| AI tasks executed | 50,000/month | 500,000/month |
| Average worker earnings improvement | +50% | +80% |

### 12.2 Technical Metrics

| Metric | Target |
|--------|--------|
| Extension load time | <1 second |
| AI analysis latency | <2 seconds |
| Browser Use execution success | >75% |
| Uptime (backend) | 99.5% |
| Chrome Web Store rating | 4.0+ stars |

---

## 13. Implementation Phases

### Phase 1: MVP (Weeks 1–6)

**Goal:** Ship a working extension to closed beta users

| Week | Deliverable |
|------|-------------|
| 1 | Project scaffold, manifest v3 setup, Chrome DevTools integration |
| 2 | DOM capture engine (content script → background) |
| 3 | Gemini API integration (task analyzer) |
| 4 | Pre-fill mode (fields populated, worker submits) |
| 5 | Browser Use API integration (auto-execute) |
| 6 | Popup UI, basic earnings tracker, license key validation |

**Beta Criteria:**
- Extension loads without errors
- Task analysis works on 80%+ of MTurk survey pages
- Pre-fill correctly populates 90%+ of Likert scale fields
- License validation works

### Phase 2: Beta Iteration (Weeks 7–10)

- TurkerView integration
- Attention check detection
- Batch mode
- Bug fixes from beta feedback
- Expand to Clickworker UHRS

### Phase 3: Paid Launch (Weeks 11–14)

- Stripe subscription integration
- All 4 pricing tiers activated
- Public Chrome Web Store listing
- Launch marketing

---

## 14. Open Questions

1. **Browser Use API pricing for workers** — Pass-through cost or TurkerAI bundles? (Recommendation: bundles, take margin on API)
2. **MTurk ToS enforcement depth** — Does Amazon use behavioral detection? (Recommendation: assume yes, build delays)
3. **Multi-account support** — Allow workers to manage multiple MTurk accounts? (Legal gray area, defer to Phase 2)
4. **Mobile companion app** — Native app or PWA? (PWA lower effort, defer)
5. **Self-hosted execution agent** — Allow power users to self-host Browser Use open-source? (Technical complexity, Phase 3)

---

## 15. Appendix

### 15.1 Competitive Reference Pricing

| Tool | Price | Notes |
|------|-------|-------|
| BZTurk | $7/mo | Auto-accept only, no AI |
| Queuebicle | $15–30/mo | Windows only, no AI |
| TurkerAI (proposed) | $14/mo | AI task execution |
| TurkerAI (power) | $29/mo | Full-featured AI |

### 15.2 Key Research Sources

- Fowler et al. (2022) — MTurk worker demographics and frustration survey (n=1,369)
- Mordor Intelligence — Micro-tasking market size 2024–2031
- Browser Use Cloud — 78% success rate benchmark on 100 hard tasks
- TurkerView API documentation v3

### 15.3 Glossary

| Term | Definition |
|------|------------|
| HIT | Human Intelligence Task — a single microtask on MTurk |
| Requester | Company/person who posts HITs |
| TurkerView | Third-party reputation database for MTurk requesters |
| UHRS | Amazon's Human Microtask Crowd Source — Clickworker's platform |
| Likert scale | Survey question with 5–7 point scale (strongly disagree → strongly agree) |
| "Bubble hell" | Worker slang for surveys with hundreds of identical Likert items |
| Approval rate | % of worker's submitted HITs that weren't rejected |

---

*Document Status: Ready for Engineering Review*  
*Next Step: Architecture Design Document (ADD) and Sprint 1 planning*
