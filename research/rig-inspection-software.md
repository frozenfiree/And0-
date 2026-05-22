# Oil Drilling Rig Inspection & Reporting Software — Deep Research

> A research brief for a software product that replaces an experienced field inspector's pen / paper / photos / Microsoft Word workflow with a structured digital inspection and reporting system. Prioritises operational reality over startup hype. Every section ends with concrete implications for software design. All claims cited inline.

---

## TL;DR

The opportunity is **a professional rig inspector's report-writing tool** — not another safety checklist app, not another enterprise EAM. It sits in a gap the market has visibly left open: between $24/user generic inspection SaaS (SafetyCulture, GoCanvas, Fulcrum) that produces outputs needing post-export Word/Excel cleanup, and six-figure enterprise platforms (IBM Maximo, SAP PM, Sphera, Cority, Cenosco) that demand 6–18-month implementations and were never designed for a single inspector to walk on board, work for 5 days, and leave with a CEO-ready PDF.

Five highest-leverage facts:

1. **The deliverable is the product.** Inspectors are not paid for the day on the rig — they are paid for the bound, signed, sectioned PDF that lands in the CEO's inbox. The defect register, photo plates, executive summary, and standards-cited findings are the things money is exchanged for. Optimise for the deliverable, not the form-filling.
2. **Structure beats AI.** The highest-value automations — component-aware templates, photo→finding linkage at capture time, reusable findings library, deterministic PDF render, standards-clause citation, severity rule support — are all pure deterministic software. They ship in 90 days and save 15–30 hours per inspection. AI defect detection is overhyped and unsafe to ship to drilling contractors in v1.
3. **Offline-first is architecture, not a feature.** Connectivity inside a cellar, mud tank, or sub-base is zero; even Starlink Maritime stops at the steel. The product must launch, capture, photograph, sign and timestamp with no signal, then sync reliably when it returns. Conflict resolution must never silently overwrite the inspector's work.
4. **Hazardous-area zoning forces device choices.** Zone 1 / Class I Div 1 areas (cellar, BOP area, parts of the rig floor) forbid consumer phones. The product must run on Android (i.safe MOBILE, ecom) **and** Windows (Aegex, Getac) intrinsically-safe tablets — $3,800–$8,000 hardware, often one OS-version behind, with poor cameras.
5. **Start bottom-up with the inspector, expand to firms, contractor-direct comes last.** Bottom-up is 90 days to revenue. Inspection-firm sales (Bureau Veritas / ModuSpec, Add Energy, RigCert tier) is year two. Direct sales to drilling contractors (H&P, Patterson-UTI, Transocean) is year three — full SOC 2 Type II, MSA, IT security gate, 12–24-month cycle. Bottom-up dogfooding produces the case studies that unlock the rest.

The recommended 90-day MVP is laid out in §1 below.

---

## 1. MVP Recommendation

### 1.1 In one sentence

A **mobile-first, offline-capable inspection workbook** that replaces the inspector's notebook, photo roll, and Word template — and renders a CEO-ready PDF on the laptop the same evening.

Anything else in v1 is gold-plating.

### 1.2 Core entities

```
Inspection      (id, rig_id, inspector_id, scope, start_date, end_date, status,
                 ptw_number, jsa_ref, toolbox_talk_attendees)
Rig             (id, name, type[land|jackup|semisub|drillship|workover], owner,
                 builder, year, operating_hours_at_start)
Component       (id, rig_id, parent_id, code, name, system, specs, tag, serial,
                 last_recert_date, recert_interval_days)
Finding         (id, inspection_id, component_id, severity[critical|major|minor|obs],
                 risk_score_5x5, observation_text, recommendation_text,
                 reference_clause_id, status[open|closed|verified],
                 created_at, signed_by, signed_at)
Photo           (id, inspection_id, finding_id_nullable, captured_at, geo, exif,
                 file_ref, caption, plate_number, annotation_layer)
Recommendation  (id, finding_id, action_text, owner_role, due_date, est_cost,
                 est_downtime_days)
Report          (id, inspection_id, version, pdf_ref, generated_at, generated_by,
                 archetype[walkdown|audit|due_diligence])
StandardClause  (id, source[API_RP_4G|API_RP_8B|API_STD_53|IADC|OSHA_1910|
                 OSHA_1926|BSEE_30CFR250|NORSOK_D010|DNV_OSE101|ABS_MODU],
                 clause_ref, summary)
InspectorCert   (id, inspector_id, scheme[ASNT_SNT-TC-1A|CP-189|API_4G|IADC_RigPass|
                 BOSIET|HUET], method[VT|MT|PT|UT|RT|ET], level[I|II|III],
                 expiry_date)
CalibrationCert (id, instrument_serial, type, calibrated_at, expires_at, traceability)
```

Notes:

- `Photo.finding_id` **must be nullable** so capture is never blocked by "which finding does this belong to?" — link later in triage.
- `Component` is recursive (`parent_id`) so a rig-type template can populate `BOP → ram preventer → ram block` naturally.
- `Finding.severity` is a controlled vocabulary and `risk_score_5x5` is a separate field — operators ask for both.
- Reference clauses are first-class data so reports can defend each finding against the right standard (e.g. "API RP 4G §6.4.3" or "30 CFR 250.737(b)").
- Inspector certifications and instrument calibrations are modelled because reports are rejected by clients when a Level I signs an MT acceptance or a UT reading came from an out-of-cal probe.

### 1.3 The 5–7 mobile screens that matter

1. **Inspection dashboard** — list of components by system, progress bar, "X findings, Y critical."
2. **Component browser** — drill into the rig tree; tap a component to see its findings or create one.
3. **New / edit finding** — severity picker, component (pre-selected), observation (text + push-to-talk dictation), recommendation (text + library suggestions), photos strip, reference-clause picker.
4. **Camera** — in-app camera that auto-attaches to the currently-open finding *or* to a temporal-proximity buffer. Burst mode. One-tap retake. Immediate markup (arrows, circles).
5. **Photo triage** — grid of unlinked photos with one-tap link to a finding.
6. **Findings register** — sortable/filterable list of all findings; export to PDF preview.
7. **Sync status** — explicit, honest sync state (uploaded / pending / failed) with retry. Inspector trust collapses the first time a finding silently disappears.

A standards / findings library admin screen is needed for power users but **not for the first inspection**.

### 1.4 Report template structure (deterministic render, server-side)

Render via headless Chromium / WeasyPrint / similar — never the mobile app — for parity and version control. Output PDF/A for archival.

1. **Cover & control page** — rig name, IMO/Reg ID, owner, client, dates, scope, document revision, distribution list, confidentiality notice.
2. **Executive summary** (≤ 2 pages prose + 1-page dashboard) — scope sentence anchored to standards, headline rating, system-level RAG bar, top critical findings (5–10 bullets), explicit **go / no-go / go-with-conditions** statement.
3. **Methodology** — standards referenced, checklists used, NDT methods, sampling rationale, exclusions.
4. **Findings register** — sortable table: ID, system, component, severity, observation summary, recommendation, photo refs, API/IADC/OSHA clause, status, target close-out.
5. **Detailed findings**, one per page or per spread — full observation, recommendation, 1–4 photos, clause citation, status, verified-by.
6. **Photo plates appendix** — continuous plate numbering, captions with location (GA grid ref) and finding ID cross-reference, wide→medium→close grouping.
7. **Component checklist completion** — what was inspected, what was not, why.
8. **Appendix** — standards referenced, abbreviations, inspector credentials with expiry, instrument calibration register, sign-off page.

### 1.5 Deliberate exclusions from v1

- Multi-tenant teams / role-based permissions (one user, one tenant for now).
- Real-time collaboration / live multi-inspector sync.
- AI defect detection in photos (corrosion, cracks, leaks) — see §10.
- Voice "magic notebook" continuous transcription (only push-to-talk into structured fields).
- BOP-specific NDT modules / NORSOK well-barrier schematic editor.
- Analytics dashboards / fleet benchmarking.
- Integrations with SAP PM, IBM Maximo, ServiceNow, Power BI.
- Customer portal for clients to view reports.
- ISO 27001 / SOC 2 (gate later, when you sell to firms).
- White-label / theming.
- App-store distribution (start with TestFlight + direct APK; you have one user).
- Multi-language. Start English-only.

### 1.6 Critical non-functional requirements

- **Offline-first.** Every screen works with no signal. Sync is a background concern that must never block the inspector. Last-write-wins per field with a visible audit trail; never silently overwrite a completed inspection.
- **Photo durability.** Photos written to local disk *before* the UI confirms the finding is saved. Upload queue survives app kill, OS update, dead battery. Assume the inspector will drop the tablet at least once per inspection.
- **PDF parity with current Word output.** Non-negotiable adoption gate. If the first generated PDF looks worse than what the inspector already produces, the project is dead. Build a reference rendering from the founder's own back-catalogue and match it pixel-for-pixel before showing it to anyone.
- **Fast launch.** Cold-start under 2 seconds; new-finding flow under 5 seconds from app icon tap to camera live.
- **Hand-holdable UX.** Touch targets ≥ 11 mm (above the gloved-finger threshold), high-contrast in sunlight, no modal dialogs that block work, all primary actions in the bottom 70% of the screen (one-thumb reachable).
- **Multi-platform.** Android (i.safe MOBILE, ecom Tab-Ex) and Windows (Aegex, Getac F110-EX/ZX10-Ex) intrinsically-safe tablets. iOS is welcome but cannot be the only target — IS iPhones do not exist.
- **Data export.** "Give me everything as a zip of PDF + JSON + photos." Non-negotiable for trust. Buyers in this industry have been burned by SaaS vendors who go bust.
- **Time-zone honesty.** Inspections cross time zones; rigs are on UTC. Store UTC, render local.

### 1.7 Shortest path to operational adoption — first three inspections

- **Inspection #1 (founder's own, paper safety net).** Founder uses v1 alongside paper as backup. Goal: prove the data model survives a real rig and the PDF renders.
- **Inspection #2 (founder, no paper).** Drop the safety net. Goal: prove the workflow stands alone. Founder personally fixes anything broken within 48 hours.
- **Inspection #3 (a friendly second inspector — ex-colleague, peer in a small firm).** Goal: prove the tool works for someone whose mental model is not the founder's. This is where the first real product feedback arrives.

By the end of three real inspections you have three CEO-delivered reports, three testimonials, and a credible story to start selling to peer inspectors.

### 1.8 Architecture sketch (framework-neutral)

```
┌─ Mobile app ─────────────────────────┐         ┌─ Server ──────────────────────┐
│ Local-first datastore (e.g. SQLite + │         │ API (REST or RPC)             │
│   per-row sync metadata)             │ ─sync─▶ │ Postgres                      │
│ Photo capture + local file store     │         │ Object storage (S3-compatible)│
│ Sync engine (push/pull, conflict     │         │ Sync engine (LWW + tombstones │
│   resolution, retry, durable queue)  │         │   or CRDT, whichever suffices)│
│ Inspector-only auth (device cert     │         │ PDF render service            │
│   + passphrase)                      │         │   (headless browser,          │
│ Outbound: nothing required to        │         │    deterministic template)    │
│   function; sync when network        │         │ LLM call layer (optional, for │
│   present                            │         │   narrative + exec summary)   │
└──────────────────────────────────────┘         │ Backup / export (zip of all)  │
                                                 └───────────────────────────────┘
```

Design rules:
- Local-first is the architecture, not a feature.
- Server can fall over for 48 hours without the inspector noticing on the rig.
- PDF rendering happens server-side so template updates don't require app updates.
- LLM calls are server-side, batched, and always against structured input.
- Photo CDN is the single largest cost line — budget for it.

### 1.9 90-day plan

**Weeks 1–4 — Foundation.**
- Data model finalised; component templates for one rig type (land rig) hand-built.
- Mobile app: dashboard, component browser, new-finding flow, camera, photo triage, sync status.
- Server: API, Postgres, object storage, basic auth, deterministic PDF render of a hard-coded test inspection.
- Deliverable end of week 4: founder can complete a mock inspection on a desktop simulator and produce a PDF.

**Weeks 5–8 — First real inspection.**
- Polish offline durability; test by yanking network mid-finding repeatedly.
- Reference-report parity: render founder's last real Word report from manually entered data; iterate template until visually equal.
- Build standards-clause library: API RP 4G + IADC checklist mapped to component codes.
- Add findings/recommendations library (text snippets, frequency-ranked).
- Deliverable end of week 8: founder completes Inspection #1 (with paper safety net) and ships the PDF to a real client.

**Weeks 9–12 — Hardening and the second/third inspections.**
- Inspection #2 (no paper). Fix everything that breaks within 48 hours.
- Ship structured-field push-to-talk dictation (Whisper or Deepgram).
- Ship LLM narrative drafting + exec summary (review-gated, visible "drafted by AI, reviewed by [inspector name]" attribution).
- OCR of nameplates feature (cloud OCR, populate component records).
- Onboard inspector #2; ship Inspection #3 with them.
- Deliverable end of week 12: three CEO-delivered reports from the platform, two paying users (founder + inspector #2), a coherent demo, and the case-study material to start selling to a small inspection firm in month 4.

---

## 2. Industry Workflow & Inspection Lifecycle

### 2.1 Who hires inspectors

- **Operators (E&P companies / IOCs / NOCs)** are the most common buyer. They hire third parties before bringing a rig onto contract ("pre-spud" / acceptance surveys) and periodically thereafter. The convention is that "the rig inspection survey must be carried out by a recognised competent third-party in conjunction with the Well Operations and HSE functions" ([Rig Acceptance Standards — DrillingForGas](https://drillingforgas.com/drilling/standards/rig-acceptance-standards/)).
- **Drilling contractors** (Transocean, Valaris, H&P, Nabors, Precision, Patterson-UTI) hire inspectors for self-assessment and pre-tender readiness.
- **Regulators.** In US federal waters, [BSEE](https://www.bsee.gov/what-we-do/offshore-regulatory-programs/offshore-safety-improvement/inspection-policy-branch) performs annual scheduled and unscheduled inspections plus **monthly drilling rig inspections**. Onshore, OSHA uses an [IADC-aligned framework](https://iadc.org/wp-content/uploads/2014/04/OIL-Gas-rig-audit-OSHA-IADC-09-2013.pdf).
- **Classification societies** (ABS, DNV, Lloyd's Register, Bureau Veritas) — ABS issues a [CDS notation](https://info.keystoneenergytools.com/blog/the-american-bureau-of-shipping-abs-and-drilling-systems); DNV publishes [DNV-OS-E101 / DNVGL-OS-E101](https://rules.dnv.com/docs/pdf/dnvpm/codes/docs/2009-10/Os-E101.pdf).
- **Banks, lessors, insurers, prospective buyers** for pre-purchase / pre-lease condition surveys. Lloyd's Register markets this explicitly to "demonstrate asset condition to potential financiers or investors" ([LR Critical Equipment Survey](https://www.lr.org/en/latest-news/critical-equipment-survey/)).
- **Rig builders / OEMs** for shipyard witness at new build and FAT.

### 2.2 The seven-phase lifecycle

Best documented in the [OMV / ModuSpec joint rig-auditing case study](https://drillingcontractor.org/7-ways-to-set-up-a-successful-joint-rig-auditing-task-force-lessons-learned-from-omv-case-study-6963):

1. **Pre-inspection planning & scope-of-work agreement.** Operator + inspection firm agree on system boundary, reference standards (API RP 4G, RP 8B, RP 54, API Std 53, manufacturer specs), required NDT extent, deliverable format. A representative onshore template is [WSP's Drilling Pre-Mobilization HSE Checklist](https://www.wsp.com/-/media/policy/canada/document/drilling-checklists/wsp-drilling-pre-mobilization-hse-checklist---november-2025.pdf).
2. **Document review (desktop phase).** Inspectors request rig dossiers: equipment registers, prior inspection reports, BOP test charts, hoisting-equipment certs (API 8B / 7K), API 4F design certificates, NDT history, CMMS extracts, crew certifications, SMS / HSE Case documents.
3. **Mobilisation.** Onshore drive-in, offshore helicopter with BOSIET/HUET. "Access is mainly done through helicopters, requiring highly trained personnel" ([Vidya — Challenges of Offshore Inspection](https://vidyatec.com/blog/the-challenges-of-offshore-oil-and-gas-inspection/)).
4. **On-site execution.** Kick-off with OIM / Rig Manager, then discipline walkdowns (drilling/mechanical, electrical, well control, safety, structural, marine). Function-tests, pressure tests, load tests, insulation-resistance checks.
5. **Exit / close-out meeting.** Preliminary findings presented to rig management on the last day. Critical (red) items discussed face-to-face so the contractor can begin remediation immediately and dispute findings before the report is finalised.
6. **Post-inspection reporting.** Draft report written off-site, typically 1–3 weeks. The defect register is the workhorse output.
7. **Follow-up / close-out.** Contractor remediates; submits evidence (photos, NDT re-test, work order numbers). Inspector verifies critical items and issues the final report. "Rig acceptance with outstanding 'critical' items will not be allowed unless written dispensation has been given" ([DrillingForGas](https://drillingforgas.com/drilling/standards/rig-acceptance-standards/)).

### 2.3 Typical durations

- Daily walk-around (rig crew): minutes per shift.
- Weekly / monthly self-inspection: hours.
- Quick walkdown / pre-spud verification by third party: 1–2 days, single inspector.
- **Comprehensive rig audit / condition survey: 5–14 days on-site, team of 4–12 surveyors.** [IFP Training's Rig Inspection (Audit)](https://www.ifptraining.com/report/pdf/951) course is itself a 5-day program reflecting the structure of a real audit.
- 5-year Cat IV / Cat IV-equivalent recertification with full NDT: weeks; mast "shall be disassembled and cleaned to the extent necessary to conduct NDT of all defined critical areas" ([Global1 IRM](https://global1irm.com/inspections/api-4g-cat-iii-iv-inspection-mast-derrick-substructure/)).
- Drone-supported inspection can collapse rope-access weeks: TotalEnergies "reduces inspection costs by 40% using the Elios 3 UT drone" and rope-access inspections "can take up to eight weeks and involve shutting down production – downtime costs $7 million per day" ([Flyability](https://www.flyability.com/oil-and-gas-drones), [Struction Solutions](https://structionsolutions.com/blog/how-drones-reduce-downtime-on-offshore-oil-rigs/)).

### 2.4 Operational bottlenecks

- **Weather windows.** "Traditional inspection methods require multi-day weather windows… a single storm system can delay scheduled inspections by 1–2 weeks" ([Struction Solutions](https://structionsolutions.com/blog/how-drones-reduce-downtime-on-offshore-oil-rigs/)).
- **Permits & SIMOPS conflicts.** Hot-work, work-at-height and Permit-to-Work systems force scheduling around weather and operations.
- **Well operations interference.** Tripping pipe, casing runs, cementing and BOP tests block the rig floor, derrick, BOP stack and pump room respectively.
- **Crew change cycles.** Offshore 14/14 or 21/21 rotations mean key knowledge holders may be off-rotation; helicopter availability constrains inspector arrival/departure.
- **Equipment isolation / LOTO.** Electrical and mechanical inspections requiring LOTO usually wait for non-drilling periods.

### 2.5 The deliverable stack

1. Executive summary — overall condition statement, go/no-go.
2. Defect / observation register — line-item findings with severity, system, location, photo refs, recommendation, target close-out, responsibility ([Inspectly360](https://www.inspectly360.com/checklists/energy-utilities/drilling-rig-safety-inspection/), [SafetyCulture daily rig inspection](https://safetyculture.com/library/mining/daily-rig-inspection-ormazt6uwvbtlhcq/)).
3. Photo log — every finding has ≥ 1 photo, often annotated.
4. Discipline narrative sections — drilling/mechanical, electrical, well control, structural, HSE, marine.
5. NDT reports with readings.
6. Certificate of conformance / compliance.
7. Condition score / rating. [RIG MANUFACTURING](https://www.rigmanufacturing.com/drilling-rig-inspection-services/) describes a 1–5 grade across twenty major systems combined to an overall grade.
8. Action plan / punch list.

### 2.6 Downstream stakeholders

| Reader | What they want |
|---|---|
| Rig Manager / OIM | Owns close-out of findings |
| Drilling Superintendent (operator) | Accept / reject / dispensations |
| Maintenance Supervisor / Reliability Eng | Converts findings into CMMS work orders ([OxMaint CMMS for O&G](https://www.oxmaint.com/blog/post/blog-post-cmms-oil-gas-upstream-downstream-maintenance)) |
| HSE Manager | Safety findings; HSE Case updates |
| Well Control / Well Engineering | BOP and well-control equipment findings |
| Procurement | Quotes long-lead items |
| CEO / Asset Mgr / Buyer / Insurer | Reads the executive summary and condition score |

### 2.7 How findings drive decisions

| Tier | Trigger | Examples |
|---|---|---|
| **Shutdown / no-start (Critical / Red / Cat A)** | Equipment unsafe; well operations cease or do not start | BOP ram-seal failure; drawworks brake out of tolerance; mast weld crack on critical joint |
| **Repair before drill / pre-spud (Major / Amber / Cat B)** | Repairable in days; ops may not commence until closed | Top-drive swivel weep; sheave-groove wear near API 8B reject; accumulator pre-charge low |
| **Monitor / next opportunity (Minor / Yellow / Cat C)** | Add to backlog; track for trend | Surface corrosion; missing cable-tray cover (non-hazardous); loose grating bolts |
| **Observation / housekeeping (Green)** | Best-practice opportunity | Missing labels; faded paint; ergonomics |

> **Implications for software design — Workflow**
> - Model an **inspection-as-project lifecycle**: scoping → desktop review → on-site execution → exit meeting → draft → close-out, with state per finding.
> - Findings are first-class objects with system, sub-system, location, severity, photos, recommendation, owner, due date, evidence-of-closure, signatures.
> - Reports must export a deliverable bundle (exec summary, register, photo log, narratives, NDT data, certificate, action plan).
> - Configurable condition-score rollup (per system → overall) — not hardcoded.
> - CMMS integration via work-order numbers as evidence of closure.
> - Offline-first because connectivity on rigs is unreliable.
> - Multiple stakeholders consume different views of the same data.

---

## 3. Types of Inspections (Taxonomy)

| Type | Performed by | Cert | Tools | Duration | Output |
|---|---|---|---|---|---|
| **Visual / walk-around** | Crew, third-party inspector, OEM, regulator | API RP 54 awareness; IADC RigPass ([IADC](https://iadc.org/accreditation/rigpass/)) | Torch, camera, checklist | Hours–day | Checklist with SAT/UNS/CDI/N-A per [OSHA-IADC](https://iadc.org/wp-content/uploads/2014/04/OIL-Gas-rig-audit-OSHA-IADC-09-2013.pdf) |
| **Operational / functional** | Experienced field inspector | Industry experience | Stopwatch, multimeter, gauges, IR | 1–3 days | Narrative + function-test sheets |
| **Mechanical (API 7K / RP 8B)** | Mechanical inspector + NDT for Cat III/IV | API 7K / 8B competency | Calipers, micrometers, torque tools, borescope | Days | Dimensional + condition report |
| **Electrical** | Licensed electrical inspector; IECEx/ATEX competency offshore | Hazardous-area cert | Megger, IR camera, multimeter, hi-pot | 2–5 days | IR table per motor, Ex inspection register (IEC 60079-17) |
| **Structural (API 4F / RP 4G)** | Structural inspectors | API RP 4G competency | MPI yoke, UT gauge, total station, torque multipliers | Days–weeks | NDT findings, dimensional check, photos |
| **NDT** | ASNT Level I / II / III ([ASNT](https://certification.asnt.org/certification)) | SNT-TC-1A or CP-189 ([details](https://www.trainingndt.com/easy-as-123-ndt-certification-levels-explained/)) | UT, MPI, PT, RT, ET, hardness | Hours–days per method | Method-specific report (UT C-scan, MPI indication map, hardness traverse) |
| **Safety / HSE** | HSE professional; NEBOSH + RigPass | API RP 54 ([RP 54 PDF](https://www.api.org/-/media/files/publications/rp-54_e4.pdf)) | Detectors, IR camera | Days | HSE narrative |
| **Regulatory compliance** | Regulator (BSEE, OSHA, HSE, Havtil) | Government inspector | Per regulator | Per program | Compliance certificate / notice |
| **Pre-purchase / pre-sale** | Survey firm (LR, ABS, DNV, BV, ABL) | Class-society or equivalent | Full audit toolkit + valuation methodology | 1–3 weeks | Valuation-grade report + capex backlog |
| **Recertification (5-year)** | Class society + NDT | Class-society surveyor | Full toolkit + extensive NDT | Weeks–months | Class certificate / API RP 4G Cat IV report |
| **Acceptance (new build)** | Witness inspectors | Survey firm | Witnessing FAT, SIT, sea trials | Months | Acceptance report per FAT/SIT milestone ([ModuSpec reactivation](https://www.moduspec.com/services/rig-reactivation)) |
| **Rig-up / rig-down (land)** | Crew + occasional third party | Per RP 4G Cat I/II | Hand tools | Hours | Move report; triggered every well or after stuck pipe / lightning / high winds |
| **Dropped-object (DROPS)** | Rope-access dropped-object specialist | DROPS RP | Tether kits, camera | Days | Aloft register + DROPS calculator outputs ([Eiwaa](https://eiwaagroup.com/service/dropped-object-survey/), [DROPS Calculator](https://www.dropsonline.org/drops-guidance-and-resources/drops-calculator/)) |

**Where the user's work fits.** The user's "between operational and detailed mechanical/electrical" description most closely matches the **rig condition / acceptance survey** — the same product Lloyd's Register, ModuSpec, ABL Group, OCS Group, Aqualis, Athens Group, OES Group, Applus+, Vysus Group and Add Energy market. These engagements span functional checks (Cat II/III hoisting, Cat III structural, BOP operational verification, electrical spot-checks) without going as deep as a Cat IV strip-and-NDT, and produce a defect register + photo + narrative + condition score deliverable in the 100–300 page range ([Lloyd's Register Critical Equipment Survey](https://www.lr.org/en/latest-news/critical-equipment-survey/), [Aqualis](https://aqualisoffshore.com/services/rig-inspection-services/), [Rig QA International](https://rigqa.com/services/rig-inspections/), [ABL Group](https://abl-group.com/abl/loss-prevention/marine-surveys-inspections-and-audits/rig-inspections-and-assurance/)).

> **Implications for software design — Taxonomy**
> - Inspection templates must be scoped by type **and** by API/IADC reference; one tool needs to express daily walk-arounds, monthly Cat II, annual Cat III, 5-year Cat IV.
> - Inspector certification is data: store ASNT Level II/III, NACE/AMPP CIP I/II, API RP 4G competency, IADC RigPass, BOSIET/HUET with expiry. Operators reject reports signed by uncertified inspectors.
> - Different inspection types produce different artefacts (NDT needs calibration + readings; coatings needs DFT + chloride; DROPS needs aloft register + tether status).
> - Provide a library of canonical templates (API 4G Cat III, API 8B Cat III, IADC checklist, DROPS, NACE coating, BSEE-style) that inspectors compose into project scopes.
> - The user's likely product is a rig **condition / acceptance survey workflow**, not a single-purpose checklist.

---

## 4. Rig Components & Severity Logic

### 4.1 Major systems and where inspectors look

| System | Look at | Common failure modes |
|---|---|---|
| **Mast / derrick & substructure (API 4F / RP 4G)** | Load-bearing members, welds at critical joints, mast pins/bushings, gin pole, A-frame, ladder/cage, crown block beams, sub-base, pad eyes, raising lines, telescoping lock pins | Weld cracking at high-stress joints, mast pin wear/ovalisation, corrosion at "wet areas," missing/loose bolts, deformation from snag loads ([Sovonex](https://www.sovonex.com/drilling-equipment/api-land-drilling-rigs/electric-mechanical-drilling-rigs/), [RP 4G PDF](https://www.ipgmservicios.com/wp-content/uploads/2024/03/API-RP-4G-2019-Operation-Inspection-Maintenance-and-Repair-of-Drilling-and-Well-Servicing-Subestructures.pdf)) |
| **Drawworks** | Brake bands/linings, brake calipers/disc, brake cooling, drum grooves, clutch, chain/sprockets, gearbox, crown-saver, fast-line anchor, drillers' controls | "Drawworks failure accounts for 15% of safety-critical drilling equipment problems" — worn brake bands, brake temperature > 400°F, clutch release issues ([Metadrill](https://metadrill.org/blogs/9-most-common-drill-rig-failures-and-the-parts-that-fix-them), [IADC Drawworks](https://iadc.org/safety-meeting-topics/drawworks-and-accessories-operation-and-maintenance-safety/)) |
| **Top drive / kelly / rotary table** | Swivel main bearing, washpipe, gooseneck, motor brushes/VFD logs, gearbox, IBOP, slip-ring, torque calibration, link tilt | Swivel leak (mud past washpipe), bearing noise/vibration, gear misalignment, hydraulic leaks, VFD faults, heat indicating overload or lube failure ([Dynamox](https://dynamox.net/en/blog/drilling-rig-failure-modes-and-solution-for-monitoring)) |
| **Hoisting (API 8C / RP 8B)** | Crown block sheaves & bearings, travelling block, hook, links, elevators, slips, tongs, drill line, deadline anchor, weight indicator calibration | Sheave-groove wear, bearing fatigue, hook/elevator latch wear, drill line broken wires/birdcaging, slip die wear ([IADC Drilling Line Care](https://iadc.org/safety-meeting-topics/drilling-line-care-inspection-and-replacement/)) |
| **Mud pumps & mud system (API 7K)** | Fluid end (modules, valves & seats, pistons, liners), power end, pulsation dampener, suction strainer, discharge piping, pop-off valve set point, shakers, desander/desilter, centrifuge, mud pits/agitators, MGS, trip tank, flow line | Valve seat washouts, liner scoring, expendables life, dampener charge loss, shaker screen tears, MGS liquid-seal loss ([Sinomechanical](https://www.sinomechanical.com/news/Common-faults-of-mud-pumps.html)) |
| **BOP stack & well control (API Std 53; 16A/C/D)** | Annular preventer, ram preventers (pipe / blind-shear / casing), choke & kill outlets, choke manifold, hydraulic control system (accumulators, pumps, valves), driller's & remote panels, hydraulic SPM valves, accumulator pre-charge, function/pressure test charts | Ram seal extrusion/tearing/degradation, ram-face vs bonnet-seal leaks, misalignment, accumulator pre-charge loss, hydraulic SPM leakage, control panel solenoid failure ([BSEE TAP 693ab BOP FMECA](https://www.bsee.gov/sites/bsee.gov/files/tap-technical-assessment-program//693ab.pdf), [Sinomechanical Ram BOP](https://www.sinomechanical.com/news/Reasons-and-Solutions-for-Seal-Failure-of-Double-Ram-Blowout-Preventers.html)) |
| **Power generation, SCR/VFD, electrical** | Gensets (cylinder pressures, lube oil analysis, exhaust, AVR), main switchboard, SCR or VFD cabinets, MCCs, transformers, batteries, UPS, grounding, Ex equipment (IEC 60079-17), cable trays | Insulation breakdown, VFD harmonics/cooling failure, switchgear arc-flash risk, Ex enclosure gland/seal issues, neutral-earthing problems ([Zeefax](https://www.zeefax.com/more-about-scr-systems-simulation-and-training/)) |
| **Hydraulics / pneumatics / instrumentation** | HPU pressures, accumulator pre-charge, hose condition, air compressors/dryers, air receivers, instrument air quality, PLC fault logs, ESD interlock function tests | Hose chafe, accumulator leaks, instrument air contamination |
| **Cranes & lifting gear** | Structural condition, hoist wire rope (UK LOLER 6-monthly thorough exam), brake test, limit switches, slings, shackles, lifting register | Rope wear, brake glaze, missing inspection tags |
| **Fire / safety / lifesaving** | Firewater pumps, deluge test, gas/fire detector loop tests, lifeboats (annual fall + 5-year load), TEMPSC, life rafts, EEBDs, helideck (HCA) | Pump start failures, detector drift, lifeboat release-gear faults |
| **Living quarters / marine (offshore)** | Per MODU Code + class IIP | Hull corrosion, watertight integrity, accommodation safety |

### 4.2 Severity classification frameworks actually used

No single universal scheme. The real-world set:

- **OSHA-IADC condition codes** (US onshore walkdowns): **SAT** (satisfactory), **UNS** (unsatisfactory), **CDI** (corrected during inspection / 24 h), **DSB** (applies to Drilling / Servicing / Both) ([OSHA-IADC PDF](https://iadc.org/wp-content/uploads/2014/04/OIL-Gas-rig-audit-OSHA-IADC-09-2013.pdf)).
- **Pass / Fail / N-A** at the checkpoint level.
- **Critical / Major / Minor / Observation** at the finding level. "Critical deficiencies include inoperative emergency stops, hydraulic ram failures, damaged delivery line hoses or connections, track damage, leaking drivers, missing or damaged warning signs, or faulty levers and controls requiring immediate corrective measures" ([Inspectly360](https://www.inspectly360.com/checklists/mining/drill-rig-and-ancillary-equipment-inspection/)).
- **Acceptance "critical" flag.** "Items identified as having significant impact on mechanical integrity or safety of the rig will be classified as 'critical'. Rig acceptance with outstanding 'critical' items will not be allowed unless written dispensation has been given" ([DrillingForGas](https://drillingforgas.com/drilling/standards/rig-acceptance-standards/)).
- **RAG (Red / Amber / Green)** for dashboards and rollups, not contractual rubric ([Mastt](https://www.mastt.com/blogs/project-rag-status-dashboard)).
- **5×5 risk matrix (probability × consequence).** Default risk-assessment language; cells scored 1–25 with bands 1-4 Low / 5-9 Moderate / 10-16 High / 17-25 Critical ([SafetyCulture 5x5](https://safetyculture.com/topics/risk-assessment/5x5-risk-matrix)).
- **Condition rating 1–5** for system/asset rollup ([RIG MANUFACTURING](https://www.rigmanufacturing.com/drilling-rig-inspection-services/)).
- **API RP 8B / RP 4G Category I–IV** — frequency-and-depth labels, not severity. Often appear in recommendations ("recommend Cat III inspection of drawworks").
- **DROPS severity** for at-height items: Slight / Minor / Major / Fatality per the [DROPS Calculator](https://www.dropsonline.org/drops-guidance-and-resources/drops-calculator/).
- The **A / B / C** category sometimes mentioned is plausible (used in some operator-internal templates) but is not industry-canonical. Treat as a customer-specific variant.

### 4.3 How findings feed CMMS / maintenance backlog

Findings flow into the rig's CMMS as corrective work orders: "Corrective work orders address problems that have already been identified but haven't yet caused complete failure and are created in response to inspection findings…" ([OxMaint](https://www.oxmaint.com/blog/post/blog-post-cmms-oil-gas-upstream-downstream-maintenance)). Common drilling-specific CMMS platforms include Saxon STAR Suite ([STAR Suite](https://saxonsoftware.com/star-suite/)); IBM Maximo and SAP PM are the enterprise defaults.

Closeout flow: inspection finding → severity → recommended action → CMMS work order → execution → evidence (photo, NDT re-test, signature) → inspector/operator verifies → finding closed. This is why inspectors care so much about asset/tag IDs and work-order numbers — they have to map findings to the maintenance system already on the rig.

> **Implications for software design — Components & Severity**
> - Asset hierarchy: Rig → System → Sub-system → Component → Inspection point. Findings attach at the right level.
> - Two independent classifications per finding: (1) severity (Critical/Major/Minor/Observation, plus RAG and 5×5 supported) and (2) recommended action (Shutdown / Repair-before-drill / Monitor / Observe).
> - Support multiple severity rubrics simultaneously — operators differ; some require both.
> - Distinguish inspection category (frequency/depth, e.g. RP 4G Cat III) from finding severity.
> - Pre-populated component library for common systems with canonical inspection points and known failure modes — inspectors pick from a catalogue rather than free-type.
> - Severity → default recommended action mapping, with inspector override.
> - CMMS interop fields: `asset_tag_ID`, `work_order_number`, `cmms_system`. Export to CSV/Excel and to common CMMS APIs (Maximo, SAP PM, Amos, STAR).
> - Standards references are first-class metadata on every checklist item.
> - NDT data is structured (UT thickness grid, MPI indication list, hardness traverse), not buried in narrative.
> - Photos: ≥ 1 per finding; annotation (arrows, circles) and EXIF location/time stamps.

---

## 5. Standards & Compliance Landscape

### 5.1 API standards — the dominant North-American framework

**Structural systems**
- **API Spec 4F — Drilling and Well Servicing Structures.** Design, manufacture, ratings, PSL-1/PSL-2 for steel derricks, portable masts, crown blocks and substructures. 5th edition aligned to AISC ASD ([API Spec 4F page](https://www.api.org/products-and-services/standards/important-standards-announcements/spec4f), [Oilfield Technology summary](https://www.oilfieldtechnology.com/special-reports/10072020/api-updates-4f-manufacturing-standard-for-onshore-and-offshore-steel-structures/)).
- **API RP 4G — Operation, Inspection, Maintenance and Repair.** Four categories:
  - **Cat I** — daily observation by rig crew.
  - **Cat II** — by personnel with mast/derrick experience.
  - **Cat III** — thorough visual of all load-bearing components; **every ~730 operating days (2 years)**.
  - **Cat IV** — Cat III + NDE of all defined critical areas with disassembly/cleaning; **every ~3650 operating days (10 years)**. Frequency must be increased in corrosive/H₂S environments. ([RP 4G PDF](https://www.ipgmservicios.com/wp-content/uploads/2024/03/API-RP-4G-2019-Operation-Inspection-Maintenance-and-Repair-of-Drilling-and-Well-Servicing-Subestructures.pdf), [Global1 IRM](https://global1irm.com/inspections/api-4g-cat-iii-iv-inspection-mast-derrick-substructure/), [Lee C. Moore](https://www.lcm-wci.com/field-services/api-rp-4g-category-iii-iv-inspections/))

**Hoisting equipment**
- **API Spec 8C — Drilling and Production Hoisting Equipment (PSL-1, PSL-2).** Crown blocks, travelling blocks, hooks, links, elevators, swivels, top drives, spiders.
- **API RP 8B — Inspections, Maintenance, Repair and Remanufacture of Hoisting Equipment.** Four categories with mandated frequencies on 1/7/30/90/180/365/730/1825-day intervals depending on category × equipment. Cat III ≈ semi-annual; Cat IV ≈ 5-year for hooks/blocks ([RP 8B PDF copy](https://www.ipgmservicios.com/wp-content/uploads/2024/03/API-RP-8B-2014-Add.1-March-2019-Errata-1-August-2019.pdf), [API 8C purchasing guidelines](https://www.api.org/~/media/files/certification/monogram-apiqr/program-updates/8c%205th%20edition%20purch%20guidelines%20r1%2020131024.pdf)).

**Rotary / drilling equipment**
- **API Spec 7K** — design/manufacture for rotary tables, bushings, mud pumps, drawworks, manual/power tongs, slips, BOP handling, mud hoses, pressure-relief devices ([Spec 7K PDF](https://www.api.org/~/media/files/publications/whats%20new/7k_e6%20pa.pdf)).
- **API RP 7L** — corresponding in-service inspection RP.

**Well control / BOP**
- **API Spec 16A — Drill-Through Equipment** ([16A PDF](https://www.saigaogroup.com/uploads/file/api-16a-standard.pdf)).
- **API Spec 16C — Choke and Kill Systems.**
- **API Spec 16D — Control Systems for Drilling Well Control Equipment.**
- **API Std 53 — Blowout Prevention Equipment Systems for Drilling Wells.** 5th edition shifted BOP requirements from working pressure to **maximum anticipated surface pressure**. Mandatory under [30 CFR 250 Subpart G](https://www.ecfr.gov/current/title-30/chapter-II/subchapter-B/part-250/subpart-G/subject-group-ECFR045ffcd99ad03d3) for US OCS ([Piping Tech](https://pipingtechs.com/api-standard-53-blowout-prevention-equipment-systems-for-drilling-wells/)).
- **API Std 16AR** — repair, remanufacture, recertification of used BOPs.

**Quality and HSE**
- **API Spec Q2** — QMS for E&P service supply organizations ([Q2 page](https://www.api.org/products-and-services/standards/important-standards-announcements/specq2)).
- **API RP 54** — Occupational Safety and Health for Oil and Gas Well Drilling and Servicing Operations, 4th ed ([RP 54 PDF](https://www.api.org/-/media/files/publications/rp-54_e4.pdf), [JPT release notes](https://jpt.spe.org/api-releases-updated-recommended-practice-safety-and-health-well-drilling-and-servicing)).

### 5.2 IADC

- **IADC Drilling Rig Safety Inspection Checklist** — used for routine walkdowns against OSHA/API/ANSI. The [OSHA-IADC checklist PDF (Sep 2013)](https://iadc.org/wp-content/uploads/2014/04/OIL-Gas-rig-audit-OSHA-IADC-09-2013.pdf) is the de facto reference for North-American land rig walkdowns, with SAT/UNS/CDI/DSB condition codes.
- **IADC HSE Case Guidelines** — separate volumes for Land Drilling Units and MODUs; defines HSE case structure incl. Appendix 4 risk-assessment guidance ([HSE Case for Land](https://store.iadc.org/product/iadc-hse-case-guidelines-for-land-drilling-units), [HSE Case for MODUs](https://store.iadc.org/product/iadc-hse-case-guidelines-for-mobile-offshore-drilling-units)).
- **IADC RigPass®** — 8-hour safety-orientation accreditation; Land/Offshore endorsements; SEMS II elements ([RigPass](https://iadc.org/accreditation/rigpass/)).
- **DROPS** — industry initiative and parallel IADC RP. **DROPS Calculator** classifies a potential dropped-object event by mass × fall-height into Slight / Minor / Major / Fatality bands ([DROPS Calculator](https://www.dropsonline.org/drops-guidance-and-resources/drops-calculator/), [Metric Calculator PDF](https://www.dropsonline.org/assets/documents/DROPS-Metric-Calculator-A4-June-2021.pdf)).

### 5.3 ASNT — NDT personnel certification

| Level | Authority |
|---|---|
| **Level I** | Operator/technician; performs calibrations and tests under written instructions; **cannot** sign acceptance/rejection |
| **Level II** | Sets up tests, interprets results, accepts/rejects per code |
| **Level III** | Develops/approves procedures, qualifies Level I/II personnel, signs off technical content (≥ 4 years as Level II per method) |

Both **ASNT SNT-TC-1A** (employer-based recommended practice) and **ANSI/ASNT CP-189** (national standard, more prescriptive) define these levels ([ASNT standards](https://www.asnt.org/standards-publications/standards), [CP-189 listing](https://source.asnt.org/1pekngo/), [Training NDT](https://www.trainingndt.com/easy-as-123-ndt-certification-levels-explained/)).

**This matters for software:** the sign-off authority on every NDT finding must be a Level II or above for that specific method (a UT-II cannot sign MT findings unless also MT-certified). Model person × method × level × expiry.

### 5.4 NDT methods on a rig

| Method | Use | Performed by | Report artefacts |
|---|---|---|---|
| **VT (Visual)** | Daily Cat I; baseline of every Cat III walkdown | Level II VT | Worksheet + photos, condition codes |
| **MT / MPI** | Surface/near-surface cracks on welds, threaded connections, ferromagnetic load-bearing members | Level II MT | Procedure ref, indications log, photos, accept/reject vs ASTM E709 / API DS-1 |
| **PT** | Surface cracks on non-ferromagnetic parts | Level II PT | Indications log + photos |
| **UT / UT-thickness / PAUT** | Internal flaws in welds; thickness mapping / corrosion mapping of pipework, BOP bodies, choke/kill, riser | Level II UT (often + PAUT specialist) | Thickness grid, C-scan image, calibration record, transducer cert |
| **RT** | Critical welds where UT impractical; weld qualification | Level II RT + radiation safety | Films/digital radiographs, shot map, density/IQI |
| **ET** | Heat exchanger tubes, surface inspection of conductive non-ferromagnetics | Level II ET | C-scan / impedance plot |
| **Hardness** | Material verification post-repair, weld HAZ | Inspector | Hardness map + tester cal |

Documentation requirement (all methods): location, method, procedure number, settings, technician name + cert ID + level + method, calibration block IDs, environmental conditions, results, accept/reject decision, standard reference.

### 5.5 OSHA — US workplace law

- **29 CFR 1910 (General Industry)** — default for drilling/servicing. Key subparts: D (walking-working surfaces), H (haz materials), I (PPE), J (general environmental incl. 1910.146 confined spaces, 1910.147 LOTO), L (fire), N (materials handling), O (machinery), Q (welding/cutting), S (electrical), Z (toxic substances incl. 1910.1153 silica).
- **29 CFR 1926 (Construction)** — applies during rig-up/rig-down, well-pad construction, pipeline tie-ins. Key subparts: C, E, L (scaffolds), M (fall protection — 6-ft trigger vs 4-ft in general industry), R (steel erection), S (underground), CC (cranes & derricks), 1926.1153 (silica) ([1926 Subpart CC](https://www.osha.gov/laws-regs/regulations/standardnumber/1926/1926SubpartCC), [Vector Solutions explainer](https://www.vectorsolutions.com/resources/blogs/osha-part-1910-general-industry-part-1926-construction-standards/)).

### 5.6 BSEE — US offshore (Outer Continental Shelf)

**30 CFR Part 250** subparts: A (general), D (drilling), E (completion), F (workover), **G (well operations & equipment — the BOP heart of the Well Control Rule)**, H (production safety systems), Q (decommissioning), S (SEMS).

Key inspection-software facts in **[30 CFR 250.737](https://www.ecfr.gov/current/title-30/chapter-II/subchapter-B/part-250/subpart-G/subject-group-ECFR045ffcd99ad03d3/section-250.737):**
- BOP pressure test **every 14 days** (every **30 days** for blind shear rams).
- Each test holds the required pressure for **5 minutes** (3 minutes acceptable on surface BOPs) recorded on a chart not exceeding 4 hours or digital.
- May request 21-day BOP testing frequency.
- Major detailed inspection of riser/BOP/LMRP/control pods every **5 years**, may be phased; **third-party independent reviewer required**.

Risk-Based Inspection (RBI) Program layers additional scrutiny on higher-risk facilities ([BSEE RBI](https://www.bsee.gov/what-we-do/offshore-regulatory-programs/offshore-safety-improvement/inspection-programs/risk-based-inspection-program)).

### 5.7 UK HSE — North Sea

- **Offshore Installations and Wells (Design and Construction etc) Regulations 1996 (DCR)** — well lifecycle, well examination scheme, blowout-prevention duty.
- **Offshore Installations (Offshore Safety Directive)(Safety Case etc) Regulations 2015 (SCR 2015)** — safety case must be accepted by competent authority (HSE + OPRED) before operating ([HSE L154](https://www.hse.gov.uk/pubns/books/l154.htm), [HSE L84 PDF](https://books.hse.gov.uk/gempdf/l84.pdf), [IADC UK/Norway summary](https://iadc.org/wp-content/uploads/2015/04/Rules-and-Regulations-for-UK-Norwegian-Waters.pdf)).

### 5.8 Norway — NORSOK and Havtil

- **NORSOK D-010 — Well Integrity in Drilling and Well Operations.** Defines functional and performance requirements for well barriers. Mandates **well barrier schematics** (primary/secondary envelopes) — visual artefacts the report has to embed and update ([D-010 release notes PDF](https://www.offshorenorge.no/contentassets/f60cf93f2c9c4c129b0a73303ff8081c/02---norsok-d-010-status-new-revision.pdf), [SPE on barrier schematics](https://jpt.spe.org/using-schematics-managing-well-barriers)).
- **NORSOK D-001 — Drilling Facilities.** Design/manufacture/installation/testing for drilling facilities.
- Petroleum Safety Authority (now **Havtil**) Facilities Regulations reference NORSOK directly.

### 5.9 DNV — classification standards

- **DNV-OS-E101 / DNVGL-OS-E101 — Drilling Plant / Drilling Facilities.** Design and construction of drilling facilities on fixed and floating offshore installations; covers surface and subsea drilling systems down to (but not including) the wellhead ([2009 PDF](https://rules.dnv.com/docs/pdf/dnvpm/codes/docs/2009-10/Os-E101.pdf), [2015 PDF](https://rd-engineering.co/wp-content/uploads/2019/04/DNVGL-OS-E101JULY2015.pdf)).
- **DNV Condition Assessment Programme (CAP)** — graded condition rating across hull, machinery, cargo systems, electrical, with photo evidence and summary ([CAP service page](https://www.dnv.com/services/condition-assessment-programme-cap--17741/)).

### 5.10 ABS — class society surveys for MODUs

- **ABS Rules for Building and Classing MODUs** + **Guide for the Classification of Drilling Systems (CDS).**
- Survey cadence: **Annual** (general external/operating), **Intermediate** (year ~2.5–3 more detailed), **Special Periodical Survey** (within 5 years of build or previous SS; comprehensive; drydock or UWILD). At least two bottom examinations in any 5-year period, interval ≤ 36 months ([ABS MODU rules](https://www.dco.uscg.mil/Portals/9/DCO%20Documents/5p/5ps/Alternate%20Compliance%20Program/absmodu.pdf), [ABS MODU brochure](https://ww2.eagle.org/content/dam/eagle/publications/brochures/MODU2014.pdf), [ABS CDS Guide PDF](https://ww2.eagle.org/content/dam/eagle/rules-and-guides/current/offshore/57_Classification_of_Drilling_Systems_2021/cds-guide-feb21.pdf)).

Lloyd's Register and Bureau Veritas operate analogous schemes under the IACS framework.

### 5.11 Documentation and traceability expectations (common to all standards)

Reports must carry:
1. Inspector identity, qualification level, certification expiry, signature per finding (and per NDT method) — required by SNT-TC-1A / CP-189.
2. Reference to the clause/standard the finding is judged against.
3. Calibration evidence — each measurement instrument's traceability to a national standard, calibration cert number + expiry, calibration block IDs for UT.
4. Equipment identity — manufacturer, serial number, year of manufacture, last recertification date.
5. Operating-hours clock for category-driven intervals (RP 4G's "operating day" = 24 operating hours).
6. Third-party independence — required for the 5-year BOP recertification under BSEE.

> **Implications for software design — Standards**
> - Asset hierarchy with API-aligned categories baked in (mast/derrick for RP 4G; hoisting line items for RP 8B; BOP stack for Std 53/16A/C/D; drilling equipment for 7K/7L).
> - Track operating-hours separately from calendar time so RP 4G Cat III/IV and RP 8B intervals fire correctly.
> - Store inspector qualification per method × level × expiry and prevent a Level I from signing acceptance/rejection findings.
> - Force a reference standard + clause dropdown per finding (RP 4G clause X, RP 8B Table 1 row Y, OSHA 1910.147, 30 CFR 250.737, NORSOK D-010 §15).
> - Maintain calibration register linked to each measurement instrument; reject readings from out-of-cal instruments.
> - Carry multiple severity taxonomies natively and convert between them: OSHA-IADC (SAT/UNS/CDI/DSB), DROPS, Critical/Major/Minor, 5×5.
> - Track recertification cycles: 14-day BOP pressure tests; 30-day blind shear ram tests; 5-year major BOP recert; 5-year ABS Special Survey + UWILD; ~2-year RP 4G Cat III; ~10-year RP 4G Cat IV.
> - Embed and version well-barrier schematics (NORSOK D-010) as first-class diagram objects, not PDF attachments.
> - Generate signed deliverables with PDF/A long-term archival, inspector signature, certificate ID, equipment ID and reference clause on every finding.

---

## 6. Field Realities & UX Constraints

### 6.1 Environments

Land rigs in Permian / Bakken / Middle East / North Africa / Siberian permafrost / equatorial jungle. Offshore: jackup (shallow water, legs to seabed), semisubmersible / drillship (deepwater, floating). Per-rig micro-environments: open weather (rig floor), dark/oily/low overhead (cellar, sub-base), wind at heights (mast), slippery and chemically aggressive (shaker/mud room), moonpool (offshore). Offshore adds saline corrosion and 40+ knot wind. North Sea harsh-environment jackups: ambient below −10 °C in winter ([Westwood](https://www.westwoodenergy.com/news/westwood-insight/westwood-insight-rig-dayrates-have-risen-so-when-are-the-new-rig-orders-coming)).

**Implication:** device must be IP65, screen ≥ 1000 nits ([Panasonic Toughbook](https://connect.na.panasonic.com/blog/toughbook/defining-rugged-toughbook-computers-and-laptops)), operational range −20 to +60 °C ([i.safe IS530.1](https://www.isafe-mobile.com/en/products/products-zone-1/21/is5301)). No tabletop assumptions. Auto-timestamp + auto-geotag every photo and note.

### 6.2 Connectivity

VSAT historically dominant offshore: ~500–700 ms latency, ~1 Mbps shared rig-wide ([Speedify](https://speedify.com/blog/enterprise-internet-bonding-software/can-i-use-starlink-internet-on-an-oil-rig-yes-with-speedify/), [Offshore Magazine](https://www.offshore-mag.com/home/article/16757572/for-oil-and-gas-satcom-users-how-much-bandwidth-is-enough)). Starlink Maritime is changing this — Speedcast/COSL Drilling Europe integration in late 2025 reports 20–60 ms latency, 40–220 Mbps real-world, peaks of 1.6 Gbps ([Speedcast/COSL](https://www.speedcast.com/newsroom/press-releases/2025/speedcast-integrates-new-global-high-throughput-service-from-starlink-as-part-of-cosl-drillings-hybrid-solution/), [Clarus Networks](https://www.clarus-networks.com/2025/07/14/pushing-the-boundaries-of-maritime-connectivity-with-the-starlink-performance-kit/)). But even with generous rig bandwidth, the inspector is inside steel — cellar, mud tanks, BOP area, mast interior — where Wi-Fi doesn't penetrate. Land rigs in remote areas often have one VSAT and one 4G hotspot at the company-man's trailer, dead 200 m away.

**Implication:** offline-first is hard requirement. Launch, render every form, capture photos/voice/signatures/timestamps without network. Sync is background. Conflict resolution defaults to last-write-wins per field with audit trail, never silently overwriting. Incremental, pause/resume sync — 50 MB inspection bundle over 1 Mbps shared VSAT will fail dozens of times.

### 6.3 Hazardous-area zoning

Zone 0 / Class I Div 1 (continuous risk): mud tank interiors, vent masts. Zone 1 / Class I Div 1 (likely in normal operation): enclosed spaces with open mud-system openings, parts of cellar, BOP area, well bay, pump rooms. Zone 2 / Class I Div 2: surrounding area incl. parts of rig floor ([Maritime Executive](https://maritime-executive.com/features/understanding-hazardous-area-classification-on-ships-and-offshore), [46 CFR 111.105](https://www.ecfr.gov/current/title-46/chapter-I/subchapter-J/part-111/subpart-111.105), [Plantfce FPSO guide](https://plantfce.com/blog/posts/fpso-hazardous-area-classification/)). A consumer iPhone/iPad is **not permitted in Zone 1**; many operators forbid them in Zone 2 too.

Intrinsically safe (IS) devices certified for Zone 1:

| Device | OS | Form | Cert | Indicative price |
|---|---|---|---|---|
| i.safe MOBILE IS530.1 | Android 10 | 4.5" smartphone, 13 MP | ATEX/IECEx Zone 1/21 | ~$1,678 ([Able Instruments](https://ableinstruments.com/product/i-safe-mobile-is530-1-intrinsically-safe-smartphone/)) |
| i.safe MOBILE IS540.1 | Android 5G | Larger | ATEX/IECEx Zone 1/21 | ~$2,697 |
| Aegex 100M | Windows 11 IoT | 10.1" tablet, Intel Apollo Lake, 8 GB / 256 GB | Zone 1/21 + Class I Zone 1 | Quote (legacy Aegex 10 listed ~$3,500) ([Aegex datasheet](https://aegex.com/images/uploads/aegex100m/Aegex_100M_DataSheet.pdf)) |
| Pepperl+Fuchs / ecom Tab-Ex 05 DZ1 | Android 5G | 8" tablet | Zone 1/21 & Div 1 | Quote ([ecom-ex](https://www.ecom-ex.com/products/mobile-computing/tablets-android/tab-ex-05-dz1/)) |
| ecom Smart-Ex 03 DZ1 | Android | Smartphone | Zone 1 | Quote |
| Getac F110-EX / ZX10-Ex | Windows | Tablet | ATEX/IECEx | Quote ([Getac](https://www.getac.com/us/certifications/atex-iecex/)) |
| Bartec Pixavi / RugGear | Various | Various | Zone 2 typical | Quote |

Total Zone 1 tablet range: **~$3,800–$8,000**, 4–8 week lead times ([Intrinsically Safe Store 2025](https://intrinsicallysafestore.com/blog/atex-zone-1-tablet-price/)).

**Implication:** must run on Android (i.safe, ecom) **and** Windows (Aegex, Getac). iOS-only is a non-starter. Cannot rely on latest mobile OS APIs — Aegex 100M is Windows 11 IoT; IS Android devices often one to two OS versions behind. Camera quality on IS devices is poor (5–13 MP, fixed focus) — tolerate low-res blurry photos and provide annotation tools to compensate.

### 6.4 PPE and ergonomics

Hard hat, safety glasses, FRC coverall (Nomex), steel-toed boots, hearing protection, impact gloves with nitrile/rubberized palms. Heat strain significant — 81% of PPE-wearing workers report being "slightly warm, warm or hot," 88% report discomfort ([PMC heat strain study](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9877975/)). Touchscreen-compatible gloves exist but capacitive performance is unreliable with nitrile, moisture, and EMI from VFDs ([PIP Global](https://us.pipglobal.com/en/about-us/news-and-events/?nID=112)). Rig floor noise routinely 95–110 dB (NIOSH limit 85 dB) — workers cannot remove ear protection to use voice input, and ambient noise alone defeats most consumer ASR ([The Driller](https://www.thedriller.com/articles/86218-hearing-protection-and-air-rotary-drilling-part-1), [CDC/NIOSH PDF](https://stacks.cdc.gov/view/cdc/9719/cdc_9719_DS1.pdf), [PMC offshore inspector hearing study](https://pmc.ncbi.nlm.nih.gov/articles/PMC5333758/)).

**Implication:** touch targets ≥ 11–12 mm (above 9 mm gloved-finger threshold) with generous padding. No hover, long-press, pinch-zoom, or two-finger gestures. Voice input is a "back-of-truck" feature for after descending — not for primary capture on deck. Tune contrast and font weight for safety glasses at outdoor illumination.

### 6.5 Physical access

Inspectors climb the derrick in fall-arrest harness (Working at Heights certification), enter the cellar through 24-inch hatches, sit in confined mud tanks, and routinely use one hand on a handrail while the other holds a torch, gauge, or tablet. Oil, drilling mud, grease, seawater on everything. The inspector cannot put a tablet down on the rig floor — it gets kicked, slid, splashed.

**Implication:** strap/lanyard mounting; UI works in landscape **and** portrait, from chest-strap angle. All primary actions reachable with one thumb (bottom 70% of screen). Auto-save on every field change. "Resume where I left off" is the launch state, not "Welcome, please log in."

### 6.6 Photo capture realities

A single inspection: 100–400 photos from 30+ m derrick down to 2 mm weld crack, under torchlight in cellar or 100 klx direct sun. Dust on lens, oily fingerprints on screen. Hours later the inspector struggles to remember which photo goes with which finding (Equinor CIO: "80% of employee time in [oil and gas] is spent looking through unstructured data," cited in [Matterport blog](https://matterport.com/blog/oil-gas-document-management)). Paper inspectors often photograph a finding plus a sticky note with a reference number — fragile when the note blows away.

**Implication:** photos captured from inside a finding/checklist item context, attached to a known item at capture time, never at the desk later. Scale-reference helper. In-app markup (arrows, circles, text) immediately after capture — preserve original + annotated. Thumbnail in draft report so the inspector verifies before sync.

### 6.7 PTW / JSA / toolbox talks

Inspector cannot touch the rig, walk the floor, or open a panel without active PTW, JSA, and morning toolbox talk attendance. Permits time-windowed (12 h) and area-bounded ([DNV PTW guide](https://www.dnv.com/article/a-complete-guide-to-permit-to-work-ptw-systems/)).

**Implication:** record PTW number, JSA reference, toolbox-talk attendance as opening fields of an inspection — these are real metadata the auditor will want. Allow scoping an inspection to a permitted area + time window. Push notifications largely useless — the inspector's personal phone may be locked in the radio room and only the IS device is on deck.

### 6.8 Time pressure and rig dayrates

Sixth-gen drillships earned $400k–$490k/day in H2 2025; seventh-gen $430k–$540k/day. Harsh-environment jackups $140k–$200k/day ([Westwood dayrate forecast](https://www.westwoodenergy.com/business-division/research/global-offshore-drilling-rig-dayrate-forecast)). A one-hour wait for inspection paperwork on a 7th-gen drillship costs ~$22,000. Smaller land rigs $25k–$80k/day, still meaningful.

**Implication:** speed is the metric. No long login, no downloading 200 MB template, no syncing asset library at start. Launch < 3 s to first usable screen. "Start an inspection" ≤ 2 taps from cold start. Report assembly that used to take 4–8 hours back in the office is the primary value lever.

### 6.9 Crew change cycles

Offshore 14/14, 21/21, 28/28 with 12-hour shifts ([Rigzone QA/QC](https://www.rigzone.com/insights/job-descriptions-2/responsibilities-of-a-qaqc-inspector-in-offshore-projects-225)). If the inspector needs to interview the toolpusher about a finding, that toolpusher may be off-rotation for 2–4 weeks.

**Implication:** findings attribute to roles, not just names. Async "ask the toolpusher" comment threads with answers possibly arriving weeks later. Crew/personnel directory importable from operator HR, including rotation schedule.

### 6.10 Data sovereignty and IT lockdown

Operators run third-party security reviews of 3–9 months before cloud SaaS touches their data. Many forbid foreign-hosted cloud SaaS; some require on-premise or in-country deployment. Post-Operation Epic Fury, 94% of operators reviewing unplanned OT security funding from third-party-access risk ([Industrial Cyber](https://industrialcyber.co/news/oil-and-gas-operators-ramp-up-ot-security-spending-post-epic-fury-but-critical-detection-gap-persists/)).

**Implication:** offline-first becomes a regulatory benefit, not just environmental. Per-customer data residency (EU, US, KSA, Norway). Export to neutral formats (PDF/A, CSV, structured JSON) is first-class. SOC 2 / ISO 27001 / SSAE 18 documentation needs to be ready-made.

---

## 7. Competitive Landscape

### 7.1 Categories

**Generic digital inspection / form builders.** SafetyCulture (iAuditor), TrueContext (ProntoForms), Fulcrum, GoCanvas, Device Magic, Nintex Mobile, Form.io.

**Industrial inspection specialists / EHS suites.** Cority, Intelex, Sphera, Enablon, eVision, Fieldwire, AuditBoard.

**Oil & gas–specific inspection / asset integrity.** Cenosco IMS, Pegasus Vertex (now LINQX), Wellsite Report, RigER, Petrolink, P2/Enverus, Quorum, IFS Ultimo, GE Digital APM (Meridium), AVEVA (PI / APM), Hexagon ALI / SmartPlant.

**EAM/CMMS used in O&G.** IBM Maximo, SAP PM / S/4HANA Asset Management, Infor EAM, Oracle eAM, UpKeep, Fiix, Limble, eMaint.

**Drone / visual AI inspection.** Cyberhawk iHawk, Flyability Elios 3, Percepto AIM, Skydio, Sterblue.

**NDT data management.** Mistras OneSuite, Acuren, Olympus / Evident Scientific, Eddyfi.

**Field service / mobile inspector tools.** ServiceMax (PTC), Salesforce Field Service, Praxedo, IFS FSM, Trimble, FieldAware.

**Digital twin / rig-specific.** Cognite Data Fusion, Kongsberg Kognitwin, Akselos, Aize, AVEVA PI Vision.

**Drilling-specific operational reporting (adjacent, not inspection).** Pason DataHub, DataCloud/Rigminder, NOV Max, Halliburton iEnergy.

### 7.2 The 12 most relevant competitors

| Product | Category | Offline | Photo workflow | O&G adoption | Weakness | Price |
|---|---|---|---|---|---|---|
| **SafetyCulture iAuditor** | Generic checklist | Yes (sync issues) | Photo per question + annotation | Wide "good enough" baseline | Reports need Excel cleanup; 3-device cap; rigid scoring | $24/user/mo Premium ([source](https://safetyculture.com/pricing)) |
| **TrueContext (ProntoForms)** | Field forms / workflow | Yes, strong | Photo + GPS | Real O&G technicians on rigs | Clunky on complex forms | From $400/mo team, $4,800/yr min ([source](https://truecontext.com/pricing/)) |
| **Fulcrum** | Geo-first forms | Yes | Photo + map pin | Some O&G, more env/utility | Not built for compliance reports | $43/user/mo, 5 min ([source](https://www.fulcrumapp.com/pricing/)) |
| **GoCanvas** | Generic forms | Yes | Per-field photo | Some oilfield services | No OSHA logs, no incident mgmt | $49–$79/user/mo, 3 min |
| **Cority** | Enterprise EHS | Mobile add-on | Per-finding | Large operators | Heavy implementation, weak field UX | $30k+/yr enterprise |
| **Sphera** | Enterprise EHS/PSM | Limited mobile | Per-finding | Refining, midstream | High 6-figure annual; consultant-led | $500k+/yr typical |
| **IBM Maximo Mobile** | Enterprise EAM | Yes | Attached to work order | Wide upstream | "Not a true app"; reporting needs coding | $50/user/mo+, $200–500k implementation ([source](https://www.itqlick.com/ibm-maximo/pricing)) |
| **SAP PM (S/4HANA AM)** | Enterprise EAM | Via third-party | Via add-on | Operator-standardized | 6–18 month rollout, not mobile-native | Custom enterprise |
| **Cenosco IMS** | O&G asset integrity | Desktop-first | N/A field-first | Integrity engineering teams | Not a field-inspector tool | Enterprise quoted |
| **GE Digital APM (Meridium)** | O&G APM | Limited | Via work order | Refining, upstream | Consultant-heavy | 6–7 figures |
| **ServiceMax FieldFX** | Field service for O&G | Yes | Per-ticket | Oilfield services | Built for ticketing/billing, not audits | $300–500/user/mo |
| **Flyability Elios 3** | Drone visual/UT | N/A (hardware) | Onboard 4K + UT/LiDAR | Tank/confined-space inspection | Hardware, not workflow software | Quote-based |
| **ModuSpec Argus** | O&G-specific digital inspection (incumbent) | Yes | Native | High inside BV/ModuSpec ecosystem | Captive to ModuSpec services | Bundled |

### 7.3 Gaps inspectors complain about

Cross-cutting complaints in G2 / Capterra / SoftwareAdvice and industry write-ups:

1. **Reports need post-export cleanup in Excel/Word.** The output is structured for the form, not for the regulator's expected layout. Inspectors rebuild the PDF anyway ([Capterra iAuditor reviews](https://www.capterra.com/p/141080/iAuditor/reviews/), [Fluix review](https://fluix.io/blog/safetyculture-review)).
2. **Sync is unreliable in real offline conditions.** "Crashing during offline use", "syncing delays" recur in SafetyCulture reviews ([G2](https://www.g2.com/products/safetyculture-2025-01-20/reviews?qs=pros-and-cons)).
3. **Templates can't express conditional engineering logic.** Inspectors need formulas (corrosion rate from two thickness readings), tables (24-bolt torque spec), reference drawings — generic tools represent these as plain text.
4. **No native concept of an asset hierarchy.** A rig has hundreds of taggable items. Generic tools treat them as free text; EAM tools have it but lock it inside expensive implementations.
5. **Photo-to-finding traceability is fragile.** Inspectors photograph on the camera roll and reconcile later — a major source of error and rework.
6. **Multi-day inspections don't fit.** A full IADC drilling rig inspection takes 3–7 days with hundreds of findings; most apps assume a single-pass form completed in minutes.
7. **No purpose-built O&G content.** Generic tools require building templates from scratch; specialist tools require enterprise rollout to use ([BasinCheck oil & gas review](https://basincheck.com/safetyculture-alternative)).

### 7.4 Why SafetyCulture is "good enough" but breaks at rig-audit complexity

SafetyCulture wins because the inspector can build a template in 20 minutes, capture photos and signatures, and email a PDF without IT involvement — a meaningful jump over Word + camera roll. But a rig audit is not a checklist. It's hundreds of structured findings each requiring severity, photo, recommended action, regulatory reference (API/IADC/OSHA clause), responsible party, target date, and verification photo. SafetyCulture's reporting cannot generate the long-form professional inspection report that a third-party inspector (DNV, ABS, ModuSpec) is paid to deliver, and the post-export Excel cleanup negates much of the time saved during capture.

### 7.5 Why enterprise EAMs (Maximo, SAP PM) are too heavy

Maximo and SAP PM each require $200k–$500k+ in implementation consulting, 6–18 months to deploy, and ongoing internal admin staff. The mobile experience is consistently the weakest user-reported aspect ("not a true app", "clunky work-order close-out"). They were architected for the asset owner's CMMS, not for an independent inspector who shows up on a rig for three days, generates a report for the client, and leaves.

### 7.6 Pricing patterns

- **Per-seat checklist apps:** $15–$80/user/month (SafetyCulture $24, Fulcrum $43, GoCanvas $49–$79).
- **Field service per-seat:** $300–$500/user/month (ServiceMax, IFS).
- **CMMS per-user/asset:** $20–$75/user/month SMB; six-figure annual at enterprise.
- **Per-inspection:** common in NDT/integrity services (Mistras charges per scope of work bundled with software).
- **Enterprise license:** Sphera, Cority, Maximo, SAP PM — six figures + multi-six-figure implementation.

### 7.7 White-space opportunities

1. **Inspector-grade report generator, not a checklist app.** Polished, sectioned PDF generated directly from field capture with zero Word/Excel post-processing.
2. **Rig-native data model out of the box.** Pre-built asset hierarchy (derrick, substructure, drawworks, mud system, BOP, well control, electrical, safety) keyed to IADC/API/OSHA inspection categories.
3. **Multi-day, multi-inspector audit primitive.** First-class "inspection campaign" object — split sections, merge findings, hand off shifts, produce one consolidated report.
4. **IS-device friendliness as a first-class constraint.** Run on Android (i.safe, ecom) and Windows (Aegex, Getac) IS tablets.
5. **Bring-your-own-cloud / data residency.** Per-customer EU/US/KSA/Norway, optional self-host, neutral export formats.
6. **Photo capture inside the finding context, never the camera roll.** Born attached to a specific item with timestamp, GPS, inspector ID.
7. **One-tap resume, sub-3-second launch, all primary actions thumb-reachable.**
8. **Voice as an after-the-floor reflective tool.** On-device transcription of dictated narrative findings in the truck/trailer, not on the deck.

No current vendor sits at the intersection of (1)+(2)+(3) — SafetyCulture is too generic; Cenosco/Maximo/SAP too heavy; ServiceMax/IFS oriented to dispatch rather than audit; ModuSpec Argus is captive to its services arm.

---

## 8. Reporting Intelligence

### 8.1 Anatomy of a professional rig inspection report

Canonical structure (consistent across ModuSpec, OCS Group, Applus+, Bureau Veritas, Lloyd's Register, Global Offshore Engineering):

1. **Cover & control page.**
2. **Executive summary (1–3 pages).**
3. **Table of contents + acronyms.**
4. **Scope of work** as agreed pre-mob.
5. **Methodology** — standards referenced, checklists used, NDT methods, sampling, exclusions.
6. **Rig description.**
7. **System-by-system findings** — each major system (well control / hoisting / rotary & top drive / mud / cementing / power / HVAC / safety / structure / marine / I&C / handling / accommodation) with condition narrative, photo plates, findings table extract.
8. **Consolidated defect register.**
9. **NDT reports appendix** with calibration certs and Level II/III sign-off.
10. **Certificates & class status appendix** — current ABS/DNV/LR certs, BOP test charts, mast certs, lift certs.
11. **Photo log** — every photo numbered, captioned, cross-referenced.
12. **Recommendations / road-map** prioritised by severity and target close-out.

Sources: [ModuSpec rig inspection & intake](https://www.moduspec.com/services/rig-inspection-and-intake), [OCS Group rig condition & acceptance](https://ocsgroup.com/service/inspections-audits-and-compliance/rig-inspections/rig-condition-and-acceptance-inspections/), [GPS rig acceptance](https://gammaps.net/rig-acceptance.html), [GOE rig condition surveys](http://www.goe-group.com/rig-condition-surveys/), [Applus+ rig audit white paper](https://www.scribd.com/document/419970538/Rig-Audit), [Bureau Veritas Middle East technical rig audit](https://middle-east.bureauveritas.com/your-needs/inspection/technical-rig-audit), [InspectionTrack — structuring an O&G inspection report](https://www.inspectionstrack.com/how-to-structure-a-detailed-oil-and-gas-inspection-report/).

### 8.2 Executive summary conventions

- **Length:** ≤ 2 pages prose + 1-page dashboard. If the CEO has to turn a third page they stop reading.
- **Scope sentence** anchored to specific standards — "Rig X, Type Y, scope was a 5-day operational walkdown to API RP 4G Cat I/II + OSHA-IADC + DROPS, with NDT limited to MPI of crown sheaves."
- **Headline rating** — single label (Excellent / Good / Fair / Poor) + 1-pager system-level RAG dashboard. ABS and DNV CAP use numeric scores; ModuSpec / OCS / BV typically a narrative label backed by a percent compliance figure (e.g. "92.6% compliance").
- **Go / no-go / go-with-conditions** statement.
- **Top critical findings** — bullet list of 5–10 issues that drive risk; each line shows title, system, severity, recommended action, est. close-out time.
- **Key recommendations** — 10–20 lines binned by short/medium/long term.

### 8.3 Defect / findings register — typical columns

| Column | Notes |
|---|---|
| Finding ID | sequential or system-coded (e.g. `BOP-031`) |
| System | controlled list mapping to client EAM hierarchy |
| Sub-system / Component | |
| Tag number / Serial | links to client's asset register |
| Photo refs | one finding → many plate numbers |
| Finding / Observation | factual |
| Cause (if known) | |
| Reference clause | API RP 4G §6.x, OSHA 1910.147, etc. |
| Severity | Critical / Major / Minor (or A/B/C; plus DROPS for at-height) |
| Risk score | likelihood × consequence per 5×5 |
| Recommendation | what to do |
| Target close-out date | |
| Responsible party | rig owner / OEM / inspector |
| Status | open / in progress / closed / verified |
| Verified-by + date | |

### 8.4 Photo evidence practices

- Each photo gets a **plate number** ("Plate 4-12" = section 4, photo 12), continuously numbered.
- Caption includes location (deck level, GA grid ref, system tag), what-it-shows sentence, finding ID cross-reference.
- Annotations: arrows + callouts (red boxes, dimension lines, scale rule, date stamp + photographer initials).
- Before/after pairs for items closed-out during the inspection (CDI items per OSHA-IADC).
- Wide → medium → close progression so the reader can orient.
- Sidecar raw photo bundle — full-res, EXIF-intact, geo-tagged.

### 8.5 Condition scoring and operational readiness

Two forms travel:
- **Aggregate score** — % compliant, or a weighted score across critical/major/minor (e.g. "92.6% on safety compliance").
- **System-level RAG dashboard** — 12–20 row table showing each major system as Red / Amber / Green or Excellent / Good / Fair / Poor.
- **Operational readiness statement** — yes/no/yes-with-conditions, with the conditions explicit.

For pre-purchase due diligence, the rating drives **valuation impact** directly — the gap between a "Good" rig and a "Fair" rig is millions of dollars of capex backlog plus downtime risk.

### 8.6 Data visualisation typical in reports

- RAG matrix by system (heatmap).
- Heatmap of finding density across the rig (bar chart of defects per system).
- Before/after photo comparisons for in-inspection corrections.
- Trend graphs of % compliance vs prior inspections.
- NDT thickness contour / C-scan map overlaid on GA drawings ([Corrosion mapping in action](https://www.worldpipelines.com/special-reports/20062024/corrosion-mapping-in-action/)).
- Well-barrier schematic (NORSOK-aligned reports) — two-colour (primary/secondary) barrier envelope diagram ([SPE on barrier schematics](https://jpt.spe.org/using-schematics-managing-well-barriers)).
- Risk matrix with finding-count cells.

### 8.7 Three report archetypes

| Type | Page range | Trigger | Scope | NDT | Deliverables |
|---|---|---|---|---|---|
| **Operational walkdown / safety inspection** | ~10–30 pp | Routine, weekly/monthly | OSHA-IADC checklist, RP 54, DROPS walkdown | None or minimal VT | PDF report + defect list, often next-day delivery |
| **Rig condition / acceptance audit** | ~100–300 pp | Pre-contract, end-of-contract, third-party audit | Full systems audit to API/IADC/class standards; equipment open-ups; function/load tests; partial NDT (Cat III scope) | MPI/UT/VT on critical load-bearing items | Bound PDF + Excel defect register + photo log + NDT appendices + cert copies |
| **Pre-purchase / due-diligence** | ~50–150 pp | M&A, asset sale, refinancing | Condition assessment + capex backlog quantification + valuation input | Targeted NDT on highest-risk components | All of above + capex backlog spreadsheet by year + executive briefing slides |

### 8.8 What CEOs and management actually read

1. **Executive summary headline + go/no-go.** CEO will look at this and the system-level RAG bar.
2. **Top critical findings list.** 5–10 items. If any have $ figures attached (e.g. "estimated 14 days downtime and $1.2M to remediate"), those get circled.
3. **Photos of any "critical" findings.** The visceral case for spending money.
4. **Capex backlog total** (due-diligence reports).
5. **Recommendations / road-map** — usually delegated to a VP of operations.

What CEOs ignore: detailed NDT appendices, methodology section, certificates. Those are for asset-integrity team and regulators.

What triggers action: a **Red** on a safety-critical system, an unscheduled **5-year recert overrun** (BOP or mast), a **third-party regulator finding** referenced by a clause (BSEE notice, HSE improvement notice), or a **DROPS Major/Fatality** category dropped-object risk.

### 8.9 Delivery formats

- **PDF** — bound, signed, controlled-revision; primary deliverable.
- **Bound hardcopy** — still requested for due-diligence dossiers and some IOC clients.
- **Excel defect register** — the operational artefact that planners actually work from.
- **Sidecar photo bundle** — full-res JPG/RAW, EXIF, sometimes geo-tagged, delivered on USB / SharePoint.
- **Portal access** — some service providers and operator-owned platforms push findings to a web portal where they can be ticked off as closed ([RIG Trac by OCS Group](https://ocsgroup.com/service/software-solutions/hazard-trac-2/), [Inspections Track](https://www.inspectionstrack.com/), [Arcus](http://arcus.live/)).
- **Client EAM ingest format** — increasingly, operators want the defect register in a format their CMMS/EAM can ingest directly so each finding becomes a notification/work order without re-keying.

### 8.10 The knowledge-loss problem

The after-life of a typical inspection report is poor:
- The PDF lands, gets filed, gets emailed around, becomes "the inspection we did in March."
- The Excel defect register drifts; closures aren't fed back; next year's inspector has no way to know what was closed properly vs left to drift.
- Photos sit on a OneDrive folder; no one looks at them again.
- The inspector's tacit knowledge — "the bearing on the port crown sheave has been on the watch list since 2019" — disappears when the inspector retires.

> **Implications for software design — Reporting**
> - Render an executive summary in ≤ 2 pages with system-level RAG dashboard, top-N critical findings, explicit go/no-go statement.
> - Defect register as first-class structured object, not a Word table.
> - Photo plates with continuous numbering, captions linked to GA grid refs and finding IDs, annotation tools, wide→medium→close grouping, exportable sidecar bundle preserving EXIF.
> - Multi-format deliverable from one data model: PDF/A signed report; Excel defect register with stable column schema; portal view; sidecar photo bundle; **EAM-ingest CSV/JSON** mapping cleanly into Maximo work-order / SAP PM notification / Infor EAM work-request formats.
> - Distinct report templates for the three archetypes (walkdown, audit, due-diligence) on the same data layer.
> - Versioned, append-only history of findings per asset/component so the next inspection inherits open items, marks close-outs with evidence, shows trend over time.
> - Inspector signatures + qualification stamps auto-applied per finding, with certificate ID and expiry in audit trail.
> - Standards library as data (controlled clause selection per finding), not free text.
> - Capex backlog roll-up for due-diligence mode — each finding can carry estimated cost and downtime; report rolls up by year and severity.
> - Well-barrier schematic as native diagram object (NORSOK D-010).
> - Live closure loop — defect register portal where rig owner closes items with evidence; closure captured into same record so next inspector inherits the truth.
> - Calibration & certification register linked to both inspector and instrument; block readings from out-of-cal instruments; warn on expiring certifications.

---

## 9. Workflow Automation Opportunities

### 9.1 Pain-point ranking matrix

Time-cost estimates are per inspection. "Leverage" distinguishes pure time saving from unlocking a capability that previously did not exist.

| # | Pain point | Stage | Hours saved | Error reduction | Difficulty | Leverage |
|---|---|---|---|---|---|---|
| 1 | Retyping field notes into Word | After-hours | 8–14 | High | Low | Time-only |
| 2 | Photo→finding linkage | Capture / After-hours | 4–8 | Very high | Low | Time + quality |
| 3 | Photo plate layout in Word | Compilation | 3–6 | Medium | Low | Time-only |
| 4 | Defect register table build | Compilation | 2–4 | High | Low | Time + queryable data |
| 5 | Executive summary drafting | Compilation | 1–3 | Low | Medium (LLM) | Time-only |
| 6 | Severity decision consistency | After-hours | 0.5–1 | Very high | Low (rules) | Quality + defensibility |
| 7 | Reusable findings library | Compilation | 2–4 | High | Low | Compounding moat |
| 8 | Standard-clause auto-citation (API/IADC) | Compilation | 1–2 | High | Medium | Credibility, defensibility |
| 9 | Component-aware templates per rig type | Pre-inspection | 1–3 | High | Medium | Quality, completeness |
| 10 | Historical comparison (this rig vs last visit) | Pre / Delivery | 1–2 once, compounds | Medium | Medium | New capability |
| 11 | Cross-rig / fleet benchmarking | Delivery | n/a single-visit | Medium | High | Net-new product |
| 12 | PDF export with parity to current Word | Compilation | 0.5–1 | Low | Medium | Adoption blocker |
| 13 | Close-out tracking (findings → response → resolved) | Follow-up | 1–2 | Medium | Medium | Stickiness |
| 14 | Prior-report retrieval | Pre-inspection | 0.5–1 | Low | Low | Stickiness |
| 15 | Voice-memo transcription with finding linkage | Capture / After-hours | 1–3 | Medium | Medium | Quality of life |

Aggregate: an MVP that nails items 1–4, 6, 7, 9 and 12 saves **15–30 hours per inspection**. At a third-party inspector day-rate of ~$1.2k–$2.5k, that is **$1.8k–$7.5k of recovered margin per job** before any AI.

### 9.2 Highest-leverage automations (ranked)

1. **Structured component-aware templates per rig type.** A jack-up template instantiates "BOP stack, derrick, drawworks, top drive, mud pumps, SCRs, accommodation, lifeboats, helideck…"; a land rig template instantiates a different tree. Template is checklist *and* report spine.
2. **Photo auto-tagging via temporal proximity.** Every photo taken within *N* minutes of a finding creation is provisionally linked to that finding, subject to one-tap confirmation. No ML needed.
3. **Auto-generated report from structured findings.** If the data model is right, the report is a deterministic render. No "report writing" — only "report reviewing."
4. **Reusable findings/recommendations library** with standard phrasing, indexed by component and severity. The compounding moat — each inspection enriches the library and shortens the next.
5. **Rule-based severity decision support** ("derrick weld crack + load-bearing → CRITICAL"). Deterministic, defensible, much safer than an AI severity classifier.
6. **Historical comparison.** "Repeat finding from 2024-09 inspection — not closed" is enormously valuable to a Drilling Superintendent and proves out-of-the-box value the moment a rig is re-inspected.
7. **Standard-clause auto-citation.** Tag each finding with the relevant API RP 4G or IADC clause; render inline in the report.

The pattern: **structure first, AI later.** Every one of these is achievable with deterministic software, and each one makes future AI features dramatically more useful by giving them structured input.

---

## 10. Realistic AI / Data Potential (no hype)

Starting assumptions: small initial dataset (one inspector's history, maybe hundreds of historical reports if extractable from Word); industrial-safety false-negative cost is asymmetric (a missed crack can be a fatality and a lawsuit); buyers are skeptical of AI in safety contexts; regulators will eventually ask "show me the model card." Bias is toward AI that *assists humans* rather than AI that *makes safety calls*.

### 10.1 Verdict table

| Use case | Value | Risk | Feasibility solo | Verdict |
|---|---|---|---|---|
| LLM narrative drafting from structured findings | High | Low | High | **BUILD NOW** |
| LLM executive summary | High | Medium | High | **BUILD NOW** (review-gated) |
| Photo classification (component type) | Medium | Low | Medium | BUILD LATER (v1.5) |
| Defect detection in photos (corrosion / cracks / leaks) | High if it worked | Very high | Low | **DO NOT BUILD** |
| OCR of nameplates/serials/calibration stickers | High | Low | High | **BUILD NOW** (v1.1) |
| Voice-to-text dictation (push-to-talk, structured field) | High | Medium | High | **BUILD NOW** |
| Predictive maintenance from inspection data | High if it worked | High | Very low | **DO NOT BUILD** |
| Fleet benchmarking / trend analysis | Medium | Low | Medium | BUILD LATER (v2) |
| RAG over historical reports + standards corpus | Medium | Medium | Medium | BUILD LATER (v1.5–v2) |

### 10.2 The honest take on defect detection

This is where the industry over-claims. ImageVision's marketing cites "95%+ defect detection accuracy in real-time" ([ImageVision](https://imagevision.ai/blog/transforming-oil-and-gas-asset-management-with-drone-inspection-using-vision-ai/)); GE/Avitas claimed up to 25% inspection cost reduction with "automated defect recognition" ([Commercial UAV News](https://www.commercialuavnews.com/infrastructure/ge-uses-drones-reduce-inspection-costs)). Peer-reviewed work on corrosion is more honest — one study reported 95% of corroded images correctly classified but 5% false negatives, with authors noting "corrosion detection model algorithms will output a considerable number of false positives and false negatives when challenged in the field" ([Nature npj Materials Degradation](https://www.nature.com/articles/s41529-022-00232-6)).

Industrial benchmarks target mAP@0.5:0.95 > 0.7 ([Roboflow](https://blog.roboflow.com/corrosion-detection/)) — meaningful in lab, far from "trust this for a kill decision." Cyberhawk publishes no headline accuracy number and positions its drones + iHawk as a *data-collection-and-analysis platform with human engineering review*, not autonomous defect detection ([Cyberhawk](https://thecyberhawk.com/oil-gas-marine)). Percepto's hard published number is methane OGI down to 100 g/hr with 90% reliability under EPA OOOOa/b — a regulatory-grade claim that took years of co-development with Chevron and EPA approval ([DroneLife](https://dronelife.com/2025/10/29/autonomous-methane-detection-drones-earn-epa-approval-for-oil-and-gas-compliance/)).

**Verdict: DO NOT BUILD in v1, probably not v2.** The cost of a false negative in a rig safety context is catastrophic, the dataset to do this honestly is years away for a solo founder, and the buyers who have actually deployed it (Chevron) did so with multi-year co-development and explicit human-in-the-loop framing. If you must touch it, ship "AI-assisted photo triage" that flags candidate corrosion for inspector review — never auto-classifies severity.

### 10.3 Voice — what actually works in industrial noise

Whisper-large-v3 hits ~5–6% WER on clean English with documented robustness to ESC-50 environmental noise ([MLCommons](https://mlcommons.org/2025/09/whisper-inferencev5-1/), [Whisper-AT paper](https://arxiv.org/pdf/2307.03183)). Deepgram Nova-2 reports ~8.4% median WER across mixed real-world domains with sub-300 ms latency ([Deepgram Nova-2](https://deepgram.com/learn/nova-2-speech-to-text-api)). Neither publishes a number for "next to a running mud pump at 95 dB."

Realistic expectation: usable for short structured dictations (5–30 s) into a finding's "observation" field when the inspector steps away from the loudest noise source; unusable for continuous transcription on the rig floor. **Push-to-talk, on-device Whisper-small fallback for offline. Build it as a structured-field dictation feature, not a "magic notebook."**

---

## 11. Product & Business Strategy

### 11.1 Buyer personas

| Persona | What they care about | Budget? | Sales cycle | Day-1 fit? |
|---|---|---|---|---|
| Solo / small-shop third-party rig inspector (the user) | Time saved per job, report quality, professional image | Personal credit card → subsidises consulting fee | Days | **Yes** |
| Mid-size inspection firm (RigQA, regional Add Energy practice) | Consistency across inspectors, scaling without hiring senior reviewers, branded report templates | Practice lead / partner | Weeks–months | **Yes (v1.5)** |
| Tier-1 inspection firm (ModuSpec/BV, Add Energy, Stewart Buchanan, Applus+, ABL) | Workflow standardisation, audit trail, integration with their existing portal (ModuSpec Argus, ABL online reporting) | Practice head + IT + Legal | 6–18 months | Hard — they already have something |
| Drilling contractor (H&P, Patterson-UTI, Nabors, Precision, Transocean, Valaris) | Fleet-wide inspection consistency, defect register hygiene, integration with existing platforms (H&P DrillScan, Nabors RigCLOUD) | VP Drilling + Asset Integrity Manager + IT Security | 12–24 months | No (v3+) |
| Operator (Chevron, Shell, Petrobras, Equinor) | Vendor-management overlay, third-party assurance, data sovereignty | Operations Excellence / Wells Integrity / HSE | 12–24+ months | No — they buy from Tier-1s |
| Insurance / lender / classification society | Standardised, attestable inspection trail; defect-history audit | Underwriting | 6–18 months | No (v3+) |
| Rig builder / refurbisher (Keppel, Lamprell, NOV) | Acceptance inspection workflow for new builds; reactivation surveys | Project management office | 6–12 months | Possible (v2) |

**Budget ownership.** For mid-market, the practical buyer is the inspection practice lead or business owner who personally feels the pain. For drilling contractors, the spend split is roughly: HSE/Wells Integrity owns the operational case ("we keep finding the same defects six months later"), Asset Integrity Manager owns the technical fit, IT/InfoSec owns the security gate, VP Drilling or VP HSE signs the cheque.

### 11.2 Procurement reality

Enterprise O&G procurement is security-questionnaire-first. Average enterprise security questionnaire: 200+ items across 19 risk domains, 15–40 hours to complete manually; >70% of B2B SaaS deals require SOC 2 before contracts get signed; SOC 2 Type II audits take 6–12 months and cost $7k–$50k+ ([Iterators](https://www.iteratorshq.com/blog/soc2-compliance-for-saas-why-enterprise-customers-demand-it-and-how-to-get-certified/), [Workstreet SOC 2 guide](https://www.workstreet.com/blog/soc-2-for-startups)). Add O&G-specific gates: MSA negotiation, supplier qualification (Achilles, ISN), cyber-insurance COI, ISO 27001 for EMEA, data-residency clauses for Middle East and Brazil. Real-world cycle to a drilling contractor: 12–24 months, and the deal does not start until SOC 2 Type II is in hand.

**Adoption inertia.** O&G is slow on SaaS for legitimate reasons: (1) audit-trail and 20-year retention requirements that don't fit "delete-after-30-days" SaaS defaults, (2) data-sovereignty rules (Brazil ANP, Saudi data residency), (3) integration with SAP PM and IBM Maximo entrenched as CMMS for any majors ([IBM Maximo for O&G](https://www.ibm.com/products/maximo/oil-gas)), (4) conservative culture, and (5) downturn-burned IT budgets making buyers gun-shy about Series-A vendors.

### 11.3 Switching barriers and stickiness

The moat for an inspection tool sits in roughly this order:
1. **Report archive** — the inspector's own history lives in your system; leaving means losing it.
2. **Custom findings/recommendations library** — the inspector's own phrasings, refined over months. The most personal lock-in.
3. **Component templates** — rig-specific component trees built and refined.
4. **Standards reference data** — the curated API/IADC clause map that takes months to build.
5. **CMMS / client-side integration** — once findings flow into the operator's Maximo as work orders, you become infrastructure.
6. **Brand trust with the inspector's clients** — when the CEO of a drilling contractor recognises the report layout, the inspector cannot leave the tool without explaining a downgrade.

### 11.4 Pricing benchmarks

- **SafetyCulture / iAuditor:** ~$19–$24 per user per month; 30-person team on Premium ~$720/month ([SafetyCulture pricing](https://safetyculture.com/pricing)).
- **BasinCheck** (purpose-built O&G): $149–$599/month flat team pricing ([BasinCheck](https://basincheck.com/resources/best-safety-management-software-oil-gas)).
- **FleetRabbit** (oilfield fleet inspection): $3 per vehicle per month ([FleetRabbit](https://fleetrabbit.com/industry/oil-and-gas/oilfield-fleet-software-pricing-guide)).
- **Quorum, Cognite, IBM Maximo:** enterprise quote-only; Forrester on Cognite suggested $21.6M NPV at 400% ROI for asset-heavy adopters ([Cognite ROI](https://www.cognite.com/en/company/newsroom/independent-study-finds-asset-heavy-organizations-can-expect-21.6m-in-value-after-adopting-cognite-data-fusion)).

**Recommended pricing for v1 sold to individual inspectors:** $99–$199/month per inspector, annual prepay discount, no per-inspection meter. Signal you want: "I'd pay this out of my own pocket because it pays for itself in one job." Per-inspection metering punishes power-users — exactly your champions. Per-seat works for inspection firms in v1.5. Save per-rig pricing for v3 when you sell fleet view to drilling contractors.

### 11.5 Go-to-market motions

(a) **Bottom-up tool for individual inspectors.** Fast adoption (days), low ACV ($1.2k–$2.4k), no procurement gate, immediate feedback loop, founder-market fit. Compounds into (b) over 12–18 months. **Recommended starting motion.**

(b) **Sell to inspection firms.** Moderate ACV ($30k–$150k/yr per firm), 3–9 month cycle, moderate security gate, accelerates report quality across multiple inspectors. **Year-2 motion**, seeded by inspectors in (a) who join firms.

(c) **Sell directly to drilling contractors.** High ACV ($250k–$2M/yr), 12–24 month cycle, full SOC 2 + ISO 27001 + cyber insurance gate. **Year-3+ motion.** Do not start here.

(d) **Inspection-as-a-service hybrid.** Productize the founder's own inspection business. Advantage: captive customer + demo. Disadvantage: confuses buyer about whether you're software or services and caps growth at billable hours. **Use as runway / case-study generator, not as the company.**

### 11.6 Defensibility lessons from the market

- **Cognite Data Fusion** succeeded by being the "contextualisation layer" for huge industrial data lakes with explicit big-customer co-development (Aker BP origin, supermajor case studies). Lesson: in O&G, marquee logos compound, and ROI must be in eight-figure NPV for majors ([Cognite scaling case study](https://www.cognite.com/en/resources/customer-stories/dataops-oil-gas-scaling)).
- **Quorum Software** consolidated upstream-back-office software via Thoma Bravo-funded M&A (Aucerna merger, TietoEVRY O&G assets) — a roll-up, not an organic moat. Lesson: O&G software gets acquired more than it IPOs.
- **Peloton (Calgary)** built deep, narrow workflow tools (WellView, ProdView) and survives on stickiness in a specific job-to-be-done. Lesson: narrow + deep beats broad + shallow.
- **PetroLink** survived as the "neutral wellsite data" layer by being non-threatening to incumbents — positioning lesson for any startup.
- **H&P (DrillScan acquisition)** and **Nabors (RigCLOUD)** are drilling-contractor-internal platforms; biggest contractors will *build/buy* their way into your category if you let them. Your moat must be vertical-deep on the inspection workflow itself, not horizontal data-platform.
- **Failures** (rarely written up): well-funded "AI for oil and gas" startups (Avitas Systems, multiple drone-defect-detection plays) that pitched autonomous defect detection and quietly pivoted to data-services or got absorbed. Lesson: **leading with "AI replaces the inspector" gets you a polite meeting and no PO.**

---

## 12. Risk Register & Implementation Insights

### 12.1 Risk register

| Risk | Likelihood | Severity | Mitigation |
|---|---|---|---|
| PDF formatting parity with Word fails — CEO rejects report | Medium | Fatal | Match a real founder-authored report exactly in week 1; CEO showcase before broad release |
| Photo loss in field | Medium | Fatal | Local-first writes, durable upload queue, sync-status UI, manual export |
| Inspector rejects tool mid-job and reverts to paper | Medium | High | Keep paper as fallback through Inspection #2; founder must dogfood first |
| IT security gate when selling to firm | High at year 2 | Sales-killing | SOC 2 Type I in progress before first firm conversation; data-residency-friendly hosting |
| AI hallucination in exec summary embarrasses inspector with CEO | Medium | High | Always inspector-reviewed; visible "drafted by AI" attribution; never auto-send |
| Founder runs out of money before product-market fit | Medium | Fatal | Use own consulting fees as runway; price v1 to be paid-for by use |
| Large incumbent (ModuSpec Argus, BasinCheck) ships the same feature | Low–medium | Medium | Move faster, stay inspector-shaped (not enterprise-shaped) |
| Rugged tablet hardware variance breaks app | Medium | Medium | Standardise on Samsung Galaxy XCover or Panasonic Toughpad for v1 testing; certify IS variants later |
| Standards drift (API RP 4G new edition) makes clause library stale | Low | Low | Manual library curation, quarterly review |
| Operator refuses to accept reports generated by an unknown vendor | Medium | High | Output is plain PDF + standards-cited findings + signature — same shape as a Word report; vendor brand irrelevant to deliverable |
| IS device certification doesn't allow your APK | Low | Medium | i.safe MOBILE and ecom devices accept standard Android APKs; confirm before promising Zone 1 use |

### 12.2 Implementation insights

- **Dogfood inspection #1 with the founder.** No second user until the first three reports have shipped from the platform.
- **Reference-report parity is the adoption gate.** Spend the first 30 days of the build on data model + PDF render against the founder's last real Word report. Iterate template until visually equal. Everything else is downstream of this.
- **Treat sync state as a UX problem.** Inspectors who lose data once stop using the tool forever. The sync-status screen is as important as the new-finding screen.
- **Severity vocabulary is configurable per client.** Don't hardcode A/B/C; some clients use Critical/Major/Minor; many use 5×5. Ship with a default but let the inspector switch per inspection.
- **Standards library is a curated asset, not a marketing checkbox.** Build the API RP 4G + RP 8B + Std 53 + IADC + 30 CFR 250 clause map by hand. This is your hardest-to-copy moat and is what makes reports defensible at audit.
- **Resist "platform" thinking.** Every "what if we also did X" idea (cross-rig benchmarking, fleet dashboards, integrations with Maximo, AI defect detection) is a year-2 question. The 90-day window is single-inspector-replaces-Word.
- **Findings library compounds.** Each inspection enriches the snippet library. By inspection #20, autocomplete is suggesting 80% of the recommendation text. This is invisible day-1 value that becomes the strongest reason to stay.
- **Ship the export-everything button on day one.** The "give me a zip of my data" feature is what closes the trust gap with inspectors who have been burned by vendors before.

### 12.3 The opinionated bet

Pick the **bottom-up, single-inspector workflow tool** path. Defend it on three grounds:

1. **The founder is the customer.** The cardinal sin of O&G software startups is building for buyers they have never been. You avoid it.
2. **The highest-leverage automations** — structured templates, photo→finding linkage, deterministic PDF render, findings library — are all pure software with no AI dependence and ship in 90 days.
3. **Every higher-ACV motion is reachable from this start without rewriting.** The entities that work for one inspector also work for a firm and a contractor — what changes is permissioning, integration, and security posture, all addressable later with funding.

The dangerous alternative — building "AI rig inspector" for drilling contractors — burns 18 months, requires SOC 2 Type II on day zero, faces incumbents (Maximo, RigCLOUD, DrillScan) with installed bases, and asks an industry that has been burned by AI promises to trust your model with safety decisions. **It is the wrong shape for a solo founder.**

The product that wins is the boring one that ships in 90 days, replaces an evening of Word formatting on inspection #1, and quietly accumulates a findings library, a standards-clause map, and an archive of comparable inspections that nobody else has.

---

## 13. References

### Industry workflow & inspection firms
- [WSP Drilling Pre-Mobilization HSE Checklist (Nov 2025)](https://www.wsp.com/-/media/policy/canada/document/drilling-checklists/wsp-drilling-pre-mobilization-hse-checklist---november-2025.pdf)
- [DrillingContractor — OMV / ModuSpec joint rig-auditing case study](https://drillingcontractor.org/7-ways-to-set-up-a-successful-joint-rig-auditing-task-force-lessons-learned-from-omv-case-study-6963)
- [Rig Acceptance Standards — DrillingForGas](https://drillingforgas.com/drilling/standards/rig-acceptance-standards/)
- [RIG MANUFACTURING — Drilling Rig Inspection Services (1–5 rating)](https://www.rigmanufacturing.com/drilling-rig-inspection-services/)
- [Inspectly360 — Drilling Rig Safety Inspection](https://www.inspectly360.com/checklists/energy-utilities/drilling-rig-safety-inspection/)
- [SafetyCulture — Daily Rig Inspection template](https://safetyculture.com/library/mining/daily-rig-inspection-ormazt6uwvbtlhcq/)
- [IFP Training — Rig Inspection (Audit) 5-day course PDF](https://www.ifptraining.com/report/pdf/951)
- [Lloyd's Register — Critical Equipment Survey](https://www.lr.org/en/latest-news/critical-equipment-survey/)
- [ModuSpec — Rig Inspection and Intake](https://www.moduspec.com/services/rig-inspection-and-intake)
- [ModuSpec Argus digital inspection](https://www.moduspec.com/services/moduspec-argus)
- [Aqualis Offshore — Rig Inspection Services](https://aqualisoffshore.com/services/rig-inspection-services/)
- [ABL Group — Rig Inspections & Assurance](https://abl-group.com/abl/loss-prevention/marine-surveys-inspections-and-audits/rig-inspections-and-assurance/)
- [OCS Group — Rig Condition and Acceptance Inspections](https://ocsgroup.com/service/inspections-audits-and-compliance/rig-inspections/rig-condition-and-acceptance-inspections/)
- [Athens Group — Rig Inspection and Acceptance](https://athensgroupservices.com/services/rig-inspection-acceptance/)
- [Rig QA International — Rig Inspections](https://rigqa.com/services/rig-inspections/)
- [NOV — Annual Rig Inspections](https://www.nov.com/products/annual-rig-inspections)
- [RIGCERT certification services](https://www.rigcert.org/en/)
- [Vidya — Challenges of Offshore Inspection](https://vidyatec.com/blog/the-challenges-of-offshore-oil-and-gas-inspection/)

### Standards & regulation
- [API Spec 4F page](https://www.api.org/products-and-services/standards/important-standards-announcements/spec4f)
- [API RP 4G 2019 PDF copy](https://www.ipgmservicios.com/wp-content/uploads/2024/03/API-RP-4G-2019-Operation-Inspection-Maintenance-and-Repair-of-Drilling-and-Well-Servicing-Subestructures.pdf)
- [API RP 4G errata](https://www.api.org/~/media/files/publications/addenda-and-errata/4g%20e4%20errata.pdf)
- [API RP 8B 2014 + errata PDF copy](https://www.ipgmservicios.com/wp-content/uploads/2024/03/API-RP-8B-2014-Add.1-March-2019-Errata-1-August-2019.pdf)
- [API Spec 7K publication notice](https://www.api.org/~/media/files/publications/whats%20new/7k_e6%20pa.pdf)
- [API Std 53 — Piping Technology summary](https://pipingtechs.com/api-standard-53-blowout-prevention-equipment-systems-for-drilling-wells/)
- [API Spec 16A PDF mirror](https://www.saigaogroup.com/uploads/file/api-16a-standard.pdf)
- [API Spec Q2 page](https://www.api.org/products-and-services/standards/important-standards-announcements/specq2)
- [API RP 54 PDF](https://www.api.org/-/media/files/publications/rp-54_e4.pdf)
- [IADC RigPass](https://iadc.org/accreditation/rigpass/)
- [OSHA + IADC Oil & Gas Rig Inspection Checklist (Sep 2013)](https://iadc.org/wp-content/uploads/2014/04/OIL-Gas-rig-audit-OSHA-IADC-09-2013.pdf)
- [IADC HSE Case for Land Drilling Units](https://store.iadc.org/product/iadc-hse-case-guidelines-for-land-drilling-units)
- [IADC HSE Case for MODUs](https://store.iadc.org/product/iadc-hse-case-guidelines-for-mobile-offshore-drilling-units)
- [DROPS Calculator](https://www.dropsonline.org/drops-guidance-and-resources/drops-calculator/)
- [DROPS Metric Calculator PDF](https://www.dropsonline.org/assets/documents/DROPS-Metric-Calculator-A4-June-2021.pdf)
- [ASNT NDT Certification](https://certification.asnt.org/certification)
- [ASNT CP-189 listing](https://source.asnt.org/1pekngo/)
- [Training NDT — NDT Levels Explained](https://www.trainingndt.com/easy-as-123-ndt-certification-levels-explained/)
- [29 CFR 1910 index — OSHA](https://www.osha.gov/laws-regs/regulations/standardnumber/1910)
- [29 CFR 1926 Subpart CC — OSHA](https://www.osha.gov/laws-regs/regulations/standardnumber/1926/1926SubpartCC)
- [Vector Solutions — 1910 vs 1926](https://www.vectorsolutions.com/resources/blogs/osha-part-1910-general-industry-part-1926-construction-standards/)
- [BSEE Inspection Policy Branch](https://www.bsee.gov/what-we-do/offshore-regulatory-programs/offshore-safety-improvement/inspection-policy-branch)
- [BSEE Risk-Based Inspection Program](https://www.bsee.gov/what-we-do/offshore-regulatory-programs/offshore-safety-improvement/inspection-programs/risk-based-inspection-program)
- [30 CFR 250 Subpart G](https://www.ecfr.gov/current/title-30/chapter-II/subchapter-B/part-250/subpart-G/subject-group-ECFR045ffcd99ad03d3)
- [30 CFR 250.737 — BOP testing](https://www.ecfr.gov/current/title-30/chapter-II/subchapter-B/part-250/subpart-G/subject-group-ECFR045ffcd99ad03d3/section-250.737)
- [BSEE TAP Report 693ab — BOP FMECA](https://www.bsee.gov/sites/bsee.gov/files/tap-technical-assessment-program//693ab.pdf)
- [HSE L154 — SCR 2015 guidance](https://www.hse.gov.uk/pubns/books/l154.htm)
- [HSE L84 — well aspects of DCR PDF](https://books.hse.gov.uk/gempdf/l84.pdf)
- [NORSOK D-010 release notes PDF](https://www.offshorenorge.no/contentassets/f60cf93f2c9c4c129b0a73303ff8081c/02---norsok-d-010-status-new-revision.pdf)
- [Standard Norway D-010 2021 page](https://handle.standard.no/en/sectors/energi-og-klima/petroleum/norsok-standard-categories/d-drilling/d-0105/)
- [SPE on barrier schematics](https://jpt.spe.org/using-schematics-managing-well-barriers)
- [DNV-OS-E101 2009 PDF](https://rules.dnv.com/docs/pdf/dnvpm/codes/docs/2009-10/Os-E101.pdf)
- [DNVGL-OS-E101 2015 PDF](https://rd-engineering.co/wp-content/uploads/2019/04/DNVGL-OS-E101JULY2015.pdf)
- [DNV Condition Assessment Programme (CAP)](https://www.dnv.com/services/condition-assessment-programme-cap--17741/)
- [ABS MODU rules — USCG mirror](https://www.dco.uscg.mil/Portals/9/DCO%20Documents/5p/5ps/Alternate%20Compliance%20Program/absmodu.pdf)
- [ABS Classification of Drilling Systems Guide PDF](https://ww2.eagle.org/content/dam/eagle/rules-and-guides/current/offshore/57_Classification_of_Drilling_Systems_2021/cds-guide-feb21.pdf)
- [ABS MODU brochure](https://ww2.eagle.org/content/dam/eagle/publications/brochures/MODU2014.pdf)
- [IADC UK/Norway rules summary PDF](https://iadc.org/wp-content/uploads/2015/04/Rules-and-Regulations-for-UK-Norwegian-Waters.pdf)

### Field realities & devices
- [Maritime Executive — Hazardous Area Classification](https://maritime-executive.com/features/understanding-hazardous-area-classification-on-ships-and-offshore)
- [46 CFR 111.105](https://www.ecfr.gov/current/title-46/chapter-I/subchapter-J/part-111/subpart-111.105)
- [Plantfce — FPSO hazardous area classification](https://plantfce.com/blog/posts/fpso-hazardous-area-classification/)
- [i.safe MOBILE IS530.1 spec](https://www.isafe-mobile.com/en/products/products-zone-1/21/is5301)
- [Aegex 100M datasheet PDF](https://aegex.com/images/uploads/aegex100m/Aegex_100M_DataSheet.pdf)
- [ecom Tab-Ex 05 DZ1](https://www.ecom-ex.com/products/mobile-computing/tablets-android/tab-ex-05-dz1/)
- [Getac ATEX/IECEx certifications](https://www.getac.com/us/certifications/atex-iecex/)
- [Intrinsically Safe Store — 2025 ATEX Zone 1 tablet price guide](https://intrinsicallysafestore.com/blog/atex-zone-1-tablet-price/)
- [Speedify — Starlink on oil rigs](https://speedify.com/blog/enterprise-internet-bonding-software/can-i-use-starlink-internet-on-an-oil-rig-yes-with-speedify/)
- [Offshore Magazine — Satcom bandwidth](https://www.offshore-mag.com/home/article/16757572/for-oil-and-gas-satcom-users-how-much-bandwidth-is-enough)
- [Speedcast/COSL integration](https://www.speedcast.com/newsroom/press-releases/2025/speedcast-integrates-new-global-high-throughput-service-from-starlink-as-part-of-cosl-drillings-hybrid-solution/)
- [PMC heat strain study](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9877975/)
- [CDC/NIOSH hearing PDF](https://stacks.cdc.gov/view/cdc/9719/cdc_9719_DS1.pdf)
- [PMC offshore inspector hearing study](https://pmc.ncbi.nlm.nih.gov/articles/PMC5333758/)
- [PIP Global — capacitive gloves](https://us.pipglobal.com/en/about-us/news-and-events/?nID=112)
- [Industrial Cyber — Operation Epic Fury OT security](https://industrialcyber.co/news/oil-and-gas-operators-ramp-up-ot-security-spending-post-epic-fury-but-critical-detection-gap-persists/)
- [Westwood — Rig dayrate forecast](https://www.westwoodenergy.com/business-division/research/global-offshore-drilling-rig-dayrate-forecast)
- [DNV Permit-to-Work guide](https://www.dnv.com/article/a-complete-guide-to-permit-to-work-ptw-systems/)

### Competitors
- [SafetyCulture pricing](https://safetyculture.com/pricing)
- [TrueContext (ProntoForms) pricing](https://truecontext.com/pricing/)
- [Fulcrum pricing](https://www.fulcrumapp.com/pricing/)
- [Capterra iAuditor reviews](https://www.capterra.com/p/141080/iAuditor/reviews/)
- [G2 SafetyCulture pros and cons](https://www.g2.com/products/safetyculture-2025-01-20/reviews?qs=pros-and-cons)
- [BasinCheck — SafetyCulture alternative for O&G](https://basincheck.com/safetyculture-alternative)
- [Cenosco products](https://cenosco.com/products)
- [IBM Maximo for Oil & Gas](https://www.ibm.com/products/maximo/oil-gas)
- [ITQlick Maximo pricing](https://www.itqlick.com/ibm-maximo/pricing)
- [MaintainX — best maintenance software for upstream O&G](https://www.getmaintainx.com/blog/best-maintenance-software-for-upstream-oil-and-gas)
- [IFS Ultimo EAM](https://www.ifs.com/solutions/ifs-ultimo-eam)
- [Cyberhawk Oil, Gas & Marine](https://thecyberhawk.com/oil-gas-marine)
- [Flyability Oil & Gas Drones](https://www.flyability.com/oil-and-gas-drones)
- [Percepto AIM](https://percepto.co/aim/)
- [Skydio Oil & Gas](https://www.skydio.com/solutions/oil-and-gas)
- [Mistras OneSuite](https://www.mistrasgroup.com/how-we-help/data-management/onesuite/)
- [Eddyfi NDT software](https://www.eddyfi.com/en/product/eddyfi-technologies-nondestructive-testing-inspection-software)
- [ServiceMax FieldFX](https://appexchange.salesforce.com/appxListingDetail?listingId=a0N3000000B4IEpEAN)
- [Cognite Oil & Gas](https://www.cognite.com/en/industries/oil-and-gas)
- [Cognite DataOps scaling case study](https://www.cognite.com/en/resources/customer-stories/dataops-oil-gas-scaling)
- [Kongsberg Kognitwin renaming](https://kongsbergdigital.com/news/kongsberg-digitals-digital-twin-solutions-renamed-to-kognitwin)
- [Pason DataHub with Pason Live](https://pason.com/products/datahub-with-pason-live/)
- [H&P acquires DrillScan](https://drillingcontractor.org/helmerich-payne-expands-portfolio-of-drilling-optimization-software-and-capabilities-with-the-acquisition-of-drillscan-53325)
- [Nabors RigCLOUD for operators](https://www.nabors.com/for-operators/directional-automation/rigcloud/)
- [Quorum/Aucerna merger](https://www.businesswire.com/news/home/20210215005048/en/Thoma-Bravo-Owned-Quorum-Software-and-Aucerna-to-Merge-and-Acquire-TietoEVRYs-Oil-and-Gas-Software-Business)

### Reporting & deliverables
- [GPS rig acceptance surveys](https://gammaps.net/rig-acceptance.html)
- [Global Offshore Engineering rig condition surveys](http://www.goe-group.com/rig-condition-surveys/)
- [Applus+ Rig Inspection Audits & Commissioning](https://www.applus.com/us/en/what-we-do/service-sheet/rig-inspection-audits-and-rig-commissioning)
- [Applus+ rig audit white paper on Scribd](https://www.scribd.com/document/419970538/Rig-Audit)
- [Bureau Veritas Middle East technical rig audit](https://middle-east.bureauveritas.com/your-needs/inspection/technical-rig-audit)
- [Derrick Services UK rig condition surveys](https://www.derricksl.com/inspections/rig-condition-surveys-2/)
- [InspectionTrack — structuring an O&G inspection report](https://www.inspectionstrack.com/how-to-structure-a-detailed-oil-and-gas-inspection-report/)
- [FieldEquip oilfield inspection management](https://www.fieldequip.com/oilfield-inspection-management/)
- [FTQ360 — NIOSH-compliant rig inspection best practices](https://blog.ftq360.com/blog/8-best-practises-niosh-compliant-rig-inspections)
- [Matterport — O&G document management ($180k lost paper sheets)](https://matterport.com/blog/oil-gas-document-management)
- [World Pipelines — Corrosion mapping in action](https://www.worldpipelines.com/special-reports/20062024/corrosion-mapping-in-action/)
- [Mastt — Project RAG Status Dashboard](https://www.mastt.com/blogs/project-rag-status-dashboard)

### Rig systems / failure modes
- [Sovonex Electric & Mechanical Drilling Rigs](https://www.sovonex.com/drilling-equipment/api-land-drilling-rigs/electric-mechanical-drilling-rigs/)
- [Metadrill — 9 Common Drill Rig Failures](https://metadrill.org/blogs/9-most-common-drill-rig-failures-and-the-parts-that-fix-them)
- [Dynamox — Drilling rig failure modes](https://dynamox.net/en/blog/drilling-rig-failure-modes-and-solution-for-monitoring)
- [IADC Drawworks Operation and Maintenance Safety](https://iadc.org/safety-meeting-topics/drawworks-and-accessories-operation-and-maintenance-safety/)
- [IADC Drilling Line Care, Inspection and Replacement](https://iadc.org/safety-meeting-topics/drilling-line-care-inspection-and-replacement/)
- [Sinomechanical — Common faults of mud pumps](https://www.sinomechanical.com/news/Common-faults-of-mud-pumps.html)
- [Sinomechanical — Ram BOP Seal Failure](https://www.sinomechanical.com/news/Reasons-and-Solutions-for-Seal-Failure-of-Double-Ram-Blowout-Preventers.html)
- [Zeefax SCR Systems on Rigs](https://www.zeefax.com/more-about-scr-systems-simulation-and-training/)
- [OxMaint — CMMS for Oil & Gas](https://www.oxmaint.com/blog/post/blog-post-cmms-oil-gas-upstream-downstream-maintenance)
- [Saxon STAR Suite — Drilling EAM](https://saxonsoftware.com/star-suite/)

### AI feasibility
- [ImageVision.ai O&G Vision AI](https://imagevision.ai/blog/transforming-oil-and-gas-asset-management-with-drone-inspection-using-vision-ai/)
- [Commercial UAV News — GE/Avitas](https://www.commercialuavnews.com/infrastructure/ge-uses-drones-reduce-inspection-costs)
- [Nature npj Materials Degradation — Corrosion detection with confidence](https://www.nature.com/articles/s41529-022-00232-6)
- [Roboflow corrosion detection benchmark](https://blog.roboflow.com/corrosion-detection/)
- [DroneLife — Percepto EPA approval](https://dronelife.com/2025/10/29/autonomous-methane-detection-drones-earn-epa-approval-for-oil-and-gas-compliance/)
- [MLCommons Whisper benchmark](https://mlcommons.org/2025/09/whisper-inferencev5-1/)
- [Whisper-AT paper](https://arxiv.org/pdf/2307.03183)
- [Deepgram Nova-2](https://deepgram.com/learn/nova-2-speech-to-text-api)
- [Golden-Retriever RAG for industrial KB](https://arxiv.org/pdf/2408.00798)

### Procurement
- [Iterators — SOC 2 for enterprise procurement](https://www.iteratorshq.com/blog/soc2-compliance-for-saas-why-enterprise-customers-demand-it-and-how-to-get-certified/)
- [Workstreet — SOC 2 for startups](https://www.workstreet.com/blog/soc-2-for-startups)
- [Cognite Forrester $21.6M NPV study](https://www.cognite.com/en/company/newsroom/independent-study-finds-asset-heavy-organizations-can-expect-21.6m-in-value-after-adopting-cognite-data-fusion)
