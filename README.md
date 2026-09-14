# Aging & Longevity Research Digest

A GitHub Actions workflow that searches a curated list of aging, longevity, nutrition, cardiovascular, and metabolic health journals on PubMed, filters out widely covered stories, runs a single Claude pass (writer + fact-checker + pitch generator), and publishes results to a GitHub Pages dashboard.

---

## How it works

1. **PubMed search** — Queries journals by ISSN for studies published in the past 7 days
2. **Title screening** — Prioritises studies with novelty signals (first-in-class, counterintuitive, overturns prior research). Excludes animal-only studies.
3. **SERPAPI media filter** — Checks Google News and skips any study with 3+ news results
4. **Abstract fetch** — Retrieves full abstracts for shortlisted studies
5. **Claude pass (writer + fact-checker + pitcher)** — Writes structured JSON: headline, summary, why it matters, caveats, relevance score, and pitch angles per publication type
6. **Artifact upload** — Saves JSON results as a GitHub Actions artifact
7. **Deploy job** — Downloads all job artifacts, merges and deduplicates by PMID, commits `data/results.json`, serves via GitHub Pages
8. **Email notification** — Short email with study count and link to dashboard

---

## Dashboard

Features:
- Card view per study with headline, summary, caveats, fact-check notes
- Expandable pitch angles section (one block per publication type — AARP The Magazine, Prevention, Next Avenue, EatingWell, Everyday Health, Health.com, Verywell Health, Women's Health Magazine, Sixty and Me, Woman's World, First for Women)
- Filter by category, groundbreaking type, status, and date range
- Search across all study text and pitches
- Status tracking (New / Saved / Pitched / Passed) saved to localStorage
- Deduplication across runs — same PMID won't appear twice

---

## Schedule

Runs automatically every **morning at 7:00 AM ET**. All jobs run in parallel; the deploy job merges results and publishes the dashboard once complete.

Can also be triggered manually via **Actions → Aging & Longevity Research Digest → Run workflow**.

---

## Categories

| Category | Journals | Jobs |
|---|---|---|
| Seniors & Aging | 83 | 1 |
| Geriatrics | 72 | 1 |
| Nutritional Sciences | 62 | 1 |
| Endocrinology & Metabolism | 126 | 2 (chunks 1–2) |
| Cardiology | 155 | 2 (chunks 1–2) |

Large categories are split into chunks so each job processes ~60–80 journals, keeping run times under 20 minutes.

The CSVs in `data/` are now the hand-maintained source of truth. The workbook `extract_journals.py` was written for (`PubMed_Journals_Categorized.xlsx`) no longer exists, so re-running that script would wipe hand-added rows — add journals by appending rows to the CSVs instead. Every row is searched with no topic filter, so a journal's entire weekly PubMed output enters the digest.

---

## Journal list audit (2026-09-14)

**Method:** Pulled OpenAlex's top sources for this digest's subject areas (cardiology, endocrinology/diabetes, nutrition, geriatrics/aging) over the prior 12 months, diffed them against the CSVs by ISSN and title, and kept only titles NCBI indexes with 20+ PubMed articles in that window. The ten aging titles added the same day by the sibling senior-research-digest audit were also checked here against NLM Catalog and PubMed; five made the cut.

**Added (15):**

| Journal | CSV | PubMed/yr |
|---|---|---|
| Hypertension Research | Cardiology | ~514 |
| American Journal of Preventive Cardiology | Cardiology | ~335 |
| American Journal of Hypertension | Cardiology | ~243 |
| Journal of Clinical Hypertension | Cardiology | ~190 |
| Journal of Human Hypertension | Cardiology | ~166 |
| Journal of the Endocrine Society | Endocrinology & Metabolism | ~288 |
| Menopause | Endocrinology & Metabolism | ~285 |
| Diabetes & Metabolism Journal | Endocrinology & Metabolism | ~147 |
| Climacteric | Endocrinology & Metabolism | ~141 |
| BMJ Open Diabetes Research & Care | Endocrinology & Metabolism | ~90 |
| Journal of Geriatric Cardiology | Geriatrics | ~78 |
| Immunity & Ageing | Geriatrics | ~59 |
| European Journal of Ageing | Geriatrics | ~54 |
| European Review of Aging and Physical Activity | Geriatrics | ~49 |
| Dementia & Neuropsychologia | Seniors & Aging | ~93 |

No category grew by more than ~7%, so the workflow chunking is unchanged.

**Notable exclusions:**
- **Mega-journal / volume:** Frontiers in Cardiovascular Medicine (~2,500/yr) would swamp the 30 candidate slots; JACC Advances (~940/yr, broad general cardiology) left out for the same reason; Journal of Cardiovascular Development and Disease (MDPI) skipped.
- **Case reports / supplements:** JACC Case Reports, European Heart Journal – Case Reports, HeartRhythm Case Reports, JCEM Case Reports, Journal of Cardiology Cases, European Heart Journal Supplements, AACE Endocrinology and Diabetes (formerly AACE Clinical Case Reports).
- **Off-beat:** pharmacy practice and education titles (JAPhA, International Journal of Pharmacy Practice, Research in Social & Administrative Pharmacy, American Journal of Pharmaceutical Education), Cardio-Oncology, lower-extremity wounds, pharmacology/toxicology methods, cereal chemistry, and procedure- or imaging-technique cardiology titles (Structural Heart, echocardiography and cardiovascular imaging journals, electrophysiology journals); from the sibling list, npj Parkinson's Disease and Geriatric Orthopaedic Surgery & Rehabilitation.
- **Too small (<20 PubMed articles/yr):** Aging Brain (~16), Alzheimer's & Dementia: Behavior & Socioeconomics of Aging (~18), Diabetology, Diabetes, Obesity and Cardiometabolic CARE.
- **Not in PubMed (can never return results here):** Revista Argentina de Cardiología (~700 on-topic articles/yr in OpenAlex, 0 in PubMed), Russian Journal of Cardiology, Pakistan Heart Journal, Journal of Cereal Science, Clinical Thyroidology, Die Diabetologie, and the European Heart Journal's Valvular and Structural Heart Disease title.
- **Lower priority for now:** a set of smaller regional or society open-access cardiology and diabetes titles (e.g. Circulation Reports, CJC Open, JACC Asia, European Heart Journal Open, Heart Rhythm O2, Endocrine Connections, Diabetes Therapy) that are in PubMed but add less to this digest's consumer healthy-aging beat.

---

## Manual trigger

Go to **Actions → Aging & Longevity Research Digest → Run workflow**.

- Leave **category** blank to run all jobs
- Enter an exact category name (e.g. `Cardiology`) to run just that category

---

## GitHub Pages setup

1. Go to **Settings → Pages**
2. Set source to **Deploy from a branch**
3. Branch: `main`, folder: `/ (root)`
4. Save — GitHub will serve `index.html` at the dashboard URL

---

## Required secrets

Add these in **Settings → Secrets and variables → Actions**:

| Secret | Description |
|---|---|
| `ANTHROPIC_API_KEY` | Anthropic API key (`sk-ant-...`) |
| `SERPAPI_KEY` | SerpAPI key for Google News filtering |
| `SUPABASE_URL` | Supabase project URL (dashboard save/delete personalization) |
| `SUPABASE_KEY` | Supabase API key (read-only) |
| `DASHBOARD_REPO_TOKEN` | Token with push access to the shared `research-digest-dashboard` repo |

---

## Repo structure

```
.github/
  workflows/
    aging-longevity-digest.yml   # GitHub Actions workflow (matrix + deploy)
scripts/
  aging_longevity_digest.py      # Main pipeline: PubMed → Claude → JSON artifact
  merge_results.py               # Deploy job: merges artifacts → data/results.json
  extract_journals.py            # Historical one-time extractor (source workbook gone; do NOT re-run, CSVs are hand-maintained)
data/
  Seniors & Aging.csv
  Geriatrics.csv
  Nutritional Sciences.csv
  Endocrinology & Metabolism.csv
  Cardiology.csv
  results.json                   # Auto-generated by deploy job; read by dashboard
index.html                       # GitHub Pages dashboard
requirements.txt
```

---

## Dashboard study card fields

Each study card shows:

- **Headline** — plain-language present-tense summary
- **Relevance score** — 1–10, weighted for aging/longevity journalism fit
- **Category & journal** — source metadata
- **Groundbreaking type** — Counterintuitive / Overturns prior research / First-in-class / Aging finding
- **Media coverage** — SERPAPI verification status
- **The study** — what was done, who participated (N=X, age range), key finding
- **Why it matters** — real-world significance for healthy aging
- **Caveats** — limitations flagged automatically
- **Fact-check note** — corrections made during the Claude pass
- **Pitch angles** — one expandable block per publication type (AARP The Magazine, Prevention, Next Avenue, EatingWell, Everyday Health, Health.com, Verywell Health, Women's Health Magazine, Sixty and Me, Woman's World, First for Women)
- **Status** — New / Saved / Pitched / Passed (tracked in your browser)
