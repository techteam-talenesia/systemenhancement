# Talenesia Business Process & Data Systems Brief

**Audience:** engineers/engineering team being hired to build Talenesia's data & systems platform
**Purpose:** explain the business (sales/acquisition → student graduation), the current data problem, why two prior attempts to fix it failed, and what a workable solution needs to look like — so a new engineer can get productive fast without re-discovering all of this by trial and error.

**Status:** first draft, compiled from an interview with Talenesia's CLO plus direct research into Paper.id's API and Indonesian data-protection law. Open questions are marked explicitly at the end — this is a brief to build shared understanding, not a locked spec.

---

## 1. The two problems this brief exists to solve

1. **Data is scattered and not interoperable.** Student, sales, and learning data lives across many disconnected spreadsheets and a few real systems, with mostly manual import/export between them. This directly hurts team efficiency and the accuracy/quality of service to students.
2. **Two prior attempts to fix this failed to get adopted** — an ERP build and an AI-based platform. Understanding *why* they failed is treated as first-class input to this brief, not a footnote (see §5).

---

## 2. The business, end to end

Talenesia runs a paid, cohort-based training program ("KSDK") that takes a student from lead → admitted student → trained professional → placed graduate. Three roles are involved throughout: the **student/customer**, the **Sales/EC (Education Consultant) team**, and **Educators** (Mentor, Tutor, SME sub-roles).

### The 9-phase journey

| Phase | What happens | Current system of record |
|---|---|---|
| **1. Lead acquisition & sales** | Leads arrive via website, social, ads (unique link per channel/post/campaign), events, Instagram DM, media partners/KOL — each with a unique lead ID. EC works the lead via omni-channel, ideally logs hot leads into CRM, gives a quotation (price + payment scheme). Branches: accept → Sales Order (SO) approved; reject → alternate payment scheme offered (different scheme = different Product ID); still reject → ends. | **Qontak (CRM)** |
| **2. Admission** | Demographic data + cognitive test. Fail → 1 retake. Still fail → check passing grade for other majors → offer alternative or end. Pass → book career-guidance (bimkar) slot, get mentor assignment. | Mastersheet Admission (Google Sheet) |
| **3. Career guidance / bimkar (pre-enrollment)** | Mentor reviews student data, runs the session. Student gives feedback; mentor files a report and sets final major + package. A PKS (cooperation agreement/contract) is generated from that; student reviews and signs. If student disagrees with the final quotation → new quotation loop; if agrees → payment. | Google Form response sheet ("[Pelatihan Lengkap] Pendaftaran Fullstack") |
| **4. Payment & registration** | Invoice generated, 1st payment processed. Student creates a **Paper.id** account and pays. Student signs up to the student dashboard. Payment/status (DG/UG/Selesai) recorded. Sales enters bundle + payment scheme into the SO. Paper.id auto-bills installments. Special flows: batch deferral; full refund if student fails cognitive test after paying (registration fee of 85k is forfeited). | "DATA MASTER SALES" (Google Sheet) |
| **5. Onboarding** | WhatsApp group → pre-onboarding session (alumni sharing etc.) → LMS access → auto-grouped into squad + mentor + tutorial group → gets a mentor coordinator → schedule/deadlines/journey info → squad WA group for coordinating tutoring/mentoring. | — (WhatsApp + LMS access provisioning, largely manual) |
| **6. Work-simulation classes** | 7 projects, each preceded by class + tutoring cycles: class (attendance tracked), case study + reflection, group tutoring (attendance), group mentoring (attendance), 1:1 mentoring (1×/month), mini-project (scored on softskills + hardskills). After project 1, students are graded into 4 groups for internship placement: **A – Berdaya, B – Afirmasi, C – Bertumbuh, D – Kasus Khusus**, based on mini-project scores, softskills, CV, submission timeliness, and *economic condition*. | **Separate dashboard per batch** (Batch 9/10/11/12 each have their own sheet); underlying grades live in **Moodle**, replicated to **Google Cloud / BigQuery** (`moodle-raw-data` project) |
| **7. Internship matchmaking (3 rounds)** | Industry partners are classified by BD: A–Enterprise, B–Growth, C–Starter, D–Committed-for-special-talents. Round 1 (wk 1–3): A-students → A-partners, etc. Round 2 (wk 3–4): unmatched students/partners opened to everyone. Round 3 (wk 4–6): still-unmatched students manually placed by Talenesia; D-group students manually placed with committed partners. Accepted → partner issues contract, onboarding with student + mentor + partner, roles agreed. | Per-batch matchmaking master sheet |
| **8. Internship** | Month 1: journal, mentor monitoring, feedback, innovation project agreed with supervisor, 1st-month evaluation, IDP (Individual Development Plan), mentoring continues. Month 2 (async): feedback + innovation project work. Final: presentation, mentor + supervisor grade, student gets report card + internship certificate + a documented list of demonstrated competencies. | Per-batch internship monitoring dashboard (.xlsx) |
| **9. Job search** | Student picks a job-search package. If "guarantee" package: full support (tracker, mentoring, monitoring); placed → program ends (remaining installments, if any, restructured to a higher rate); not placed but mentoring quota remains → continues; breaks package rules → downgraded to non-guarantee. If no guarantee package → program ends after internship. | Per-batch job-search monitoring dashboard |

### Supporting (cross-cutting) processes

- **Payment relief:** student requests relief → options are tenor extension, deferral to "T-Later," or installment freeze (only if actively studying and eligible) → simulation generated → if approved, contract amended + invoice adjusted from the current billing month.
- **Downgrade/upgrade/batch deferral:** decision + consequences communicated to student; Sales edits the SO bundle and pushes updates to the accounting system and Paper.id — by hand.
- **Post-course billing:** remaining installments after the program ends are still tracked and auto-invoiced via Paper.id; non-payment → access to services is cut off.
- **Master reconciliation:** "Master Rekonsiliasi Talenesia" is meant to be the cross-functional reconciliation point for sales, finance, and learning data (see §4 — this is currently broken).
- **Contractor payroll (mentors/tutors/SMEs):** ~100 active freelance/contracted educators, paid monthly based on service hours, calculated from a separate tracking sheet against a separate fee-calculation sheet. *(Scope for this engineering effort — TBD, see open questions.)*

---

## 3. Team shape (who touches this data)

| Function | Headcount |
|---|---|
| Sales | 1 manager, 2–3 sales force, 1 sales data admin |
| BD (industry partnerships) | 1 manager, 2 staff |
| Learning Operations | 1 coordinator, 2 staff |
| Learning Program Manager / Educator Manager | 3 |
| Data Analyst | de facto owner of cross-system reconciliation (see §4) — headcount/reporting line TBD |
| Finance | responsible on paper for AR reconciliation, currently blocked by data quality (see §4) |
| Chief Learning Officer | 1 (this brief's author/product owner for the new engineering effort) |
| CEO | 1 |
| HR Ops | 1 (payroll) |
| Mentor / Tutor / SME | ~100 active, freelance/contracted directly, paid monthly by service hours |

**~16 core internal staff** plus a ~100-person contractor pool interact with this data pipeline in some form.

---

## 4. Where "scattered and not interoperable" actually bites

This isn't abstract — it's visible directly in the table above:

- **A new spreadsheet gets created per batch, per phase.** Batches 9, 10, 11, and 12 each have their own work-simulation dashboard, matchmaking sheet, internship dashboard, and job-search dashboard, rather than one system filtered by batch. Every batch launch means someone manually rebuilds trackers from scratch.
- **No single system carries a student through all 9 phases**, even though a **Student ID does exist** and does link Sales Order, invoice, and certificate. The gap is enforcement: when someone creates a new ad-hoc dashboard (which happens often, per phase 6–9), there's no guarantee they include the Student ID field — it's a discipline problem, not a missing-feature problem.
- **The "Master Rekonsiliasi Talenesia" is the clearest evidence of the core problem.** It was designed to be the shared reconciliation point across sales, finance, and learning — each team is supposed to keep their section current. In practice: it isn't wired to Paper.id, isn't wired to the working dashboards, and depends entirely on manual discipline that doesn't happen. As a result, **Finance — the team whose actual job is AR reconciliation — can't do it**, and a Data Analyst has become the unofficial, manual integration layer between systems that should talk to each other automatically.
- **This bites both across functions and within them.** Across: sales → admission → finance (Paper.id) → BD/matchmaking → mentors, each a manual handoff. Within: sales (initial SO quotation vs. the final PKS quotation reconciling in different sheets); within learning (per-batch dashboard fragmentation, plus Moodle's BigQuery data sitting separate from the batch monitoring sheets).
- **CRM usage may itself be inconsistent.** The process describes EC "ideally" logging hot leads into Qontak — phrasing that suggests inconsistent compliance, though this isn't confirmed since the CLO doesn't directly supervise sales. *(Open item — needs verification.)*

The practical cost: inaccurate/slow reporting, reconciliation that depends on one person's manual effort, and service-quality risk (e.g., a student's real payment or academic status not matching what staff see, because the record they're looking at isn't the current one).

---

## 5. Why the two prior fixes didn't stick — design principles for this one

### ERP (staged rollout)
It launched in stages, so at every stage the team had to keep using **all their existing tools plus the ERP** — pure overhead, no removed step, no visible benefit. It was too complex for the team to adopt on top of an already-fragmented toolset.

> **Design principle:** any new system component must **replace and reduce** the number of tools/steps for at least one team from day one. A staged rollout is fine, but each stage must retire something, not just add something. If a phase's rollout doesn't let someone stop using a spreadsheet, it hasn't shipped anything real yet.

### AI-based platform
The engineer wired up API calls but never did the actual work of grounding the system in Talenesia's real data (proper schema, context, RAG/fine-tuning) — so it couldn't reliably produce the answers the team needed.

> **Design principle:** an AI feature is a **data-engineering project first, an API call second.** Whoever builds it owns getting the underlying data clean, structured, and correctly modeled before wiring any model to it. Don't repeat this by hiring for API integration skills alone.

### A general lesson from both
Talenesia's non-technical, non-data-savvy staff (Sales, BD, LOps, mentors) are the actual daily users. A "correct" backend that they won't use is worthless. Whatever front end this project produces needs to be evaluated primarily on adoption, not feature completeness.

### An operational risk worth naming
A previous engineer did not share GitHub access to prior work, so recoverability of past ERP/AI-platform code and infrastructure is uncertain. **This repository (`techteam-talenesia/systemenhancement`) is itself the fix** — all engineering work for this effort should live here, in a Talenesia-owned account, from day one, with no exceptions for "I'll share it later."

---

## 6. Scale

- **150–160 students per batch**
- **New batch opens every 3 months**; each batch runs a **9-month journey** (3 months work-simulation, 3 months internship, 3 months job search)
- **3 batches running concurrently** at any given time → roughly **450–480 active students** in the pipeline at once
- ~100 active contracted mentors/tutors/SMEs

This is a data-modeling and workflow-integration problem, not a big-data problem — volume is modest and well within reach of a straightforward relational/warehouse design.

---

## 7. What's already confirmed technically feasible

### Payment integration (Paper.id)
Full API exploration is documented in [`platform/tooling/api/paperid_payment/`](../tooling/api/paperid_payment/README.md). Highlights:
- Auth is a simple static `client_id`/`client_secret` header pair (no OAuth) — straightforward to integrate, but the keys *are* the entire security boundary and need careful secrets handling.
- A staging environment exists (no KYC, payment simulation) — safe to build and test against before production.
- Endpoints exist to automate exactly what's manual today: Sales Invoice creation/status, Partner (customer) sync, balance/withdrawal, post-course billing.
- **Real gap to design around:** Paper.id's webhooks (Payment In, Payment Out, Invoice Paid, Disbursement) have **no documented signature verification**. Any real-time payment-status sync built on these needs to treat webhook payloads as untrusted and reconcile against Paper.id's own API rather than trusting callbacks blindly — worth a direct question to Paper.id's support team.

### Regulatory constraints (Indonesia's UU PDP)
Full research is documented in [`platform/docs/compliance/indonesia-data-protection-uu-pdp.md`](compliance/indonesia-data-protection-uu-pdp.md) — **compiled from public sources, not a legal opinion; needs review by Indonesian counsel before being treated as final.** Highlights:
- Talenesia is a **Data Controller** under UU PDP (Law 27/2022) for student, lead, and mentor data.
- **"Kondisi ekonomi" (economic condition) data is explicitly sensitive personal data** under the law; cognitive test scores should be treated as sensitive out of caution given they drive admission decisions.
- A 2025 Constitutional Court ruling likely means Talenesia **crosses the threshold requiring a Data Protection Officer**, given the scale of sensitive data processed.
- **Cross-border transfer is a live, unresolved question**: Moodle data replicates to Google Cloud/BigQuery, and the team needs to confirm the dataset's region and that Google's Data Processing Addendum is executed.
- **Breach notification is a 3×24-hour window** — the current all-manual, multi-spreadsheet state cannot realistically meet this.
- Practical build-in requirements: data-subject deletion/export capability, encryption + RBAC + audit logging for sensitive fields (test scores, economic condition, financial data, NIK), and structured consent capture at lead intake — especially for Instagram DM / KOL-link inbound leads, which currently have no consent moment at all.

---

## 8. What a solution needs to look like

Based on everything above, the CLO's framing is: **one properly structured backend/data warehouse that integrates all of this data, fronted by a UI simple enough for a non-technical, non-data-savvy team to actually adopt.**

Concretely, that implies:

- **Qontak, Paper.id, and Moodle likely stay as systems of record** for the teams that work in them daily (sales, finance, students) — the new layer integrates with them via API rather than replacing what already works, pulling their data into a central, properly modeled warehouse. *(To confirm explicitly — see open questions.)*
- **Every phase-specific tracking artifact (the per-batch dashboards) gets replaced by one system, filtered by batch** — not recreated by hand each cohort.
- **The Student ID becomes structurally impossible to omit** — if the new system is the only sanctioned place to create a tracking view, there's no more ad hoc spreadsheet to forget it in.
- **The Master Rekonsiliasi becomes a live, automatically-updating view**, not a manually-maintained spreadsheet dependent on team discipline — this alone would unblock Finance's actual AR reconciliation job.
- **Front-end design must optimize for adoption over completeness**, given the ERP's failure mode. Open question: should it feel spreadsheet-like (gentler transition for Sheets-native staff) or a guided, form-based app with fewer choices (addresses the discipline/compliance problem more directly)? Recommend the incoming engineer propose this explicitly rather than assume.
- **Defensive integration with Paper.id**, given the unverified webhook signatures (§7).
- **Compliance-aware from day one**: consent capture at lead intake, encryption/RBAC on sensitive fields, and a deletion/export capability — not bolted on later.

---

## 9. Success criteria

**Stated goal: data is reconciled and connected across platforms within 6 months.**

This should be broken down (by the incoming engineer, in collaboration with the CLO) into concrete, checkable milestones — e.g., which phases have a live single source of truth by which month, when Finance can run AR reconciliation without the Data Analyst manually bridging gaps, when a new batch launch no longer requires hand-building a new dashboard.

---

## 10. Open questions

These need answers before or during the engineering engagement — flagged here rather than guessed at:

1. **Contractor payroll** — is the mentor/tutor/SME fee-calculation process (service hours → monthly pay) in scope for this engineering effort, or a separate/later initiative?
2. **Budget and timeline for this hire** — even a rough range, to calibrate ambition (a warehouse + one integration is a very different sized engagement than warehouse + integrations + new front end).
3. **Front-end UX direction** — spreadsheet-like (familiar, gentler adoption curve) vs. guided/form-based app (fewer choices, addresses the discipline problem more directly)? Recommend the incoming engineer propose this rather than assume it.
4. **Qontak CRM compliance** — is "EC ideally logs hot leads into Qontak" actually followed, or are leads slipping through when EC skips CRM entry? Needs verification with the sales team directly (the CLO doesn't supervise sales day-to-day).
5. **Reusable assets** — is any part of the prior ERP or AI-platform build (infra, schema, data) recoverable, given the access issue with the previous engineer? Should be checked before assuming a fully greenfield build.
6. **Data Analyst's role going forward** — headcount, reporting line, and current tooling (manual spreadsheet work vs. any real SQL/BigQuery/scripting capability) — this person is likely a key internal collaborator for whoever gets hired.

---

## Appendix: source materials

- Original process document: *Business Process Program KSDK* (provided by CLO)
- Data landscape sources named in interview: Qontak (CRM), Paper.id (payment gateway — see API research below), Moodle + Google Cloud/BigQuery (`moodle-raw-data` project), and several Google Sheets (acquisition, admission, sales master, per-batch dashboards, mentor roster/fee calculation, master reconciliation) — not directly accessible to this brief's author (private/authenticated); structural details were relayed by the CLO rather than inspected directly.
- [`platform/tooling/api/paperid_payment/`](../tooling/api/paperid_payment/) — full Paper.id Open API reference (auth, invoices, payments, webhooks, partners, errors, reference data)
- [`platform/docs/compliance/indonesia-data-protection-uu-pdp.md`](compliance/indonesia-data-protection-uu-pdp.md) — Indonesian data protection law research
