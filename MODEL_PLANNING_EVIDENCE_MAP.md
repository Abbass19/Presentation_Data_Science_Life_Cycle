# MODEL PLANNING — EVIDENCE MAP

Maps statements and numbers used in the outline to their source. Keys as in the dossier (OR-P, OR-R, XL, N0–N16 = Obsidian notes, D01–D12 = formal docs by file number, PA). Tags: **[A]** stated in file · **[U]** author said it in conversation (not in any file) · **[B]** my derivation · **[C]** general theory.
Confidence: **H** consistent across sources · **M** single source or minor inconsistency · **L** known inconsistency; use cautiously.

## 1. Context, goals, constraints

| Statement | Source | Tag | Conf. | Notes |
|---|---|---|---|---|
| System must run fully offline for privacy, security, IT policy | OR-P §4 | A | H | Original requirement |
| Hospital VM has no internet, no GPU | D08 §2–4, D03 §3 | A | H | |
| RAM limited; ~2–7 GB total estimated for all components | D08 §4.2 | A | M | Estimates, not measured [U] |
| RAM ≈ 8 GB or less; Whisper medium hard to load; LLM never tried | [U] | U | M | "8 GB" to be confirmed |
| Latency target: sentence transformer < 3 s | N15 | A | M | Stated as a to-do |
| Embedding ≈ 0.5–2 s per text on CPU | D08 §4.1 | A | M | Estimate |
| Start-up 50–120 s | D08 §11.2 | A | M | Estimate |
| Embeddings pre-computed and stored; retraining reuses them | D02 §11, D03 §5.3, D03 §10 | A | H | |
| Decision to store embeddings/build retraining engine taken in first month | [U] | U | M | Not dated in files |
| Retraining at 1.5×/2×/2.5× data | D03 §10, N4 | A | H | |
| Predictions are suggestions; human gates for High severity, High Harm, Red Flag, Never Event | D03 §11, D11 §8 | A | H | |
| Regression gate: no model −5 pp F1 | D10 §8.3 | A | M | |
| Manual classification of nine labels was slow and inconsistent | D05 §2, D03 §2 | A | M | Retrospective docs |

## 2. Data

| Statement | Source | Tag | Conf. | Notes |
|---|---|---|---|---|
| 111 records before the job | OR-R p.2; [U] | A/U | H | |
| ~500 records when the job started | N2, N4; [U] | A/U | H | |
| 485 archived, 476 clean, 380/96 split | D02 §4, §7 | A | H | Early test sets n=90–93 |
| JMIR dataset 1,816 English GP complaints | D02 §14, N4 | A | H | |
| Arabic: MSA + Lebanese/Iraqi dialect + English terms | D02 §6.1 | A | M | |
| Avg lengths 332 / 155 / 138 chars | D02 §5.3 | A | M | |
| Labels assigned by complaint officers, no inter-annotator agreement | D02 §2, §13 | A | H | |
| Classes: Domain 3, Category 7, Sub 26, fine-grained 78 (73 EN) | D01 §4–6 | A | M | Other docs/notes say 21/27/31 for sub (C11) |
| Domain: MANAGEMENT 285 (59.9%), RELATIONAL 102, CLINICAL 89 | D01 §5.1 | A | M | |
| Severity: LOW 317 (66.6%), MEDIUM 125, HIGH 28 | D01 §7 | A | M | |
| Improvement type: Ordinary 92.6% (441); Red Flag 19 | D01 §10 | A | M | |
| Harm: Severe 186, Death 128, Moderate 7 (cardiac population) | D01 §9 | A | M | |
| Category counts (177/100/58/47/35/30/29) | D01 §5.2 | A | **L** | Inconsistent with Domain totals; not used (C2) |
| Records per class ≈ 6 for 78 classes; ~50 per class needed → ~4,000 records | D12 §2.3 | A | H | |
| Nine targets from two texts: patient → 4, hospital → 5; Status dropped | N4, N5 | A | H | |

## 3. Candidates and alternatives

| Statement | Source | Tag | Conf. | Notes |
|---|---|---|---|---|
| TF-IDF + LR benchmarked (Wave 0): Domain acc 0.391, Category 0.435 | OR-R p.7 | A | M | |
| Multi-head AraBERT implemented (5 heads), Domain 0.609, Category 0.478 | OR-R pp.5–6 | A | M | Table identical to SBERT (C9) |
| SBERT + LR selected | OR-R p.9 | A | M | |
| Multi-head BERT fine-tuned locally on CPU; Domain F1 0.27, acc 0.674 | N5 §2–3; XL | A | M | Early work |
| Stacked sequential models benchmarked | D04 §5 | A | H | |
| Hierarchical pipeline implemented and deployed | D04 §6, D03 §6.1 | A | H | |
| Five chosen solutions; three abandoned (parallel multi-head, Bayesian nets, CRF) | N6, N6.2 | A | H | |
| Bayesian network / CRF were *considered*, not built | N6.2 | A | H | N6.2 says CRF "evaluated as a theoretical option" |
| AraBERT, mBERT, AraBART/CAMeL, ada, larger ST rejected with reasons | D03 §5.1, D04 §9.4 | A | M | AraBERT actually benchmarked earlier (C8) |
| No LLM tested in HCAT; excluded by offline/CPU/RAM | D03 §3, D08 §3; [U] | A/U | H | |
| Local LLM (Mistral 7B / Llama 3.1 8B) only as future if GPU | D12 §9.5 | A | H | |
| Hybrid LLM+ML listed as "Phase 4" | N5 §7 | A | M | Never executed |
| Synthetic augmentation considered, not done | N4, N5, D12 §2.1 | A | H | |

## 4. Experiments and results

| Statement | Source | Tag | Conf. | Notes |
|---|---|---|---|---|
| Flat LR: Domain 79.6%/0.696; Category 60.2%/0.448; Sub 45.2%/0.220; Class_ar 31.2%/0.152; Stage 56.7%/0.390; Harm 47.3%/0.244 | D04 §3.1 | A | M | Earlier notes give 81.7%/0.722 for Domain |
| RF Severity 82.8%/0.401; Improvement type RF 98.9%/0.497 | D04 §3.1; N4; XL | A | H | Excel flagged some columns; these match across sources |
| Stacked Category 63.4%/0.485 vs flat 60.2%/0.448 | D04 §5.2; XL | A | M | "Statistically significant" is unsupported (C12) |
| Hierarchical Domain LR 82.0%/0.722 | D04 §6.1 | A | M | Production Feb 2026: 57.3%/0.582 unexplained (C6) |
| Category MANAGEMENT RF 85%/0.83, n=61 | D04 §6.2 | A | M | |
| Majority baseline for that branch ≈ 64% (39/61) | N4 supports (Env 22, IP 39); D04 n=61 | **B** | M | Derived |
| Environment sub-category 95%/0.88, n=21 | D04 §6.3 | A | M | |
| Severity ordinal: per-class F1 LOW 0.85, MEDIUM 0, HIGH 0 | D04 §7.1, D09 §8.1 | A | H | |
| Severity "F1" 0.624/0.637 = weighted F1; macro ≈ 0.28 | D04 §7.1 per-class table | **B** | M | (0.85+0+0)/3 = 0.283; weighted = 67×0.85/91 ≈ 0.63 |
| Harm binary 93.4–93.8% acc; High-Harm recall 0/6 | D04 §7.3, D09 §8.3, D11 §9.1 | A | H | |
| Harm binary macro-F1 ≈ 0.49 | D09 §8.3 per-class F1 (0.97, 0.00) | **B** | M | Reported "0.90" is weighted |
| Feb 2026 table: Recall = Accuracy in each row | D04 §8.1 | A/B | M | Signature of weighted recall; author doesn't recall [U] |
| Stage projection acc 45.6%, F1 ≈0.25 (macro) | D04 §7.2, D09 §8.2 | A | M | Per-class: 0.22, 0.40, 0.64, 0, 0 → macro 0.25 ✔ |
| Stage corrected baseline (hospital-text LR) ≈ 0.46 F1 / 0.578 acc | XL block (b) | A | M | Excel is "old"; use cautiously [U] |
| Stage baseline used to motivate redesign: acc 0.444, F1 0.189 | N10 | A | M | Matches Excel block flagged as wrong text |
| 18 production models; average F1 0.635 (26 Feb 2026) | D04 §8, D09 §7 | A | M | Metric definition unclear (C4) |
| Classification_ar F1 0.152 (flat), 0.124 (stacked); Classification_en 0.143 (prod) | D04 §3, §5, §8 | A | H | |
| Text-source findings: Domain/Category/Sub → patient; Severity/Harm → combined; Stage → hospital | D02 §12, N4, N10 | A | H | |
| Retrain at milestones; self-improving loop | D03 §10 | A | H | |

## 5. External benchmark

| Statement | Source | Tag | Conf. | Notes |
|---|---|---|---|---|
| JMIR/Koh 2025: GPT-4o Domain 79.4% (κ 0.623), Category 69.8% (κ 0.571), Severity 53.9% (κ 0.226), Stage 66.1% (κ 0.534), Harm 74.2% (κ 0.162) | N4, D04 §10 | A | H | |
| Mean concordance GPT-4o ≈ 68.8%, GPT-3.5 61.9% | N4, D04 §10 | A | H | |
| HCAT "beats" GPT-4o claim not like-for-like | D04 §10, D09 §10 vs details | **B** | M | Language, taxonomy, n, sub-problems, no κ |
| ≥0.8 accuracy threshold, justified by JMIR | N4, N5, D09 §11.2 | A | H | Who set it: open question |

## 6. Decisions and process

| Statement | Source | Tag | Conf. | Notes |
|---|---|---|---|---|
| After meeting with the supervisor: continue with ML models as they are; retrain with more data; test NER/STT; decide on launch later | N4 "Decision Taking" | A | H | |
| "Each model is a project of its own" realisation | N4 "Models Evaluation" | A | H | |
| Stacked > LR observation triggered the dependency insight | N6 | A | H | |
| Hierarchy discovered by a quick (~10 min) manual check | N8 | A | M | D04 says "decision tree built" |
| Self-critique: parallel evaluation over-generalised; should have profiled High vs Low severity | N6.1 | A | H | |
| Per-target plan (embeddings, algorithms, class weights, stacking inputs) | N7, N6.2 | A | H | |
| Severity ordinal; Stage rule+projection; Harm two-stage | N10 | A | H | |
| Single 80/20 split; no CV mentioned | D02 §7, D09 §2.2 | A | H | Absence verified in all files |
| Selection of "best per feature" from a comparison table of test metrics | N6.2, N7 | A | M | vs D09 §1 claim (C5) |
| Cohen's κ listed as a metric but not computed for HCAT | D04 §2.3 vs all result tables | **B** | M | |
| Documentation written via staged LLM workflow in May 2026 | PA | A | H | Provenance of D01–D12 |

## 7. Statements to avoid or qualify

| Statement | Why |
|---|---|
| "HCAT beats GPT-4o" | Not comparable; see §5 |
| "Stacking is statistically significant" | No test; +0.037 F1 on ~93 records |
| "Macro-F1 for Severity 0.62" | Appears to be weighted; macro ≈ 0.28 [B] |
| Category counts / "Quality of Care dominates" | Inconsistent; avoid |
| "AraBERT was only considered" | Implemented and benchmarked in Wave 0 |
| "Domain production accuracy" | 82% vs 57.3% unexplained; author doesn't recall [U] |
| RAM/latency "measured" | Not measured [U] |
