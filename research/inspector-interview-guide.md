# Inspector Interview — Building Software for Your Workflow

Thanks for the time.

We're working on a tool to replace the pen, paper, photos, and Microsoft Word workflow that you (and inspectors like you) currently use to produce CEO-facing rig inspection reports. Rather than guess at what you actually do day-to-day, we'd rather build the product from the real shape of your work.

This document is designed two ways:
- A **60–90 minute conversation** — we'll walk through it together.
- An **async questionnaire** — answer the ones that matter to you, skip the rest. Even 10 thoughtful answers are more valuable than 50 quick ones.

A few ground rules:
- **No wrong answers.** "It depends" is a perfectly good answer.
- **Specifics from your last few inspections beat generalisations.** "On the *Atwood Beacon* in November we…" is gold.
- **Skip anything you'd rather not answer**, especially anything client-identifying.
- **We will not share your answers** outside our small build team, and any artifacts you share will be redacted before they enter any internal document.
- After the conversation we'll send you a written summary of what we changed in the spec because of your input.

---

## Our current thinking (please challenge it)

Before the questions, here's the hypothesis we're testing. If any of it is wrong, that's the most useful thing you can tell us.

1. The biggest time costs in your week are the **evening debrief** (transferring photos, retyping notes, building defect-register entries in Word/Excel) and **report writing** (laying out photo plates, formatting tables, writing exec summary). Together: 20–40 hours per inspection.
2. The single biggest source of error is **photo-to-finding linkage** — knowing which photo on the camera roll belongs to which finding hours later.
3. **Severity often drifts** between when you spot it in the field and when you write it up.
4. The **CEO reads only the executive summary** and maybe the top critical findings list.
5. You produce broadly three shapes of report: a short walkdown (~10–30 pp), a full audit (~100–300 pp), and a pre-purchase / due-diligence (~50–150 pp).
6. You'd rather have a tool that **does fewer things really well** than one that tries to be a generic checklist app.

If any of these feel wrong, please push back early — we'd rather discover it now than build around a wrong premise.

---

## Section 1 — Your background and current setup (~5 min)

1. How long have you been doing rig inspections, and what kinds of rigs are you on most (land, jackup, semisub, drillship, workover)?
2. Do you work solo, as part of a small firm, or as a contractor placed through a third-party house (Bureau Veritas / ModuSpec, Add Energy, ABL Group, OCS Group, RigCert, etc.)?
3. What's the rough mix of your engagements right now — operator-commissioned, drilling-contractor-commissioned, pre-purchase / due-diligence, recertification, other?
4. Roughly how many inspections do you complete in a typical month or year?
5. What's your day rate, or how do you bill? A rough range is fine.
6. What gear do you carry onto a rig today — phone model, tablet, laptop, camera, anything intrinsically safe, any test instruments you bring yourself (UT gauge, IR thermometer, multimeter)?

---

## Section 2 — Walk us through your last inspection (~20 min)

Pick whichever recent inspection you remember most clearly. The more concrete the details, the more we learn.

7. Where was it (region, rig type), who was the client (operator / contractor / owner), how many days on board, how many days end-to-end including travel and reporting?
8. How did the **scope** get agreed before mobilisation — email, Word SOW, framework agreement? Did it change at any point?
9. What documents did you pull together in the **desktop-review** phase before going on board? Where did they live (OneDrive, email attachments, USB drive)? Roughly how long did that take?
10. What did **Day 1 on the rig** look like — induction, PTW, kick-off meeting? At what point in Day 1 did you start logging actual findings?
11. How did you split your time across disciplines (structure, mechanical, electrical, well control, safety) day by day?
12. Roughly **how many findings** did you raise in total, and **how many photos** were on your phone at the end of the last day?
13. Did anything in the rig schedule (BOP test timing, weather, SIMOPS, crew change) force you to reorder your work?
14. What did the **exit meeting** look like — who attended, how were any disputed findings handled, were any findings dispensed or downgraded under client pressure?
15. Anything unusual about that particular inspection that wouldn't be true for most others?

---

## Section 3 — The evening debrief and the report (~20 min)

This is where we suspect the biggest pain lives. Push back hard if we've got that wrong.

16. After each day on the rig, what did you do in the evening before sleeping? Walk us through 19:00 to 23:00 on Day 3 of an average inspection.
17. How do you **transfer photos off your phone**? How do you rename and sort them — by day, by system, by finding?
18. How do you turn **handwritten notes into findings** — Word? Excel? Both? At what point do you type them up?
19. How do you decide **severity**? Do you set it in the moment, or revisit at the end of the day, or wait until report-writing?
20. After the on-rig week, **how long until the report is delivered**, and what does the report-writing schedule look like — full days, evenings around other work?
21. How do you actually **build the report**? Is there a master Word template you start from? How much do you copy or adapt from previous reports?
22. How do you **lay out photo plates** in Word? How do you annotate photos (arrows, circles, dimension lines)? What tool do you use for the annotation?
23. Do you have a **peer reviewer**, or does the report ship straight from you to the client?
24. How does the client take **delivery** — email PDF, USB drop-off, portal upload? Do they want an Excel defect register alongside the PDF? Do they want raw photos?
25. What's the **longest** part of report-writing? Where do you most often get stuck, bored, or have to redo work?
26. After delivery, how do you handle **close-out** — clients sending evidence that critical items have been fixed? Does the loop usually close, or do items drift?

---

## Section 4 — Specific tools and field reality (~10 min)

27. What's your **daily phone** right now? Has any operator ever asked you not to bring it onto the rig floor or into Zone 1?
28. Have you ever used an **intrinsically-safe phone or tablet** (Aegex, Getac F110-EX / ZX10-Ex, i.safe MOBILE, ecom Tab-Ex)? If so, what was it like to live with for a week?
29. What's **connectivity** like on a typical rig you visit — VSAT, 4G hotspot at the company-man's trailer, Starlink Maritime? Are you uploading anything from on board, or saving it all for after?
30. Do you wear **gloves** while logging findings on a phone or tablet? Which gloves? Do they actually work with the screen?
31. Have you ever **lost work** because of a dead battery, a dropped phone, a lost notebook, an OneDrive sync failure? What happened?
32. Do you use **voice dictation** today, or have you tried? At what noise level does it become useless?
33. What software (apps, Word, Excel, Adobe Acrobat, OneDrive, Dropbox) do you actually use during an inspection? Rough list.

---

## Section 5 — Severity, vocabulary, standards (~10 min)

34. Which **severity scheme** do you use most often — Critical / Major / Minor, A / B / C, 1–5 condition rating, 5×5 risk matrix? Does it vary by client?
35. Which **standards** do you cite most often in reports — API RP 4G, RP 8B, Std 53, Spec 16A, IADC checklist, OSHA 1910, BSEE 30 CFR 250, NORSOK, DNV, ABS, client-specific specs?
36. Are there clauses you **wish you cited more often** but skip because it's tedious to look up?
37. Do you keep a **personal library of recommendation phrasings** you re-use? Where does it live — a Word doc, scattered across previous reports, in your head?
38. What **component-naming conventions** do you stick to? Any standard component names you wish were just always pre-populated when you start an inspection (mast pins, BOP rams, crown sheaves, etc.)?
39. How do you handle **NDT data** today — your own measurements, or someone else's? Do they hand you a PDF you embed in the report, or raw readings you have to format?
40. What inspection **categories** (API RP 4G Cat III/IV, API RP 8B Cat I–IV) do you most often scope into?

---

## Section 6 — Mistakes, disputes, defensibility (~10 min)

41. What's the **worst report mistake** you've ever made or seen — wrong photo on a finding, severity downgrade you regretted, missing critical item, stale text from a previous report? What was the root cause?
42. Have you ever had a finding **disputed by a client** and overturned at exit meeting? How was that resolved? Was there a written dispensation?
43. Have you ever had a finding **come back to bite you** months later — regulator finding, incident on that rig, insurance dispute?
44. What's the one thing that, **if it failed in the field**, would make you abandon a software tool and reach for your notebook?
45. What do you wish a report had that you currently can't put in it because it's too tedious — historical comparison with last inspection, capex backlog roll-up, regulatory clause map, something else?

---

## Section 7 — Money, adoption, and your network (~10 min)

46. What **software do you currently pay for** out of pocket (Adobe Acrobat Pro, Microsoft 365, OneDrive, anything else)? Rough monthly cost?
47. Have you ever paid for an **inspection tool** — SafetyCulture / iAuditor, FAT FINGER, BasinCheck, Floodlight, ModuSpec Argus, GoCanvas, Fulcrum, anything else? What did you think? Why did you stop, if you did?
48. If a tool genuinely saved you **20–40 hours per inspection** and produced a PDF at least as good as your current Word output, what would you pay per month for it? At what price would you start hesitating?
49. Would you rather pay **per inspection or flat monthly subscription**? Annual or month-to-month?
50. How many **other inspectors** are in your immediate network who might also use a tool like this? Any specific people you'd recommend we speak to?
51. Would you be willing to **dogfood the first version** on one of your real inspections — paper still in your bag as a safety net?
52. Could we **shadow one of your inspections** in the next 3–6 months, even remotely (you carry a second device, we watch over your shoulder)?

---

## Section 8 — Show and tell (small but high-value)

If you're willing, the single most valuable thing you can share is real artifacts. Nothing needs to leave our build team. We will redact any client-identifying details before anything enters an internal document.

53. **A recent report (PDF)** — even with all names and logos blacked out, the structure, length, and language tell us a lot.
54. **Your current Word template** — the bones we'd be matching for visual parity in our PDF render.
55. **Your personal snippets file**, if you have one, for recommendation phrasings.
56. **A screenshot of your folder structure** on OneDrive / laptop / phone for a single inspection — how you organise photos and notes today.
57. **The format of an Excel defect register** a client has asked you to deliver — column headers and one or two example rows is plenty.
58. **A copy of an inspection SOW** you've signed (redacted), so we can understand how scope is framed.

---

## Closing

59. What did we **not ask** that we should have?
60. **If we built only one thing**, what's the single feature that would make you adopt this tool on your next inspection?

---

Thanks again. We'll send a written summary of what we change in the spec because of this conversation. If you have follow-up thoughts after the call, even days later, send them anytime — they're equally welcome.
