# AI SDR Agent – End‑to‑End Build Plan (Tutor‑Led)

> **Context:** Internal tool (Project 3) to supercharge Sales & Marketing with an AI SDR Agent. Built primarily on **platform.openai.com** (Responses API, Files/RAG, tools), with a *thin Node layer* later for optional Outlook/Zoho/Teams. No new product—just a simple, powerful agent you run internally.

---

## 0) Principles & Guardrails

* **Internal copilot, not autopilot:** AI researches + drafts; **humans approve** everything external.
* **Keep it simple:** Use OpenAI’s built‑ins (Responses, Files, tools). Only add Node when absolutely needed.
* **Structured outputs:** All agent responses are strict JSON (schemas below) or clearly templated markdown for easy review.
* **Grounding over guessing:** Prefer files you upload + explicit snippets over general knowledge.
* **Document as you go:** Each milestone includes a **GitHub Update** block to paste into your repo docs.

---

## 1) Tools, Accounts, and Folders

**Why:** Clean separation + repeatable steps.

### 1.1 Accounts

* **OpenAI:** platform.openai.com (create a new *Project*: `SDR Agent (Project 3)`).
* **GitHub:** one repo: `sdr-agent-internal`.
* **(Later) Microsoft Entra / Graph, Zoho CRM, LinkedIn Sales Navigator:** create app registrations only when we hit Milestone 6.

### 1.2 Local Folders (suggested)

```
/sdr-agent-internal
  /docs            # this plan + decisions + runbooks
  /prompts         # system prompts, few-shots
  /schemas         # JSON schema definitions
  /samples         # example campaign inputs & outputs
  /scripts         # node scripts (later)
  /knowledge       # persona one-pagers, case studies (also upload to OpenAI Files)
```

### 1.3 Repo Setup (first commit)

* Create repo `sdr-agent-internal` (private). Add a `README.md` and `/docs/PLAN.md`.
* Copy this entire build plan into `/docs/PLAN.md`.

**GitHub Update (paste):**

```md
# Commit: repo bootstrap
- Created repo `sdr-agent-internal`
- Added `/docs/PLAN.md` with end-to-end tutor-led plan
- Established folder structure for prompts/schemas/samples/knowledge
```

---

## 2) Output Contracts (Schemas & Templates)

**Why:** Schemas make the agent reliable and easy to plug into UI and CRM later.

Create `/schemas/sdr-package.schema.json` with the combined schema:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "SDRPackage",
  "type": "object",
  "properties": {
    "campaign_summary": {
      "type": "object",
      "properties": {
        "campaign_name": {"type": "string"},
        "goal": {"type": "string"},
        "icp_summary": {"type": "string"},
        "personas": {"type": "array", "items": {"type": "string"}}
      },
      "required": ["campaign_name", "goal", "icp_summary", "personas"]
    },
    "persona_pack": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "persona": {"type": "string"},
          "problems": {"type": "array", "items": {"type": "string"}},
          "metrics": {"type": "array", "items": {"type": "string"}},
          "talk_tracks": {"type": "array", "items": {"type": "string"}},
          "proof_points": {"type": "array", "items": {"type": "string"}}
        },
        "required": ["persona", "problems", "metrics", "talk_tracks"]
      }
    },
    "account_plan": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "company": {"type": "string"},
          "tier": {"type": "string", "enum": ["1","2","3"]},
          "why": {"type": "string"},
          "angles": {"type": "array", "items": {"type": "string"}},
          "openers": {"type": "array", "items": {"type": "string"}}
        },
        "required": ["company", "tier", "why"]
      }
    },
    "research_cards": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "company": {"type": "string"},
          "person": {"type": "string"},
          "role": {"type": "string"},
          "highlights": {"type": "array", "items": {"type": "string"}},
          "sources": {"type": "array", "items": {"type": "string"}},
          "risk_notes": {"type": "string"}
        },
        "required": ["company", "highlights", "sources"]
      }
    },
    "message_map": {
      "type": "object",
      "properties": {
        "observations": {"type": "array", "items": {"type": "string"}},
        "problems": {"type": "array", "items": {"type": "string"}},
        "proof": {"type": "array", "items": {"type": "string"}},
        "cta": {"type": "string"}
      },
      "required": ["observations", "problems", "proof", "cta"]
    },
    "assets": {
      "type": "object",
      "properties": {
        "email_first": {
          "type": "object",
          "properties": {
            "subject": {"type": "string"},
            "body_markdown": {"type": "string"},
            "personalized_highlights": {"type": "array", "items": {"type": "string"}}
          },
          "required": ["subject","body_markdown"]
        },
        "email_bump1": {"$ref": "#/properties/assets/properties/email_first"},
        "email_bump2": {"$ref": "#/properties/assets/properties/email_first"},
        "linkedin_first": {"type": "string"},
        "call_opener": {"type": "string"},
        "loom_script_60s": {"type": "string"}
      },
      "required": ["email_first","linkedin_first","call_opener","loom_script_60s"]
    }
  },
  "required": ["campaign_summary","persona_pack","account_plan","research_cards","message_map","assets"],
  "additionalProperties": false
}
```

Create `/samples/campaign.input.sample.json`:

```json
{
  "campaign_name": "UK SaaS CMOs – Q4 Retention Push",
  "goal": "Book net-new meetings for retention analytics",
  "product_summary": "Analytics that predicts churn risk and automates playbooks",
  "icp": {
    "industries": ["B2B SaaS"],
    "regions": ["UK"],
    "company_size": "200-2000",
    "persona_roles": ["CMO", "Head of Marketing Ops"]
  },
  "value_props": [
    "Reduce churn 2–5 pts in 90 days",
    "Lift NRR via targeted expansion plays"
  ],
  "targets": {
    "companies": ["Acme.io", "BetaCloud.co.uk", "GammaSoft.com"],
    "people": []
  },
  "constraints": {
    "email_word_limit": 120,
    "tone": "concise, credible, UK English",
    "cta": "15-min intro call"
  }
}
```

**GitHub Update (paste):**

```md
# Commit: output contracts
- Added `/schemas/sdr-package.schema.json` (master schema for SDR outputs)
- Added `/samples/campaign.input.sample.json` (seed input)
- Decision: all agent outputs must validate against these schemas
```

---

## 3) OpenAI Project & Files (RAG) – Click Paths

**Why:** Give the agent your internal knowledge so it grounds outputs.

### 3.1 Create/Open Project (once)

* Go to **platform.openai.com** → top-left **Projects** → **Create project** → name: `SDR Agent (Project 3)`.
* Open **Settings → Billing** and ensure a payment method is active.

### 3.2 Upload Grounding Files

* In left nav, click **Files** → **Upload**.
* Upload 3–6 short docs (PDF/MD/TXT):

  * Persona one-pagers for your ICP roles
  * 1–2 case studies
  * 2 winning outbound emails (anonymised)
  * (Optional) The provided PDFs (*AE Frameworks*, *Modern Outbound*)
* Note the **File IDs** (we’ll reference them in prompts).

**GitHub Update (paste):**

```md
# Action: OpenAI Files
- Uploaded persona sheets, case studies, sample winning emails
- Recorded File IDs in `/docs/files.md`
- Decision: Use OpenAI File Search (built-in vector store) as primary RAG
```

---

## 4) System Prompt (Tutor-Led) & Few-Shots

**Why:** This is the agent’s brain + style rules.

Create `/prompts/system.sdr.md`:

```md
You are an internal **AI SDR Copilot** for our sales & marketing team.

**Role & Boundaries**
- You do **research** (web + our files) and produce **structured outputs**.
- You never send messages externally; you only provide drafts for human review.
- You **never invent facts**. Use only what you find in web/files/user input.

**Style**
- UK English, concise, plain language.
- First surface 2–3 **specific personalization insights**.
- Emails: <120 words for first touch, **one CTA**, no fluff.

**Workflow**
1) Parse campaign/task → clarify internally → plan your steps.
2) Research personas/accounts/signals (use tools). Capture sources.
3) Build **message_map**.
4) Draft **assets** (email, LinkedIn, call opener, Loom script).
5) Output **strict JSON** conforming to the provided schema.

**Grounding**
- Prioritize our uploaded files + direct quotes from web results.
- If unsure, say so in `risk_notes`.
```

Create `/prompts/fewshots.md` with 1 good example of: persona_pack item, research_card, email_first.

**GitHub Update (paste):**

```md
# Commit: prompts
- Added `/prompts/system.sdr.md` (role, style, workflow, grounding)
- Added `/prompts/fewshots.md` (example outputs)
- Decision: all runs include these prompts to standardise tone & structure
```

---

## 5) Prototype in OpenAI **Playground → Responses**

**Why:** Validate behavior before any code.

**Click Path:**

1. Go to **Playground** → **Responses**.
2. **Model:** select a GPT‑4 class model (e.g., `gpt-4o-mini`).
3. **System message:** paste `/prompts/system.sdr.md` content.
4. **User message:** paste JSON from `/samples/campaign.input.sample.json` (or your live campaign brief).
5. **Tools:** enable **Web / Search** and **File Search** (if available in your UI). Attach the **File IDs**.
6. **Structured Outputs:** if UI supports schema, select **JSON schema** and paste `/schemas/sdr-package.schema.json`. If not, instruct the model to output valid JSON and validate manually.
7. Click **Run**.

**Success Criteria:** You get a single JSON object with all top-level keys: `campaign_summary`, `persona_pack`, `account_plan`, `research_cards`, `message_map`, `assets`.

**Troubleshooting Tips:**

* If output is not JSON, prepend: *“Return ONLY valid JSON conforming to this schema:”* and paste schema.
* If research is thin, add more specific brief notes (e.g., known news) or upload more files.

**GitHub Update (paste):**

```md
# Test: Playground run
- Ran Responses with system prompt + sample campaign input
- Enabled Web + File Search; attached uploaded files
- Received valid SDRPackage JSON ✅ (screenshots stored in `/docs/playground/`)
```

---

## 6) Thin Node Runner (Optional but Recommended)

**Why:** Save/validate outputs, and pave the way for Outlook/Zoho later.

Create `/scripts/run-campaign.mjs` (Node 18+):

```js
import fs from 'node:fs/promises';
import OpenAI from 'openai';
import schema from '../schemas/sdr-package.schema.json' assert { type: 'json' };

const client = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

const system = await fs.readFile('./prompts/system.sdr.md', 'utf8');
const campaign = JSON.parse(await fs.readFile('./samples/campaign.input.sample.json', 'utf8'));

const response = await client.responses.create({
  model: 'gpt-4o-mini',
  input: [
    { role: 'system', content: system },
    { role: 'user', content: `Use this campaign input and output strict JSON for SDRPackage.\n${JSON.stringify(campaign)}` }
  ],
  // If your SDK supports structured outputs with a JSON schema, pass it here.
  // Otherwise, validate after the fact with a JSON schema validator.
});

const text = response.output_text || JSON.stringify(response, null, 2);
await fs.writeFile('./samples/campaign.output.json', text);
console.log('Saved to ./samples/campaign.output.json');
```

Add a JSON Schema validator later (Ajv) to fail the build if the output doesn’t match.

**GitHub Update (paste):**

```md
# Commit: thin runner
- Added `/scripts/run-campaign.mjs` to call Responses API and save outputs
- Next: add Ajv validation & small CLI flags for input/output paths
```

---

## 7) Review UX (No new app—just simple docs for now)

**Why:** Human-in-loop review before sending.

* Create `/docs/review-checklist.md`:

```md
# SDR Review Checklist
- [ ] Facts correct (company, names, dates)?
- [ ] Personalization highlights are real & relevant?
- [ ] Tone matches UK concise style? (<120 words first-touch)
- [ ] CTA is single, clear, and low-friction?
- [ ] Remove any risky claims or jargon.
```

* For now, copy `email_first` into Outlook manually. (Integration comes later.)

**GitHub Update (paste):**

```md
# Commit: review UX
- Added `/docs/review-checklist.md`
- Interim flow: human reviews JSON → copies email_first into Outlook → send
```

---

## 8) Enriching Research (Iterate)

**Why:** Better inputs → better outputs.

* Add more `knowledge/` docs (persona nuances, product FAQs, competitor snapshots). Upload to OpenAI Files.
* Create a **signals** note field in your campaign input (e.g., recent funding, new exec). Seed with things you already know to steer the model.
* Save **good outputs** in `/samples/`—they’ll become future few-shots.

**GitHub Update (paste):**

```md
# Action: enrichment
- Uploaded additional knowledge docs; updated File IDs
- Saved best outputs as few-shot examples in `/prompts/fewshots.md`
```

---

## 9) Optional Integrations (when you’re ready)

**Why:** Remove copy/paste and close the loop.

### 9.1 Outlook Drafts (Microsoft Graph)

* Register an app in **Microsoft Entra**; grant `Mail.ReadWrite`.
* Add a Node function (tool) `save_outlook_draft(to, subject, body_md)` and let the model call it *after approval*.

### 9.2 Zoho CRM (read-only first)

* Create a Zoho client; scope: Accounts/Leads read.
* Tool: `get_crm_context(company)` → last touch, owner, status. Feed into research stage.

### 9.3 Teams (review card)

* Post JSON summary + Approve/Regenerate buttons in a Teams bot card.

**GitHub Update (paste):**

```md
# Plan: integrations
- Defined tool specs: save_outlook_draft, get_crm_context
- Will implement after stable research/messaging quality
```

---

## 10) Quality & Safety

* **Guardrails:** strip unnecessary PII before sending to OpenAI; never store secrets in prompts.
* **Validation:** enforce schema with Ajv; block if invalid; log reasons.
* **Edits-as-learning:** keep a `/docs/edits-log.md` of human edits → adjust prompts/few-shots monthly.
* **Style lint:** create a short rubric (“no fluff, 1 CTA, specific insight first”).

**GitHub Update (paste):**

```md
# Commit: QA & safety
- Added Ajv validation plan; edits-log process; style rubric
```

---

## 11) What “Done” Looks Like (v1)

* You paste a **campaign JSON** (or brief) into the agent.
* Agent returns a **complete SDRPackage JSON** with:

  * Persona packs, account plan, research cards (with sources), message map, and ready-to-review assets.
* You review with the checklist and send from Outlook.
* No new app, minimal glue code, fully documented in your repo.

---

## 12) Tutor-Led Execution Order (we’ll do these together)

1. Repo bootstrap + folder structure ✅
2. Output schema + sample input ✅
3. OpenAI Files upload (RAG setup) ✅
4. System prompt + few-shots ✅
5. Playground run to validate JSON ✅
6. Thin Node runner + (later) Ajv validation
7. Review checklist & cadence
8. Enrichment loop (add docs, refine prompts)
9. Optional: Outlook/Zoho/Teams tools

> When you complete each step, paste the matching **GitHub Update** block into a commit or `/docs/CHANGELOG.md`. I’ll guide you step-by-step through each click and check.
