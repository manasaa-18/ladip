# Longitudinal Adverse Drug Interaction Predictor (LADIP) 💊⏱️

[![Live Demo](https://img.shields.io/badge/Live_Demo-Vercel-000000?style=for-the-badge&logo=vercel&logoColor=white)](https://web-psi-gilt-e25eky7q2j.vercel.app)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge)](LICENSE)
[![Python 3.10+](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Next.js 14](https://img.shields.io/badge/Next.js-14-000000?style=for-the-badge&logo=next.js&logoColor=white)](https://nextjs.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![Expo Go](https://img.shields.io/badge/Expo-000000?style=for-the-badge&logo=expo&logoColor=white)](https://expo.dev/)

> A longitudinal, multi-drug pharmacovigilance system combining real-world FDA FAERS adverse event data with patient medication timelines to detect hidden drug-drug-symptom interactions while eliminating **Clinical Alert Fatigue**.

🔗 **Live Application Portal:** [web-psi-gilt-e25eky7q2j.vercel.app](https://web-psi-gilt-e25eky7q2j.vercel.app)[cite: 1]

---

## 🎯 The Core Problem: Eliminating Clinical Alert Fatigue

In modern Electronic Health Record (EHR) systems, **90% to 96% of drug interaction alerts are overridden and ignored** by clinicians. Conventional checkers trigger flood-level warnings for trivial, non-urgent, or long-standing stable combinations. When an overburdened physician receives 50 low-confidence alerts for an 8-medication patient, they ignore all of them—frequently missing the one critical interaction.

LADIP solves this through three clinical pillars:

1. **Quantitative Disproportionality Scoring:** Replaces binary interaction lists with empirical signal detection statistics ($\text{PRR}$, $\text{ROR}$, $\chi^2$ with Yates' correction, $p$-value, Bayesian Information Component $\text{IC}$) and Evans' SRS criteria ($\text{PRR} \ge 2.0$, $\chi^2 \ge 4.0$, $N \ge 3$).
2. **MedDRA & Outcome Severity Tiering:** Classifies signals into `CRITICAL`, `HIGH`, `MODERATE`, and `LOW` tiers using FDA FAERS outcome codes (`DE` Death, `LT` Life-Threatening, `HO` Hospitalization) and MedDRA System Organ Class (SOC) ontologies.
3. **Temporal Plausibility & Longitudinal Suppression:** Evaluates the Drug-Symptom Temporal Association Score (DTAS) and the Naranjo ADR Probability Scale (10 questions). If a patient has taken a combination stably for $>6$ months with no adverse symptoms, benign background alerts are automatically suppressed.

---

## 🏛️ Segregated System Architecture (Next.js Web + Expo Mobile + FastAPI Backend)

### Why Segregate into Next.js + FastAPI?

* **Independent Frontend/Backend Scaling:** Streamlit re-executes the entire Python script on every widget interaction and couples UI state with backend compute in a single process. By segregating the Next.js App Router Web UI (`web/`) from the FastAPI REST Service (`backend/main.py` & `src/api.py`), the frontend renders with client-side spring physics (`framer-motion`), instant tab transitions, and SSR/static asset caching while FastAPI scales independently across workers.
* **Unified Multi-Client REST Contract:** Both the Next.js Web Portal (`web/`) and the Expo React Native Mobile App (`mobile/`) consume the exact same versioned FastAPI endpoints (`/api/v1/...`).

+------------------------------------+    +------------------------------------+
|     Clinician Web Portal           |    |      Patient Mobile App            |
|  (Next.js 14 + Tailwind + Motion)  |    |   (Expo Go / React Native)         |
|  HORMN Theme + Taste-Skill UI      |    |                                    |
+-----------------+------------------+    +-----------------+------------------+
|                                         |
+--------------------+--------------------+
|  HTTP / REST (/api/v1/...)
v
+------------------------------+
|  Segregated FastAPI Backend  |
|  (backend/main.py, src/api)  |
+--------------+---------------+
|
v
+----------------------------------------------------------------------+
|                     LADIP Core Intelligence Engine                   |
|                                                                      |
|  +------------------------+  +-------------------+  +--------------+ |
|  | Disproportionality     |  | Temporal Analysis |  | RxNorm Brand | |
|  | Engine (PRR, ROR, Chi2)|  | (DTAS & Naranjo)  |  | Normalizer   | |
|  +------------------------+  +-------------------+  +--------------+ |
|                                                                      |
|  +------------------------+  +-------------------+  +--------------+ |
|  | Longitudinal Patient   |  | Prospective Drug  |  | Gemini AI    | |
|  | Memory Store           |  | Safety Checker    |  | Explainer    | |
|  +------------------------+  +-------------------+  +--------------+ |
+---------------------------------------+------------------------------+
|
v
+--------------------------------------------+
|    FDA FAERS Benchmark SQLite Database     |
|     & OpenFDA Throttled Caching Client     |
+--------------------------------------------+

---

## 📁 Repository Structure

```text
ladip/
├── web/                          # Segregated Next.js 14 App Router Web UI (HORMN-Inspired Theme)
│   ├── package.json              # Next.js, React 18, Framer Motion, Phosphor Icons, Tailwind CSS
│   ├── next.config.mjs           # API rewrites proxying /api/v1/* to FastAPI Backend (:8000)
│   ├── tailwind.config.ts        # HORMN clinical pastel tokens & Outfit / Jakarta / Mono fonts
│   └── src/
│       ├── app/                  # Next.js App Router (layout.tsx, page.tsx, not-found.tsx, globals.css)
│       ├── components/           # LadipWorkspace.tsx, BklitCharts.tsx, HormnIllustrations.tsx
│       └── lib/                  # Segregated REST API client (api.ts) & TypeScript contracts (types.ts)
├── backend/                      # Segregated FastAPI Backend Entrypoint
│   ├── __init__.py
│   └── main.py                   # Uvicorn server entrypoint (uvicorn backend.main:app --port 8000)
├── src/
│   ├── config.py                 # Configuration, thresholds, and paths
│   ├── api.py                    # Core FastAPI REST service for Next.js Web & Expo Mobile clients
│   ├── app.py                    # Legacy/Companion Streamlit clinical dashboard (HORMN theme)
│   ├── components/
│   │   └── bklit_charts.py       # Composable Bklit.UI charts & Motion.dev animations (CCv2)
│   ├── normalization/
│   │   └── rxnorm.py             # RxNorm REST API & brand-to-generic mapper
│   ├── faers/
│   │   ├── client.py             # openFDA API client with disk caching & rate throttling
│   │   └── bulk_loader.py        # SQLite schema & benchmark adverse signal store
│   ├── analysis/
│   │   ├── disproportionality.py # PRR, ROR, Chi2, IC, Evans' & multi-drug synergy
│   │   ├── severity.py           # MedDRA SOC & FAERS outcome severity tiering
│   │   ├── temporal.py           # DTAS, overlap windows, dechallenge/rechallenge
│   │   ├── naranjo.py            # 10-point Naranjo ADR Probability Scale
│   │   └── signal_matcher.py     # Patient-to-signal matcher with fatigue filter
│   ├── patient/
│   │   ├── models.py             # Medication, Symptom, PatientProfile schemas
│   │   ├── memory.py             # Patient timeline store & longitudinal deduplication
│   │   └── report_parser.py      # PDF (PyMuPDF) / OCR (Tesseract) / Gemini LLM parser
│   ├── safety/
│   │   └── drug_checker.py       # Prospective "Add New Drug" safety simulator
│   └── explanations/
│       └── pharmacology.py       # Gemini AI & rule-based physiological rationale
├── mobile/                       # React Native / Expo Go Patient Mobile App
│   ├── App.js                    # Cross-platform patient portal & timeline viewer
│   ├── app.json                  # Expo project metadata, favicon & camera permissions
│   ├── package.json              # Mobile dependencies
│   └── src/
│       ├── api/client.js         # Mobile client API bridge with offline clinical fallback
│       ├── components/           # Header, Footer, BklitChart & MotionView spring physics
│       └── screens/              # Schedule, DrugChecker, Scanner, Profile & NotFound (404)
├── scripts/
│   ├── build_signal_db.py        # Seed local SQLite database with benchmark signals
│   ├── generate_synthetic_patients.py # Generate realistic clinical test profiles
│   └── download_faers.py         # FAERS quarterly & live openFDA signal fetcher
├── tests/
│   ├── test_disproportionality.py# Verified against hand-calculated 2x2 tables
│   ├── test_temporal.py          # Validates DTAS & Naranjo scoring
│   ├── test_signal_matcher.py    # Validates alert priority & suppression rules
│   └── test_api.py               # Complete FastAPI, Next.js Web UI & Streamlit test suite
├── data/
│   ├── faers.db                  # Pre-seeded SQLite database with 16 benchmark signals
│   └── patients/                 # Realistic Indian patient cohort JSON files
├── requirements.txt              # Backend dependencies
├── .env.example                  # Environment configuration template
├── LICENSE                       # MIT Open Source License (© 2026 LADIP Contributors)
└── README.md

🧮 Mathematical & Statistical Foundations1. $2 \times 2$ Contingency Table for PharmacovigilanceMetricEvent E ReportedEvent E Not ReportedTotalDrug Combination $D$ Present$a$$b$$a + b$Drug Combination $D$ Absent$c$$d$$c + d$Total$a + c$$b + d$$N = a + b + c + d$2. Proportional Reporting Ratio ($\text{PRR}$)$$\text{PRR} = \frac{\frac{a}{a + b}}{\frac{c}{c + d}}$$3. Reporting Odds Ratio ($\text{ROR}$)$$\text{ROR} = \frac{a \cdot d}{b \cdot c}$$4. Chi-Squared ($\chi^2$) with Yates' Continuity Correction$$\chi^2 = \frac{N \cdot \left( \vert{}a \cdot d - b \cdot c\vert{} - \frac{N}{2} \right)^2}{(a + b)(c + d)(a + c)(b + d)}$$5. Multi-Drug Synergy Ratio$$\text{Synergy} = \frac{\text{PRR}(d_1 + d_2 + \dots + d_k)}{\max_{i < j} \text{PRR}(d_i + d_j)}$$Detects emergent interactions that only manifest when 3 or more drugs are co-prescribed concurrently.🔬 Benchmark Clinical Demo Cohort (Indian Patients)LADIP includes pre-configured realistic clinical test profiles showcasing complex multi-drug challenges:Patient IDName & DemographicsRegimenAdverse ReactionAlert PriorityClinical MechanismPT_BLEED_001Ramesh Sharma(68M, Hyderabad)Warfarin + Aspirin + IbuprofenGastrointestinal HemorrhageCRITICAL(98.5/100)Triple hemostatic failure (COX-1 inhibition + Vit-K antagonism). Acute onset 3 days after adding Ibuprofen.PT_STATIN_002Sunita Patel(62F, Ahmedabad)Simvastatin + Amiodarone + AmlodipineRhabdomyolysisCRITICAL(96.2/100)Severe CYP3A4 & P-gp inhibition causing massive simvastatin accumulation and CK surge (4,820 U/L).PT_MTX_003Kavitha Reddy(54F, Warangal)Methotrexate + TMP-SMX + NaproxenPancytopeniaCRITICAL(97.8/100)Renal clearance blockade + antifolate synergy causing lethal bone marrow suppression.PT_CARDIO_005Arjun Nair(52M, Bengaluru)Clopidogrel + OmeprazoleAttenuated Antiplatelet EffectHIGH(78.0/100)Competitive CYP2C19 bioactivation blockade risking acute stent thrombosis.PT_STABLE_004Rajesh Varma(58M, Secunderabad)Metformin + Lisinopril + AtorvastatinNone (Negative Control)SUPPRESSED(LOW)Alert Fatigue Suppression: Tolerated for 2+ years without symptoms. Suppressed so doctors aren't spammed!🚀 Quickstart & Execution1. PrerequisitesPython: 3.10 or higherNode.js: 18+ & npm (for Next.js Web UI & Expo Mobile App)API Key (Optional): Gemini API key for natural language pharmacological rationales
2. Installation
# Clone the repository
git clone [https://github.com/manasaa-18/ladip.git](https://github.com/manasaa-18/ladip.git)
cd ladip

# Install Python backend dependencies
pip install -r requirements.txt

# Install Next.js Web Frontend dependencies
cd web && npm install && cd ..
3. Run Automated Tests
Verify mathematical engines, UI components, and API endpoints:
python3 -m pytest tests/ -v
4. Start the Segregated FastAPI REST Backend
uvicorn backend.main:app --reload --port 8000
API Root: http://localhost:8000

Interactive OpenAPI Docs: http://localhost:8000/docs

5. Launch the Segregated Next.js Clinician Web UI
In a new terminal window:
cd web
npm run dev
Opens at http://localhost:3000

Built with Next.js 14 App Router, Tailwind CSS, Framer Motion spring physics, Bklit.UI Composable Charts, and the HORMN-inspired Clinical Design System (Outfit + Plus Jakarta Sans + JetBrains Mono).

6. Launch Mobile Patient App (Expo Go)
In a new terminal window:
cd mobile
npm install
npx expo start
📡 REST API ReferenceMethodEndpointDescriptionGET/API portal status, copyright metadata, and module linksGET/api/v1/healthService health check and FAERS database statisticsGET/api/v1/patientsList all patient profiles in the clinical cohort registryGET/api/v1/patients/{patient_id}Retrieve complete patient EHR, timeline, and current medicationsGET/api/v1/patients/{patient_id}/scheduleRetrieve daily dosing schedule slots (Morning, Afternoon, Evening, Bedtime)GET/api/v1/patients/{patient_id}/alertsCompute multi-drug disproportionality, DTAS, Naranjo causality, and alert prioritiesPOST/api/v1/patients/{patient_id}/check-drugProspective drug safety check: simulate adding a new medicationPOST/api/v1/patients/extract-timelineParse PDF/Image/Text discharge summary and save new PatientProfilePOST/api/v1/patients/{patient_id}/scan-reportUpload prescription/report file or text for OCR timeline mergePOST/api/v1/patients/{patient_id}/scan-base64Upload base64-encoded prescription image from mobile cameraPOST/api/v1/simulateAd-hoc prospective simulation for arbitrary drug combinations (with live openFDA option)
🛡️ License
This project is licensed under the MIT License — Copyright © 2026 LADIP Contributors. See the LICENSE file for details.
