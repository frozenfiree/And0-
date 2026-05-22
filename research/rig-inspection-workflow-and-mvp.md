# Rig Inspection Software — Workflow Extraction & Product Definition

> Companion document to `research/rig-inspection-software.md`. Where the prior brief surveyed the industry, this document compresses that research into an operational workflow archetype and a buildable MVP spec — data model, screen-by-screen mobile flow, report template architecture, controlled vocabulary, reliability requirements, and build sequence.
>
> **Methodology note.** The workflow below is reconstructed from industry research (third-party inspection firm methodologies, public sample reports, IADC/API/ABS documentation, and standard practice in rig condition audits). A 90-minute call with the actual target inspector — walking through their last three real inspections, screen by screen and folder by folder — will sharpen this further. Treat this document as a strong v1 hypothesis to react against, not as a faithful capture of one specific inspector's habits.

---

## Table of contents

- [A. Workflow Map](#a-workflow-map) — end-to-end lifecycle, phase by phase
- [B. Inspector Behaviour Model](#b-inspector-behaviour-model) — how the inspector thinks and moves
- [C. Workflow Compression Opportunities](#c-workflow-compression-opportunities)
- [D. Inspection Data Model](#d-inspection-data-model)
- [E. Mobile Workflow Design (screen-by-screen)](#e-mobile-workflow-design-screen-by-screen)
- [F. Report Template Architecture](#f-report-template-architecture)
- [G. Controlled Vocabulary & Standardisation](#g-controlled-vocabulary--standardisation)
- [H. Pain Point → Feature Mapping](#h-pain-point--feature-mapping)
- [I. Reliability Requirements](#i-reliability-requirements)
- [J. MVP Scope Definition](#j-mvp-scope-definition)
- [K. The Real Product Moat](#k-the-real-product-moat)
- [L. Product Strategy — why inspectors adopt and pay](#l-product-strategy--why-inspectors-adopt-and-pay)
- [M. Build Sequence](#m-build-sequence)

---

## A. Workflow Map

The lifecycle of a single 5-day rig condition / acceptance inspection that produces a ~150-page CEO-facing PDF. Times are realistic ranges, not averages. Each phase lists: **goal · tools · data created · what slows it down · what causes mistakes · what must be defensible later**.

### Phase 1 — Scope & contracting (T-21 to T-7 days)

- **Goal.** Agree what is being inspected, against which standards, with what deliverable.
- **Tools.** Email, Word SOW, Outlook calendar.
- **Data created.** SOW (rig name, type, dates, scope boundary, standards reference list, NDT extent, deliverable format), client contact list, day-rate / lump-sum quote.
- **Friction.** Scope creep ("can you also look at the cranes?"), standards mismatch (client wants IADC + API + their own corporate checklist), unclear "out of scope" boundary.
- **Mistakes.** Wrong rig type assumed (jackup vs semisub vs land), wrong client signatory, NDT extent under-quoted.
- **Defensibility.** The SOW is the contract. Every finding the inspector raises later must trace back to a scope item or an obvious safety issue.

### Phase 2 — Pre-inspection desktop review (T-14 to T-3 days)

- **Goal.** Walk on board already knowing the rig.
- **Tools.** PDF reader, Word, Excel, OneDrive/SharePoint, email.
- **Data consumed.** Prior inspection reports (often 2–3 years of history), CMMS extract (open work orders), BOP test charts, hoisting-equipment recert certificates (API 8B Cat III/IV), API 4F design data sheet for mast/substructure, NDT history (UT thickness maps, MPI reports), planned-maintenance system extract, crew certifications, SMS / HSE Case.
- **Data created.** Pre-inspection checklist (what to verify on-board), risk hotlist (what to look at first based on prior history), template selection (which inspection-form pack to bring).
- **Friction.** Documents arrive in 6 different formats from 4 different contacts. Some are missing. The "latest" version is unclear. Photos in prior reports aren't searchable. The CMMS extract is a 12,000-row Excel that nobody can usefully read.
- **Mistakes.** Reading the wrong prior report (last year's, not the most recent). Missing a flagged "watch item" because it was buried on page 87 of a 200-page PDF.
- **Defensibility.** Knowing what the prior inspector flagged is what makes the inspector look professional in the kick-off meeting.

### Phase 3 — Travel & site access (T-2 to T-0)

- **Goal.** Get on the rig with the right kit, the right certifications, and the right permits.
- **Tools.** Itinerary, BOSIET/HUET certificates, vendor-access forms, PPE bag.
- **Data created.** Travel receipts, time logs.
- **Friction.** Helicopter delays (offshore), BOSIET expiry surprise, vendor-access form not pre-submitted, PPE missing one item (e.g. H₂S monitor).
- **Mistakes.** Showing up at the heliport without the right vendor-access ID. Forgetting a charger / spare battery.
- **Defensibility.** Travel time and arrival timestamp end up in the report's "dates on board" section.

### Phase 4 — Safety induction & arrival (Day 1, morning)

- **Goal.** Be cleared to walk the rig.
- **Tools.** Induction form, PPE issue sheet, ID badge, permit-to-work system.
- **Data created.** Induction completion record, PTW number, JSA reference, toolbox-talk attendance log.
- **Friction.** Induction can take 2–4 hours. Permit office may not open until 07:00. JSA needs to be written before any inspection task.
- **Mistakes.** Trying to inspect before PTW is active. Stepping into Zone 1 with a non-IS phone.
- **Defensibility.** PTW number and induction record need to appear in the report metadata. "Inspector was inducted to site SMS X on date Y" is a credibility marker.

### Phase 5 — Kick-off meeting (Day 1, mid-morning)

- **Goal.** Agree the inspection plan with the OIM / Rig Manager and the client representative.
- **Tools.** Printed SOW, pen, notebook.
- **Data created.** Inspection schedule (which system on which day), SIMOPS conflicts list, escort assignments, document-retrieval list (asking the rig clerk for things the desktop review couldn't access).
- **Friction.** Rig is mid-trip; rig floor unavailable until afternoon. BOP test scheduled tomorrow — that's when the inspector should be in the BOP area, not before. Toolpusher off-rotation; deputy is new.
- **Mistakes.** Scheduling electrical inspection on a day with planned LOTO that doesn't happen.
- **Defensibility.** The kick-off establishes who said what; the inspector takes notes during this meeting because exit-meeting disputes start here.

### Phase 6 — On-site execution (Days 2–N)

This is the heart of the inspection. Sub-phases:

#### 6a. Orientation walk-around (Day 1 afternoon or Day 2 morning)

- **Goal.** Build a mental map of the rig.
- **Tools.** Hard hat, camera, notebook, GA drawing (often a folded paper printout).
- **Data created.** Wide-context photos (mast, rig floor, BOP area, mud room, accommodation, helideck for offshore) to ground later detail photos.
- **Friction.** Climbing the derrick is permitted-only; the inspector may have to defer the structural orientation until later.
- **Mistakes.** Taking 100 photos without notes; later, the inspector can't remember which deck level.

#### 6b. Discipline walkdowns (Days 2–N, parallel or serial)

Each discipline is its own walk-and-document session. Typical discipline split for a 5-day audit:

| Day | Disciplines |
|---|---|
| Day 2 | Structure (mast, derrick, substructure), hoisting equipment, drawworks |
| Day 3 | Top drive / rotary, mud pumps & mud system, cementing |
| Day 4 | BOP stack & well control, choke manifold, hydraulics |
| Day 5 | Electrical (gensets, SCR/VFD, MCCs, hazardous-area), safety/HSE, lifesaving (offshore), marine (offshore) |
| Day 5 evening | Exit meeting prep |

- **Goal per discipline.** Walk-through, look at every item on the discipline checklist, photograph defects, talk to the relevant crew member (rig mechanic for mechanical, RSTM for electrical, subsea engineer for BOP, etc.).
- **Tools.** Discipline checklist (paper), camera, notebook, sometimes UT thickness gauge, IR thermometer, multimeter, torque wrench.
- **Data created.** Per-item check (compliant/non-compliant), 1–10 photos per finding, hand-written notes, instrument readings (UT thickness, IR temp).
- **Friction.** Item is in use ("can't open the BOP control cabinet, we're closing in the well"), item is dirty (can't see the nameplate), crew member is on shift change, photos backlit / blurry, gloves on, mud everywhere.
- **Mistakes.** Forgetting to photograph the nameplate before walking away ("which serial number was that?"); forgetting which deck level the photo was taken on; capturing 7 photos of the same defect from slightly different angles; failing to take a scale-reference photo.
- **Defensibility.** Every finding needs ≥ 1 photo. Many findings need a measurement (crack length, corrosion area, thickness, torque value).

#### 6c. Function tests & pressure tests

- **Goal.** Watch the rig do something and confirm it works.
- **Tools.** Stopwatch, multimeter, gauge, pressure-test chart (sometimes printed in real time by the rig).
- **Data created.** Pass/fail outcome, observed value, witnessed-by signature.
- **Friction.** Tests are scheduled around well operations. BOP test may be 02:00. Crown-saver drop-test requires permission and a slow trip.
- **Mistakes.** Inspector doesn't witness the full duration of a test (e.g. BOP held for 5 minutes — inspector arrived at minute 3).
- **Defensibility.** The chart from the test is the evidence. If inspector wasn't there, finding is not signable.

#### 6d. Crew interviews

- **Goal.** Cross-check that documented procedures match real practice; surface known issues.
- **Tools.** Notebook.
- **Data created.** Quotes (with attribution), unrecorded watch items.
- **Friction.** The right person is off-rotation. The driller is busy. Interviews can't happen on shift change.
- **Mistakes.** Misattributing a quote.

### Phase 7 — Daily debrief (every evening, 2–5 hours)

The single biggest time sink in the current workflow.

- **Goal.** Convert the day's chaos into clean notes and photo sets.
- **Tools.** Laptop, USB-C cable, phone, Word, Excel, File Explorer.
- **Data created.** Per-day folder of renamed photos (`Day2_Structure_Mast_PinWear_001.jpg`), typed-up findings (Word doc), updated defect register (Excel).
- **Friction.**
  - Transfer 200–400 photos from phone (camera roll has no folders).
  - Rename each photo with a meaningful filename.
  - Sort into per-system folders.
  - Retype handwriting (some illegible after a day in oily gloves).
  - Build defect-register entries in Excel.
  - Cross-reference notes to photos (which photo is the swivel weep?).
  - Decide severity for each finding now that the rush is over.
  - Plan tomorrow.
- **Mistakes.** Photos lost in the transfer ("did I AirDrop those?"). Wrong photo attached to wrong finding. Severity drifts ("I called it Major at the time but I think it's a Minor now").
- **Defensibility.** This is where the report's bones are formed. Sloppy debrief → sloppy report.

### Phase 8 — Exit meeting (Last day, late afternoon)

- **Goal.** Present preliminary findings to the rig manager and client representative. Resolve disputes.
- **Tools.** Laptop, printed defect register, projector (if available).
- **Data created.** Updated defect register (some findings re-classified after rig pushback), dispensation requests, agreed close-out timelines.
- **Friction.** Inspector is exhausted by Day 5. Findings discussed without all the photos to hand. Critical findings can become heated.
- **Mistakes.** Capitulating to pushback on a finding that should hold. Accepting a verbal close-out that isn't documented.
- **Defensibility.** Every dispensation must be in writing and named. "Critical finding X was dispensed by client-rep Y in the presence of OIM Z on date W" is the only way to protect the inspector if a regulator follows up.

### Phase 9 — Demobilisation (post-exit)

- **Goal.** Get home with all data intact.
- **Tools.** Bag.
- **Data at risk.** Phone full of photos that haven't been backed up; notebook that can't be replaced.
- **Friction.** Helicopter weather. Lost luggage with paper notes.
- **Mistakes.** Phone wiped, notebook left in the seat pocket.
- **Defensibility.** The single biggest data-loss event in the entire lifecycle. Inspectors lose work here more often than they admit.

### Phase 10 — Report writing (T+3 to T+21 days)

The other half of the time cost.

- **Goal.** Produce a CEO-grade PDF.
- **Tools.** Word (template), Excel (defect register), Photoshop or Snipping Tool (photo annotation), Adobe Acrobat (PDF assembly), email.
- **Sub-tasks.**
  - Start from previous report's Word template (saves time but introduces stale content if not pruned).
  - Write executive summary.
  - Per-discipline narrative.
  - Build defect register as a Word table (Excel → paste → reformat). Manual.
  - Lay out photo plates (2-up or 4-up grids in Word). Excruciating.
  - Caption each photo with finding ID.
  - Annotate photos (arrows on defects). Done in a separate tool, then re-imported.
  - Cross-reference clauses (API RP 4G §6.x). Often dropped because tedious.
  - Run spell-check; check terminology consistency.
  - Internal peer review (in a small firm, often skipped).
  - PDF export.
  - Sign (sometimes wet signature scanned in; sometimes digital).
  - Compress photos because the PDF is 80 MB.
- **Friction.** Word table reformatting eats hours. Photo plate layout breaks every time a photo is moved. The "Final_v3_FINAL_reviewedSB_v2.docx" filename anti-pattern. Inspector forgets which folder has the latest version. Photo compression destroys defect detail.
- **Mistakes.** Wrong photo on a finding (the worst kind of mistake — turns a credible report into a joke). Stale text from previous report ("the BOP was last tested on 2023-04-12" — true for the previous rig, not this one). Inconsistent severity vocabulary between sections.
- **Defensibility.** Every figure number cited in the text must exist. Every finding must have a photo or a recorded measurement. The signature page must list inspector name + qualifications + date.

### Phase 11 — Delivery (T+21)

- **Goal.** Ship the report.
- **Tools.** Email, USB, sometimes SharePoint, sometimes operator portal upload.
- **Data created.** Sent email log, delivery acknowledgement.
- **Friction.** PDF too large for email; needs file-share link. Operator portal has a 50 MB upload limit. Wrong distribution list.
- **Mistakes.** Sending to the wrong client contact. Sending the draft instead of the final.
- **Defensibility.** A delivery log (date sent, to whom, file hash) is what protects the inspector if the client later says "we never got it."

### Phase 12 — Close-out (T+30 to T+90)

- **Goal.** Verify the client has actioned critical findings.
- **Tools.** Email, sometimes the client's closure-evidence portal.
- **Data created.** Closure evidence (photos of repair, work order numbers, NDT re-test reports), updated finding status (open → closed → verified).
- **Friction.** Closure evidence trickles in. Some never comes. Some is for the wrong finding ID. Some critical items remain open at the next inspection.
- **Mistakes.** Marking a finding closed without verifying evidence.
- **Defensibility.** Open-finding history compounds. The next inspection (next year) inherits unclosed items.

### Workflow at a glance

```
T-21d   T-14d   T-2d   D1    D2   D3   D4   D5   T+3d   T+10d  T+21d  T+30-90d
  │       │       │     │     │    │    │    │      │      │      │      │
SCOPE → DESKTOP → TRAVEL → INDUCT → ── EXECUTION (with daily debrief) ── → EXIT → DEMOB → REPORT WRITING → DELIVER → CLOSE-OUT
                                                                           │      │
                                                                           │      └─ Data-loss risk
                                                                           └─ The exit meeting decides what survives into the report
```

Two visible time sinks dominate:
1. **Daily debrief (2–5 h × N days = 10–25 h).** Notebook + camera roll → Word + Excel.
2. **Report writing (12–25 h).** Word table layout, photo plates, narrative, captioning, severity normalisation.

These two together are **22–50 hours per inspection** — almost a second inspection's worth of effort, and both are software-addressable without any AI.

---

## B. Inspector Behaviour Model

How the inspector actually thinks and moves. Useful for design choices because a "logical" UI that doesn't match this gets abandoned.

### B.1 How they organise findings mentally

- **By system, then by location, then by severity.** Walking the mud system in order: shakers → desander → desilter → mud pits → pumps → discharge piping. The order is physical proximity, not a checklist sequence. Software that forces strict sequential checklist order fights this.
- **Critical findings get a separate mental bucket.** Inspectors maintain a small in-head list of "the 5 things the CEO must know" the entire inspection. By Day 5, this list has 8–12 items and they prune to 5–10 for the exec summary.
- **"Watch items" — neither critical nor minor.** Things that aren't actionable now but the inspector wants to remember. In paper workflow these end up in marginalia. In software, this needs to be a first-class severity tier (Observation).

### B.2 How they prioritise issues

The implicit decision tree:
1. Is anyone in immediate danger? → Stop work, escalate to OIM.
2. Would this stop drilling if discovered tomorrow? → Critical.
3. Will this be a problem within 30 days? → Major.
4. Will this be a problem within 12 months? → Minor.
5. Is this housekeeping / best practice? → Observation.

Severity is judged with reference to: physical condition × consequence-if-it-fails × time-to-failure × ease-of-repair. None of this is written down anywhere — it lives in the inspector's experience. Software should make this judgment visible (severity picker with rationale field) without trying to automate it.

### B.3 How they move through the rig

- **Sweep + dive.** Walk through a system area once at orientation speed; come back to specific items. This means findings get logged in a different order than the checklist suggests.
- **Tag-along to crew tasks.** "The mud engineer is dumping the desander now — I'll walk that with him." Inspector schedule bends to operational opportunities.
- **Climb once, look at everything aloft.** Going up the mast is a permit + harness event. The inspector does the entire derrick inspection in one climb, even if it crosses three checklist sections.
- **Cellar / sub-base last.** Confined-space entry needs gas-test and standby person. Often done late in the day.

Implication: the mobile app must not enforce sequential checklist completion. Inspectors will skip ahead, jump between systems, and come back. The data model must be unordered.

### B.4 How they capture evidence

Three habits, near-universal:

1. **Photo before note.** Camera comes out first because the rig will move on. Note is written later, sometimes minutes later, sometimes that evening.
2. **Photo set per finding.** Wide context → medium → close-up of defect → scale reference (a glove or ruler in frame). Usually 3–5 photos per significant finding.
3. **Nameplate photo as fact-capture.** Make / model / serial / hours from the nameplate is photographed even when nothing is wrong, because it might be needed later for cross-reference.

Implication: photos and findings have a many-to-one relationship; the app must support photo-bursts attached to one finding; OCR of nameplates is a high-ROI feature.

### B.5 How they remember follow-ups

Inspector heuristics:
- A folded corner of a notebook page = "come back to this."
- A finger-jab in the notebook + a photo = "ask the toolpusher about this when I see him."
- A specific photo with a scribbled "?" annotation = "I don't know what this part is."
- A mental flag like "I need to see the BOP test before I close the well-control section."

In software this needs to map to:
- **"Needs follow-up" flag** on a finding (separate from severity).
- **"Open question"** type of note (different from an observation).
- **"Awaiting witness"** state on a function-test finding.

### B.6 How they handle interruptions

Interruptions are constant on a rig: crew change, weather delay, sudden well event, helicopter arrival, emergency drill. Inspector strategies:
- **Park task at any granularity.** Often mid-photo, mid-sentence in a note.
- **Resume cold.** Sometimes hours later. Sometimes the next day.

Implication: every screen must auto-save on every keystroke. "Resume where I left off" is the launch state. There is no "save" button worth designing — saving is implicit and continuous.

### B.7 Where the senior-inspector instinct lives

Two places software cannot replicate:
- **"That doesn't sound right."** Vibration, noise, smell, tank levels off — the inspector notices things the checklist doesn't list. Software should give them an "ad-hoc finding" path that doesn't require selecting a component from a tree (or that lets them snap a photo, log it, and link the component later).
- **"That's what we said last year."** Recognising a repeat finding by recall. Software can support this once a rig has a history (historical-comparison view), and it becomes a quiet but powerful retention feature.

### B.8 Reporting fatigue

By Day 5 of the inspection, the inspector has:
- ~80–150 findings
- ~300–600 photos
- 30–60 pages of notes
- A list of disputes from the exit meeting
- Travel home ahead of them

Then they need to write a 150-page report. The first draft is hardest because it's a blank Word document. Any tool that lets them open the laptop and see "your report is 70% drafted" instead of a blank page changes the psychological economics. This is the single largest emotional driver of adoption.

---

## C. Workflow Compression Opportunities

What can be compressed, by how much, and how.

### C.1 Daily debrief: from 2–5 h to 15–30 min

Today: photo transfer + rename + sort + retype notes + build register.

Compressed: photos already linked to findings at capture; notes already typed (or dictated) into the finding; severity already set; register auto-builds from the structured findings.

What's left in the evening: triage unlinked photos (5 min), review today's findings list (10 min), plan tomorrow (5 min). **Saved: ~2–4.5 h/day × 5 days = 10–22 h per inspection.**

### C.2 Report writing: from 12–25 h to 2–5 h

Today: write exec summary + build register table + lay out photo plates + write narrative + annotate photos + cross-reference clauses.

Compressed: register is rendered from data; photo plates auto-laid (4-up grid per system); clause citations attached at finding time; narrative drafted by LLM from structured findings (inspector edits); exec summary drafted by LLM from severity counts and top critical findings (inspector edits).

What's left: review and edit (LLM draft is never final), executive judgement calls, sign-off. **Saved: ~10–20 h per inspection.**

### C.3 Photo organisation: from 1–2 h to 0 h

Today: rename and sort folders.

Compressed: photos are born inside a finding context with timestamp, GPS, and inspector ID. Folder structure ceases to exist as a user-visible concept.

**Saved: 1–2 h per inspection, with the bonus that "which photo belongs to what finding" becomes impossible to get wrong.**

### C.4 Severity normalisation: from 1 h to 5 min

Today: inspector revisits the day's findings and adjusts severity post-hoc (which causes inconsistency across the inspection).

Compressed: severity is selected at finding-creation time from a structured picker with definitions visible. Rule-based defaults (e.g. "weld crack on load-bearing structure → Critical") prefill the picker; inspector can override.

**Saved: 1 h, but the larger win is consistency across the inspection.**

### C.5 Clause citation: from "we usually skip it" to automatic

Today: inspector mentions API RP 4G when they remember, drops it when they don't. Reports vary in regulatory defensibility.

Compressed: every finding ties to a component, every component has default applicable clauses (RP 4G for derrick, RP 8B for hoisting, Std 53 for BOP, etc.), the picker offers the right clause first.

**Saved: minutes per inspection, but the report quality improves measurably and the firm-level differentiator grows.**

### C.6 Aggregate

| Compression | Hours saved per inspection |
|---|---|
| Daily debrief | 10–22 |
| Report writing | 10–20 |
| Photo organisation | 1–2 |
| Severity normalisation | 1 |
| Clause citation | (quality, not time) |
| **Total** | **22–45 h** |

Twenty-two hours is a credible minimum even before LLM-assisted writing. Forty-five hours is the realistic upper end with LLM assistance and after the inspector has built up a findings library.

---

## D. Inspection Data Model

The minimum structured model needed to support capture, severity logic, and report generation. Optimised for one-inspector use; designed so multi-inspector and multi-tenant fit cleanly later without re-shaping core entities.

### D.1 Entities

```
Client
  id, name, type[operator|drilling_contractor|owner|investor|insurer|builder]
  primary_contact_name, primary_contact_email, brand_assets_ref

Rig
  id, name, type[land|jackup|semisub|drillship|workover]
  imo_number_nullable, reg_id_nullable, owner_client_id, builder, year_built
  notes_md

Inspection
  id, rig_id, client_id, inspector_id, archetype[walkdown|audit|due_diligence]
  scope_md, standards_refs[array of StandardClause.id]
  ptw_number_nullable, jsa_ref_nullable, toolbox_talk_attendees_md
  start_date_local, end_date_local, timezone
  status[draft|in_field|report_writing|delivered|closed]
  prior_inspection_id_nullable  // for repeat-rig context

Component
  id, rig_id, parent_component_id_nullable
  code, name, system[bop|derrick|drawworks|topdrive|mud|electrical|safety|...]
  manufacturer_nullable, model_nullable, serial_nullable
  year_installed_nullable, last_recert_date_nullable
  recert_interval_days_nullable, operating_hours_at_install_nullable
  notes_md
  // Recursive; pre-populated from rig-type template; inspector can add ad hoc.

Finding
  id, inspection_id, component_id_nullable  // nullable for ad-hoc findings
  title  // short, one-line
  observation_md  // what was seen
  recommendation_md  // what to do
  cause_md_nullable
  severity[critical|major|minor|observation]
  recommended_action[shutdown|repair_before_drill|monitor|housekeeping]
  risk_likelihood_1to5_nullable, risk_consequence_1to5_nullable  // 5x5 matrix
  drops_band_nullable[slight|minor|major|fatality]  // for at-height findings
  follow_up_flag_bool  // separate from severity; "I need to come back to this"
  target_close_out_date_nullable
  responsible_party_nullable
  status[open|in_progress|closed|verified|dispensed]
  inherited_from_finding_id_nullable  // for repeat findings across inspections
  created_at_utc, created_at_local
  signed_by_inspector_id_nullable, signed_at_utc_nullable

FindingClauseLink
  finding_id, clause_id

StandardClause
  id, source[api_rp_4g|api_rp_8b|api_std_53|api_spec_16a|iadc_checklist|
            osha_1910|osha_1926|bsee_30cfr250|norsok_d010|dnv_os_e101|abs_modu|
            client_spec]
  source_version, clause_ref, summary_md, full_text_ref_nullable
  // We store the citation taxonomy; we do NOT redistribute copyrighted text.

Photo
  id, inspection_id, finding_id_nullable
  captured_at_utc, captured_at_local
  geo_lat_nullable, geo_lon_nullable, exif_json
  file_ref  // path to original
  thumbnail_ref
  annotated_file_ref_nullable  // markup layer baked in for the report
  caption, plate_number_nullable  // assigned at report-build time
  inspector_id
  ocr_text_nullable  // for nameplate photos

Measurement
  id, finding_id_nullable, component_id, type[ut_thickness|hardness|torque|
                                              insulation_resistance|temperature|
                                              dimension|pressure]
  value, unit, instrument_serial_nullable
  taken_at_utc, taken_by_inspector_id

FunctionTest
  id, inspection_id, component_id, test_type[bop_pressure|crown_saver_drop|
                                             insulation_resistance|lifeboat_fall|
                                             firewater_pump|esd|...]
  result[pass|fail|not_witnessed]
  observed_value_nullable, expected_value_nullable
  witnessed_at_utc, witnessed_by_inspector_id
  witnessed_by_rig_personnel_md  // free-text, with role
  chart_attachment_ref_nullable

Note
  id, inspection_id, kind[interview|observation|open_question|todo]
  body_md, attributed_to_role_nullable, attributed_to_name_nullable
  created_at_utc, voice_clip_ref_nullable, voice_transcript_nullable

Recommendation  // a structured recommendation, separable from finding text
  id, finding_id, action_md, owner_role, due_date_nullable
  est_cost_nullable, est_downtime_days_nullable
  status[open|in_progress|closed|verified]

Report
  id, inspection_id, version, generated_at_utc, generated_by_inspector_id
  pdf_ref, archetype[walkdown|audit|due_diligence]
  exec_summary_md_draft, exec_summary_md_final  // separate so the LLM draft is preserved
  is_signed_bool, signature_image_ref_nullable
  delivered_at_utc_nullable, delivery_log_md

InspectorCertification
  id, inspector_id
  scheme[asnt_snt_tc_1a|asnt_cp_189|api_4g|api_8b|iadc_rigpass|bosiet|huet|nace_cip]
  method_nullable[vt|mt|pt|ut|rt|et]
  level_nullable[i|ii|iii]
  certificate_number, issued_at, expires_at
  scan_ref  // PDF of the cert

InstrumentCalibration
  id, instrument_serial, instrument_type
  calibrated_at, expires_at, traceability_authority
  cert_ref  // PDF of the cal cert

ChangeLog  // for audit trail / defensibility
  id, entity_type, entity_id, field, old_value, new_value
  changed_at_utc, changed_by_inspector_id
```

### D.2 Relationships at a glance

```
Client 1 ── n Rig
Rig 1 ── n Inspection
Rig 1 ── n Component (tree)
Inspection 1 ── n Finding 1 ── n Photo
                              ── n Recommendation
                              ── n Measurement
                              ── n FindingClauseLink ── 1 StandardClause
Inspection 1 ── n FunctionTest
Inspection 1 ── n Note
Inspection 1 ── 1..n Report (versioned)
Inspector 1 ── n InspectorCertification
Inspector / Instrument ── n calibration certs
```

### D.3 Status models

- **Inspection.status:** `draft → in_field → report_writing → delivered → closed`. Each transition gates UI features (e.g. you can't `delivered` until at least one `Report` exists; you can't `closed` until all `critical` findings are `closed|verified|dispensed`).
- **Finding.status:** `open → in_progress → closed → verified`, with a parallel `dispensed` state for items waived by client. `inherited_from_finding_id` carries the closure history across inspections.
- **Report.is_signed:** false until the inspector explicitly signs; after signing, edits create a new `version`, never overwrite.

### D.4 Severity logic

Severity is a **controlled vocabulary with a default-suggester**, not a calculator:

```
default_severity(component.system, finding.observation_keywords) →
  if component.system == 'bop' and keywords ⊂ {ram_seal, leak, fail_test}: critical
  if component.system == 'derrick' and keywords ⊂ {weld_crack, load_bearing}: critical
  if component.system == 'electrical' and keywords ⊂ {insulation_breakdown}: critical
  ...
  default: (inspector_must_pick)
```

The default is a *suggestion*, not a binding rule. The inspector always has the final say. The system records both the suggested and chosen severity so quality drift can be reviewed later.

### D.5 What's deliberately NOT in the data model

- **Users / roles / teams.** Not needed for one-inspector v1. Add a `team_id` column on every multi-tenant entity later — clean migration.
- **Permissions / sharing model.** Same.
- **Custom-fields engine.** Generic "form builder" plumbing kills the product's focus and turns it into SafetyCulture. Inspectors should ask us for new fields; we add them with a release.
- **AI inference table.** When AI runs, it writes drafts into existing `_draft` fields. No separate "AI outputs" entity.
- **Webhooks / integration layer.** Not until the first firm or contractor asks.
- **Internationalisation.** English only.

---

## E. Mobile Workflow Design (screen-by-screen)

Built for **one-handed, gloved, sun-bright, dirty, interrupted** use. Every screen passes the test: "if the inspector is hanging off a handrail with one thumb available, can they still use this?"

### E.1 Screen 1 — Inspection home

**When you tap the app icon.** Cold launch ≤ 2 s.

- **Top.** Inspection name + rig name (large). Day X of Y. Days remaining.
- **Body.** "Today's plan" — the inspector's freeform plan for the day. Editable in place.
- **Centre.** System grid (large tiles): Derrick · Drawworks · Top Drive · Mud · BOP · Electrical · Safety · Marine (for offshore). Each tile shows finding count + colour stripe (red/amber/green based on findings logged so far in that system).
- **Bottom.** Big primary button: **"+ Quick finding"** (component picker happens later). Secondary: **"Camera"** (free-shoot mode for things you don't know where to put yet). Tertiary: **"Photos to triage (12)"** badge.

Design rules:
- No top-nav hamburger menu. No login on this screen — already authenticated.
- The system grid is the inspector's mental map of the rig. Tap one to enter the component tree.
- The primary "Quick finding" button bypasses the component tree for situations where the inspector knows the finding before they know where to file it.

### E.2 Screen 2 — Component browser (per system)

**Entered by tapping a system tile.**

- **Top.** Breadcrumb (Rig > System).
- **Body.** Component tree, expandable. Each row: component name + finding count + last-checked timestamp + small severity stripe.
- **Tap behaviour.** Single tap → expand. Long-press → "+ Finding for this component."
- **Bottom.** Filter chips: Open · Critical · Major · Not yet checked.

Design rules:
- The component tree is pre-populated from the rig-type template; inspector can add ad-hoc components (some rigs have one-off modifications). Added components are flagged so the template can be improved over time.
- Each row needs to be tappable with a glove — min 12 mm tall, padding generous.
- Tree state persists (last expansion remembered).

### E.3 Screen 3 — Finding editor

**Entered from "+ Finding" anywhere.**

A single scrollable screen with everything in one place — not a wizard. Wizards force ordering the inspector doesn't have.

Layout, top to bottom:

1. **Component.** Pre-selected if entered from component browser; otherwise a "Pick component" chip the inspector taps later. May be left blank.
2. **Title.** One short line. Required.
3. **Severity picker.** Four big chips: Critical / Major / Minor / Observation. Default is whatever the rule-engine suggests; visually marked as "suggested" so the inspector knows to confirm.
4. **Photo strip.** Horizontal scroll of photos already attached. Big "+" tile at the end opens the camera.
5. **Observation.** Multi-line text. Push-to-talk mic button to the right.
6. **Recommendation.** Multi-line text. Library-suggest dropdown ("similar findings have recommended…").
7. **Reference clause picker.** Default clause prefilled from component (e.g. "API RP 4G §6.4" for a derrick weld). Tap to change.
8. **More fields (collapsed).** Recommended action · Risk score (5×5) · DROPS band (if at-height) · Target close-out · Responsible party · Inherited from? · Follow-up flag.
9. **Sign off.** "Mark complete" button. Pre-sign state is "draft"; signed state is "complete."

Design rules:
- Auto-save on every keystroke. No "Save" button.
- The screen never blocks. If the camera is opened mid-typing, the typed text persists.
- Reference-clause picker is the most-skipped field in current workflows. Defaulting it makes citation costless.

### E.4 Screen 4 — Camera

**Entered from photo strip in finding, from system tile, or from home.**

- **Camera viewfinder full-screen.** Big shutter button bottom-centre, thumb-reachable.
- **Top-left.** Currently-attached-to indicator: "Attaching to: BOP Ram Block weld crack" or "No finding (will triage later)."
- **Top-right.** Burst-mode toggle.
- **Bottom-left.** Last photo thumbnail (tap to review).
- **Bottom-right.** Switch to markup mode on the just-taken photo (arrows, circles, dimension lines).
- **Behaviour.** If a finding was opened ≤ 5 min ago, photos auto-link to it. Otherwise they go to the triage queue. The 5-min window is configurable.

Design rules:
- Save full-resolution to local disk *before* the camera UI confirms. If the app is killed mid-save, the photo survives.
- Markup is non-destructive: original always preserved, annotated copy saved alongside.

### E.5 Screen 5 — Photo triage

**Entered from the "Photos to triage" badge on home.**

- **Grid of unlinked photos**, biggest tiles, with capture time and rough location ("BOP area, 14:32, today").
- **Tap a photo** → full-screen with three actions: **Link to finding** (opens a finding picker filtered by recent), **Link to component** (creates a draft finding under that component), **Discard**.
- **Multi-select** for batch link (long-press first photo, tap others, then "Link all to…").

Design rules:
- The badge count must be visible from the home screen because the most common failure mode is photos drifting unattached.
- A photo with no link after 24 h shows up in a "stale photos" view to nudge cleanup.

### E.6 Screen 6 — Findings register

**Entered from a "Findings" tab on the home dock.**

- **Header.** Counts by severity (Critical 4 · Major 11 · Minor 23 · Obs 17).
- **List.** One row per finding: title, system, severity stripe, photo count, status indicator. Tappable to open the editor.
- **Filters.** Severity, system, status, follow-up flag, has-photo.
- **Sort.** By severity desc default; alternative: by created-time, by component.
- **Bottom-right floating button.** "Preview report PDF."

Design rules:
- The preview is rendered server-side when online, locally (if feasible) when offline. Rendering 150 pages takes seconds, not minutes.
- The preview is the inspector's primary feedback mechanism — they will hit it dozens of times across an inspection.

### E.7 Screen 7 — Sync & data status

**Always reachable from a small icon in the home header.**

- **State chip.** Online / Offline / Syncing / Stalled.
- **Counts.** N findings, M photos, K measurements pending upload.
- **Last-synced timestamp.**
- **Per-photo upload progress** (collapsed list).
- **Retry now** button.
- **Export everything** button — produces a zip of PDF + JSON + photos. Always available, including offline.

Design rules:
- Sync state must never be silent. An inspector who finds out at the airport that yesterday's findings didn't sync will never trust the tool again.
- "Export everything" is the trust feature. It is not a feature requested in user research; it is a feature that wins the inspector before they even commit.

### E.8 Screen interactions — what is NOT in v1

- No team / collaboration screens.
- No client / portal-handoff screens.
- No analytics / dashboards.
- No template editor (templates curated by us; "add this component" is the only customisation).
- No settings page beyond inspector profile + sign-in to sync server.

---

## F. Report Template Architecture

The report is a deterministic render from the data model. It must match a professional inspector's current Word output well enough that the first PDF is at least as good as the inspector's existing template.

### F.1 Three archetypes, one engine

Three report shapes share the same data model and the same component templates. Selection happens on `Report.archetype`.

| Archetype | Pages | Sections present | LLM use |
|---|---|---|---|
| **Walkdown** | ~10–30 | Cover, exec summary (½ page), findings register, photo plates, sign-off | Minimal (caption polish only) |
| **Audit** | ~100–300 | All sections | Exec summary + per-discipline narratives (draft) |
| **Due-diligence** | ~50–150 | All audit sections + capex backlog + valuation summary | Audit sections + capex narrative |

### F.2 Section-by-section template (audit archetype, full)

#### 1. Cover & control page

- Rig name, type, IMO/Reg number (if offshore), owner.
- Client name + logo (uploaded asset).
- Inspection dates (on-board → off-board) with time zone.
- Document revision, distribution list.
- Confidentiality notice.
- Inspector name + qualifications.
- Report version + generation timestamp.

**Auto / Manual:** 100 % auto from `Inspection` + `Client` + `InspectorCertification`. Inspector edits the "purpose" line only.

#### 2. Executive summary (≤ 2 pages prose + 1 page dashboard)

- **Scope sentence.** Auto-generated from `Inspection.scope_md` + `standards_refs`.
- **Methodology summary.** 3–4 lines.
- **Headline rating.** Generated from severity counts + per-system RAG roll-up.
- **System-level RAG dashboard.** 12–20 row table, one row per `Component.system`, status colour from worst finding severity in that system.
- **Top critical findings.** Top 5–10 findings sorted by severity, then by recommended action. Each line: title · system · severity · recommended action · estimated close-out time.
- **Go / no-go statement.** Inspector-written (LLM-suggested with disclaimer).
- **Key recommendations.** Binned short-term (< 30 days) / medium (< 12 months) / long-term (> 12 months).

**Auto / Manual:** Scope, RAG dashboard, top findings list, severity counts — all auto. Methodology summary and go/no-go are LLM-drafted, inspector-reviewed. Inspector retains final edit authority.

#### 3. Table of contents + acronyms

**Auto.**

#### 4. Scope of work as agreed

**Auto from `Inspection.scope_md`.** Renders the SOW that was agreed pre-mob.

#### 5. Methodology

- Standards referenced.
- Inspection categories applied (e.g. "API RP 4G Cat III applied to derrick; API RP 8B Cat III applied to hoisting").
- NDT methods used + scope sampling rationale.
- Exclusions ("electrical Ex-zone inspection deferred to next visit per scope").

**Auto from `Inspection.standards_refs` and the curated standards library.** Inspector edits narrative.

#### 6. Rig description

- Type, build year, builder.
- Major systems present (auto from `Component` tree).
- Operating history summary (auto if prior inspection data exists).

**Auto from `Rig` + `Component` aggregates.**

#### 7. System-by-system findings (the heart of the audit)

For each `Component.system`:
- Condition narrative (LLM draft, inspector edits).
- Findings table (filtered to this system from the master register).
- Embedded photo plates per significant finding.
- Sub-system breakdowns.

**Auto:** the findings table, the photo embeds, the per-finding clause citations.
**LLM draft, inspector edits:** narrative paragraphs.

#### 8. Consolidated defect register

- The master table.
- Columns: ID, system, component, tag/serial, photo refs (plate numbers), finding, severity, risk score, recommendation, target close-out, responsible party, status, reference clause.
- Sortable by severity for the PDF version; included as Excel sidecar in the delivery bundle.

**Auto.**

#### 9. NDT reports appendix

- For each NDT method used: procedure ref, technician name + cert ID + level, calibration block IDs, environmental conditions, readings table, indication map (if applicable), accept/reject vs standard.

**Auto from `Measurement` + `InspectorCertification` + `InstrumentCalibration`.**

#### 10. Certificates & class status appendix

- ABS/DNV/LR/BV class certificate (if available, uploaded).
- BOP test chart (most recent).
- Mast cert.
- Lift certs.

**Inspector uploads; auto-paginated.**

#### 11. Photo log

- Every photo numbered "Plate X-Y," captioned (auto-generated from finding context + inspector edit), cross-referenced to finding ID.
- Grouped by section.

**Auto, with inspector caption pass.**

#### 12. Recommendations / road-map

- All recommendations sorted by `target_close_out_date`, grouped by severity.
- Capex backlog roll-up (due-diligence archetype only).

**Auto.**

#### 13. Sign-off page

- Inspector name + qualifications + signature.
- Date.
- Document hash for integrity.

**Auto + inspector signature.**

### F.3 Photo plate layout

- **One-up.** Single critical finding photo, half-page, with caption block and finding ID.
- **Two-up.** Before/after comparison, side-by-side.
- **Four-up.** Wide → medium → close-up → scale-reference for a single finding.
- **Six-up grid.** Bulk photo log appendix.

Layout is deterministic — there is no inspector dragging photos around. Plate numbering is continuous through the report, assigned at render time.

### F.4 Multi-format deliverable bundle

For each report version, the system produces:

1. **PDF/A.** Signed, archival, the headline deliverable.
2. **Excel defect register.** Stable column schema, one row per finding. The operational artefact for the rig owner's planners.
3. **Photo sidecar zip.** Full-resolution, EXIF-intact, geo-tagged, named by finding ID + plate number.
4. **JSON export of the inspection.** Complete data dump for archival and re-import. The "you can always leave" guarantee.
5. **Optional EAM-ingest CSV.** Mapped to common Maximo / SAP PM column conventions (v1.5+).

### F.5 What is intentionally NOT in the report engine in v1

- Custom template editor (templates are curated, not user-editable).
- Multi-language report rendering.
- White-label theming (other than client logo on the cover).
- Branded watermarking.
- Live web-portal version (the PDF is the deliverable).
- Print-shop layout / bound-hardcopy automation.

---

## G. Controlled Vocabulary & Standardisation

Inconsistent vocabulary is the second-largest source of report quality problems (after missing photos). Standardising language at capture time eliminates a whole class of report cleanup.

### G.1 Severity vocabulary

| Label | Internal value | Meaning | Default action |
|---|---|---|---|
| **Critical** | `critical` | Equipment unsafe; well operations cease or do not start | Shutdown / no-start |
| **Major** | `major` | Repairable in days; operations may not start until closed | Repair before drill / pre-spud |
| **Minor** | `minor` | Add to backlog; track for trend | Monitor / next opportunity |
| **Observation** | `observation` | Best-practice opportunity, no immediate risk | Housekeeping |

These four are the canonical set. A/B/C, 1–5 condition rating, 5×5 risk score, and DROPS bands are **additional taxonomies stored alongside**, not replacements. The report renders whichever the client requires; the inspector always logs at least the canonical Critical/Major/Minor/Observation.

### G.2 Recommended action vocabulary

| Label | Internal value | Meaning |
|---|---|---|
| **Shutdown** | `shutdown` | Cease operations now |
| **Repair before drill** | `repair_before_drill` | Must be remediated before spud / resumption |
| **Monitor** | `monitor` | Track; address at next planned opportunity |
| **Housekeeping** | `housekeeping` | Document; remediate when convenient |

### G.3 Status vocabulary (Finding)

`open → in_progress → closed → verified`, with `dispensed` as a parallel state for items waived by the client (always with a named approver and timestamp).

### G.4 Component naming conventions

Pre-populated component templates use a canonical naming taxonomy. Examples (land rig, abbreviated):

```
Derrick & substructure
  Mast
    Mast pin (port)
    Mast pin (starboard)
    Mast leg (front-left)
    ...
    Crown block beam
    Crown sheaves (1 through N)
  Substructure
    Box-on-box
    Setback area
    V-door
  ...

Drawworks
  Drum
  Brake bands
  Brake calipers
  Brake cooling system
  Clutch
  Gearbox
  Crown-saver
  ...

Top drive
  Main bearing
  Washpipe assembly
  Gooseneck
  Motor (drive)
  Gearbox
  IBOP (upper / lower)
  Remote IBOP
  Slip ring
  ...

BOP stack
  Annular preventer
  Ram preventer (upper pipe)
  Ram preventer (lower pipe)
  Ram preventer (blind-shear)
  Choke outlet
  Kill outlet
  Hydraulic control system
    Accumulator bottles (1 through N)
    Pumps (electric / pneumatic)
    SPM valves
  Driller's panel
  Remote panel
```

Inspectors can add ad-hoc components; ad-hoc additions are flagged for template curation review.

### G.5 Recommendation phrasing library

A library of standard recommendation phrasings keyed to component × severity. Examples:

- **Component: derrick weld · Severity: critical**
  - "Cease operations and perform full NDT of the affected joint per API RP 4G Cat IV before resumption."
  - "Engage certified API 4F manufacturer's representative for structural assessment and repair specification."
- **Component: BOP ram seal · Severity: critical**
  - "Replace ram seal and re-test per API Std 53 §6 before any further well operations."
- **Component: drawworks brake band · Severity: major**
  - "Replace brake band before next spud per OEM service manual; re-test brake holding capacity."

Inspectors edit these to fit specifics; the library accelerates the first draft and enforces terminology consistency.

### G.6 Observation language standards

Observation prose follows three rules embedded in the editor as hints:

1. **Factual.** "Crack observed at upper-port mast pin, approximately 25 mm long, oriented vertically."
2. **Measured where possible.** Length, depth, area, thickness, torque — numbers if any are available.
3. **Causally agnostic.** Observations do not assign cause unless cause is directly known. Cause goes in the separate `cause_md` field.

### G.7 Standards-clause reference taxonomy

The clause picker is populated from the curated `StandardClause` table. The full text of API/IADC/OSHA standards is **not redistributed** — the system stores the citation, a short publisher-allowed summary, and (where available) a publisher link.

### G.8 Terminology consistency at the report layer

The report-render engine normalises terminology at output time:
- "Top drive" not "TDS" not "powered swivel."
- "Drawworks" not "drum" not "winch."
- "Mast" or "derrick" depending on rig type (mast = land/portable, derrick = offshore).
- "Blowout preventer" or "BOP" — first usage spelled out, subsequent abbreviated.

A small style normaliser runs over the rendered prose before PDF.

---

## H. Pain Point → Feature Mapping

| Pain point | Feature in v1 | Time saved | Cognitive load reduction |
|---|---|---|---|
| Retyping field notes into Word | Notes are typed (or dictated) directly into structured `Finding` records on the device | 8–14 h / inspection | High |
| Photo→finding linkage lost overnight | Photos auto-link by temporal proximity at capture time; triage queue handles strays | 4–8 h / inspection | Very high — eliminates "which photo was that?" |
| Word table reformatting for defect register | Register is a rendered output from `Finding` rows; no table editing | 2–4 h / inspection | High |
| Photo plate layout in Word | Deterministic plate template; auto-numbered; rendered server-side | 3–6 h / inspection | Medium |
| Severity drift across days | Severity picker at capture time with rule-suggested defaults | 1 h / inspection | High — consistency across inspection |
| Missing clause citation | Clause auto-prefilled from component context | <1 h / inspection | High — credibility win |
| Re-typing prior-inspection context | Prior open findings auto-imported as "inherited" items | 1–2 h / inspection | Medium — also unlocks repeat-finding callouts |
| Forgetting to photograph nameplate | OCR-of-nameplate as a one-tap capture; populates `Component.manufacturer/model/serial` | Small per use, large in cumulative cleanliness | Medium |
| Reporting fatigue / blank-page anxiety | LLM-drafted exec summary + per-discipline narratives, inspector-reviewed | 4–8 h / inspection | Very high — emotional, not just temporal |
| Lost work between site and home | Local-first writes; visible sync status; export-everything button | Saves catastrophe, not minutes | Very high — trust |
| Stale text from previous Word template | Report is rendered, not edited — previous reports cannot leak into new ones | Eliminates a whole class of mistakes | Very high — risk reduction |
| "Final_v3_FINAL.docx" version anti-pattern | `Report.version` is auto-incremented; only one "current" version | Saves admin pain | Medium |
| Inconsistent component naming | Canonical component templates per rig type | Quality, not time | High — report polish |
| Inspector qualifications not on report | `InspectorCertification` table renders into the sign-off page automatically | <30 min / inspection | High — defensibility |
| Calibration evidence missing from NDT findings | `InstrumentCalibration` cross-referenced from `Measurement`; rendered into the NDT appendix | <30 min / inspection | High — defensibility |
| Critical-finding dispensation lost track of | `Finding.status = dispensed` with named approver and timestamp | Minor time, major risk reduction | High |
| 80 MB PDF won't email | Photo compression at render; sidecar zip for full-res; portal upload for very large reports | Saves a 30-min struggle | Low cognitive load |
| Repeat findings across inspections | `inherited_from_finding_id` auto-flags repeats; render "REPEAT FROM 2024-09" badge in the register | New capability | Medium — story for CEO |

---

## I. Reliability Requirements

These are not features. They are properties the product must hold even when nothing else works.

### I.1 Offline behaviour

- **Every screen renders, every action saves, with no network connectivity.** No "you must be online" splash, ever, after first sign-in.
- The local data store is the source of truth on the device. The server is the source of truth for the canonical inspection state, with the device as authoritative for in-progress field capture.
- Sync resumes automatically when network returns. The inspector sees sync state but is never *blocked* by it.

### I.2 Sync handling

- **Push first, pull later.** When connectivity returns, push pending writes before pulling server updates. The inspector's work is the most precious data; never overwrite it with stale server state.
- **Conflict resolution.** Last-write-wins per *field*, not per row. Two devices editing the same finding can both make non-overlapping edits without one losing.
- **Audit trail.** Every conflict resolution writes a `ChangeLog` entry. The inspector can review what was overwritten and restore if needed.
- **Sync is incremental.** A 200 MB inspection bundle is broken into per-row + per-photo chunks. A failed photo upload does not abort the whole sync.
- **Sync is pause/resume.** A dropped connection mid-upload picks up where it left off.

### I.3 Evidence safety

- **Photos write to local disk before the UI confirms.** No "saving…" spinner that fails silently.
- **Photo uploads survive app kill, OS update, dead battery.** The upload queue is a durable database, not an in-memory list.
- **No silent deletion, ever.** Discards go to a 30-day local trash with a one-tap restore.
- **Full-resolution originals preserved.** Compressed copies for transmission; originals retained for sidecar zip and archival.

### I.4 Export guarantees

- **"Export everything" works offline and is always available.** Produces a zip with PDF + Excel + JSON + full-res photos.
- **Inspector owns their data.** No "premium tier needed to export" UX dark pattern. Ever.
- **Round-trip safety.** Exported JSON can be re-imported into a fresh install with zero data loss. Tested as a CI guarantee.

### I.5 PDF rendering

- **Renders identically across devices and over time.** Same input → same output, byte-for-byte where possible.
- **PDF/A archival format** so reports remain openable in 10 years.
- **Signed.** Digital signature; falls back to scanned wet-signature image if the inspector prefers.
- **Reproducible.** Given the inspection data + the template version, anyone can re-render the exact same PDF.

### I.6 Time-zone honesty

- **Store UTC.** Always.
- **Display local.** Always.
- **Render local in reports** unless the inspection's location is on a rig in UTC (some offshore standardise on it) — let the inspector pick at inspection-creation time.

### I.7 Crash recovery

- **Auto-save on every keystroke.** No data loss from crash, force-close, or OS-level termination.
- **Resume state at relaunch.** App reopens to the exact screen and field cursor it was on, with the unsaved value (if any) still present.
- **No "report unsaved changes?" dialogues.** Save is implicit.

### I.8 Failure modes the product must survive

| Failure mode | Behaviour |
|---|---|
| Battery dies mid-photo | Photo on disk, finding intact, app resumes at relaunch with photo attached |
| Network drops mid-sync | Sync resumes from last successful chunk |
| Device dropped, screen cracked | Data on disk readable; export-everything still works via cable |
| OS update wipes app cache | Local data in app-private storage survives; tested explicitly |
| Server is down for 48 h | Inspector continues working; sync catches up on return |
| Server data corrupted | Local-device state is canonical for in-progress inspections; can re-seed server |
| Tablet stolen | Server has yesterday's state; today's work is the loss budget |
| Cloud provider outage | Local-first means no operational impact for ≥ 48 h |

---

## J. MVP Scope Definition

### J.1 In scope (v1, 90 days)

- One inspector, one device (tablet) + one optional laptop for viewing the PDF preview.
- Land rig template fully populated; jackup template partially populated (sufficient for one real inspection).
- Five-system depth: derrick & substructure, drawworks, top drive, mud system, BOP. Other systems present as empty trees the inspector can populate ad-hoc.
- Finding capture with photos, observation, recommendation, severity, reference clause.
- Photo capture with temporal-proximity auto-link, triage queue, in-app markup.
- Push-to-talk dictation into observation field (Whisper or Deepgram).
- Standards-clause library: API RP 4G, RP 8B, Std 53, IADC checklist, OSHA 1910 selected subparts.
- Findings library: ~50 starter snippets keyed to component × severity; inspector adds as they go.
- Server-side PDF render of the full report template (audit archetype).
- Excel defect register export.
- JSON full export.
- Sidecar photo zip.
- Sync engine, conflict resolution, durable upload queue.
- Inspector profile + certifications.
- Single-tenant deployment (one inspector, one server tenant).

### J.2 Out of scope (v1)

- Multi-tenant teams, roles, sharing.
- Multi-inspector collaboration on the same inspection.
- Real-time co-editing.
- Customer portal / web view of reports.
- Custom template / form builder.
- AI defect detection in photos.
- Continuous voice transcription (only structured-field dictation).
- Fleet benchmarking, analytics dashboards.
- Predictive maintenance.
- Integrations with Maximo, SAP PM, Cognite, Power BI.
- ISO 27001 / SOC 2 audit (gate later, when selling to firms).
- White-label / theming beyond client logo on cover.
- App store distribution (TestFlight + direct APK for v1).
- Multi-language.
- BOP-specific NDT calculator modules.
- NORSOK well-barrier schematic editor.
- Drone footage / video ingest.
- Inspection scheduling / dispatch.
- Time tracking / billing.
- E-signature workflow with multi-party countersign.
- iPad-only iOS app (start with Android + Windows; iOS later if requested by buyers).

### J.3 The complexity test

Before adding any feature to v1, ask:
1. **Does it appear in the founder's last three real inspections?** If not, postpone.
2. **Is it required to ship a CEO-acceptable PDF?** If not, postpone.
3. **Does it improve trust / reliability / speed?** If not, postpone.

This is the only filter. "Customers will ask for it" is not a v1 gate.

### J.4 v1.5 (months 4–6)

- LLM-drafted exec summary and per-discipline narratives (review-gated).
- LLM-drafted finding narratives from structured fields.
- OCR of nameplates → auto-populate `Component`.
- Historical comparison view (this rig vs last visit).
- Second rig-type template completion (jackup).
- Multi-inspector single-tenant (one firm, multiple inspectors).
- Photo classification (component type) for triage acceleration.

### J.5 v2 (months 7–12)

- Multi-tenant teams.
- Customer portal for closure-evidence upload.
- Fleet view across multiple rigs.
- SOC 2 Type I preparation.
- Maximo / SAP PM ingest CSV.
- RAG over historical reports + standards library for question-answering.

---

## K. The Real Product Moat

Five candidate moats, ranked by how hard they are for an incumbent to replicate.

### K.1 The inspector's findings library

Every inspection enriches the library of phrasings, recommendations, clause-citations, and component templates the inspector personally trusts. By inspection #20, autocomplete suggests 80% of the recommendation text — and the inspector's editing makes those suggestions better. **This compounds. SafetyCulture cannot have my findings library. ModuSpec cannot either.**

### K.2 The standards-clause map curated to component context

The mapping from "BOP ram block crack" → API Std 53 §6.x + 30 CFR 250.737 + API Spec 16A material requirements is **months of curation work** that no competitor will do unless they have an inspector-founder. This is also the highest signal of professionalism in the report — clients and regulators visibly notice it.

### K.3 The inspection archive itself

Two years in, the inspector has a corpus of comparable inspections. That corpus enables historical comparison ("repeat finding from 2024-09"), benchmarking ("your top-drive saver-sub findings are 2.1× fleet median"), and RAG over reports. **The corpus does not exist without the tool. Leaving the tool means abandoning the corpus.** This is the single hardest moat to detach from.

### K.4 PDF parity that exceeds the inspector's current Word output

If the rendered PDF is visibly cleaner, more consistent, and better-laid-out than what the inspector previously produced, the inspector's clients start preferring it. Inspector cannot leave the tool without explaining a downgrade to clients. This is brand-trust moat at one remove — the *inspector's brand* becomes dependent on the tool's output.

### K.5 Habit formation

The fewer keystrokes per finding, the more findings get logged. Over months, the inspector accumulates muscle memory: severity-chip → photo → push-to-talk → mark complete. Switching to another tool means re-learning the rhythm. This is the weakest moat individually but the most underestimated in aggregate.

### K.6 What is NOT a moat

- **AI defect detection.** Even if it worked, it's a feature, not a moat — and it doesn't reliably work yet.
- **Pretty UI.** Copyable.
- **Cloud sync.** Table stakes.
- **Integrations.** Useful, but each integration is an arms race against the incumbent's roadmap.

The defensibility is **inspector-shape**, not platform-shape. Compete on workflow density, vocabulary fidelity, and accumulated context — not on horizontal "platform" surface.

---

## L. Product Strategy — why inspectors adopt and pay

### L.1 The adoption story

An experienced inspector adopts the tool because **the second inspection is the proof**. Inspection #1 they're cautiously trialling alongside paper. Inspection #2 they drop the paper and discover that the evening debrief took 30 minutes instead of 4 hours. They walk to dinner at 19:00 instead of 23:00. That moment is the conversion event.

### L.2 The trust story

Inspectors have been burned by:
- SaaS vendors that go bust.
- Tools that lock data behind exports requiring a premium tier.
- Apps that lose photos.
- Reports that look worse than what they made in Word.

The tool wins trust by:
- Local-first architecture (data lives on the inspector's device).
- "Export everything" button on day one.
- PDF rendering that visibly exceeds Word output.
- A founder who is also an inspector (or who has inspected alongside one).
- No "AI takes the photo and decides if it's bad" claims.

### L.3 The price logic

At **$99–$199/month per inspector**, paid out of personal credit card or small-firm overhead, the tool pays for itself in **one job**. A third-party inspector billing $1,200–$2,500/day saves 22–45 hours per inspection — at the lower end, ~$3,000 of recovered margin per job. At one job per month, ROI is 15–30×.

Avoid per-inspection pricing — it punishes power users (your champions) and creates resistance at exactly the moment the inspector should be reaching for the tool. Avoid free tiers — they signal the tool is not for serious inspectors. Avoid annual-only — month-to-month with a generous annual discount fits the unpredictable revenue of independent inspectors.

### L.4 The acquisition story

Three reachable channels for v1:

1. **Direct sales by the founder.** The first 10 customers come from the founder's existing inspector network. No marketing spend required.
2. **Case-study driven.** Each early customer's inspection becomes a public case study (with permission, with rig identifying details scrubbed). "How [Inspector X] cut report writing from 25 hours to 3" is a credible story for the inspector community.
3. **LinkedIn / SPE drilling forum presence.** The inspector community is small, networked, and reads each other's posts. A founder who posts technical notes (not marketing) gets visibility.

Avoid:
- Paid advertising. Inspectors don't click.
- Cold outreach to drilling contractors. Wrong buyer, wrong cycle.
- Conference booths. Too expensive for the audience reach.

### L.5 The retention story

Inspectors stay because:
- Their findings library is in the tool.
- Their report template is in the tool.
- Their inspection history is in the tool.
- Their clients now expect the report to look the way the tool renders it.

Inspectors leave only if:
- The tool loses their data (catastrophic, unrecoverable).
- The tool's PDF quality regresses.
- The tool ships features they didn't ask for and the UI becomes cluttered.
- A competitor with significantly better mobile UX appears (low-probability, but the threat motivates UX investment).

### L.6 The expansion story (after PMF)

After the bottom-up motion has produced 50–100 paying inspectors and a body of case studies:

- **Year 2.** Sell to small inspection firms (5–25 inspectors). ACV $30k–$150k/firm. Add multi-inspector single-tenant features (firm branding on report, peer review workflow, simple admin).
- **Year 3.** Sell to drilling contractors. ACV $250k–$2M/firm. Requires SOC 2 Type II, multi-tenant, Maximo/SAP integration, fleet view. The entities and workflows from v1 still hold — only the scaffolding around them changes.

Do not skip stages. Each stage's revenue funds the next stage's compliance, integration, and security investment.

---

## M. Build Sequence

The order that minimises risk and produces shippable progress every two weeks.

### M.1 Weeks 1–2 — Data model + render skeleton

- Implement `Rig`, `Component`, `Inspection`, `Finding`, `Photo`, `StandardClause` tables (Postgres).
- Implement the recursive `Component` tree with a seeded land-rig template (~80 components across 5 systems).
- Implement a Python (or your preferred stack) HTML→PDF renderer with a hard-coded test inspection.
- **Deliverable:** founder can `curl` an API to create an inspection, post 5 findings with 1 photo each, and `GET /report` returns a 10-page PDF.

### M.2 Weeks 3–4 — Mobile app shell

- Implement local-first datastore on Android (SQLite + sync metadata).
- Implement the 7 mobile screens (home, component browser, finding editor, camera, photo triage, register, sync status) with minimum viable visual polish.
- Implement basic sync to the server.
- **Deliverable:** founder can install the APK on a tablet, complete a 20-finding mock inspection, see findings sync to the server, and download the rendered PDF.

### M.3 Weeks 5–6 — Reference-report parity

- Take the founder's last real Word report.
- Manually enter the data behind it into the system.
- Iterate the PDF template until the output is visually comparable (or better) than the Word original.
- **Deliverable:** a side-by-side comparison the founder is willing to put in front of a peer.

### M.4 Weeks 7–8 — Field hardening

- Test offline behaviour by yanking network mid-finding repeatedly.
- Test photo durability by force-closing the app at varying capture points.
- Test sync resume after 24-h offline period with 200+ pending uploads.
- Add the "Export everything" feature.
- Add the sync-status screen.
- Run **Inspection #1** (founder's own, paper safety net).
- **Deliverable:** first CEO-facing PDF rendered from the platform and delivered to a real client.

### M.5 Weeks 9–10 — Findings & clauses

- Build the findings/recommendations library (starter set ~50 snippets).
- Populate `StandardClause` table with API RP 4G, RP 8B, Std 53, IADC checklist, OSHA 1910 subparts.
- Wire clause-prefill into the finding editor.
- Implement severity rule-suggester for common cases.
- Run **Inspection #2** (founder, no paper).
- **Deliverable:** second client report; debrief takes < 1 h on the final day.

### M.6 Weeks 11–12 — Voice + LLM draft

- Add push-to-talk dictation to the observation field (Whisper or Deepgram, with on-device fallback).
- Add LLM-drafted exec summary (server-side, review-gated, "drafted by AI, reviewed by [inspector]" attribution).
- Add LLM-drafted per-discipline narratives.
- Onboard inspector #2 (peer); run **Inspection #3** with them.
- **Deliverable:** three CEO-delivered reports; two paying users; a coherent demo; case-study material for selling to small firms in month 4.

### M.7 Month 4 — Onward

- v1.5 priorities (LLM narrative everywhere, OCR of nameplates, historical-comparison view, second rig-type template).
- Begin selling to small inspection firms via the founder's network.
- Begin SOC 2 Type I gap analysis.

---

## N. Cross-cutting design principles

Five rules that override any specific feature decision:

1. **The deliverable is the product.** Optimise for the PDF. Every other screen is in service of that.
2. **Structure before AI.** Deterministic software ships faster, is more defensible, and gives AI features better input later.
3. **Offline-first is architecture, not a feature.** Designing around connectivity is too late.
4. **Match the inspector's mental model, not the checklist's order.** Component-system-physical-proximity, not numbered-checklist-row.
5. **Trust is built in the first failure.** Honest sync state, durable photos, export-everything. The inspector who experiences a crash and finds all their work intact never leaves.

---

## Open questions to validate with the actual inspector

A short list of questions worth resolving in a 60–90 minute call with the target inspector before locking the spec:

1. Show me your last three reports. What did you copy from the previous report? What did you re-write?
2. Walk me through the last evening of an inspection. What was the first thing you did when you opened the laptop? The last thing before you closed it?
3. How many photos did you take on your last inspection? How many ended up in the report? What happened to the rest?
4. Which severity vocabulary do your most demanding clients use? Critical/Major/Minor, A/B/C, or something else?
5. What does the inside of your inspection folder structure look like today?
6. Which standards do you cite most often? Which do you wish you cited more often but skip because it's tedious?
7. What's the worst inspection report mistake you've ever made? What was the cause?
8. What does your current Word template look like? Can I have a copy with sensitive details redacted?
9. If you could automate one thing tomorrow, what would it be?
10. What inspections are you doing in the next 60 days? Could I shadow one?

The answers to these calibrate every decision above.
