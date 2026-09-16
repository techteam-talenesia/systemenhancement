# Indonesia Data Protection (UU PDP) — Research Brief for System Architecture

> **Disclaimer:** This document is general research compiled from public sources (Indonesian government publications, established law-firm client alerts, and reputable compliance publications) to inform engineering and architecture decisions. **It is not legal advice.** UU PDP's implementing regulations are still being finalized (Indonesia's dedicated data protection authority, "Lembaga PDP," has not yet been formally established as of September 2026), and several provisions (DPO triggers, cross-border transfer mechanics, consent specifics) have recently been reshaped by Constitutional Court rulings. **Verify all of the below with qualified Indonesian counsel before finalizing system architecture, vendor contracts, or public-facing consent language.**

---

## What This Means Practically (Act-On-This-Now Summary)

- **Talenesia is almost certainly a Data Controller ("Pengendali Data Pribadi")** under UU PDP for student, lead, and mentor data — this brings direct obligations (lawful basis, data subject rights, security, breach notification), not just "best practice" concerns.
- **Cognitive test scores and "kondisi ekonomi" (economic condition) data are strong candidates for "specific/sensitive personal data"** treatment (financial data is explicitly listed; test/psychometric and economic-means data are adjacent and should be treated as sensitive out of caution) — this triggers stricter handling: explicit consent, tighter security, and it counts toward the DPO trigger.
- **A Data Protection Officer (DPO/PPDP) appointment is plausibly required** given a July 2025 Constitutional Court ruling that broadened the trigger to "any one of" three conditions (rather than all three) — large-scale sensitive-data processing (student cognitive + financial data across hundreds of students) likely satisfies this alone. Budget for this role or a fractional/outsourced DPO.
- **Replicating Moodle data to Google Cloud/BigQuery is a cross-border transfer question that needs an explicit answer**, not an assumption — check the actual GCP project/dataset region, and confirm Google's Data Processing Addendum (DPA) terms are executed and cover Indonesian PDP Law obligations.
- **Breach notification is fast: 3×24 hours (72 hours) to affected individuals and regulators** once a breach is confirmed — the current manual multi-spreadsheet workflow is a real exposure risk that should be prioritized for remediation before, not after, a warehouse migration.
- **Data deletion/right-to-be-forgotten must be a first-class system capability**, not a manual process — build a data-subject deletion/export function into the new architecture from day one rather than retrofitting it.
- **Lead-intake consent (Instagram DMs, ads, KOL links) needs a documented, granular, timestamped consent record** — bundled "by submitting this form you agree to everything" language is not defensible under UU PDP's consent standard.
- **Penalties are real but calibrated**: administrative fines up to 2% of annual revenue plus criminal liability (up to 5–6 years imprisonment / IDR 5–6 billion in fines for unlawful data obtaining/disclosure, higher for corporations). This is not GDPR-scale (4% of global turnover) but is not trivial either — weigh proportionately in the brief.

---

## 1. Is Talenesia a Data Controller, and What Are Core Obligations?

UU PDP (Undang-Undang Nomor 27 Tahun 2022 tentang Pelindungan Data Pribadi) defines the **Pengendali Data Pribadi (Data Controller)** as the party that determines the purpose of, and controls, personal data processing. Talenesia — determining why and how student, lead, and mentor data is collected and used — fits this role for essentially all the data flows in scope (leads, students, mentors). Where a vendor (e.g., Moodle host, GCP, Paper.id, an ATS/HRIS) merely processes data on Talenesia's instructions, that vendor is typically a **Data Processor (Pemroses Data Pribadi)**, with the Controller remaining primarily accountable.

Core Controller obligations under UU PDP include:
- **Lawful basis** for processing (consent is the primary basis for most of Talenesia's use cases; other bases exist for contractual necessity, legal obligation, etc.) and processing that is limited, specific, lawful, and transparent (purpose limitation).
- **Accuracy, completeness, and consistency** of personal data, and maintaining records of processing activities.
- **Security measures** to prevent unauthorized access, loss, or disclosure.
- **Data subject rights**, including the right to: access information about how data is processed; correct/update personal data; request deletion/erasure; withdraw consent; object to certain processing; and (per Article 16(2)(g) principles) data portability-adjacent rights around processing lifecycle (collection → processing → dissemination → deletion).

Sources: [Mondaq — Law No. 27 of 2022 High-Level Overview](https://www.mondaq.com/data-protection/1242654/law-no-27-of-2022-on-personal-data-protection-a-high-level-overview-on-the-new-personal-data-protection-law); [Library of Congress — Indonesia PDP Act Enters Into Force](https://www.loc.gov/item/global-legal-monitor/2022-12-18/indonesia-personal-data-protection-act-enters-into-force/); [XPND — UU PDP 27/2022 Guide](https://xpnd.co.id/regulatory/uu-pdp-27-2022-indonesia/)

**Practical note:** The two-year transition period after enactment ended **17 October 2024** — UU PDP is now in full legal effect with no grace period remaining.

---

## 2. Sensitive/Specific Personal Data — Do Test Scores and "Kondisi Ekonomi" Qualify?

**Article 4 of UU PDP** splits personal data into general and **"data pribadi yang bersifat spesifik" (specific/sensitive personal data)**. The explicitly listed specific categories are: health/medical data, biometric data, genetic data, criminal records, **personal financial data** (savings, deposits, credit card data, and similar), and children's data — plus a catch-all for "other data as determined by law."

Applying this to Talenesia's data:
- **"Kondisi ekonomi" (economic condition / means-testing data)** falls squarely within or immediately adjacent to the explicit "personal financial data" category — it should be treated as **sensitive data** requiring stricter handling.
- **Cognitive test scores** are not explicitly named in the public summaries of Article 4, but they function analogously to psychometric/assessment data used for high-stakes decisions (admission, career guidance) about an individual. Given the law's purpose-based, risk-based framing, and the fact that a wrong classification carries real penalty exposure, **the conservative and recommended posture is to treat cognitive test scores as sensitive data** pending explicit counsel confirmation, especially since it feeds admission/placement decisions.
- Specific/sensitive data processing carries **additional safeguards**: mandatory data protection impact assessments (DPIAs) for higher-risk processing, and it is one of the factors that can independently trigger the DPO requirement (see Section 3) when processed at scale.

Sources: [Library of Congress — PDP Act](https://www.loc.gov/item/global-legal-monitor/2022-12-18/indonesia-personal-data-protection-act-enters-into-force/); [ASEAN Briefing — Indonesia PDP Law Guide](https://www.aseanbriefing.com/doing-business-guide/indonesia/company-establishment/personal-data-protection-law); [Regulations.AI — UU PDP 27/2022 text summary](https://regulations.ai/regulations/RAI-ID-NA-URIN2XX-2022)

**Note on minors:** If any leads/students are under 18 at point of contact (common in bootcamp/education lead funnels reaching high-school-adjacent audiences), **children's data is explicitly listed as sensitive**, and UU PDP requires parental/guardian consent for processing children's data. A separate implementing regulation track ("PP TUNAS") specifically addresses children's digital data protection. Confirm Talenesia's actual lead age profile and build age-gating/parental consent flows if relevant.

Source: [Robere & Associates — PP TUNAS: Protecting Children's Data](https://robere.co.id/pp-tunas-child-data-protection-digital-age/)

---

## 3. Data Protection Officer (DPO) Requirement — Does Talenesia Cross the Threshold?

**Article 53 of UU PDP** requires Controllers and Processors to appoint a DPO ("Pejabat/Petugas Pelindungan Data Pribadi") when any of three conditions is met:
1. Processing is for **public interest** service delivery;
2. The **nature, scope, and/or purpose** of processing requires **regular and systematic monitoring** of data subjects on a large scale; or
3. The **core activities** involve **large-scale processing of specific/sensitive personal data** and/or data related to criminal offenses.

**Critical recent development:** On **30 July 2025**, Indonesia's Constitutional Court (Decision No. 151/PUU-XXII/2024) ruled that these three conditions should be read as **"and/or" rather than cumulative** — meaning **meeting just one condition triggers the obligation**, materially lowering the practical bar compared to earlier (stricter, cumulative) readings.

**Applied to Talenesia:** With ~450–480 concurrently active students whose records include cognitive test scores and economic/financial condition data (arguably sensitive, per Section 2), processed on an ongoing basis across the full student journey (admission → career guidance → payment → onboarding → simulation → internship matching), this plausibly satisfies condition 3 (large-scale sensitive-data processing) on its own under the post-2025 "and/or" standard — independent of whether it also involves "regular and systematic monitoring." **This is a scale/activity judgment call that should be confirmed with counsel**, but the risk-managed default is to plan for a DPO role (which can be a designated internal role with appropriate expertise, or an outsourced/fractional DPO service) rather than assume exemption.

Sources: [Assegaf Hamzah & Partners — Broader DPO Mandate Confirmed](https://www.ahp.id/indonesias-pdp-law-update-broader-dpo-mandate-confirmed-further-clarity-expected-on-data-disclosure-and-cross-border-transfers/); [Hogan Lovells — Indonesia Lowers Threshold for Mandatory DPO Appointment](https://www.hlc.com/en/publications/indonesia-lowers-threshold-for-mandatory-data-protection-officer-appointment); [DLA Piper — Data Protection Officers in Indonesia](https://dlapiperdataprotection.com/?c=ID&t=data-protection-officers)

---

## 4. Cross-Border Data Transfer Rules — Moodle → GCP/BigQuery, Paper.id

**Article 56 of UU PDP** establishes a **tiered ("sequential") test** for transferring personal data outside Indonesia:
1. **Adequacy**: the destination country/jurisdiction has data protection law equal to or higher than Indonesia's — Controller must verify this (currently the Government/future Lembaga PDP handles formal adequacy determinations; in the interim, Controllers are expected to undertake their own technical/legal verification of the destination's standards).
2. **If no adequacy**, the Controller must ensure **adequate and binding safeguards** exist — e.g., binding corporate rules or standard contractual clauses (SCC-equivalent) with the recipient.
3. **If neither of the above is satisfied**, the Controller must obtain the **data subject's explicit consent** for that specific cross-border transfer.

**Status as of September 2026 (relevant caveats):**
- Indonesia's dedicated **Data Protection Authority (Lembaga PDP) still does not formally exist** — a draft Presidential Regulation was submitted for presidential approval in mid-2026 but remained unsigned as of August 2026. In the interim, oversight sits with the **Directorate General of Digital Space Supervision** (under the Ministry of Communication and Digital Affairs / Komdigi), which can receive complaints and impose administrative sanctions.
- A February 2026 **US–Indonesia Trade Agreement** reportedly includes Indonesia recognizing the US as offering "adequate data protection" for trade-related transfers — but this requires parliamentary ratification and may not automatically satisfy UU PDP's formal adequacy-assessment requirement for general-purpose transfers (i.e., don't rely on this for GCP/BigQuery hosted in the US without separate verification).
- The Constitutional Court (2025/2026 rulings) upheld the tiered framework as constitutional and confirmed adequacy assessment is an executive/government function, while leaving some implementation details to forthcoming regulations.

**Engineering action items:**
- **Confirm the actual GCP project/dataset region** for the BigQuery replica of Moodle data. If it is not an Indonesia region (Google Cloud does not currently operate a full-region presence with all services inside Indonesia in the way "data residency" is sometimes assumed — this must be verified against current GCP region offerings), this is a cross-border transfer under Article 56 and needs a documented basis (safeguard mechanism or consent).
- **Execute/confirm Google Cloud's Cloud Data Processing Addendum (CDPA)** — available in Bahasa Indonesia — as the contractual safeguard layer for the Google relationship, and check whether it references Indonesia PDP Law specifically (Google has published compliance materials referencing Indonesia's PDP Law).
- **Paper.id is an Indonesian-incorporated entity**, so payment/invoice data processed within Paper.id's own infrastructure is likely to stay onshore by default — but confirm where Paper.id's own downstream processors (if any) are located, and get Paper.id's data processing terms in writing.
- Treat "where does this data actually live, and under whose jurisdiction" as a **standing architecture question** for every new integration (not just GCP/Paper.id) — industry partner data, email/marketing tools, and any future SaaS integration should go through the same check.

Sources: [Rajah & Tann Asia — Indonesia's PDP Law Updates: DPA, US Trade-Related Data Transfers, Recent Court Rulings](https://www.rajahtannasia.com/viewpoints/indonesias-pdp-law-updates-dpa-u-s-trade%E2%80%91related-data-transfers-and-recent-court-rulings/); [Makarim & Taira S. — PDP Law: Cross-Border Transfer Requirements](https://www.makarim.com/news/personal-data-protection-law-cross-border-transfer-requirements); [APPDI — Unpacking Constitutional Court Decision on Adequacy (137/PUU-XXIII/2025)](https://appdi.org/unpacking-constitutional-court-decision-on-adequacy-appropriate-safeguards-and-consent-for-international-data-transfer-decision-no-137-puu-xxiii-2025_/); [Adaptist Consulting — Indonesia's PDP Law Is Fully Enforceable, Its Regulator Still Doesn't Exist](https://adaptistconsulting.com/reports/indonesias-pdp-law-is-fully-enforceable-without-regulator/); [Google Cloud — Cloud Data Processing Addendum](https://cloud.google.com/terms/data-processing-addendum); [Google Cloud — Indonesia PDPL Compliance](https://cloud.google.com/security/compliance/indonesia-pdpl)

---

## 5. Breach Notification Obligations

**Article 46 of UU PDP** requires the Data Controller to deliver **written notification within 3×24 hours (72 hours)** of a personal data breach ("kegagalan pelindungan data pribadi") to:
- **Affected data subjects**, and
- **The relevant supervisory authority** (currently the interim Komdigi/Directorate General of Digital Space Supervision function, pending Lembaga PDP's formal establishment).

The notification must include, at minimum: **what data was disclosed/breached**, **when and how the breach occurred**, and **remediation/recovery steps taken**. If the breach disrupts public services or significantly affects the public interest, **the Controller must also notify the public**.

One important clarifying nuance from draft implementing regulation (RPP PDP) commentary: the 3×24-hour clock is understood to start once the breach is **confirmed with reasonable certainty** through an incident documentation/investigation process — not necessarily the exact moment of first suspicion. This does not excuse delay, but it does mean an organization needs a **defined incident-detection-to-confirmation process** to know when the clock legally starts.

**Direct relevance to Talenesia today:** the described manual multi-spreadsheet workflow for student/financial/economic data is a real breach-surface risk (shared spreadsheets are easy to over-share, hard to audit, and hard to remediate quickly within a 72-hour window). This is a strong argument for prioritizing **access control, audit logging, and centralization** as part of (or ahead of) the data warehouse build — not just for efficiency, but because the current state makes timely, accurate breach response (data was exposed, to whom, when, how) very difficult to execute.

Sources: [Mondaq/Lexology — PDP Law and Data Breach Notification Requirements in Indonesia](https://www.mondaq.com/data-protection/1237980/pdp-law-and-data-breach-notification-requirements-in-indonesia); [Conventus Law — PDP Law and Data Breach Notification Requirements in Indonesia](https://conventuslaw.com/featured-content/pdp-law-and-data-breach-notification-requirements-in-indonesia/); [Transatlantic Law — PDP Law and Data Breach Notification Requirements](https://www.transatlanticlaw.com/content/pdp-law-and-data-breach-notification-requirements-indonesia/)

---

## 6. Data Retention / Deletion / Right to Be Forgotten

**Article 16** defines the full lifecycle of "processing" as including collection, processing/analysis, updating/correction, dissemination/disclosure/transfer, **and deletion/destruction** — deletion is treated as a core processing stage, not an afterthought.

**Article 43(1)** (per secondary-source summaries) requires personal data to be **deleted or destroyed** when:
- The data is **no longer needed** for the purpose it was collected for;
- The data subject **withdraws consent**;
- The data subject **requests deletion**; or
- The data was **obtained/processed unlawfully**.

This is Indonesia's functional equivalent of a **"right to be forgotten,"** grounded in Article 16(2)(g)'s deletion principle. There is no single universal statutory retention period specified in the law itself for private-sector education/commercial data (retention periods for specific data types — e.g., tax/financial records — may instead be set by sector-specific regulations, such as accounting/tax record-keeping rules, which can be in tension with a "delete when no longer needed" instruction — this interaction should be confirmed with counsel, particularly for Paper.id-linked financial/invoice records).

**Engineering action items:**
- Define **explicit, purpose-tied retention periods** per data category (lead data, student PII, cognitive test scores, financial/payment records, mentor/contractor data) rather than "keep everything indefinitely."
- Build a **data-subject deletion/export function** (a "right to erasure/portability" capability) into the new data warehouse and integration layer from the start — this is far cheaper to build in at the schema/pipeline design stage than to retrofit later across Moodle, BigQuery, Paper.id, and any CRM.
- Reconcile retention against **financial record-keeping obligations** (Indonesian tax/accounting law may require multi-year retention of invoice/payment records even where PDP's "no longer needed" principle would otherwise suggest deletion) — this needs a documented, counsel-reviewed retention schedule, not an engineering guess.

Sources: [FPF — Indonesia's PDP Bill Overview](https://fpf.org/blog/indonesias-personal-data-protection-bill-overview-key-takeaways-and-context/); [Center for Digital Society — Right to Be Forgotten: Privacy Protection or Track Record Deletion?](https://digitalsociety.id/2022/12/24/right-to-be-forgotten-privacy-protection-or-track-record-deletion/); [KRTHA Bhayangkara Journal — The Right to Be Forgotten: Regulation of Personal Data Deletion in Indonesia](https://ejurnal.ubharajaya.ac.id/index.php/KRTHA/article/view/3291)

---

## 7. Consent Mechanics for Lead Acquisition (Website, Ads, Instagram DMs, KOL Links)

UU PDP's consent standard requires consent to be **specific, informed, and unambiguous** — general secondary-source guidance consistently states this **rules out pre-ticked/pre-checked boxes and bundled ("agree to everything at once") consent**. Practical implications:

- **Granular opt-in**: separate consent for different purposes (e.g., "process my contact info to respond to my inquiry" vs. "send me marketing/promotional messages" vs. "share my data with a KOL/partner for attribution tracking") should not be bundled into a single unavoidable checkbox.
- **Documented consent record**: Talenesia needs to be able to show **what** a lead consented to and **when** — a consent log tied to the lead/student record, not just a UI checkbox that leaves no trace.
- **Easy withdrawal**: withdrawing consent must be **at least as easy** as giving it, and withdrawal must actually **stop** the relevant processing (e.g., stop marketing sends, not just hide a UI toggle).
- **First-touchpoint channel diversity is the hard part here**: Talenesia's leads arrive via website forms, paid social ads, **Instagram DMs**, and **KOL/media-partner tracking links** — each of these has a different, weaker "natural" consent capture moment than a structured web form:
  - **Website/landing-page forms**: straightforward to fix — add explicit, unbundled consent checkboxes + timestamp capture.
  - **Instagram DM / social inbound**: no native structured consent UI exists on the platform; Talenesia needs a **defined process** (e.g., an automated first-reply message linking to a data-processing notice plus a follow-up structured intake form) before substantive personal data is captured into internal systems, rather than relying on the DM thread itself as "consent."
  - **KOL/media-partner tracking links**: since a third party (the KOL/partner) is the first point of contact, Talenesia should have **partner agreements** clarifying that the partner will not pass personal data to Talenesia without the lead's awareness/consent, and that any data received via a partner link still needs Talenesia's own consent capture at the point the lead actually provides personal data (e.g., on landing on Talenesia's own form).
- Because Article 4/56-related implementing detail is still evolving (a 2025 Constitutional Court decision explicitly left **detailed consent specifications to future implementing regulations**), treat current consent-language templates as a **living document** to be revisited once implementing regulations or Lembaga PDP guidance are published.

Sources: [DEV Community — Building for Indonesia: What UU PDP Actually Requires From Developers](https://dev.to/hem_081a27fed379/building-for-indonesia-what-uu-pdp-actually-requires-from-developers-a-practical-checklist-1i0o) *(practitioner-oriented secondary source — verify specifics with counsel)*; [APPDI — Unpacking Constitutional Court Decision on Consent for International Data Transfer](https://appdi.org/unpacking-constitutional-court-decision-on-adequacy-appropriate-safeguards-and-consent-for-international-data-transfer-decision-no-137-puu-xxiii-2025_/); [Rajah & Tann Asia — Indonesia's PDP Law Updates](https://www.rajahtannasia.com/viewpoints/indonesias-pdp-law-updates-dpa-u-s-trade%E2%80%91related-data-transfers-and-recent-court-rulings/)

---

## 8. Contractor/Mentor Data (HR-Adjacent) — Does UU PDP Treat It Differently From Student Data?

UU PDP does **not** carve out a separate legal regime for employment/contractor data — the same Controller obligations, lawful-basis requirements, and data subject rights apply. However, the **practical basis for processing differs**:

- For the ~100 freelance/contracted mentors, processing tied directly to the service contract (identity verification, hours worked, payment calculation) can generally rely on **contractual necessity** as the lawful basis, rather than needing standalone consent for every processing activity connected to fulfilling the contract — similar to how employee data is typically justified via employment-relationship necessity plus legal obligations (tax reporting, etc.).
- Where mentor data includes **NIK (national ID number), NPWP (tax ID), bank account details for payroll**, this is sensitive/financial-adjacent data requiring the same stricter security posture as student financial data.
- If a payroll or HR platform/vendor is used to calculate and disburse mentor payments, that vendor is a **Data Processor**, and Talenesia (as Controller) remains primarily accountable — a **Data Processing Agreement** with that vendor is advisable, mirroring the Paper.id/GCP vendor-contract needs discussed above.
- **Distinction from student data for the brief**: student data (especially cognitive scores, economic condition) leans more heavily on **consent** as the lawful basis and carries stronger "specific/sensitive data" classification risk; mentor/contractor data leans more on **contractual/legal-obligation** bases, which is somewhat more stable operationally but does **not** reduce the security or breach-notification obligations — a payroll data leak is just as reportable within 3×24 hours as a student data leak.

Sources: [Talenta — UU PDP and HR: A Practical Compliance Roadmap](https://www.talenta.co/en/blog/hr-uu-pdp-compliance-roadmap/); [ADCO Law — From Recruitment to Offboarding: Compliance Challenges under the PDP Law](https://adcolaw.com/blog/from-recruitment-to-offboarding-compliance-challenges-under-the-personal-data-protection-law/); [Adaptist Consulting — Implementing Indonesia's PDP Law: Obligations, Challenges, Where to Begin](https://adaptistconsulting.com/blog/implementing-indonesias-personal-data-protection-law-obligations-challenges-and-where-to-begin/)

---

## 9. Enforcement and Penalties — What's Actually at Stake

UU PDP has a **layered enforcement regime**: administrative sanctions plus, for the most serious violations, criminal liability.

**Administrative sanctions** (for general non-compliance — e.g., inadequate security, failure to honor data subject rights, inadequate consent practices):
- Range from a **written warning**, to **temporary suspension of processing activities**, up to **administrative fines of up to 2% of annual revenue/turnover**.

**Criminal liability** (for the more serious, typically intentional, categories of violation — e.g., unlawfully obtaining/collecting someone else's personal data for personal benefit, or unlawfully disclosing personal data):
- Figures cited across sources vary somewhat by which specific article is triggered, but the consistent range reported is **up to 5–6 years imprisonment and/or fines in the range of IDR 5–6 billion (roughly USD 320,000–400,000)** for individuals.
- **For corporations found in breach** of the intentional-violation provisions, penalties can be **multiplied up to 10x** the individual maximum, plus **confiscation of profits/assets**, **freezing of part or all of the business**, and **closure of part or all of the business** — this is a materially higher-stakes exposure category than the administrative-fine track.

**Calibration note for the brief**: this is **not GDPR-scale** (GDPR's headline fines run up to 4% of *global annual turnover* or €20M, whichever is higher), but the **2% of annual revenue administrative fine plus real criminal exposure for intentional/serious violations** is a genuine business risk that should be represented accurately — neither dismissed as "just a compliance nice-to-have" nor inflated into an existential threat for good-faith operational gaps. The bigger near-term practical risk for Talenesia is likely **reputational and operational** (breach affecting hundreds of students' cognitive/financial data, with a 72-hour disclosure clock and no dedicated regulator yet to guide a "soft landing") rather than the criminal-liability tail risk, provided the company is not engaged in intentional misuse.

Sources: [Schinder Law Firm — Sanctions and Compliance with Indonesia's PDP Law by October 16, 2024](https://schinderlawfirm.com/blog/sanctions-and-compliance-with-indonesias-personal-data-protection-law-uu-pdp-by-october-16-2024/); [BDO Indonesia — Introduction of the Official PDP Act](https://www.bdo.co.id/en-gb/insights/introduction-of-the-official-personal-data-protection-act-(uu-pdp)); [Library of Congress — Indonesia PDP Act Enters Into Force](https://www.loc.gov/item/global-legal-monitor/2022-12-18/indonesia-personal-data-protection-act-enters-into-force/); [Fortra — Indonesia's PDP: Requirements, Penalties & Compliance](https://www.fortra.com/blog/navigating-indonesias-personal-data-protection-law-key-takeaways-organizations)

---

## Related Context: Electronic Systems Registration (PP 71/2019) and Regulatory Landscape

Separately from UU PDP itself, **Government Regulation No. 71 of 2019 ("GR/PP 71/2019") on Electronic Systems and Transactions** requires **Electronic System Operators (ESOs)** — which likely includes Talenesia's own digital platforms (website, learning portal, internal systems if privately operated as an electronic service) — to **register with Kominfo/Komdigi (PSE registration)**. This is a distinct but related obligation from PDP compliance and should be confirmed as part of the same architecture/compliance review (i.e., check PSE registration status alongside PDP compliance, not as an afterthought).

Sources: [SIP Law Firm — Key Points of GR No. 71 of 2019](https://siplawfirm.id/key-points-of-government-regulation-no-71-of-2019-on-organization-of-electronic-systems-and-transactions); [Jagamaya — Understanding Indonesia's PP 71/2019](https://jagamaya.com/understanding-indonesias-pp-71-2019-what-it-means-for-your-data/)

---

## Recommended Technical Safeguards Checklist

Use this as a starting punch-list for the data warehouse + integration layer design. Each item should still be reviewed with counsel for completeness, but these are concrete, buildable items derived directly from the sections above.

**Consent & lead intake**
- [ ] Add explicit, **unbundled** consent checkboxes (not pre-checked) at every lead-intake form, with separate checkboxes for "contact me about my inquiry" vs. "marketing communications" vs. "share with partner/KOL for attribution."
- [ ] Capture and store a **consent record** (what was agreed to, exact text/version shown, timestamp, channel/source) linked to the Lead/Student ID — not just a boolean flag.
- [ ] Build a defined intake flow for **Instagram DM and other unstructured social inbound** that routes the lead to a structured consent-bearing form before personal data is written into core systems.
- [ ] Add data-processing disclosure language to KOL/media-partner agreements and confirm partners aren't passing personal data to Talenesia without the lead's awareness.
- [ ] Build a **consent withdrawal** mechanism that is as easy to use as opt-in, and wire it to actually stop downstream processing (not just flip a UI flag).

**Data subject rights**
- [ ] Build a **data-subject access/export function** (self-service or ops-assisted) covering all core systems (CRM/lead data, student records, Moodle/BigQuery learning data, Paper.id-linked financial records).
- [ ] Build a **data-subject deletion/erasure function**, including a defined process for propagating deletion across Moodle, BigQuery, CRM, and any other downstream copies — and a documented exception process for data that must be retained for legal/financial record-keeping reasons.
- [ ] Add a **correction/update** workflow so students/leads/mentors can request and have PII corrected, with an audit trail of the change.

**Sensitive data handling**
- [ ] Classify and tag **cognitive test scores, "kondisi ekonomi" data, and financial/payment data** as sensitive fields in the data model (a dedicated schema flag, not just documentation).
- [ ] **Encrypt sensitive fields at rest** (cognitive test scores, economic-condition/means-testing data, payment/financial data, national ID numbers) and enforce **role-based access control** so only staff with a legitimate need (e.g., admissions counselors, finance team) can view them — not blanket internal access.
- [ ] Add **access logging/audit trails** for reads/writes to sensitive fields, so a breach investigation can answer "who accessed what, when" within the 72-hour notification window.
- [ ] Evaluate whether a **Data Protection Impact Assessment (DPIA)** is warranted for the admissions/cognitive-testing and financial/economic-data processing flows, given the sensitive-data classification.

**Cross-border transfer / vendor architecture**
- [ ] Confirm the **actual GCP region** hosting the BigQuery replica of Moodle data; document whether this constitutes a cross-border transfer under Article 56.
- [ ] Confirm **Google Cloud's Data Processing Addendum (CDPA)** is executed and review its Indonesia PDP Law coverage.
- [ ] Confirm **Paper.id's own data-hosting/subprocessor locations** and obtain their data processing terms in writing.
- [ ] Build a standing **"data location and legal basis" checklist** applied to every new SaaS/vendor integration going forward (not just the current GCP/Paper.id/Moodle stack).
- [ ] If any transfer cannot be justified by adequacy or contractual safeguards, add an **explicit-consent capture step** for that specific transfer purpose.

**Breach readiness**
- [ ] Replace/retire the manual multi-spreadsheet workflows for sensitive student/financial data with access-controlled, auditable systems as a near-term priority — this is both a breach-likelihood reducer and a breach-response enabler.
- [ ] Define an **incident detection → confirmation → notification** process with clear ownership, so the 3×24-hour clock can realistically be met once a breach is confirmed.
- [ ] Pre-draft breach notification templates (to data subjects and to the interim regulator/Komdigi contact point) covering the legally required content: what data, when/how breached, remediation steps taken.

**Retention & governance**
- [ ] Define an explicit **retention schedule per data category** (leads, student PII, test scores, financial/payment records, mentor/contractor data), reconciled against Indonesian tax/accounting record-keeping requirements for financial data.
- [ ] Build **automated purge/anonymization jobs** tied to the retention schedule, rather than relying on manual cleanup.
- [ ] Maintain a **record of processing activities (ROPA)**-style register mapping each data category to: purpose, lawful basis, storage location, retention period, and downstream sharing (Paper.id, GCP, industry partners).

**Governance / org readiness**
- [ ] Evaluate and budget for a **DPO role** (internal designee or outsourced/fractional), given the plausible trigger from large-scale sensitive-data processing.
- [ ] Confirm **PSE (Electronic System Operator) registration status** with Komdigi for Talenesia's digital platforms, alongside the PDP compliance review.
- [ ] Establish a **Data Processing Agreement (DPA)** template for all vendors/processors (GCP, Paper.id, any payroll/HR platform, any future CRM/marketing tool).
- [ ] Treat this document as a **living reference** — revisit once Indonesia's Lembaga PDP is formally established and implementing regulations (especially around consent specifics and cross-border adequacy determinations) are published, as several open questions above are explicitly pending regulatory clarification.
