# Governance — learner Q&A and model outputs

Kaltura Djinn sends **learner questions** and receives **model-generated answers** server-side. Your institution should treat these as **education records** under **FERPA** (U.S.) and any local policy.

## What to decide (document before wide rollout)

1. **Logging** — Does Djinn or your reverse proxy log request bodies, URLs, or headers? The embedded LLM proxy emits **structured JSON lines** to **stdout** for each upstream request/response (see `src/llm-gateway/logging.ts`). Confirm whether your log aggregator retains those lines and who can query them.
2. **Retention** — How long are logs and any backups kept? Align with records-management and privacy office guidance.
3. **Third parties** — Questions and caption excerpts are sent to your configured **LLM provider** (e.g. OpenAI). Ensure **DPAs / terms** cover instructional use and subprocessors.
4. **Minors / sensitive topics** — Course teams should follow existing norms for AI use in the classroom; this document does not replace syllabus or IRB requirements.
5. **Incident response** — Who is contacted if keys leak, logs expose content, or an integration misroutes data?

## Operational hygiene

- Store **`LLM_*`** and **`KALTURA_*`** secrets in a **managed secret store**, not in ticket systems or chat.
- Rotate keys if exposure is suspected.
- Restrict production log access to need-to-know roles.

## Disclaimer

This file is **operational guidance**, not legal advice. Consult **privacy / legal / compliance** for your organization.
