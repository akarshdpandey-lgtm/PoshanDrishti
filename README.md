# PoshanDrishti Health — Mother + Child Screening in One Home Visit

A screening-support web app for ASHA and Anganwadi frontline health workers.
In a single home visit, the worker screens **a mother and her child(ren)
together — one linked record (a "dyad")** — for child undernutrition and for
the mother's anaemia risk, and gets a clear next action for each.

It runs entirely in the browser: one self-contained `index.html`, no server,
no login, no account. Records are stored only on the device (browser
localStorage), so after the first load it works fully offline — in airplane
mode, in low-connectivity villages, on any basic smartphone.

It also carries the field tools of the original PoshanDrishti system,
ported to run entirely on-device: an ASHA Saathi help/FAQ with emergency
helplines and government schemes, a Hidden Hunger (micronutrient) checklist,
a pregnancy care checklist, a per-child diet-plan generator, and a progress
score for follow-up prioritisation — see
[What's inside — from the original PoshanDrishti](#whats-inside--from-the-original-poshandrishti).

**This is screening support, not a diagnosis.** Every result screen says so,
in English and in Hindi: *"Screening support — not a diagnosis." /
"यह जाँच में सहायता है — निदान नहीं है।"*

## Why a dyad?

Mother and child nutrition are biologically and practically linked: a mother
with anaemia is more likely to have an anaemic child (NFHS-5 analyses report
an adjusted odds ratio of about 1.75), and the same home visit already brings
the worker to both. Most tools screen the child alone, or record mother and
child in separate systems. PoshanDrishti Health makes the pair the unit of
the visit: one form, one saved family record, both results side by side, and
a visit history that grows for the child over time.

## What it computes

### Child module (0–59 months)

Inputs: date of birth **or** age in months, sex, weight (kg), length/height
(cm) and how it was measured (lying/standing), MUAC (mm), and bilateral
pitting oedema (yes/no).

- **Z-scores** — weight-for-age (WAZ), length/height-for-age (HAZ) and
  weight-for-length/height (WHZ) are computed from the **WHO Child Growth
  Standards LMS tables** using the LMS formula
  `z = ((X/M)^L − 1) / (L·S)`, with the WHO restricted adjustment applied
  beyond |z| = 3 (z is extended linearly using the distance between the
  values at ±2 and ±3 SD).
- **Length/height rule** — under 24 months the recumbent length is used
  (height = length − 0.7 cm where needed); from 24 months standing height is
  used (length = height + 0.7 cm where needed), exactly as in the WHO
  standards.
- **Classification (WHO 2023)** — SAM = WHZ < −3 **or** MUAC < 115 mm **or**
  oedema present; MAM = WHZ −3 to < −2 **or** MUAC 115 to < 125 mm. MUAC
  cut-offs are applied only at 6–59 months. Each result carries its action
  card: SAM → referral to a facility / Nutrition Rehabilitation Centre now;
  MAM → supplementary nutrition, counselling and closer follow-up; normal →
  continue routine growth monitoring. Stunting and underweight are also
  shown from HAZ and WAZ.
- **Re-measure prompts** — values outside the WHO flag ranges (WHZ beyond
  ±5, HAZ beyond ±6, WAZ below −6 or above +5) are flagged with a plain
  "check the measurement and re-enter" message.
- **Trajectory** — every visit is kept on-device. From the second visit the
  app draws the child's WAZ and WHZ trend and shows a plain-language prompt
  when the latest visit shows no weight gain since the previous visit, or a
  fall of 0.5 z or more in WHZ or WAZ. The prompt says only: visit sooner,
  counsel, re-weigh. It is a prompt to look again — **not a forecast** of the
  child's future growth.

### Mother module — anaemia RISK TRIAGE (no camera, no photo analysis)

The worker records what she can see and ask:

- **Pallor checklist** — lower eyelid (conjunctiva), palms, nailbeds:
  present/absent at each site.
- **Symptoms** — unusual fatigue, breathlessness on routine work, dizziness,
  palpitations.
- **Risk factors** — delivered in the last 6 months, history of heavy
  bleeding, not taking IFA tablets, no animal-source foods or milk in the
  diet, short gap since the last child; age under 20 is added automatically.

These are combined by a **transparent rule whose working is shown on the
result screen** (pallor counts double; two or more pallor sites, or a high
total, means Higher risk). The output is one of three tiers, and every tier
ends in an action:

- **Higher risk** → haemoglobin test at the nearest facility/PHC/sub-centre
  within a week, plus IFA guidance per programme.
- **Moderate risk** → IFA and diet counselling now; Hb test at the next
  ANC/VHSND contact.
- **Lower risk** → routine prevention (IFA as per programme, iron-rich
  foods), with the honest note that absence of pallor does not rule out
  anaemia.

**Optional measured Hb** — if a real haemoglobin value (g/dL) from any
actual test is entered, it is classified with the **WHO 2024 bands**
(non-pregnant women: anaemia < 12.0; pregnant: < 11.0, second trimester
< 10.5; with mild/moderate/severe splits) and clearly labelled as the
classification of a **measured** value. (WHO's altitude adjustment is not
applied; the app is designed for lowland use and says to follow local
programme guidance.)

### Language and voice

Full Hindi (Devanagari) and English toggle for all screens and results, and
a "Listen" button on result cards that reads the result aloud
(hi-IN / en-IN device voices) where the device supports it.

## Household cascade & closed referral loop

A SAM or MAM result is treated as a household signal, not a single-child
event. When any child in a visit screens SAM (WHZ < −3 / MUAC < 115 mm /
oedema) or MAM (WHZ −3 to < −2 / MUAC 115–< 125 mm), the visit **cannot be
marked complete** until a short household check is done — or the worker
explicitly defers it, and the deferral is stored on the family record with
its date.

1. **Sibling sweep.** The worker lists the household's other under-5
   children. Each child who is present is screened with the same rules
   (MUAC + oedema at minimum; full WHO z-scores when weight and length /
   height are measured), and the result is saved on the household record.
   A sibling who is absent becomes a follow-up task due within 7 days in
   the worker's open-tasks list.
2. **TB-contact prompt.** Three yes/no questions: any household member on
   TB treatment in the last 2 years; caregiver cough for 2+ weeks;
   fever / night sweats / weight loss in the caregiver or child. Any "yes"
   produces a referral for TB evaluation at the PHC / NTEP facility plus
   an information line on the Nikshay Poshan Yojana nutrition support
   (₹1,000/month for TB patients). It is labelled, in both languages:
   **"TB risk prompt — not a TB test or diagnosis."**
3. **Closed referral loop.** Every referral the screening creates — SAM /
   MAM index child, SAM / MAM sibling, mother at Higher anaemia risk, TB
   evaluation — becomes a trackable on-device record. States:
   *Referred → Reached* (the worker confirms, with date) or *Referred →
   not reached*, where the worker records a barrier from a fixed list
   (distance / transport cost; wage loss; no escort available; facility
   closed / no bed; family refused; child ill / other; other). Follow-up
   is prompted at day 3, day 7 and day 14 in the worker's open-referral
   list, and a referral still open after 14 days is flagged. A referral
   closes **only** as "Reached" or as **"Closed after 3 attempts —
   barrier: [x]"**. Everything works fully offline in localStorage; there
   is no server, no SMS and no facility-side integration of any kind.

Both cascade and referral screens carry the labels, verbatim, in English
and Hindi:

- "Screening support only — this is not a diagnosis"
- "Referral tracked by the health worker — PoshanDrishti does not confirm
  the facility's diagnosis or treatment."

### Why this exists — the evidence

- **Siblings share the risk.** A study from Bhopal published in *Indian
  Pediatrics* found roughly **30% SAM and 20% MAM among the under-5
  siblings of children admitted with SAM** — screening only the index
  child misses a large, findable share of cases in the same home.
- **Referral is where the chain breaks.** In a Nashik study, about **54%
  of families referred with a SAM child could not follow through** —
  a referral slip with no follow-up record silently loses these children.
  Recording the barrier (transport cost, wage loss, no escort, facility
  closed) turns an invisible failure into an actionable list.
- **TB and undernutrition travel together.** The RATIONS trial in
  Jharkhand showed household nutritional support reduced TB incidence
  among household contacts, and TB patients receive ₹1,000/month under
  the Nikshay Poshan Yojana — so a malnutrition screening visit is a
  practical moment to ask about TB in the household and point the family
  to support that already exists.

### What is NOT claimed

- **Not the first referral tracking.** Referral follow-up exists in
  CommCare, DHIS2-based programmes and others. What this prototype adds
  is narrower and specific: an **offline, worker-confirmed referral loop
  tied directly to the screening result** (including the sibling sweep
  and TB prompt it triggers), running on the same single device with no
  account, no connectivity and no cost — built for the worker who did
  the screening, not for a dashboard.
- **Not facility confirmation.** "Reached" means the health worker
  recorded that the family reached the facility. The app never hears
  back from the facility and does not confirm any diagnosis or treatment
  given there.
- **Not a TB test.** The three questions are a risk prompt that ends in
  referral for evaluation — nothing more.

## What it deliberately does NOT do

- **No camera and no photo analysis anywhere.** Nothing in the app estimates
  haemoglobin — or anything else — from a photograph. Clinical-sign
  checklists are honestly presented as risk triage that ends in referral
  for a real Hb test, and the checklist is **not a validated score**: field
  validation against measured Hb is the stated next milestone.
- **No diagnosis and no prescribing.** IFA is referenced only as "per
  programme guidance".
- **No growth forecasts and no risk scores for SAM.** The trajectory prompt
  is descriptive ("visit sooner, counsel, re-weigh"), nothing more.
- **No server, no uploads, no accounts.** Family records never leave the
  device.
- **No phantom referral confirmation.** A referral closes only when the
  worker records "Reached" on the device, or after three recorded
  attempts with a named barrier. The app has no link to any facility
  system and never implies one.

## Data source and validation

- **Tables**: WHO Child Growth Standards expanded LMS tables — weight-for-age,
  length/height-for-age, weight-for-length (45–110 cm) and weight-for-height
  (65–120 cm), by sex — taken from the data files of the WHO `anthro` R
  package (which distributes the WHO standards), retrieved 5 November 2026.
  In the app they are embedded as compact JSON: age tables on a grid of
  2 days (0–90 days), 5 days (to 24 months) and 10 days (to 59 months);
  weight-for-length/height every 0.5 cm; L, M, S are linearly interpolated
  between grid points, the same way the standards are used between rows.
- **Validation**: an independent Python implementation recomputed z-scores
  directly from the raw, full-resolution WHO rows (daily ages; 0.1 cm steps)
  using the same published LMS formula and restricted adjustment, and the
  outputs were compared with the app's own JavaScript run headlessly:
  **104 test cases, 116 z-scores compared, maximum deviation 0.0005 z** —
  the tiny residual comes only from the app's thinned grid. Median checks
  return exactly z = 0 (e.g., a boy of 9.646 kg at 365 days; a girl of
  86.4008 cm at 730 days).
- **Worked example (SAM)**: boy, 730 days, lying length 80.0 cm, weight
  7.6 kg, MUAC 118 mm, no oedema. Weight-for-length at 80.0 cm (boys):
  L = −0.3521, M = 10.4475, S = 0.08308 → unadjusted z = −4.05 → WHO
  adjustment (values at −3 SD = 8.2273 kg, −2 SD = 8.8896 kg) → **WHZ =
  −3.95 → SAM**, referral action. The app gives identical values.
- **Worked example (MAM)**: girl, 913 days, standing height 85.0 cm, weight
  9.2 kg, MUAC 128 mm, no oedema. Weight-for-height at 85.0 cm (girls):
  L = −0.3833, M = 11.3934, S = 0.08896 → **WHZ = −2.50 → MAM**,
  supplementary-nutrition action. The app gives identical values.

## Programme alignment (India)

The mother module follows **Anemia Mukt Bharat's** approach — Test, Treat,
Talk, Track: the app supports the *Test* referral (with the checklist making
the referral decision visible) and the *Track* step (results and Hb values
are kept in the family record across visits), while treatment remains with
the programme and the facility. The child module follows WHO 2023 wasting
criteria already used in Indian community screening practice (MUAC +
oedema), adding the full z-score picture and visit trajectory that a paper
register cannot show.

## Context

Built as the working prototype for an **MIT Solve 2027 Global Health
Challenge** entry. It builds on **PoshanDrishti**, Akarsh Pandey's earlier
AI project for child malnutrition screening support for ASHA/Anganwadi
workers, and extends it from child-only screening to the mother–child dyad
with anaemia risk triage. Design language matches its sibling prototype
NightMaa: warm light editorial, mobile-first, large tap targets, usable by
a worker with gloves in one hand and a MUAC tape in the other.

## Credits

- **Akarsh Pandey** — concept, design and build.
- Growth references: WHO Child Growth Standards (WHO Multicentre Growth
  Reference Study), via the WHO `anthro` package data.
- Anaemia thresholds: WHO guideline on haemoglobin cut-offs (2024).
- Programme framework: Anemia Mukt Bharat, Ministry of Health & Family
  Welfare, Government of India.

## What's inside — from the original PoshanDrishti

This prototype began as a focused mother–child screening tool. In November
2026 the features of Akarsh's **original PoshanDrishti system** — a
full-stack app (React 19 frontend, Flask 3 + SQLite backend, ML models,
deployed with Render/Vercel configs) — were reviewed module by module, and
everything that can honestly live in an offline, single-file,
screening-support app was ported into this file. The ports are adaptations
of the original code and content (credited here), re-implemented in this
app's architecture with full English + Hindi strings.

| Original PoshanDrishti feature | Where it lives in this app |
|---|---|
| **ASHA Saathi chatbot** (`Chatbot.jsx` — a fixed on-device knowledge base of 16 topics with keyword matching, plus the backend `/api/chatbot` keyword rules) | **Saathi Help** view (home → “Saathi Help”): the same 16 topics as tappable chips + type-to-search with the original keyword matcher, each answer with a Listen (TTS) button. Wording was checked against this app's rules (WHO MUAC bands; anaemia bands labelled as children's/pregnancy bands, with “only a blood test measures Hb” stated plainly). |
| **Emergency helplines** (`AlertSystem.jsx` + chatbot) | Saathi Help header strip: tap-to-call **108 / 102 / 1098 / 104 / 112**. (One number in the old chatbot text — an “ICDS helpline 1800-345-6789” — could not be verified and was dropped.) |
| **Government schemes** (from `WombToWorld.jsx`) | Saathi Help: JSY (₹1,400), PMMVY (₹5,000), ICDS, free IFA + Calcium. |
| **Hidden Hunger Predictor** (`HiddenHunger.jsx` — fully client-side in the original: 18 observable signs, diet-quality and economic levels, six nutrient groups) | **Hidden Hunger check** on each child's family card: the original scoring (base 20; diet low +20 / medium +10; economic low +15 / medium +5; Iron +25, Vitamin A +20, Calcium+Vitamin D +25, Vitamin C +15, B12/Folate +15, Zinc +10; capped at 100) with the original per-nutrient, area-specific (rural/urban/tribal) food lists and tips. Labelled as a diet-and-signs checklist, not a test. A second, simpler frequency-based variant existed on the old backend (`/api/hidden-hunger`); the richer client-side version was ported. |
| **Womb to World** (`WombToWorld.jsx` — fully client-side pregnancy risk checklist, trimester diets, recommendations, danger signs; backend twin at `/api/womb-to-world`) | **Pregnancy care checklist** in the visit form (appears when the mother is pregnant): the original scoring (base 15; measured-Hb, BMI, age, ANC, diet, economic, diabetes, IFA, tribal-area and water factors) shown as risk/protective factor lists with a care-priority band, plus the original trimester diet plans by area, care actions, and danger signs. Results are kept in the family record. |
| **Diet Plan generator** (`DietPlan.jsx` — food database of 30 common foods + urban extras with kcal and nutrient tags, area/region priority lists, severity-based meal-plan templates) | A **diet plan card on every child's result**: SAM (8 meals, 1500–1800 kcal), MAM (6 meals, 1200–1500 kcal) and Normal (5 meals, 1000–1200 kcal) plans assembled on-device by food tag from the original food list, area-priority foods first, with the original meal times and tips. The worker picks the area type (rural/urban/tribal) on the card. |
| **Health Score + segmentation** (`segmentation.py` — 0–100 score from severity/MUAC/weight-for-height/anaemia/oedema, trend vs previous assessment, segments On Track / Improving / Needs Attention, per-segment recommended actions, >30-day stale rule) | A **progress score chip on each child's family card**: same point table and segment rules, the original recommended-action texts (rendered in English/Hindi), and the stale override (“no visit for N days — follow-up due”). Labelled in-app as a simple worklist score, not clinically validated. Two other score variants existed in the old codebase (a page-level variant in `HealthScore.jsx` and a simplified one in the chatbot text); the segmentation engine's is the operative one and is what the Saathi “health score” answer now explains. |
| **EN/HI language + voice** (`locales/`, `LanguageVoice.jsx`, `VoiceService.js`) | Already native to this app (full EN/HI string tables + Listen buttons); all ported content ships with both languages. |

Two silent defaults in the original code were **deliberately not copied**,
because a screening tool must not invent measurements:

- The old pregnancy page scored Hb as 11 g/dL when no Hb was entered. Here,
  an unmeasured Hb simply skips that factor, and the result card says so
  and asks for a measured test. (If a measured Hb *was* entered in the
  mother module, it is used and classified with WHO 2024 bands as before.)
- The old health score treated a missing weight-for-height z-score as 0
  (worth +20 points). Here a missing z-score earns 0 points for that
  component.

The original pregnancy page also printed numeric “child outcome
predictions” (expected birth-weight ranges, stunting/preterm “risks”).
Those are **not ported**: the checklist ends in care actions, never in a
predicted outcome for the baby. Likewise the untouched-checklist case
records nothing — the pregnancy section only saves when the worker
actually fills it in.

## From the wider PoshanDrishti system (not embedded in this offline prototype)

These parts of the original system depend on a server, trained ML models,
third-party network services, or user accounts, so they are not in this
file and are not simulated:

- **Photo anaemia scan** (`ScanPage.jsx` + `/api/vision-analysis`, Keras
  CNN `anemia_model.h5` on conjunctiva photos, with a red-channel
  colour fallback) — this prototype's founding rule is *no photo-based
  haemoglobin estimation*; anaemia stays a clinical-sign triage ending in
  referral for a measured Hb test.
- **ML malnutrition “prediction”** (`/api/predict`, RandomForest
  `ml_model.pkl`) — a server-trained classifier; this app classifies only
  with published WHO rules that the worker can inspect.
- **30/60/90-day risk forecasting** (`risk_engine.py`, `RiskPrediction.jsx`,
  `models/risk_prediction/`) — forecasting a child's future SAM/MAM status
  is a prediction claim this prototype does not make; its growth-velocity
  idea survives only as the descriptive faltering prompt (“visit sooner,
  counsel, re-weigh”).
- **Accounts, server database and dashboards** (`UserRegistration`,
  `FamilyEntry`, `DataEntry`, `Dashboard`, server `Segmentation` analytics,
  `CommunityDashboard`, `ReportExport`/Excel) — this prototype is
  account-free and keeps records only in the phone's browser storage; the
  family view covers the per-family record, and the segmentation *scoring*
  (above) was ported even though the server analytics pages were not.
- **Hospital Finder** (`HospitalFinder.jsx`, OpenStreetMap Overpass API)
  and **DietPlan location search** (Nominatim) — live network geocoding;
  the diet plan instead asks the worker to pick the area type.
- **SMS/WhatsApp alert dispatch** (`AlertSystem.jsx` server actions) — no
  messages are sent by this app; its helpline strip and worker-confirmed
  referral loop are the offline equivalents.
- **Progress photo comparison** (`ProgressPage.jsx`) — photo storage and
  comparison; excluded with all camera features. (Its local visit-history
  idea is covered by this app's family record and trend chart.)
- **`ARCHITECTURE_PLAN.md`** in the original repo describes a planned
  React + FastAPI + MongoDB production architecture; the shipped original
  code (React + Flask + SQLite) is what was ported from.

## Running it

Open `index.html` in any modern browser, or serve the folder statically
(e.g., GitHub Pages). The only external requests the page makes are to
Google Fonts; everything else — data, logic, records — is in the file and
on the device.
