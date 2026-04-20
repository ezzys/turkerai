TurkerAI — Product Requirements Document (PRD) v2.0

Version: 2.0 
Date: April 19, 2026 
Status: Draft — Post-Audit Revision 
Product: TurkerAI Browser Extension SaaS

---

 1. Overview

 1.1 Product Definition

TurkerAI is a Chrome Extension subscription SaaS that helps microtask workers on Amazon Mechanical Turk (MTurk) and Clickworker automate the execution of repetitive tasks — particularly surveys with Likert scales, data entry, and transcription — using AI browser agents that operate locally within the workers browser.

Core differentiation vs. existing tools: All current tools (BZTurk, Queuebicle, TurkerHub) focus on finding and accepting tasks. TurkerAI is the first to automate actual task execution (filling forms, answering questions, submitting) with a human-in-the-loop, focusing on compliance and cost-effectiveness.

 1.2 Problem Statement

 Worker Reality Data 
----------------------
 Average MTurk earnings $2–6/hour (likely lower now) 
 Workers earning above federal minimum wage ($7.25/hr) Only 4 
 Top frustration: "bubble hell" (endless radio buttons) 37.84 of workers 
 Repetitive survey questions 24.16 
 Tool-assisted experienced workers earn $8–12/hour 

The gap between unaided and tool-assisted earnings is massive — but even existing tools dont close it. TurkerAI closes it by focusing on ethical, compliant automation that saves workers time and boosts earnings without relying on external cloud browser services.

 1.3 Solution

An AI agent that:
1. Captures the task page (DOM, instructions, requester info) locally within the browser.
2. Analyzes it via LLM — classifies task type, estimates hourly rate, assesses automation risk, and emits a recommendation (auto-execute / pre-fill / skip).
3. Decides action based on worker-configured thresholds (auto-execute / pre-fill / skip).
4. Executes filling fields, handling attention checks, and submitting via local DOM manipulation.
5. Logs results for earnings analytics and approval rate tracking.
6. Ensures safe state recovery on any execution failure to protect workers approval rate and data.

---

 2. Product Vision

 "Every click a worker saves is money in their pocket. TurkerAI saves thousands, compliantly."

TurkerAI positions itself as the first AI-native work tool for the microtask economy — not a hit-catcher, but an AI coworker that does the tedious work so humans can focus on higher-value tasks, with a strong emphasis on compliance and ethical use. Workers always have final say.

---

 3. User Profiles

 3.1 Primary User — The Automation-First Worker

- Age: 20–45
- Platform: MTurk (primary), Clickworker (secondary)
- Behavior: Works 20–40 hrs/week on microtasks uses multiple tools seeks efficiency
- Pain: Spends 40 of time on tedious "bubble hell" surveys
- Willingness to pay: $10–20/month if ROI is clear (realistic: 1.5x earnings improvement)
- Tech literacy: Medium — comfortable with browser extensions, can configure settings

 3.2 Secondary User — The Power Worker

- Age: 25–55
- Behavior: MTurk as primary income uses scripts, VPNs, multiple accounts
- Pain: Approval rate management, qualification maintenance, batch grinding
- Willingness to pay: Up to $50/month for edge
- Tech literacy: High — wants advanced config, API access, custom scripts

 3.3 Tertiary User — The Casual Worker

- Behavior: MTurk as side income few hours/week
- Pain: Low pay, confusing UI, rejections
- Willingness to pay: $5–10/month
- Value: Automated pre-fill reduces cognitive load

---

 4. Feature Requirements

 4.1 MVP Features (P0)

 P0.1 — Task Capture Engine
- Inject content script on MTurk/Clickworker task pages
- Extract: page DOM, task instructions text, form fields, requester name/ID, reward, estimated time, TurkerView reputation score
- Send structured payload to an offscreen document (using chrome.offscreen API) for long-running processes.

 P0.2 — AI Task Analyzer (LLM)
- Use Gemini 3 Flash (lowest cost: $0.72/1M input, $4.20/1M output) for task classification and basic risk assessment.
- Use Gemini 2.5 Pro for nuanced attention check detection.
- Prompt: classify task type, estimate hourly rate, assess automation risk, emit recommendation (auto-execute / pre-fill / skip).
- Cost target: Maximize DOM parsing for free, reserve LLM for complex (0.02-0.05/task) inference only.

 P0.3 — Human-in-the-Loop Decision Engine
- Worker configures: minimum hourly rate threshold, minimum requester reputation, automation level preference (full auto / pre-fill / manual only).
- Three modes:
 - Auto-Execute: Agent fills and submits if thresholds pass, with explicit worker opt-in and clear ToS warnings.
 - Pre-Fill: Agent fills fields, worker reviews and submits (recommended default).
 - Manual: Pass-through, no AI involvement.

 P0.4 — Local Browser AI Execution Agent
- No Browser Use Cloud API. Instead, execute via local DOM manipulation using content scripts.
- This allows for full control, eliminates per-user cloud browser costs, and mitigates MV3 service worker timeouts (using chrome.offscreen for coordination).
- Task: navigate page elements, fill inputs, handle attention checks, click submit.
- Screenshot capture before/after for worker review.
- Safe State Recovery: On any execution failure, immediately stop, screenshot current state, alert worker, and never auto-submit a partially-filled form.

 P0.5 — Earnings Dashboard
- Per-session earnings, hourly rate approximation
- Approval rate tracker
- Task type breakdown (survey vs. data entry vs. transcription)
- Export to CSV

 P0.6 — Subscription License Manager
- Freemium model: 50 free AI analyses/day, 10 free AI executions/day, then subscription.
- Tiers: Free, Pro ($14.99/mo), Power ($29.99/mo), Elite ($49.99/mo).
- Supabase Auth for worker accounts and license validation via backend API.
- API-gated AI execution (acknowledging this is a form of DRM).

---

 4.2 P1 Features (Post-MVP)

 Feature Description Priority Rationale 
-----------------------------------------
 TurkerView Integration Auto-fetch requester reputation from TurkerView API Reduces risk of bad requesters 
 Attention Check Detector LLM flags suspicious attention checks before execution Protects approval rate 
 Batch Auto-Accept Queue multiple HITs, AI analyzes and executes sequentially Power user demand 
 Clickworker Support Extend pipeline to Clickworker UHRS platform Market expansion 
 Whisper Transcription Audio tasks auto-transcribed via Whisper API before submission High-ROI task type 
 Open-Text AI Writer Generate plausible open-text responses for survey comments Medium risk — see ToS 
 Mobile Companion App Simple status dashboard on phone (PWA) Retention feature 

---

 4.3 P2 Features (Future)

- Custom LLM model fine-tuned on MTurk task patterns
- Multi-account management with session isolation
- VPN/proxy integration for IP management
- HIT finder with AI task quality scoring
- Community presets (worker-shared automation configs)

---

 5. Automation Risk Matrix

 Task Type Auto-Execute Pre-Fill Manual Only Notes 
-----------:------------::---------::-----------:-------
 Likert scale surveys ✅ ✅ ✅ Best automation target, local DOM manipulation 
 Single-sentence open text ⚠️ ✅ ✅ AI plausible, risk medium requires Gemini Pro 
 Long-form written responses ❌ ⚠️ ✅ ToS risk if auto-submitted, human review mandatory 
 Data entry / transcription ✅ ✅ ✅ High accuracy expected, local DOM manipulation 
 Audio transcription ✅ ✅ ✅ Whisper handles 
 Image categorization ⚠️ ✅ ✅ VLM-dependent, human review 
 Attention checks ⚠️ ⚠️ ✅ Human review critical AI for flagging only (Gemini Pro) 
 Captcha ❌ ❌ ❌ Never automate 

---

 6. Technical Architecture

 6.1 Extension Architecture

turkerai-extension/
├── manifest.json Chrome Extension manifest v3
├── background/
│ ├── service_worker.js Main background process (Plasmo)
│ └── offscreen.html For long-running tasks, chrome.offscreen API
├── content/
│ ├── inject.js DOM capture field injection, local execution
│ ├── page_monitor.js Detects new task pages
│ └── dom_parser.js Extracts structured task data
├── popup/
│ ├── popup.html
│ ├── popup.js Worker dashboard UI (React)
│ └── popup.css
├── options/
│ ├── options.html Settings page (React)
│ ├── options.js
│ └── options.css
├── lib/
│ ├── gemini_client.js Google Gemini API client (proxy via backend)
│ └── storage.js chrome.storage wrapper
├── assets/
│ ├── icon16.png, icon48.png, icon128.png
│ └── logo.svg
└── locales/
 └── en/messages.json

 6.2 Backend Architecture

turkerai-backend/
├── app.py FastAPI
├── routes/
│ ├── auth.py Supabase Auth integration
│ ├── license.py POST /validate_license, API-gated execution
│ ├── analytics.py POST /track_event (earnings, usage)
│ ├── llm_proxy.py Proxy for Gemini API calls (security)
│ └── subscription.py Stripe webhook handler
├── models/
│ ├── license.py
│ └── subscription.py
└── .env SUPABASE_URL, SUPABASE_KEY, STRIPE_SECRET_KEY, GEMINI_API_KEY (backend only)

Note: Backend is a full-fledged service handling security, licensing, billing, and LLM proxying. Local execution in the browser means no Browser Use Cloud dependency.

 6.3 Tech Stack

 Layer Technology Rationale 
------------------------------
 Extension shell Plasmo Framework (React, MV3) Modern MV3 toolchain, HMR, React support, offscreen document handling 
 Background process Service Worker (Plasmo) chrome.offscreen Handles MV3 timeout, long-running tasks 
 AI Analysis Gemini 3 Flash (classification), Gemini 2.5 Pro (attention checks) Cost-optimized for different tasks 
 AI Execution Local DOM manipulation (content scripts) Cost-free, full control, ToS compliance focus 
 Storage chrome.storage.sync chrome.storage.local Extension-native, no backend needed for core data 
 Backend FastAPI PostgreSQL Supabase Auth Robust, scalable, secure auth, type-safe, async 
 Payments Stripe Paddle Standard for SaaS, international tax compliance 

 6.4 Key API Integrations

 Gemini API (Task Analysis) — Proxied via Backend
- Endpoint: https://generativelanguage.googleapis.com/v1beta/models/...:generateContent
- Auth: Backend API key, accessed by extension via session token.
- Cost: Optimized for specific LLM tasks (classification only).

 TurkerView API (Requester Reputation)
- Endpoint: https://turkerview.com/api/v3
- Auth: Workers TurkerView API key stored locally.
- Use: Fetch requester rejection rate, pay review, speed.

 Stripe Paddle (Payments)
- Integrated with FastAPI backend for subscription management and webhooks.

 Supabase Auth (User Authentication)
- Integrated with FastAPI backend for secure worker accounts.

---

 7. UI/UX Requirements

 7.1 Popup Window (Primary Interface)

Size: 400600px (standard Chrome extension popup)

Sections:
1. Status Bar — Connection status, current MTurk page, AI enabled/disabled toggle
2. Task Analyzer — Shows current task analysis: type, estimated rate, recommendation badge
3. Quick Actions — Three buttons: "Auto-Execute", "Pre-Fill", "Skip"
4. Earnings Widget — Todays earnings, hourly rate estimate, tasks completed
5. Subscription Status — Plan tier, remaining AI tasks (free tier)

 7.2 Options Page (Settings)

Sections:
1. Account — Supabase Auth login/logout, subscription management link
2. AI Configuration — LLM cost limits, model preferences (Flash/Pro for specific tasks)
3. Execution Preferences — Mode (Auto/Pre-Fill/Manual), minimum hourly rate ($), minimum requester rating
4. Integrations — TurkerView API key
5. Appearance — Dark/light mode
6. Advanced — Custom LLM endpoint, proxy settings, log export, Safe State Recovery Options

 7.3 In-Page Overlay (Task Page)

When active on an MTurk task page:
- Floating pill button (bottom-right) showing AI status
- Click expands to: task analysis summary, auto/pre-fill/skip controls
- Semi-transparent overlay highlighting fields AI will fill
- Clear warnings on auto-execute mode about ToS and CFAA risk.

---

 8. Subscription Pricing

 8.1 Tiers

 Tier Price AI Analyses/Day AI Executions/Day Features 
-----------------------------------------------------------
 Free $0 50 10 Task analysis, limited pre-fill, basic dashboard 
 Pro $14.99/mo 5,000 500 Auto-execute Pre-Fill, earnings dashboard 
 Power $29.99/mo 25,000 2,500 All Pro batch mode, TurkerView integration, Whisper 
 Elite $49.99/mo Unlimited Unlimited All Power multi-account, priority support, API access 

 8.2 Conversion Strategy

- Free tier is designed to demonstrate value without incurring high costs.
- 14-day Pro trial on first install (no credit card).
- ROI calculator in popup — "Youve earned $X extra this week with TurkerAI."
- Referral program — 1 month free per referral, both get it.

---

 9. Go-to-Market Plan

 9.1 Launch Strategy

Phase 1: Closed Beta (Weeks 1–4)
- Target: 50 power MTurk workers from TurkerView/TurkerHub forums.
- Invite-only via Google Form.
- Feedback loop: Discord server, weekly check-ins.
- Goal: Validate local execution success, refine prompts, solidify safe state recovery, identify ToS compliance patterns.

Phase 2: Public Beta (Weeks 5–8)
- Chrome Web Store listing (unpacked .crx for beta).
- Reddit r/mturk, r/beermoney, Twitter/X automation communities.
- Free tier fully functional, Pro trial extended.
- Goal: 500–1,000 active users.

Phase 3: Paid Launch (Weeks 9–12)
- Stripe Paddle integration live.
- Tiered pricing activated.
- Chrome Web Store public listing.
- Launch blog post YouTube demo.
- Goal: 35 paying subscribers, $500 MRR (month 3) 200 paying subscribers, $3,000 MRR (month 6) 700 paying subscribers, $10,000 MRR (month 12).

 9.2 Community Content

- TurkerAI Blog — Guides: "How to ethically earn $15/hr on MTurk", case studies, "Understanding MTurk ToS."
- YouTube Channel — Setup tutorials, automation demos, safe use guidelines.
- Discord Community — Worker tips, feature requests, beta access.
- Reddit — r/mturk AMAs, productivity threads, compliance discussions.

---

 10. Legal Platform Risk

 10.1 MTurk Terms of Service

⚠️ CRITICAL RISK — MTurk ToS explicitly prohibits automated task completion.

Relevant ToS sections:
- "Workers may not use bots, scripts, or other automated tools to complete HITs."
- Violation consequences: account suspension, earnings forfeiture.

Risk Mitigation:
1. Human-in-the-loop as default — Auto-execute is opt-in, not default.
2. Compliance-focused design — Explicitly not using behavioral cloaking. If it needs cloaking, its not legal.
3. Worker discretion — Clear warnings in UI: "Using auto-execute may violate MTurk ToS and CFAA."
4. Legal exposure framing — Tool is positioned as "AI assistant with human oversight."
5. Multi-platform — Clickworker UHRS has more permissive ToS prioritize there.
6. ToS monitoring — Weekly automated checks on MTurk ToS pages.

 10.2 Chrome Web Store Compliance

- Deceptive practices — Must not misrepresent what the extension does.
- Data collection — Privacy policy required minimize data collected.
- Payment — Chrome Web Store payment system preferred TurkerAI uses Stripe/Paddle (allowed if disclosed).

 10.3 Data Privacy

 Data Stored Where Retention 
------------------------------
 License info Backend DB (PostgreSQL) Duration of subscription 
 AI API keys Backend only (proxied calls) Never exposed to client 
 Task analytics (earnings) chrome.storage.local Worker-controlled 
 Extension usage logs Anonymous, backend aggregate 90 days, no PII 
 Supabase Auth tokens chrome.storage.local (secure) Session-based 

---

 11. Risk Register

 Risk Likelihood Impact Mitigation 
-------------------------------------
 MTurk ToS enforcement HIGH Critical Multi-platform (Clickworker), human-in-loop, legal framing, no behavioral cloaking 
 CFAA Enforcement Medium Critical Explicit warnings, legal consultation, design for compliance (no deceptive intent) 
 EU AI Act "High-Risk" classification Low Critical Geo-restrict to non-EU markets initially, or design as "advisory tool" 
 LLM hallucination causes rejections Medium High Mandatory pre-fill review mode, approval rate monitoring, Gemini 2.5 Pro for attention checks 
 Chrome Web Store rejection Low High Full ToS compliance review before submission, Plasmo-based development 
 Competitor (BZTurk) cloning Medium Medium Speed to market, community lock-in, feature velocity, strong AI differentiation 
 Low worker retention Medium High ROI tracking, community, regular feature releases 
 Payment fraud (license sharing) Medium Medium API-gated execution, Supabase Auth with device fingerprinting, rate limiting 
 MV3 Service Worker Timeout High Critical Use chrome.offscreen API for long-running tasks 
 API Key Exposure Low Critical All LLM calls proxied via backend with session tokens 
 Local Execution Failures Medium High "Safe State Recovery" protocol, detailed error reporting to worker 
 Market Size Inflation High High Focus on realistic TAM of $17M/yr, not $34B 
 Unrealistic Cost Model High Critical Re-calculated pricing, local DOM execution, minimal LLM usage 

---

 12. Success Metrics

 12.1 Product Metrics

 Metric Target (3 months) Target (12 months) 
------------------------------------------------
 Active users (MAU) 500 5,000 
 Paying subscribers 35 700 
 MRR $500 $10,000 
 Churn rate (monthly) 8 5 
 AI tasks executed (local) 50,000/month 500,000/month 
 Average worker earnings improvement 50 80 

 12.2 Technical Metrics

 Metric Target 
----------------
 Extension load time 1 second 
 AI analysis latency 2 seconds 
 Local execution success 90 
 Uptime (backend) 99.9 
 Chrome Web Store rating 4.0 stars 

---

 13. Implementation Phases

 Phase 0: Technical Spike (Weeks 1–2)

Goal: Validate core local DOM automation techniques and MV3 architecture challenges.

 Week Deliverable 
-------------------
 1 Proof-of-concept for local DOM capture form filling on 10 MTurk tasks 
 2 POC for chrome.offscreen document for long-running tasks 

Criteria:
- Successful local form filling on 80 of test tasks.
- chrome.offscreen document successfully maintains state for 5 minutes.

 Phase 1: MVP (Weeks 3–14)

Goal: Ship a working extension to closed beta users with a compliant and cost-effective core.

 Week Deliverable 
-------------------
 3-4 Project scaffold (Plasmo), manifest v3, Chrome DevTools, initial UI 
 5-6 DOM capture engine (content script → offscreen document) local form filling 
 7-8 Gemini integration (analysis only) safe state recovery 
 9-10 Pre-fill mode worker review UI, initial earnings tracker 
 11-12 Backend (Supabase Auth, PostgreSQL, LLM proxy, license validation) 
 13-14 Basic popup UI, beta testing, bug fixes 

Beta Criteria:
- Extension loads without errors.
- Local task analysis works on 80 of MTurk survey pages.
- Pre-fill correctly populates 90 of Likert scale fields.
- Supabase Auth and license validation works.

 Phase 2: Beta Iteration (Weeks 15–18)

- TurkerView integration
- Attention check detection (Gemini Pro)
- Batch mode
- Bug fixes from beta feedback
- Expand to Clickworker UHRS

 Phase 3: Paid Launch (Weeks 19–22)

- Stripe Paddle subscription integration
- All 4 pricing tiers activated
- Public Chrome Web Store listing
- Launch marketing

---

 14. Open Questions

1. MTurk ToS enforcement depth — Does Amazon use advanced behavioral detection (Recommendation: assume yes, design for compliance not cloaking).
2. Multi-account support — Allow workers to manage multiple MTurk accounts (Legal gray area, defer to Phase 2, require explicit worker acknowledgment of risks).
3. B2C vs. B2B — Consumer subscription ($14-50/mo workers) OR enterprise sales ($500-5000/mo to requesters) (Recommendation: focus B2C for MVP, explore B2B later once product is mature and compliant).

---

 15. Appendix

 15.1 Competitive Reference Pricing

 Tool Price Notes 
--------------------
 BZTurk $7/mo Auto-accept only, no AI 
 Queuebicle $15–30/mo Windows only, no AI 
 TurkerAI (proposed Pro) $14.99/mo AI task execution (local DOM) 
 TurkerAI (proposed Power) $29.99/mo Full-featured AI (local DOM) 

 15.2 Key Research Sources

- Fowler et al. (2022) — MTurk worker demographics and frustration survey (n1,369)
- Mordor Intelligence — Micro-tasking market size 2024–2031 (with caveats on AI training data conflation)
- TurkerView API documentation v3
- Recent arXiv papers on web agents and chrome.offscreen API.

 15.3 Glossary

 Term Definition 
------------------
 HIT Human Intelligence Task — a single microtask on MTurk 
 Requester Company/person who posts HITs 
 TurkerView Third-party reputation database for MTurk requesters 
 UHRS Amazons Human Microtask Crowd Source — Clickworkers platform 
 Likert scale Survey question with 5–7 point scale (strongly disagree → strongly agree) 
 "Bubble hell" Worker slang for surveys with hundreds of identical Likert items 
 Approval rate of workers submitted HITs that werent rejected 
 CFAA Computer Fraud and Abuse Act (US Federal Law) 
 MV3 Chrome Extension Manifest V3 

---

Document Status: Ready for Technical Design and Spike Planning
