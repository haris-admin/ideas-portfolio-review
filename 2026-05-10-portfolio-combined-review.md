# 🚨 PORTFOLIO REVIEW — 5 AU RegTech Projects

**Date:** 2026-05-10 | **Reviewers:** 5x Gemini 3 Pro Subagents (Parallel)
**Status:** Hard Critique — No Sugar-Coating

---

## 📊 At a Glance

| Project | Completeness | Security | GTM Readiness | Verdict |
|---------|:-----------:|:--------:|:-------------:|:-------:|
| **AgentGate** | 60% | ❌ CRITICAL | ❌ Not ready | HOLD for security fix |
| **AuditTrail AI** | 40% | 🟡 Low risk | 🟡 Ready | GO — publish Sheets |
| **GridPass** | 15% | 🟢 No code | 🟡 Ready | GO — build calculator |
| **PitGuard** | 100% | 🟢 No code | 🟢 Ready | GO — publish now |
| **VerifyLink** | 80% | 🟢 Low risk | 🟢 Ready | GO — deploy site |

---

# 🔴 AGENTGATE — Full Critique

## Problem
AI agents operate autonomously in regulated AU enterprises with **zero governance layer**. APRA CPS 234 and AUSTRAC require immutable audit trails. No existing tool solves this for agentic AI.

## Technical Critique — Failure Points

1. **🚨 ZERO AUTHENTICATION** — Every FastAPI endpoint (`/agents`, `/audit`, `/reports`) has no `Depends()`. Anyone who can reach the API can register agents, forge audit logs, and download compliance reports. This is a **production blocker**.

2. **🚨 AUDIT CHAIN IS IN-MEMORY** — `AuditChain` stores entries in `self._entries = []` — a Python list. **Restart = total data loss.** Despite having SQLAlchemy `AuditEntryModel` in the DB schema, the core audit engine never writes to it. The "immutable audit trail" doesn't survive a container restart.

3. **🚨 HARDCODED SECRETS IN DOCKER COMPOSE** — `POSTGRES_PASSWORD: agentgate` is plain text in `docker-compose.yml`. No `.env` file reference. No vault or secrets manager.

4. **🚨 POSTGRES EXPOSED TO HOST** — Port `5432:5432` mapped directly. Any process on the host (or network, if misconfigured) can connect directly to the DB, bypassing the API layer entirely.

5. **🚨 ANYONE CAN GENERATE REPORTS** — The `/reports/generate` endpoint produces APRA/AUSTRAC PDFs. These are regulator-ready evidence packs — and any unauthenticated user can generate them for any agent.

6. **❌ No RBAC** — No concept of admin/compliance/developer roles. Everyone can do everything.

7. **❌ No audit of the audit** — No access log for who viewed or generated compliance reports.

8. **❌ No deployment documentation** — `README.md` shows `docker compose up` but no prerequisites, no config guide, no troubleshooting.

## Security Fix Recommendations (Priority Order)

1. **IMMEDIATE: Add JWT/OAuth2 middleware** to all API routes. Use FastAPI `Depends(get_current_user)`. Block all anonymous access.
2. **IMMEDIATE: Persist AuditChain to PostgreSQL.** Wire `AuditChain.append()` to write `AuditEntryModel` rows. Remove the in-memory-only path.
3. **HIGH: Move secrets to `.env`.** Remove hardcoded passwords from `docker-compose.yml`. Add `.env.example` to the repo.
4. **HIGH: Remove host port bindings** for PostgreSQL and direct API ports. Force all traffic through Nginx reverse proxy.
5. **MEDIUM: Add Nginx security headers** (HSTS, X-Content-Type-Options, CSP, CORS).

## GTM Fix

**Current GTM:** Open-source core → enterprise SaaS
**Problem:** No auth means the "open-source core" is a security incident waiting to happen. No enterprise will touch this.

**Fix:**
1. Ship a **locked-down version first** (with auth, persistent audit) as the OSS core
2. Enterprise features are: RBAC, SSO/SAML, audit viewer, compliance report scheduling
3. Pricing: Free tier (1 agent, 7-day audit retention) → Pro ($199/mo, 10 agents, 90-day) → Enterprise ($custom, unlimited)
4. **Target audience:** Not "regulated enterprises" broadly. Target **AU fintech CISOs** specifically — they're actively looking for agent governance tools right now

---

# 🟡 AUDITTRAIL AI — Full Critique

## Problem
APRA's 30 April 2026 letter demands AI governance uplift from 100+ entities. No tool maps to APRA's specific requirements.

## Technical Critique — Failure Points

1. **⚠️ THE SHEETS TEMPLATE DOESN'T EXIST.** The entire Phase 1 is documented, GRILL'd, OpenSpec'd — but there is **zero actual Excel/Sheets output**. 40% completeness is generous — there's no deliverable users can touch.

2. **⚠️ No validation interviews done.** The GRILL correctly identifies this as necessary, but it hasn't happened yet.

3. **✅ Everything else is well-structured.** The OpenSpec is thorough. The shift from Sheets to SaaS makes sense.

## GTM Fix

**Current GTM:** LinkedIn organic → Notion landing page → email capture
**Problem:** Too passive. The APRA letter is 10 days old. Every day of delay = lost first-mover advantage.

**Fix (Aggressive):**
1. **Ship the Sheets template within 48 hours.** It's AUD 250 budget — not a software project. Build the formulas, publish with "Make a Copy", done.
2. **Targeted LinkedIn outreach** — not organic posts. DM 20 compliance officers at AU banks/insurers directly. Offer free template in exchange for 15-min feedback call.
3. **Pricing hack:** Phase 2 SaaS at $2,500-5,000/mo is too wide. Price at **$2,999/mo flat**. The number is specific, not a range.
4. **Urgency hook:** "APRA expects your response within 6 months. Most compliance teams will take 4-5 months just to understand what's required. Start now."

---

# 🟠 GRIDPASS — Full Critique

## Problem
AEMC draft rule ERC0394 creates compliance confusion for data centre developers. Consultants cost $50K-200K+.

## Technical Critique — Failure Points

1. **⚠️ 15% COMPLETE.** The markdown calculator frameworks are well-written, but there is **zero working code**. No backend, no interactive frontend, no API. The `backend/` folder is a README placeholder.

2. **⚠️ HTML demo is static.** The demo shows what the tool *could* look like but provides zero actual calculation.

3. **⚠️ Scoring logic not validated.** The 20 compliance items, weights, and modifiers are well-researched but have never been tested against a real data centre project.

## GTM Fix

**Current GTM:** Open-source calculator → consulting leads
**Problem:** A static markdown calculator generates zero leads. Nobody finds it, nobody uses it.

**Fix:**
1. **Build a Streamlit calculator in 3 days.** The scoring logic is clearly defined — this is a trivial implementation. Streamlit is free and deployable to Streamlit Cloud in minutes.
2. **Email gate the detailed report.** Free score preview → enter email → full remediation report.
3. **Partner with energy consultants.** Give them the tool for their clients. They get efficiency, you get referral fees.
4. **Don't monetize the tool.** It's a **lead generation engine** for consulting referrals and data centre advisory. The calculator is free. The value is in the network.

---

# 🔴 PITGUARD — Full Critique

## Problem
Mining OT supply chain attacks are the dominant cyber threat of 2026. No OT-specific supply chain security tool exists for Australian mining.

## Technical Critique — Failure Points

1. **✅ Phase 1 framework is complete.** 29 files, zero TODOs. SBOM template, risk matrix, procurement checklist, IR playbook — all solid.

2. **⚠️ No real-world validation.** Two planned interviews haven't happened. The risk matrix weights are theoretical — they need industry feedback.

3. **⚠️ Phase 2 SaaS has no committed buyers.** The PRD correctly requires an LOI before building, but no LOI has been pursued.

## GTM Fix

**Current GTM:** Open-source → industry body adoption → authority → consulting
**Problem:** "Build it and they will come" doesn't work for OT security frameworks. This needs active industry push.

**Fix:**
1. **Publish the whitepaper TOMORROW.** LinkedIn article announcing the framework. Tag MCA, AEMO, and mining CISOs.
2. **Don't wait for interviews — use the whitepaper to get them.** "I published this framework — would love your feedback as an industry expert."
3. **Pitch to the MCA Minerals Week 2026.** The MCA already proposed AI-for-EPBC. OT supply chain security is the next logical topic.
4. **Monetization path:** Don't sell Phase 2 SaaS. Sell **assessment services** — use the framework to audit mining companies' OT supply chains. AUD 15-30K per engagement. Much faster path to revenue than building a SaaS.

---

# 🔵 VERIFYLINK — Full Critique

## Problem
Three H2 2026 regulatory shifts (AGDIS, CDR sunset, Digital ID) converge. Fintechs need a single playbook.

## Technical Critique — Failure Points

1. **⚠️ 80% for Phase 1 but SITE IS NOT DEPLOYED.** The Docusaurus site is fully built but not published to GitLab Pages. Zero traffic = zero value.

2. **⚠️ ROI calculator is markdown, not interactive.** A revenue calculator in static text is a guide, not a tool. It needs real JavaScript calculation.

3. **⚠️ No dependency audit.** `package-lock.json` exists but no CI/CD pipeline running `npm audit`. Risk of shipping with vulnerable dependencies.

## GTM Fix

**Current GTM:** Free guide → API product
**Problem:** A not-deployed guide generates zero leads. The 6-month AGDIS window is counting down.

**Fix:**
1. **DEPLOY THE SITE TODAY.** GitLab Pages = free, takes 5 minutes. `git push` the `main` branch. Get indexed by Google.
2. **SEO is everything.** Title tags: "AGDIS Integration Guide 2026 | AU Fintech Onboarding Compliance". This needs to rank for high-intent searches.
3. **Make the ROI calculator interactive.** Simple React component — 1 day of work. Without this, the site is a static brochure.
4. **Phase 2 API pricing:** Don't charge per-verification. Charge **$999/mo flat** for access to the orchestration API. Fintechs hate per-transaction pricing.
5. **Urgency hook:** "Screen-scraping sunsets in July 2026. AGDIS goes private in November 2026. You have 6 months to migrate. Here's how."

---

# 📦 Consolidated Action Plan

## Immediate (This Week)
1. **Publish PitGuard whitepaper** on LinkedIn
2. **Deploy VerifyLink** to GitLab Pages
3. **Build AuditTrail AI Google Sheets template** (48 hours)

## Short Term (Next 2 Weeks)
4. **Build GridPass Streamlit calculator** (3 days)
5. **Fix AgentGate auth** — add JWT middleware, persist AuditChain
6. **Run npm audit** on VerifyLink, add CI/CD pipeline
7. **Start validation interviews** for AuditTrail AI (target: 3 compliance officers)

## Medium Term (Next Month)
8. **Secure AgentGate docker-compose** — remove exposed ports, hardcoded secrets
9. **Deploy VerifyLink ROI calculator** as interactive React component
10. **Pitch PitGuard to MCA** for Minerals Week

## Long Term (Q3 2026)
11. **AgentGate** — Enterprise features (SSO, RBAC, audit viewer)
12. **AuditTrail AI Phase 2** — SaaS build (triggered by validation)
13. **VerifyLink Phase 2 API** — orchestration API development
14. **GridPass** — Consulting lead referral network

---

## 🔥 The Hard Truth

AgentGate is your only real **software product** play. It solves a real, growing problem and has actual code. But it cannot ship in its current state — zero auth means zero trust. Fix the security layer, then ship the lock-down OSS version, and it's the strongest project here.

The other 4 projects are **content/framework plays** — not software products. They establish authority, generate leads, and build your brand in specific AU regulatory niches. Each has a time-critical window (APRA letter: 10 days old. AEMC draft: 2 months old. AGDIS: 6 months out). Speed is everything.

**Your portfolio strategy:** One real product (AgentGate) + four lead-generation engines (AuditTrail, GridPass, PitGuard, VerifyLink). The content projects fund and feed the product pipeline.
