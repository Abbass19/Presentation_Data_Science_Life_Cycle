# MODEL PLANNING — FILE INVENTORY (Step 1)

Status: all files read one by one. No presentation material created yet.
Evidence labels used throughout the project: **[A]** the files say it · **[B]** I infer it · **[C]** general theory · **[D]** my recommendation.

## 0. How to read dates in this repo

- **Filesystem dates are NOT reliable.** All Obsidian notes show 26 Apr 2026 (the date they were copied into the repo). Note 8 shows 13 Sep 2026 (last edited, not written).
- **Better date clues [A]:** embedded image names inside the notes (`Pasted image 20251113…`, `…20251114…`, `…20251119…`), the exam brief deadline (20 Oct 2025), the "(November 2025)" title of note 5, the production report named `…26_02_2026.txt`, and the "May 2026" version stamp on the formal docs.
- **Note numbering** is the only ordering signal for undated notes. Note 1 is empty (1 byte). There is no note 11.

Legend for "Contains": **DS** dataset · **DC** deployment constraints · **MC** model candidates · **MA** model architecture · **EX** experiments · **EV** evaluation metrics · **FL** failures · **AL** alternatives · **DEC** model-selection decisions · **IMP** implementation detail · **BM** benchmark · **RK** risks/limitations · **CH** chronology

Reliability: **High** = raw results / first-hand · **Medium** = first-hand but informal, partly LLM-assisted or with internal inconsistencies · **Low** = contradicted elsewhere or off-topic.

---

## 1. Inventory

### 1.1 Old report (earliest material)

| # | File | Type | Date / version evidence | Stage | What it contains | Reliability | Contains |
|---|------|------|-------------------------|-------|------------------|-------------|----------|
| 1 | `Old Report Before I started Job/project.pdf` | PDF, 2 pp | Deadline Mon 20 Oct 2025 (file dated 13 Oct 2025) | **Earliest** | The exam brief: build an AI system for Arabic patient feedback; GLiNER NER; STT (Whisper/VOSK); multi-class feedback classification (e.g. medical / administrative / service quality); **"fully offline environment… data privacy, security, compliance with hospital IT policies"**. | High (it is the original requirement) | DC, CH |
| 2 | `Old Report Before I started Job/Report.pdf` | PDF, 11 pp | File dated 10 Nov 2025; responds to #1 | **Early** | First modelling wave: **111 records** (stated "very small"), Domain/Category imbalance, db_layer/ai_api/app_api backend. Compares **Multi-head AraBERT** (`aubmindlab/bert-base-arabertv2`, 5 heads), **TF-IDF + MultiOutput LR**, **SBERT embeddings + LR**. Picks SBERT+LR ("similar to Multi-head BERT, lighter"). GLiNER NER and Whisper (20 recorded clips) tested. Domain acc 0.609, Category acc 0.478 (best). | Medium. **Anomaly:** the Multi-head BERT and SBERT result tables are numerically identical; the text describes sentiment-aspect heads (Pricing, Appointments…) but the results shown are Domain/Category. | DS, MC, MA, EX, EV, FL, AL, DEC, BM, RK |

### 1.2 Excel

| # | File | Type | Date / version evidence | Stage | What it contains | Reliability | Contains |
|---|------|------|-------------------------|-------|------------------|-------------|----------|
| 3 | `Excel for Model Waves/Model Results.xlsx` | 1 sheet, B3:Z56 | Undated. Content spans Nov 2025 (flat/multi-head) → later (hierarchical, BERT vs MPNet) | **Early → Intermediate** | Three blocks. **(a)** Single-head LR/RF vs Multi-head LR vs Stacked vs Multi-head BERT vs Hierarchical, across 8 targets (F1 macro + accuracy). **(b)** Text-source experiment: Text 1 vs Text 23 vs Text 123. **(c)** Hierarchical sub-models: BERT vs MPNET embeddings × LR/RF/XGB (+ empty "Weight" rows). Contains a **hand-written warning**: *"All the numbers on the right, in the block are wrong, for they have used text 1 to classify features for text 2"*, and an "Out of Scope" mark on classification_ar. | High as raw evidence, **but** has flagged-wrong cells, apparent paste errors (stacked Domain F1 = 0.4848 = Text123 Domain; stacked Category F1 identical), and many empty cells | EX, EV, FL, AL, DEC, BM |

### 1.3 Obsidian notes (the real chronology)

| # | File | Date / version evidence | Stage | What it contains | Reliability | Contains |
|---|------|-------------------------|-------|------------------|-------------|----------|
| 4 | `0. Resource Action Notes & Exploration.md` | Undated; pre-modelling | Early | 4 reference sources (HCAT paper, supplement, SaferCare Victoria, NUS 2025 article). Open questions: is the 32-class target an HCAT concept or a local invention? Follow the local dataset's protocol or HCAT's? Column naming (Arabic/English mix). 27 DB columns listed. | Medium (thinking notes) | DS, RK, AL |
| 5 | `1. Virtual Environment Setup.md` | — | — | **Empty** (1 byte). | — | — |
| 6 | `2. Data Exploration.md` | Pre-training, "500 records" | Early | Questions to ask of the data; field-by-field map of which columns need a classifier, NER/rules, or matching tables. Class counts per field (3, 8, 21, 34, 3, 3, 5, 3, 3). | Medium (counts differ from later docs, see flags) | DS |
| 7 | `3. Proprocessing.md` | Pre-training | Early | Plan: Excel → database; embeddings of text 1/2/3 and combinations; mapping file for encode/decode. | Medium | DS, IMP |
| 8 | `4. Model Training.md` (30 KB, the key note) | Images dated 13–14 Nov 2025 | Early → Intermediate | The first full evaluation. Defines 9 targets from 2 perspectives (patient text → 4, hospital text → 5). Sets **≥0.8 accuracy** go/no-go (justified by JMIR paper). Quotes the **JMIR/Koh 2025 LLM benchmark**. Per-target analysis: distribution, text sample, per-class LR/RF reports, "actions if low performance", "consequence if used as-is". Domain 0.817/0.722 (LR); Category 0.602/0.448; Subcategory 0.452/0.220; Severity RF 0.828 acc / **0.401 F1**; Stage 0.567/0.390; Harm 0.473/0.244. **Decision record after meeting with Dr. Hussein.** Includes the "you only have 500 samples… 60% is solid" argument. | Medium–High (raw reports are first-hand; commentary partly LLM-written) | DS, MC, EX, EV, FL, AL, DEC, BM, RK |
| 9 | `5. GPT Report (For Dr. Hussein).md` | Titled "November 2025" | Early → Intermediate | LLM-generated summary report of note 4: 4 model types (single-head, multi-head LR, stacked, multi-head BERT fine-tuned **locally on CPU**), feasibility verdicts, 5-phase plan (augment ×20, multi-head, BERT vs ML, hybrid LLM, UI). | **Medium–Low.** Table columns look shifted; Severity row (0.420/0.710) disagrees with Excel (0.314/0.677). Inherits the flagged-wrong Stage/Harm/IOT numbers. | DS, MC, MA, EV, DEC, BM |
| 10 | `6. A Leap of Faith ; A Sudden Understanding.md` | After #9; undated | Intermediate | **The turning point.** Observation that Stacked beat LR because it sat after the strong Domain model → labels are dependent. Parallel multi-head heads can't capture this. LLM reply proposes decision pipeline / conditional heads / Bayesian network, and a **ranked list of 5 solutions** (LR, stacked, XGBoost, ordinal LR, better embeddings). Chosen embedding: `paraphrase-multilingual-mpnet-base-v2`. | High for the reasoning trail; the "no one has solved this" claims are rhetorical | MC, MA, AL, DEC, CH |
| 11 | `6.1 What are the missing steps we didn't do.md` | After #10 | Intermediate | Self-critique: should have studied High vs Low severity statistically; mistake of evaluating all models in parallel and over-generalising from the Severity finding. | High (candid process lesson) | FL, DEC |
| 12 | `6.2 The Solutions To take and Leave.md` | After #10 | Intermediate | Action list; **per-target stacked-input design** (Category←Domain; Sub←Domain+Cat; Severity←all+hospital text; Stage←hospital+Dom+Cat; Harm←everything); descriptions of the **5 chosen** solutions; **3 explicitly abandoned** classes: parallel multi-head NN, Bayesian networks, CRFs, with reasons (500 records, CPU, complexity). | High for decisions; the abandonment rationale is partly LLM prose | MC, MA, AL, DEC, RK |
| 13 | `7. Model's Plan.md` | After #12 | Intermediate | One-page plan per target (Domain "Good", Category "Needs Data", Subcategory "Needs merging", classification_ar "Will not work", Severity "Needs structure", Stage "Needs redesign", Harm "Hardest", IOT "Simple"): which embeddings, which algorithms, class weights, stacking inputs. | High (this is literally the model plan) | MC, MA, DEC |
| 14 | `8. Another Aha Moment.md` | Content ≈ Nov 2025; file edited Sep 2026 | Intermediate | **Hierarchy discovery**: Domain→Category→Sub-Category→classification_ar tree is non-overlapping; "10-minute" manual check; the independence assumption was wrong; "Hierarchical Classification Model" = ~11 models up to sub-category; lesson: *how to test this assumption quickly next time*. | High (first-hand) but the tree listing is partial and the prose is garbled | EX, FL, DEC, CH |
| 15 | `9. The After Math.md` | After #14 | Intermediate | Hierarchy "proved best fit"; open work: Severity, Stage, Harm — which text, which previous labels; plan to compute pairwise label correlations. | Medium | DEC, CH |
| 16 | `10. The New Plan.md` (15 KB) | Image dated 19 Nov 2025 | Intermediate → Late | Target-specific designs: **Severity** = ordinal LR on text123 + engineered features; **Stage** = LR on hospital text + regex rules, then the **semantic-projection** idea (22→15 metric vocabularies, centroids, dot-product projection, ≤6 sentences, max-aggregation); **Harm** = two-stage (binary High/Low + 7-class). Includes the maths and the "needed corrections" (standardise embeddings; use full sentences, not keyword bags). | High on intent. **Note:** the metric lists are struck through (unclear if done or dropped), and the 15 metrics differ from the 16 in doc 04. | MA, EX, DEC, IMP, RK |
| 17 | `12. Some Question on Embeddings.md` | After #16 | Late | Conceptual questions about embeddings (magnitude, size, sentence vs keyword models). | Low value for evidence; useful as "knowledge gap" colour | RK |
| 18 | `13. A Very Critical Moment.md` | Undated | Late? | Short reflective note: pick only three models to train; collect 500 records for each; "if it doesn't work I should say so like a professional"; could use an LLM to create data. Meaning of "3 models… mix the 3 in a binary way" is unclear. | Low (cryptic) | DEC, RK |
| 19 | `14. Database Stuff.md` | Undated; mostly software | Late | Excel→database field mapping (Hanan's Excel vs Hussein Borje's SQL schema); options: continue on existing DB or build an independent one (SQLite chosen later). | Low for Model Planning | IMP |
| 20 | `15. A List of Turns.md` | Undated | Late | To-do list: Node.js + FastAPI UI; independent DB; **"change the sentence transformer to less than 3 seconds"**; make training an easy mechanism. | Medium — the only explicit **latency target** (<3 s) | DC, IMP |
| 21 | `16. The UI Description.md` | Undated | Late | UI pages, table filters, export. | **Low — off-topic** for this presentation | — |

### 1.4 Formal technical documentation (May 2026, retrospective)

All are "v1.0, May 2026"; file times 11 May 2026 (08 → 8 May for prompts). **They are retrospective syntheses**, derived from the codebase + Obsidian notes, and the `Prompts & Answers.md` file shows they were produced through a staged **LLM-assisted workflow** ("do NOT hallucinate metrics"). Treat as *late, curated, sometimes over-polished*. Note: the **internal "Document NN" numbers do not match file numbers** (e.g. file 01 says "Document 07").

| # | File | What it contains | Reliability | Contains |
|---|------|------------------|-------------|----------|
| 22 | `01_HCAT_Taxonomy.md` | HCAT framework, RAH adaptation (Arabic, **476** clean records from Jan 2025), 9 dimensions, definitions, per-label distributions, **label dependency graph**, annotation challenges, data-quality fixes. | Medium. **Category distribution appears mislabelled** (see flag F2). | DS, MA, RK, CH |
| 23 | `02_Dataset_Documentation.md` | Origin (real, non-synthetic, labelled by complaint officers), 476 = 380 train / 96 test (80/20 random), three text fields with lengths (patient avg 332 chars), embedding columns, preprocessing stages, text-source findings, limitations, JMIR comparison. | Medium–High | DS, DC, EV, RK, BM |
| 24 | `03_AI_System_Design.md` | **Constraint → design decision flow**; embedding model choice table (AraBERT, mBERT, AraBART/CAMeL, ada, larger ST rejected, with reasons); hierarchical pipeline; Severity/Stage/Harm/IOT designs; NER; human-in-the-loop; retraining at 1.5×/2×/2.5×; AI risk table. | Medium. Figure 3 shows Domain as "XGBoost 82.0%"; conflicts with doc 04 (flag F6). | DC, MC, MA, AL, DEC, RK |
| 25 | `04_Model_Experiments.md` | **The most important formal doc.** Phases 1→4 (flat → stacked → hierarchical → target-specific), production report 26 Feb 2026 (18 models), abandoned approaches, JMIR comparison, final model selection table. | Medium. Several internal and cross-file inconsistencies (flags F3–F8). | EX, EV, FL, AL, DEC, BM, CH |
| 26 | `05_System_Overview.md` | Problem context, goals, components, roles, deployment summary. | High on context; **mostly out of scope** | DC (brief) |
| 27 | `06_Workflow_Model.md` | Complaint workflow, subcases, approvals, RCA. | **Out of scope** (only "human reviews AI predictions" step is relevant) | — |
| 28 | `07_Technical_Architecture.md` | Backend layers, DB, ML integration (in-process, preloaded at start-up). | **Out of scope** except §13 ML integration | IMP |
| 29 | `08_Deployment_Constraints.md` | Offline single VM, Windows Server 2025, **no GPU, no internet**, RAM estimates (2–7 GB total, *estimated, not measured*), 0.5–2 s per embedding, 50–120 s start-up, manual model updates. What each constraint ruled out (cloud APIs, fine-tuning, LLM >1B). | High for constraints; RAM/latency values are **estimates** | DC, RK |
| 30 | `09_Benchmark_Results.md` | Consolidated metrics for all phases + JMIR comparison + operational interpretation + the 0.8 threshold. | Medium. **Metric definitions drift** (flag F5); "beats GPT-4o" claims compare unlike things (flag F7). | EV, BM, DEC |
| 31 | `10_Testing_Methodology.md` | Software/system testing (UI, workflow, 10-complaint reporting benchmark). **§8 AI Model Validation** is the only relevant part: held-out test, macro F1 primary, regression gate (no model drops >5 pp). | **Out of scope** except §8 | EV |
| 32 | `11_Ethics_And_Privacy.md` | Privacy, human-oversight gates, bias risks, **Harm recall = 0** as an acknowledged safety risk, no ethics review. | High (candid) | RK |
| 33 | `12_Limitations_And_Future_Work.md` | Dataset/AI limitations, **~50 examples per class** rule of thumb (→ ~4,000 records for 78 classes), future: more data, Arabic BERT fine-tune if GPU, multi-label stage, harm rebalancing, **local LLM (Mistral 7B / Llama 3.1 8B) if GPU**, data synthesis, 8 lessons learned. | High for planning-relevant lessons | RK, AL, CH |
| 34 | `Prompts & Answers.md` | The 7-step LLM prompt workflow used to write the docs. "Answers" section empty. | High as provenance; no project facts | CH |

### 1.5 Duplicates / non-content

| # | File | Finding |
|---|------|---------|
| 35 | `Project Technical Documentation/PDF Version/*.pdf` (12 files) | **Renderings of files 22–33.** Page-level text sizes match the Markdown; no independent content found. Not read again line-by-line; spot-checked for key numbers (476, 0.635, recall statements). |
| 36 | `Project Technical Documentation/Documentation Procedure.rar` | **Duplicate archive** of files 22–34. File names, sizes and timestamps are identical to the loose Markdown files (verified with `7z l`). Not extracted. |
| 37 | `.gitattributes` | Git config, no content. |

**Not in the repo:** the Obsidian screenshots (`Pasted image …png`), the raw `classification_training_report_26_02_2026.txt`, per-feature `*_metrics.txt` archive logs, the codebase. Several claims therefore cannot be checked against raw output.

---

## 2. Early flags (contradictions and anomalies found while reading)

These are logged now because they change what the presentation can safely say. Full treatment goes into the dossier and `MODEL_PLANNING_OPEN_QUESTIONS.md`.

**F1 — The dataset has no single size.** 111 records (Oct–Nov 2025 report) → "500" (notes 2 and 4; test sets of n=90–93 imply ≈465–485) → **485** (`patient_feedback_encoded_Old`) → **476** (cleaned, "active"; 380/96 split). The formal docs say *all* metrics are on the 96-record test set, yet Phase 1 numbers are the Nov 2025 results (n≈93). [A] The data grew and was cleaned between waves; [B] the early waves ran on the ~485/500 version.

**F2 — Doc 01's Category distribution does not fit its own tree.** Doc 01 says MANAGEMENT = 285, but Environment + Institutional Processes = 29 + 47 = 76. Likewise CLINICAL = 89 vs Quality of Care + Safety = 207. Note 4's counts (1:30, 2:100, 3:175, 4:20, 5:60, 6:50, 7:35) *do* add up to the Domain totals under the Excel/note 8 encoding (Mgmt ≈ 100+175 = 275). [B] The doc 01 counts were attached to the wrong category names (it claims "Quality of Care dominates 37%"; the old report and note 4 imply **Institutional Processes** dominates). I would not quote doc 01's Category pie chart.

**F3 — Production per-branch labels vs training counts.** The Feb 2026 table lists Subcategory "Listening" with 82 training records and "Quality of Care" with 144, which is inconsistent with the dataset distribution under those names; also "3 classes" for branches that have 2 categories. [B] Numeric-ID → name mapping probably differs between encodings. Needs the raw report.

**F4 — "Macro F1" is not always macro F1.** Doc 04 §7.1: Severity per-class F1 = 0.85 / 0 / 0 (macro = **0.28**), yet "F1 macro 0.624/0.637" is reported and described as a real improvement over flat RF 0.401. 0.624 matches the **weighted** F1 (≈0.626). Same for Harm binary: per-class 0.97 / 0 → macro ≈ 0.49, reported 0.902–0.907 (= weighted). In the Feb 2026 table, Recall equals Accuracy in every row, which is the signature of weighted-average recall. But Stage "≈0.25" *is* macro. So tiers in doc 04/09 mix two metrics. [B, derived arithmetic; the formulas are checkable] This is the strongest "metrics mislead" example in the repo, and also a planning lesson in **metric-definition discipline**.

**F5 — Doc 09 claims the test set was never used for selection.** Notes 6.2/7 and the Excel show a "compare all models in a large table → select best per feature" process on the same test metrics, with no validation set or cross-validation mentioned anywhere. [B] Model selection and final reporting used the same 93–96 records.

**F6 — Which embedding gave the 82% Domain result?** Excel hierarchical block: Domain LR on **BERT** embeddings 0.82/0.82; on **MPNET** 0.77/0.80 (LR) and 0.78/0.79 (XGB). Docs say MPNet was used throughout and AraBERT was only "considered and ruled out" (doc 04 §9.4); but multi-head **AraBERT was implemented and benchmarked** in the Oct–Nov 2025 report and in the Excel, and note 5 says BERT was **fine-tuned on CPU** (it was poor: Domain F1 0.27). Also doc 03 Fig. 3 says Domain = "XGBoost 82.0%", doc 04 says LR 82.0%, and the **Feb 2026 production Domain is 57.3% / F1 0.582** ("different split" is the only explanation given).

**F7 — The JMIR comparison compares unlike things.** HCAT "Category 85.5%" is the 2-class Environment-vs-Institutional-Processes branch (n=61; majority baseline ≈ 64%) vs GPT-4o's full 7-class 69.8%. "Harm 93.4%" is binary with **0 of 6 High-Harm cases found** vs GPT-4o's 5-class 74.2%. Different language, hospital type, taxonomy, and test size (~93 vs 1,816). Cohen's κ is listed as a metric but **no κ is reported for HCAT Insight**. Majority-class baselines are never reported. [A for what is written; B for the critique]

**F8 — Stage redesign rests on a flagged-wrong baseline.** Note 10 justifies the semantic-projection redesign by "normal procedure was useless (Accuracy 0.444, F1 0.189)". Those figures appear in the Excel block carrying the warning that text 1 was used for text-2 features. The corrected Text-23 LR in the Excel gets F1 **0.46** / acc 0.578, higher than the final projection model (macro F1 ≈ 0.25, acc 45.6%). [B] Needs your confirmation of which cells the warning covers.

**F9 — Smaller items.** Stacked Domain/Category F1 are identical in the Excel (0.4848; likely paste error). Doc 04 says stacking was "statistically significant enough" (+0.037 F1 on n≈93; no test shown; note 6 only says "slightly better"). Severity 73.6%/0.624 vs 72.6%/0.637 in two places. Stage uses "15" metrics in docs 03/10 but the doc 04 table lists 16 and different names than note 10. "Vocab model … 991 records, balanced" appears once and is unexplained. Class counts per target vary: sub-category 21/26/27/31, severity 3/4/6, harm 5/6/7, stage 6/8, IOT 2/3. Old report and Excel's Multi-head BERT = SBERT numbers identical.

**F10 — What was never tested [A/B].** No LLM (cloud or local) was ever run on HCAT data; it was *excluded by constraints* (docs 03, 08) or listed as *future/hybrid* (notes 4–5, doc 12). Bayesian networks and CRFs were *considered and rejected* (note 6.2). Ordinal LR, XGBoost, class weights, per-feature folders were *planned in notes 6.2/7* — the Excel shows XGBoost/RF rows filled only partially and the "(Weight)" rows empty. Cross-validation, confidence intervals, and calibration are not mentioned in any file.

---

## 3. Provisional wave sketch (to be validated, not yet the final story)

Derived only from file evidence; every step cites at least one file above.

1. **Brief & constraints (Oct 2025)** — offline, privacy, Arabic, multi-class (#1).
2. **Wave 0: 111 records** — three model families on Domain/Category; SBERT+LR chosen for being lighter/semantically richer (#2).
3. **Exploration with ~500 records, 9 targets from 2 texts (Nov 2025)** — questions about taxonomy, text roles (#4, #6, #7).
4. **Wave 1: flat baselines + multi-head + stacked + BERT; go/no-go ≥0.8; JMIR benchmark (Nov 2025)** — LR/RF per target; many targets "misleading" or "unusable"; decision after meeting with Dr. Hussein (#8, #9, Excel a/b).
5. **Decision Moment: labels are dependent** (stacked > LR after strong Domain) (#10).
6. **Wave 2: plan v2** — five solutions chosen, three abandoned, MPNet adopted, per-target stacked inputs (#11–#13).
7. **Decision Moment: the taxonomy is a tree** → hierarchical pipeline (#14, #15, Excel c).
8. **Wave 3: target-specific plans** — ordinal Severity, rule+projection Stage, two-stage Harm (#16).
9. **Production (Feb 2026)** — 18 models, per-tier reliability, human-in-loop gates, retraining at 1.5×/2×/2.5× (docs 03, 04, 09, 11).
10. **Retrospective documentation (May 2026)** (docs 22–33).

---

## 4. Next step

Awaiting your go-ahead (and answers to the questions below) before writing `MODEL_PLANNING_RESEARCH_DOSSIER.md`.

**Questions I need you to answer (they affect the story):**
1. Did the "Warning" cell in the Excel apply to the Stage / Harm / IOT columns of block (a), and therefore to the numbers quoted in notes 5 and 10?
2. Which embedding model produced the 82% hierarchical Domain result: BERT/AraBERT or MPNet?
3. Which dataset version was used for Wave 1 (≈485 or 500) and which for the hierarchical wave (476)? Is the Feb 2026 production Domain (57.3%) on a different split, or was the earlier 82% a lucky split?
4. In the Feb 2026 report, are Precision/Recall/F1 weighted averages (Recall = Accuracy in every row suggests so)? Can you share the raw `classification_training_report_26_02_2026.txt`?
5. Is the Category distribution in doc 01 mislabelled (F2)? Which category is actually most frequent?
6. Do you have the VM's actual RAM/CPU specs and measured embedding latency, or only the estimates in doc 08?
7. Were any LLMs (cloud or local) ever tried informally on HCAT complaints, even just as a quick check?
