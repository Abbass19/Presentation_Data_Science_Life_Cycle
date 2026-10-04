# MODEL PLANNING RESEARCH DOSSIER — HCAT Insight

Evidence base for a presentation on **Model Planning** (Data Analytics Lifecycle, task 7), using HCAT Insight as the case study.
Companion files: `MODEL_PLANNING_FILE_INVENTORY.md`, `MODEL_PLANNING_OPEN_QUESTIONS.md`.

## Conventions

**Evidence labels:** **[A]** repository file says it · **[U]** the author (user) told me in conversation, not in any file · **[B]** my inference · **[C]** general theory · **[D]** my recommendation for the presentation.

**Status words for approaches (used strictly):** *considered* (only discussed) · *planned* (written as a to-do) · *implemented* (code existed) · *benchmarked* (numbers exist in files) · *deployed* (in production per docs) · *analytical alternative* (mine, not in the project).

**Source keys:**
`OR-P` = Old Report/project.pdf · `OR-R` = Old Report/Report.pdf · `XL` = Excel/Model Results.xlsx · `N0…N16` = Obsidian notes by number · `D01…D12` = Project Technical Documentation files by file number (D01 Taxonomy, D02 Dataset, D03 AI Design, D04 Experiments, D05 Overview, D06 Workflow, D07 Architecture, D08 Deployment, D09 Benchmarks, D10 Testing, D11 Ethics, D12 Limitations) · `PA` = Prompts & Answers.md.

**Reference-number policy (agreed with the author [U]):** the Excel is early work; where numbers conflict, **the official documentation (D01–D12) is the reference**. Excel and notes are used to show *what the plan and evidence looked like at that moment*. The production embedding model is **MPNet** (`paraphrase-multilingual-mpnet-base-v2`) [A: D02, D03; U confirms]. Caveats inside the official docs themselves are still listed in §12 because a careful audience may spot them.

---

## 1. File inventory

See `MODEL_PLANNING_FILE_INVENTORY.md` (full table). Summary: 2 old-report PDFs · 1 Excel · 19 Obsidian notes (1 empty, 3 off-topic) · 12 formal docs (5 mostly off-topic) · Prompts file · duplicates (12 PDFs, 1 RAR).

---

## 2. Project chronology (what the files say)

| When | Event | Source |
|------|-------|--------|
| Oct 2025 (deadline 20 Oct) | Exam brief: AI system for Arabic patient feedback; **fully offline** "ensuring data privacy, security and compliance with hospital IT policies"; NER, STT, multi-class classification | OR-P [A] |
| First month of the job work | Author decides **early** to store embeddings and build an ML engine that can retrain models re-using the stored embeddings | [U] (consistent with D02 §11, D03 §5.3, D03 §10) |
| ~Oct–10 Nov 2025 | **Wave 0**: 111 records; multi-head AraBERT vs TF-IDF+LR vs SBERT+LR; SBERT+LR selected | OR-R [A] |
| Nov 2025 | Real data arrives (~500 records, 9 targets from 2 texts); exploration and preprocessing into a database with embeddings | N2, N3 [A] |
| 13–14 Nov 2025 | **Wave 1**: flat LR/RF, multi-head LR, stacked, multi-head BERT. Per-target analysis. Success threshold ≥0.8 accuracy. Meeting with Dr. Hussein → decisions | N4, N5, XL [A] |
| After N5 (≈ mid-Nov) | **Insight**: labels are dependent (stacked > LR after Domain). Five solutions chosen, three abandoned. MPNet adopted | N6, N6.1, N6.2, N7 [A] |
| ≈ Nov 2025 | **Insight**: taxonomy is a tree → hierarchical pipeline | N8, N9 [A] |
| ≥19 Nov 2025 | **Wave 3**: target-specific plans: ordinal Severity; Stage rules + semantic projection; two-stage Harm | N10 [A] |
| Later (undated) | Scope-narrowing and reflection ("three models, no more"); UI/DB work | N13–N16 [A] |
| 26 Feb 2026 | Production training report: 380 training records, **18 models**, average F1 0.635 | D04 §8, D09 §7 [A] |
| May 2026 | Formal documentation written retrospectively (LLM-assisted workflow) | D01–D12, PA [A] |

---

## 3. Dataset evolution

| Stage | Records | Notes | Source |
|-------|---------|-------|--------|
| Wave 0 (Oct–Nov 2025) | **111** | "very small… data sparsity and class imbalance"; categories with 1–7 records | OR-R [A] |
| Wave 1 (Nov 2025) | "**500**" (test sets n=90–93) | 9 targets from 3 text fields; 20 columns; Arabic/English column names | N2, N4 [A] |
| Archived version | **485** (`patient_feedback_encoded_Old`) | Retained for audit | D02, D07 [A] |
| Official dataset | **476** clean (380 train / 96 test, 80/20 random split, same split across all models) | Jan 2025 onward, single specialised cardiac hospital (RAH) | D01, D02 [A] |
| Reference study | JMIR/Koh: **1,816** English GP complaints | 4× larger; GPT-4o zero-shot | D02 §14, N4 [A] |

**Dataset properties relevant to planning [A: D02 unless stated]:**
- Real operational data, labels assigned by complaint officers; **no inter-annotator agreement**; single label forced even for multi-category/stage complaints.
- Language: Modern Standard Arabic + Lebanese/Iraqi dialect + English clinical terms. Used as the reason to choose a **multilingual sentence transformer** over Arabic-only models.
- Text: patient text avg 332 chars (max 1,457), hospital immediate action avg 155, taken action avg 138. Long, emotional, causal, mixed topics (cognitive complexity for small models).
- **Two perspectives:** patient text (Text 1) and hospital response texts (Text 2, Text 3); best text input differs by target (D02 §12; N4).
- Embeddings: 768-dim, pre-computed and stored (five combined variants + six sentence-level) → ~2–4 GB SQLite DB.
- Targets: Domain 3 → Category 7 → Sub-Category 26 (strict tree); fine-grained 78 Arabic / 73 English; Severity 3 (ordinal); Stage 6; Harm 5 (ordinal); Improvement Opportunity Type 3 (Ordinary 92.6%, Red Flag 19 records, Never Event <1%) (D01).
- Verified distributions (D01 and D02, subject to caveat C2 in §12): Domain MANAGEMENT 285 / RELATIONAL 102 / CLINICAL 89; Severity LOW 317 / MEDIUM 125 / HIGH 28; Stage "Care on the Ward" 39%; Harm skewed (Severe 186, Death 128, Moderate 7) because it is a cardiac hospital; Improvement type 92.6% Ordinary.
- Dataset grows continuously; retraining at 1.5×/2×/2.5× (D03 §10).

---

## 4. Deployment constraints (and their planning consequences)

| Constraint | Evidence | Consequence in the plan | Status |
|-----------|----------|-------------------------|--------|
| **No internet** (offline hospital VM) | OR-P §4; D08 §3; D03 §3 | No cloud LLM/embedding APIs; all weights pre-downloaded; manual model updates | [A] |
| **Privacy / hospital IT policy** | OR-P §4; D11 | Offline is mandated partly for privacy; patient name excluded from ML inputs | [A] |
| **CPU only, no GPU** | D08 §4; D03 §3 | No fine-tuning, no large LLM; lightweight classifiers on frozen embeddings | [A] |
| **Limited RAM** | D03 §3 (qualitative); D08 §4.2 gives *estimates* (~2–7 GB total for all components incl. SQL Server) | Lazy loading, singleton, pre-computed embeddings | Estimates [A]; **not measured** [U] |
| **RAM prevented loading even Whisper medium; no LLM was ever loaded** | [U] ("eight [GB] of RAM or less" — I read this as ~8 GB; to be confirmed) | LLM excluded *by constraint, not by experiment*; STT limited to Faster-Whisper | [U], consistent with D08 §4.3 |
| **Latency target** | N15: sentence transformer should take **< 3 s**; D08 estimates 0.5–2 s per embedding on CPU; start-up 50–120 s | Embedding once per complaint, shared by all classifiers; all classifiers near-instant | [A] (target) / estimate |
| **Retraining must be cheap** | D02 §11; D03 §5.3, §10 | Embeddings pre-computed and stored (~2–4 GB DB) so retraining never re-embeds | [A]; decision date [U] |
| **Single VM, Windows Server 2025** | D08 §2, §5 | In-process model serving | [A] (low relevance) |
| **Human must stay in the loop** | D03 §11; D11 §8 | Predictions are suggestions; High severity/harm/Red Flag need human confirmation | [A] |

Constraint-to-design flow already drawn in D03 Fig. 1: *No internet → no cloud APIs; CPU only → no large LLMs; limited RAM → no fine-tuning; 476 records → no deep learning ⇒ pre-trained sentence embeddings + lightweight classifiers (LR, RF, XGBoost), fully offline on CPU.* **[D]** This is the best single "constraint funnel" for the presentation.

---

## 5. Model candidates and their status

| Candidate | Status | Evidence |
|-----------|--------|----------|
| TF-IDF + (MultiOutput) Logistic Regression | **Benchmarked** (Wave 0: Domain acc 0.391, Category 0.435) | OR-R |
| Multi-head AraBERT (5 linear heads on [CLS]) | **Implemented, benchmarked** (Wave 0: Domain 0.609, Category 0.478) | OR-R |
| SBERT embeddings + LR per target | **Implemented, benchmarked, selected in Wave 0** | OR-R |
| Single-head LR per target on embeddings | **Benchmarked** (Wave 1 flat baseline) | N4, D04 §3 |
| Random Forest | **Benchmarked** | N4, D04 |
| XGBoost | **Planned in N6.2/N7; benchmarked** in hierarchical wave (and Production "XGB/LR") | N6.2, D04 §6, D03 |
| Multi-head LR (2 models, 4–5 heads) | **Benchmarked** (identical to single-head for Domain/Category) | N5 |
| Multi-head BERT end-to-end (fine-tuned on CPU) | **Benchmarked** (very poor: Domain acc 0.674 / F1 0.27 at ~380 records) | N5; XL; D04 §9.2 |
| Stacked sequential (probabilities of upstream model as features) | **Implemented, benchmarked** | D04 §5 |
| Hierarchical pipeline (model per Domain/Category branch) | **Implemented, benchmarked, deployed** | D04 §6, D03 §6.1 |
| Ordinal Logistic Regression (Severity only) | **Implemented, deployed** | D04 §7.1 |
| Semantic projection Stage model (15 metric vocabularies, dot-product, max over ≤6 sentences) | **Implemented, deployed** | N10, D03 §6.3 |
| Two-stage Harm (binary safety + fine-grained ordinal) | **Implemented, deployed** | D04 §7.3 |
| Regex/rule-based stage hints | **Planned** (N10); final design uses projection instead | N10 |
| Better embeddings: MPNet multilingual | **Implemented, adopted** | N6, D03 §5 |
| Arabic SBERT / AraBERT-as-embedder | *Considered*, ruled out (token-level, needs pooling, GPU to fine-tune); note AraBERT *was* used in Wave 0 | D03 §5.1, D04 §9.4, OR-R |
| mBERT, AraBART, CAMeL, larger sentence-transformers, OpenAI ada | *Considered*, rejected with reasons | D03 §5.1 |
| Bayesian networks / PGMs | *Considered*, abandoned (500 samples, complexity) | N6.2 |
| CRF / structured prediction | *Considered*, abandoned | N6.2 |
| Parallel multi-head neural | Implemented as BERT multi-head; **rejected** (ignores label dependency; compute) | N6.2, D04 §9.2 |
| **LLM, cloud API** | *Considered* only as benchmark (JMIR) and as "future"; **excluded** by offline rule | N4, D08 §3, D12 §9.5 |
| **LLM, local (Mistral 7B / Llama 3.1 8B)** | *Considered* for future GPU VM; **never loaded** (RAM) | D12 §9.5; [U] |
| Hybrid zero-shot LLM + ML | *Planned* ("Phase 4", never executed) | N5 |
| Synthetic data (×20, paraphrase, back-translation, LLM generation) | *Considered / planned*; D12 says "No synthetic augmentation" was done | N4, N5, D12 §2.1, §9.6 |
| Ordinal regression via regression-then-round, label smoothing, multi-label stage, sequence labelling | *Considered* (N4 "actions if low performance") | N4 |

---

## 6. Experiments (official numbers, with where each came from)

All numbers from official docs unless stated. Test split n≈96 (91–93 per target after nulls).

**6.1 Wave 0 (111 records, OR-R)**
- Category accuracy: AraBERT multi-head 0.48 · TF-IDF 0.435 · SBERT+LR 0.478. Domain accuracy: 0.609 · 0.391 · 0.609.
- Conclusion in report: SBERT+LR best choice (lighter, semantically richer); TF-IDF only if resources very limited.

**6.2 Phase 1 – flat baselines (D04 §3.1, ≈Nov 2025)**
| Target | LR acc / macro-F1 | RF acc / macro-F1 | Doc's own comment |
|---|---|---|---|
| Domain (3) | 79.6% / 0.696 | 74.2% / 0.503 | Good |
| Category (7) | 60.2% / 0.448 | 56.9% / 0.241 | Needs improvement |
| Sub-Category (27+) | 45.2% / 0.220 | 44.1% / 0.174 | Poor |
| Classification_ar (78) | 31.2% / 0.152 | 25.8% / 0.083 | Unusable |
| Severity (3) | 74.2% / 0.396 | 82.8% / 0.401 | Misleading (dominant class) |
| Stage (6) | 56.7% / 0.390 | 47.8% / 0.230 | — |
| Harm (6) | 47.3% / 0.244 | 41.8% / 0.247 | Poor |
| Improvement type (2) | 97.8% / 0.744 | 98.9% / 0.497 | Misleading (dominant class) |

**6.3 Phase 2 – stacked (D04 §5)**: Category 60.2%/0.448 → **63.4%/0.485** (+0.037 F1) when Domain probabilities are appended (771 features); Classification_ar 30.1%/0.124 (no gain).

**6.4 Phase 3 – hierarchical (D04 §6)**: Domain LR 82.0%/0.722 (RF 72%/0.50, XGB 75%/0.61). Category per Domain: CLINICAL 67%/0.52 (n=18), **MANAGEMENT RF 85%/0.83 (n=61)**, RELATIONAL 64%/0.60 (n=14). Sub-category per Category: **Environment 95%/0.88 (n=21)**, Quality of Care 79%/0.53, Safety 75%/0.33 (n=4), Institutional Processes 56%/0.36, others 60–67% with n=3–6.

**6.5 Target-specific (D04 §7)**: Severity ordinal LR on text123: 73.6% (or 72.6%)/"0.62–0.64"; per-class F1 LOW 0.85, MEDIUM 0, HIGH 0. Stage projection: acc 45.6%, F1 ≈0.25 (Care on the Ward F1 0.64; Discharge, Unspecified 0). Harm binary: 93.4–93.8%, "F1 0.90"; **0 of 6 High-Harm found**. Improvement type: 96.7%/"0.951".

**6.6 Production report, 26 Feb 2026 (D04 §8, D09 §7)**: 18 models; Domain 57.3%/0.582; Category MANAGEMENT 85.5%/0.850, RELATIONAL 82.4%/0.819, CLINICAL 58.3%/0.586; Sub-category 0.50–0.84 (one branch with 0 records); Severity 72.6%/0.637; Harm binary 93.8%/0.907; Harm ordinal High 66.7%/0.533, Low 40.4%/0.282; Improvement type 96.7%/0.951; Classification_en (70 classes) 14.6%/0.143 (not deployed). Average F1 0.635. Tiering: 6 high (>0.80), 7 moderate, 3 low.

**6.7 JMIR/Koh 2025 benchmark (N4, D04 §10)**: GPT-4o zero-shot: Domain 79.4% (κ 0.623), Category 69.8% (κ 0.571), Severity 53.9% (κ 0.226), Stage 66.1% (κ 0.534), Harm 74.2% (κ 0.162); mean concordance 68.8% (GPT-3.5 61.9%). English GP data, 1,816 records.

---

## 7. Failures and abandoned approaches

| Item | What happened | Why | Source |
|------|---------------|-----|--------|
| Classification_ar (78 classes) / Classification_en (73) | Never usable (best F1 0.152; production 0.143) | <5–6 examples per class; D12: ~50 per class needed → ~4,000 records | D04 §9.1, D12 §2.3 |
| Multi-head fine-tuned BERT | Far worse than LR on embeddings (Domain F1 0.27 vs 0.72) | Few records; heads parallel and independent; CPU cost | N5, D04 §9.2 |
| Stacking all the way to Classification_ar | No gain (30.1%) | Class sparsity cannot be fixed by architecture | D04 §9.3 |
| Flat treatment of independent labels | Months of suboptimal flat experiments | Taxonomy structure was in the data from day one | D12 §13 lesson 1; N8 |
| Parallel evaluation of all models at once | Over-generalised the Severity insight to every target; skipped statistical study of High vs Low severity | Author's own self-critique | N6.1 |
| Severity MEDIUM/HIGH | Still unlearnable (F1 0 for both) | 20 and 4 test records | D04 §7.1 |
| Stage | Lower than flat baseline on the official numbers (0.25 vs 0.39) yet deployed as "architecturally preferred" | Multi-stage complaints; weak vocab lists | D04 §7.2, D12 §3.4 |
| Harm binary | 0 recall on High Harm | 85:6 imbalance | D04 §7.3, D11 §9.1 |
| Domain in production | 57.3% vs 82% archive | Explained as "different test partition" | D04 §8.1 |
| Bayesian network, CRF | Abandoned before building | 500 samples, math/engineering cost | N6.2 |
| LLM on the VM | Never attempted | RAM; even Whisper medium was hard to load | [U]; D08 |
| Synthetic augmentation | Never done | — | D12 §2.1 |

---

## 8. Model-planning decision moments (derived from the files)

Format requested: initial assumption → evidence → why insufficient → new understanding → planning decision → result.

### Decision Moment 1 — "The environment decides before the data does" (Oct 2025 – month 1)
- **Initial assumption:** Pick the most capable model for Arabic text classification (BERT-class, or LLM).
- **Evidence:** Brief demands fully offline operation for privacy/IT policy [OR-P]; CPU-only VM, no GPU [D08]; RAM too small even to load Whisper medium [U].
- **Why the old approach was insufficient:** Cloud LLMs and GPU fine-tuning are not available at all.
- **New understanding:** The feasible set is "frozen pre-trained encoder + light classifier".
- **Planning decision [U + D02/D03]:** Early (first month) decision to **pre-compute and store embeddings** and build a retraining engine around them, so retraining needs only classifiers, not the encoder.
- **Result:** All later experiments were cheap to repeat; the 18-model production pipeline and milestone retraining exist because of this choice. *Caveat:* the choice was made *before* experimental evidence; it was a constraint-driven prior, not a measured comparison against an LLM.

### Decision Moment 2 — "More complex ≠ better" (Wave 0, 111 records)
- **Initial assumption:** Multi-head AraBERT (Arabic-specific, end-to-end) should beat simpler options.
- **Evidence:** AraBERT multi-head Domain acc 0.609 / Category 0.478; SBERT+LR identical-or-similar; TF-IDF clearly worse [OR-R]. Later fine-tuned BERT multi-head: Domain F1 0.27 [N5].
- **Why insufficient:** With 111–380 records a deep model can't learn rare classes; it dominates the majority classes only.
- **New understanding:** Quality of the representation (sentence embeddings) matters more than model depth at this data size.
- **Decision:** Keep embeddings + LR as the baseline for everything; judge other models against it [OR-R; N6.2 "LR baseline … always strong"].
- **Result:** Selected SBERT+LR; later MPNet embeddings with LR/RF/XGBoost. *(Note the Wave 0 report's BERT and SBERT tables are numerically identical; see §12 C9.)*

### Decision Moment 3 — "Set the go/no-go threshold, then check whether it is reachable" (13–14 Nov 2025)
- **Initial assumption:** Success = a classifier that helps rather than burdens; threshold ≥0.8 accuracy, "acceptable… not outstanding", justified by JMIR [N4].
- **Evidence:** Only Domain (0.82) and Improvement type reached it; Category 0.60, Sub-category 0.45, Stage 0.57, Harm 0.47 [N4]. JMIR GPT-4o on 1,816 records only reached ~69% average.
- **Why insufficient:** One global threshold on accuracy hides that each target is a different problem, and it is not a fair bar for rare classes.
- **New understanding:** "Each one of those models is a project on its own" [N4]; evaluate per target with a structured questionnaire (concept complexity, which text, distribution, metrics, actions, consequences if used as is).
- **Decision (after meeting with Dr. Hussein) [N4]:** Proceed with the ML models as they are, even the ones that won't work; retrain from scratch with more data; test off-the-shelf NER and Faster-Whisper; leave "launch now vs collect/synthesise more data" open; build self-retraining at 1.5×/2×/2.5× data.
- **Result:** Pipeline became "ship with honest uncertainty + human review + retrain as data grows".

### Decision Moment 4 — "Accuracy is misleading under imbalance" (Nov 2025)
- **Initial assumption:** Accuracy is a usable headline metric (threshold 0.8).
- **Evidence:** Severity RF **82.8% accuracy, macro-F1 0.40**, predicting LOW almost exclusively; Improvement type **98.9% accuracy, F1 0.497**, always Ordinary; Harm binary 93.8% with 0/6 High Harm found [N4; D04 §3, §7].
- **Why insufficient:** The dominant class (60–93% of records) lets a trivial model look excellent while missing the safety-critical class.
- **New understanding:** Per-class recall (especially for rare, high-stakes classes) and macro-averaged metrics are mandatory.
- **Decision:** Macro-F1 declared primary metric; class weights; binary triage for Harm; mandatory human review for High severity/harm/Red Flag [D02 §8; D03 §13; D11 §8].
- **Result:** Better framing of risk. *Residual issue:* reported F1 values for Severity, Harm and Improvement type appear to be weighted-F1 (see §12 C4).

### Decision Moment 5 — "The labels are not independent" (≈ mid-Nov 2025)
- **Initial assumption:** Nine targets can be modelled independently (single head each or parallel heads) [N4: "Single head vs multi head"].
- **Evidence:** The stacked model beat LR for Category once it received the strong Domain model's output (63.4% vs 60.2%; F1 0.485 vs 0.448) [N6; D04 §5].
- **Why insufficient:** Parallel multi-head networks share a base but their heads do not condition on one another; independent models ignore the dependency.
- **New understanding:** Labels form a dependency graph that mirrors the human annotation order (Domain → Category → Sub-category → Severity → Stage → Harm) [N6, D01 §12].
- **Decision:** Reject parallel multi-head networks, Bayesian networks and CRFs (too much data/maths for 500 records); keep **stacked sequential** models with a *different* input set per target [N6.2].
- **Result:** Clear modelling hypothesis, but modest gain (+0.037 F1) and the claim of "statistical significance" in D04 has no test behind it.

### Decision Moment 6 — "It isn't just dependent, it is a tree" (≈ Nov 2025)
- **Initial assumption:** Stacking probabilities is the right way to use the dependency.
- **Evidence:** A quick manual check of Domain→Category→Sub-category co-occurrence showed non-overlapping branches (e.g. Domain 1 → Categories 5, 7 only) [N8]. Flat Category model had to rule out impossible class pairs.
- **Why insufficient:** Stacking still lets the model consider impossible labels; it adds features but does not shrink the label space.
- **New understanding:** Split the data by branch and train a model per branch (≈11 models up to sub-category) [N8, D04 §6].
- **Decision:** Hierarchical pipeline. Author also noted the meta-lesson: *how to test an assumption quickly next time* [N8].
- **Result:** Largest gain in the project: MANAGEMENT category 0.448 (flat) → **0.85 F1** [D09 §9]; Environment sub-category 95%. *Caveat:* gain is partly because the per-branch problem is smaller (2 classes), so the number is not comparable with the 7-class flat model.

### Decision Moment 7 — "Different targets, different inputs and different model families" (≈ 19 Nov 2025 →)
- **Initial assumption:** One universal input (patient text) and one algorithm family for all targets.
- **Evidence:** Text-source experiments (Text 1 vs 23 vs 123): Severity/Harm better with combined text; Stage better with hospital text; Domain/Category/Sub-category best with patient text [N4, N10, D02 §12]. Severity is ordinal and context-dependent; Harm needs hospital-documented outcomes [N6, N10].
- **Why insufficient:** Treating Severity as nominal penalises 3→2 and 3→0 equally; Harm's rare classes can't be learned.
- **New understanding:** Each target needs its own *problem definition*.
- **Decision:** Ordinal LR for Severity; two-stage binary + fine-grained for Harm; hospital-text embedding + vocabulary projection for Stage; plain LR for Improvement type [N10, D03 §6].
- **Result:** Sound designs on paper; mixed evidence: Severity still predicts LOW only; Harm binary misses all High Harm; Stage underperforms flat baseline.

### Decision Moment 8 — "Stage: when the representation is the problem" (Nov 2025)
- **Initial assumption:** A single global embedding is enough to pick the stage of care.
- **Evidence:** Flat Stage F1 0.39/0.23; complaints often span several stages while labels are single [D01 §14.2, N10].
- **Why insufficient:** Mean-pooling over a long narrative dilutes stage cues.
- **New understanding:** Measure *how much each sentence talks about each concept*, then classify the 15 resulting scores [N10, D03 §6.3].
- **Decision:** Semantic projection (centroid per metric, dot product, max over ≤6 sentences).
- **Result:** Interpretable features, but accuracy 45.6% / F1 ≈0.25, below the flat baseline; accepted as "hint only, human review" [D04 §7.2, D11 Fig. 2]. **[D]** Use as an honest example: *a principled design that did not win; plan should say what would count as success.*

### Decision Moment 9 — "Some targets should be dropped from the plan" (Nov 2025 → Feb 2026)
- **Initial assumption:** All nine HCAT-style labels will be automated.
- **Evidence:** 78 classes, ≈6 records per class; stacking and hierarchy did not help (F1 0.12–0.15) [D04 §3, §5; D12 §2.3].
- **Why insufficient:** Architecture cannot create data; "~50 examples per class" rule of thumb ⇒ ~4,000 records.
- **Decision:** Exclude Classification_ar/en from the AI pipeline (manual use only); merge/hold the rest until more data [N6.2, N7, D04 §9.1].
- **Result:** Smaller, honest scope; deployed set of 18 models.

### Decision Moment 10 — "The production environment eliminates attractive approaches" (continuous; stated in D03, D08)
- **Initial assumption:** The best published approach (zero-shot GPT-4o) is the reference solution [N4].
- **Evidence:** No internet; CPU only; limited RAM; author could not even load Whisper medium; an LLM was therefore never tried [U; D08].
- **Why insufficient:** The theoretically strongest candidate is infeasible.
- **New understanding:** Planning is selecting from the *feasible* set and measuring against what the strongest infeasible option reports.
- **Decision:** Embedding-first classical ML; use JMIR figures as an external reference only.
- **Result:** A deployed system; but there is no head-to-head measurement against any LLM on HCAT data (see §12 C7).

---

## 9. Evaluation choices (how evaluation planning evolved)

| Stage | What was used | Evidence |
|---|---|---|
| Wave 0 | Accuracy, precision/recall/F1 per class, macro/weighted F1, mAP | OR-R |
| Wave 1 | Accuracy + macro F1 + per-class report; **go/no-go: accuracy ≥ 0.8** (justified by JMIR) | N4, N5 |
| Planning guidance per target | Macro F1 and Cohen's κ for Domain; per-class F1 for Category; quadratic weighted κ for Severity; per-stage recall; weighted F1/recall for ≥moderate Harm | N4 |
| Official docs | **Macro-F1 primary**; accuracy for context; per-class F1; Cohen's κ "for JMIR comparison" | D04 §2.3, D09 §2 |
| Split | Single random 80/20 split (380/96), same split for all models; no cross-validation, no stratification, no validation set mentioned | D02 §7, D09 §2.2 |
| Regression gate for retraining | No model may fall >5 pp F1; overall mean F1 must not drop | D10 §8.3 |
| External benchmark | JMIR Koh 2025: concordance % and κ | N4, D04 §10 |
| Operational interpretation | Tier 1/2/3 reliability; "0.8 threshold met by 8 of the targets/branches" | D09 §7, §11 |

**Where metrics mislead (best teaching examples):**
1. **Improvement type:** 98.9% accuracy, macro-F1 0.497 → "predicts Ordinary exclusively". [D04 §3.1, §7.4]
2. **Severity RF:** 82.8% accuracy vs macro-F1 0.40; production ordinal model: LOW recall 1.00, MEDIUM/HIGH recall 0. [D04 §7.1]
3. **Harm binary:** 93.8% accuracy, zero recall on High Harm (0/6). [D04 §7.3, D11 §9.1]
4. **Hierarchical gain:** 85% on a 2-class branch with 61 test records; majority baseline ≈64% (39/61), never reported. [B, arithmetic from D04 §6.2, N4 supports]
5. **Metric definition drift** (C4 below).

---

## 10. Risks (risk → why it matters → how detected → planning decision)

| Risk | Why it matters | How detected | Planning decision it influenced |
|---|---|---|---|
| **Tiny dataset** (111 → 476) | Rare classes unlearnable; no deep learning | Wave 0 note; per-class supports | Frozen embeddings + classical ML; synthetic data considered; retraining at 1.5×/2×/2.5× |
| **Severe class imbalance** (60–93% dominant class) | Misleading accuracy | Wave 1 per-class reports | Macro-F1 primary; class weights; binary Harm triage |
| **Classes with a handful of records** | Zero F1 for those classes | Per-class reports | Drop 78/73-class targets; merge rare classes; ≥~50/class guideline |
| **Misleading accuracy** | False confidence | Severity RF, IOT, Harm binary | Per-class recall checks; human review gates |
| **Label dependency ignored** | Impossible class combinations; noise | Stacked > LR | Stacked → hierarchical pipeline |
| **Subjective / noisy labels** | Ceiling on achievable accuracy; no inter-annotator agreement | D01 §14; D02 §13 | Treat results as "suggestions"; plan re-labelling guidelines (N4) |
| **Single forced label for multi-stage complaints** | Label noise for Stage | D01 §14.2 | Stage redesign (projection); multi-label proposed as future |
| **High-dimensional embeddings vs few records** (768 dims vs ≈380) | Overfitting | LR/XGB with class weights; BERT multi-head collapse | LR baseline; avoid fine-tuning |
| **Safety-critical rare classes (High Harm, Red Flag, Never Event)** | Missing them is the costliest error | Harm binary 0/6 | Mandatory human confirmation; threshold calibration and rebalancing planned |
| **Model complexity vs hardware** | LLMs/fine-tuning not loadable | Whisper medium load issue [U]; D08 | Exclude LLM and fine-tuning |
| **Domain/dialect shift** (dialect, English terms, single hospital) | Generalisation | D02 §13 | Multilingual encoder; retraining |
| **Weak validation** (single 96-record split, no CV; test set also used for choosing models) | Optimistic and noisy results; 14–18 test records per branch | D02 §7; notes compare configurations on the same table | *Not addressed in plan* → candidate "lessons learned" (see §12 C5) |
| **Research performance vs operational usefulness** | A "80% model" may still miss the cases that matter | D09 §11; D11 | Tiering and per-target operating rules |
| **Domain model production drop (82% → 57.3%)** | Unexplained variance across partitions | D04 §8.1 | Explained as split difference; [B] suggests high variance of small test sets |

---

## 11. Final model choices (production, per D04 §11 and D03)

| Target | Selected design | Status / reliability (per docs) |
|---|---|---|
| Embeddings | MPNet multilingual, 768-d, frozen, CPU, pre-computed and stored | Deployed |
| Domain | Single classifier on patient text embedding (LR/XGB in different docs) | 82% (archive); 57.3% production report |
| Category | Hierarchical, one model per Domain | MANAGEMENT 0.85, RELATIONAL 0.82 (high), CLINICAL 0.59 (moderate) |
| Sub-category | Hierarchical, one model per Category (XGB) | 0.25–0.88 varying; Environment best |
| Severity | Ordinal LR on text123 | LOW reliable only; human review for MEDIUM/HIGH |
| Stage | Semantic projection (15 metrics) → LR; hospital text | Low confidence; "hint only" |
| Harm | Two-stage: binary safety (text123) + ordinal fine-grained | Binary misses High Harm; human review mandatory |
| Improvement type | LR on patient text | High on Ordinary; Red Flag needs human |
| Classification_ar / _en | **Not deployed** | Unusable |
| NER | GLiNER Arabic v2.1 + rule-based role detection (out of scope for this talk) | Deployed |
| STT | Faster-Whisper planned/loaded (D03 says planned; D07 says loaded at start-up) | Out of scope |
| Operating principle | Human-in-the-loop; retrain at 1.5×/2×/2.5× with regression gate | Designed |

---

## 12. Contradictions and uncertain history

Policy: official docs are the reference (author's instruction). The items below are weaknesses *inside* the official set, or differences between the early evidence and the official set. They may be worth acknowledging on a slide because they are teachable.

| ID | Issue | Evidence | Possible explanation | Presentation handling [D] |
|----|-------|----------|----------------------|----------------------------|
| C1 | Dataset size varies (111 / ~500 / 485 / 476) | OR-R; N2, N4; D02 | **Resolved [U]:** 111 was given before the job; the full ~500 was given at the start of the job; 485/476 are archived/cleaned versions of it | Present as "111 → ~500 → 476 cleaned"; avoid mixing numbers from different versions on one chart |
| C2 | D01 Category distribution does not fit its own Domain tree (e.g. MANAGEMENT 285 vs Environment+Inst. Processes 76) | D01 §5.1–5.2 vs N4 counts | Counts change over time; author: "any number is fine" [U]. Internal inconsistency remains [B] | Do not show Category counts next to Domain counts; use Domain/Severity/Harm/IOT distributions |
| C3 | Production per-branch names vs training counts; "3 classes" for 2-category branches | D04 §8.1 | ID↔name mapping mismatch [B] | Use branch-level highlights only if author can verify |
| C4 | "F1 macro" appears to be **weighted** F1 for Severity, Harm binary, Improvement type; Recall = Accuracy in every row of the Feb 2026 table; Stage uses macro | D04 §7.1, §8.1; D09 §7–8 | Report script printed weighted averages (the author does not remember [U]) | Strong example: show per-class table next to the headline number; state as "recomputed from the per-class table" [B] |
| C5 | D09 says test set not used for selection; notes/Excel show comparisons and best-model selection on the same test metrics; no validation set or CV | D09 §1; N6.2, N7 | Informal workflow | Use as a planning lesson (validation strategy should be planned) |
| C6 | Domain: LR vs XGB (D03 Fig. 3) and 82% vs 57.3% | D03, D04 | Different splits/versions | Use 82% as "archive hierarchical" and mention production re-run lower; or omit Domain production number unless author clarifies |
| C7 | "Beats GPT-4o" comparisons are not like-for-like (2-class branch vs 7-class; binary Harm with 0 recall; different language, hospital, taxonomy, n; no κ for HCAT; no majority baseline) | D04 §10, D09 §10 | Marketing tone of retrospective docs | Present JMIR only as context; use it to teach "benchmark comparability" |
| C8 | AraBERT "only considered" (D04 §9.4) but implemented and benchmarked in Wave 0 and as multi-head BERT in Wave 1 | OR-R, N5, XL | Docs summarise the final reasoning | Describe as "benchmarked early, replaced by MPNet" |
| C9 | OR-R: Multi-head BERT and SBERT results tables are identical | OR-R pp. 6, 8 | Possible copy/paste | Do not use the BERT-vs-SBERT numbers as proof; use only the qualitative selection rationale |
| C10 | Metric counts differ (Stage 15 vs 16 metrics; vocab names differ between N10 and D04); "991 records, balanced" appears once | D03, D04 §7.2, N10 | Evolution of the vocabulary lists | Say "≈15 metrics" |
| C11 | Class counts per target vary (Sub-cat 21/26/27/31; Severity 3/4/6; Harm 5/6/7; Stage 6/8; IOT 2/3) | N4, N5, D01–D04 | Merged/legacy labels and different cleaning stages | Use D01 definitions (3 / 7 / 26 / 3 / 6 / 5 / 3) |
| C12 | Stacking improvement called "statistically significant" without a test | D04 §4 vs N6 ("slightly better") | Overstatement | Present as "small but suggestive" (+0.037 F1) |
| C13 | Severity numbers differ (73.6%/0.624 vs 72.6%/0.637) | D04 §7.1 vs §8 | Different runs | Use one value, note approximate |
| C14 | Stage redesign was motivated by a baseline reported as 0.189 F1 (N10) from a run the Excel flags as using the wrong text; correct hospital-text LR ≈0.46 F1 vs final projection ≈0.25 | N10; XL | Author confirms the redesign was justified [U]; structural reasons (diluted embedding, multi-stage complaints) stand | Optional lesson "the baseline decides whether a redesign looks necessary"; frame as lesson, not mistake; author's call |
| C15 | RAM/latency figures are estimates, not measurements | D08; [U] | — | Say "estimated"; quote author's testimony about Whisper medium |
| C16 | Doc numbering inside D01–D12 doesn't match file numbers; STT status differs (D03 "planned", D07 "loaded at startup") | D01–D12 | Documentation drafted in several sessions | Irrelevant for the talk |

---

## 13. Strong presentation examples (ranked for a Model Planning talk)

1. **Constraint funnel** (D03 Fig. 1): no internet / CPU / RAM / 476 records ⇒ feasible set. Plus [U] "RAM could not even load Whisper medium; LLM never tried".
2. **Early planning decision with lasting consequences**: "store embeddings, build a retraining engine" — made in month 1, before results [U].
3. **Accuracy trap**: Improvement type 98.9% / F1 0.497; Severity 82.8% / F1 0.40; Harm binary 93.8% / 0 of 6.
4. **Stacked → Hierarchical**: label dependency discovery (N6) and tree discovery (N8); MANAGEMENT category 0.448 → 0.85; "structure beats volume" (D12 lesson 2).
5. **Plan-per-target** (N7): same template for each target ("what text, which features, which model, which risk") — a clean illustration of a model plan document.
6. **Per-target analysis questionnaire** (N4): definition, complexity, signal, distribution, metrics, actions, consequences — a reusable Model Planning checklist.
7. **Abandoned approaches with reasons** (N6.2): Bayesian networks, CRF, parallel multi-head — shows planning as pruning.
8. **Threshold-setting against a benchmark** (N4): ≥0.8 and JMIR; and what happened.
9. **Author's self-critique** (N6.1): parallel evaluation caused over-generalisation; should have profiled High vs Low severity first. A candid "process" lesson.
10. **"What would change without the constraints?"** (D12 §9): GPU → fine-tune Arabic BERT; local LLM → zero-shot, explanation; ~4,000 records → fine-grained labels; multi-label Stage.
11. **Metric definition drift** (C4): headline "F1 0.637" vs per-class (0.85, 0, 0).
12. **Validation planning gap** (C5): one 96-record split, 14–18 test records in branches.

---

## 14. Candidate visualisations

| Visual | Content | Source |
|---|---|---|
| Lifecycle ribbon | Discovery → Data Prep → **Model Planning** → Model Building → Communicate → Operationalise, with the Planning ↔ Building loop highlighted | [C] |
| Constraint funnel | All candidate model families → data (476, imbalance) → hardware (CPU, RAM) → offline → human-in-loop → feasible set | D03, D08 |
| Candidate map | LLM (cloud/local) · fine-tuned transformer · embeddings + classical ML · TF-IDF; with "status" colouring (benchmarked / considered / excluded) | §5 |
| Evolution timeline | Brief → Wave 0 → Wave 1 → insight 1 → plan v2 → insight 2 → hierarchy → target-specific → production | §2 |
| HCAT dependency graph / tree | Domain → Category → Sub-category; side nodes Severity, Stage, Harm, with text sources | D01 Fig. 1, 4 |
| Accuracy trap bars | Accuracy vs macro-F1 vs minority recall for IOT, Severity, Harm | D04 |
| Flat vs hierarchical | Category macro-F1 0.448 (flat) → 0.485 (stacked) → 0.85 (MANAGEMENT branch) | D09 §9 |
| Per-target plan card | One slide with a card per target: input text · model · metric · risk · human gate | N7, D03 |
| Theory → HCAT mapping table | Model Planning concept ↔ HCAT example | §15 plus theory |
| Class-size ladder | classes vs records per class (3, 7, 26, 78) with the "~50 per class" threshold line | D12 §2.3 |
| Planning loop | Plan → Build → Evidence → Revised plan (annotated with the 10 decision moments) | §8 |

---

## 15. Important numbers and sources

| Number | Meaning | Source |
|---|---|---|
| 111 | Records in Wave 0 | OR-R p.2 |
| ~500 / 485 / 476 | Wave 1 data / archived / official clean dataset | N2; D02, D07 |
| 380 / 96 | Train / test split (80/20) | D02 §7 |
| 1,816 | JMIR dataset size | D02 §14, N4 |
| 68.8%, 61.9% | JMIR mean concordance GPT-4o / GPT-3.5 | N4, D04 §10 |
| 79.4% (κ 0.623), 69.8% (κ 0.571), 53.9% (κ 0.226), 66.1% (κ 0.534), 74.2% (κ 0.162) | GPT-4o Domain/Category/Severity/Stage/Harm | N4 |
| ≥ 0.8 accuracy | Project go/no-go threshold | N4, N5, D09 §11.2 |
| 3 / 7 / 26 / 78 (73 EN) | Domain / Category / Sub-category / fine-grained classes | D01 |
| 59.9 / 21.4 / 18.7 % | Domain MANAGEMENT / RELATIONAL / CLINICAL | D01 §5.1 |
| 66.6 / 26.3 / 5.9 % | Severity LOW / MEDIUM / HIGH | D01 §7 |
| 92.6 % (441), 19 | Ordinary complaints / Red Flags | D01 §10 |
| 332 / 155 / 138 chars | Avg lengths of patient / immediate action / taken action text | D02 §5.3 |
| 768 | Embedding dimension | D02 §5.4 |
| ~2–4 GB | SQLite ML DB size (embeddings) | D08 §14.2 |
| 0.5–2 s; < 3 s target | Embedding latency estimate; stated latency target | D08 §4.1; N15 |
| 50–120 s | Server start-up with all models | D08 §11.2 |
| ~2–7 GB | Estimated RAM for all components (not measured) | D08 §4.2; [U] |
| 0.696 / 0.448 / 0.220 / 0.152 | Flat LR macro-F1: Domain / Category / Sub-cat / Class_ar | D04 §3.1 |
| 82.8 % acc / 0.401 F1 | Severity RF: the accuracy trap | D04 §3.1 |
| 98.9 % acc / 0.497 F1 | Improvement-type RF | D04 §3.1 |
| 63.4 % / 0.485 | Stacked Category (vs 60.2 % / 0.448) | D04 §5.2 |
| 82.0 % / 0.722 | Hierarchical Domain (LR, archive) | D04 §6.1 |
| 85 % / 0.83 (n=61) | Category MANAGEMENT branch | D04 §6.2 |
| 95 % / 0.88 (n=21) | Environment sub-category | D04 §6.3 |
| 57.3 % / 0.582 | Production Domain, 26 Feb 2026 | D04 §8.1 |
| 18, 0.635 | Production models, average F1 | D04 §8 |
| 93.8 %, 0 of 6 | Harm binary accuracy; High-Harm recall | D04 §7.3 |
| 45.6 % / ~0.25 | Stage semantic projection | D04 §7.2 |
| 78 classes ≈ 6 records/class; ~50/class ⇒ ~4,000 records | Why fine-grained labels were dropped | D12 §2.3 |
| 1.5× / 2× / 2.5× | Retraining milestones | D03 §10 |
| 18.6 % | Per-target test sizes: ~91–93 of ≈ 500 (derived) | [B] from N4 |

---

## 16. Material excluded as out of scope

React/FastAPI/IIS/NSSM/CORS/RBAC/workflow state machines/RCA/seasonal reports/UI/testing of reports (D05–D07, most of D08, D10, N14, N16). Used only for constraints and the human-in-the-loop rule.

## 17. What I propose for the next step

Awaiting your approval, then I will produce `MODEL_PLANNING_PRESENTATION_OUTLINE.md`, `MODEL_PLANNING_EVIDENCE_MAP.md` and finally `MODEL_PLANNING_FULL_PRESENTATION.md`. (Open questions are in `MODEL_PLANNING_OPEN_QUESTIONS.md`.)
