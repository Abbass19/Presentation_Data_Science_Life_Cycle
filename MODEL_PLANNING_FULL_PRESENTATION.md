# MODEL PLANNING IN THE DATA ANALYTICS LIFECYCLE
## Case study: HCAT Insight — full version (not time-limited)

How to use this file: each slide has **On-slide content**, **Visual**, **Speaker notes** (what to say, in spoken English) and **Sources** (keys in `MODEL_PLANNING_RESEARCH_DOSSIER.md`: OR-P, OR-R, XL, N0–N16, D01–D12). Items marked *(derived)* are my own arithmetic from the files; *(author)* means the author's own statement in conversation. Slide footers should state the data version of every number ("early wave", "archive hierarchical", "production, 26 Feb 2026").

---

# PART I — THEORY

## Slide 1 — Title
**On-slide content**
- **Model Planning in the Data Analytics Lifecycle**
- Lessons from a real, offline hospital NLP project: HCAT Insight
- [Name] · M2 Data Science · [Course] · [Date]

**Visual:** Lifecycle ribbon (Discovery → Data Preparation → **Model Planning** → Model Building → Communicate → Operationalise) in the background, Model Planning highlighted.

**Speaker notes:** "My topic is the third step of the lifecycle: Model Planning. I will give you the theory in a few minutes, then show you a real project in which the plan changed three times because of evidence. The point is not the project; it is what the project teaches about planning."

---

## Slide 2 — Where Model Planning sits
**On-slide content**
- Discovery → Data Preparation → **Model Planning** → Model Building → Communicate Results → Operationalise
- Planning **takes in:** business objective · what we know about the data · constraints
- Planning **hands over:** candidate models · evaluation plan · list of experiments
- Arrows go backwards: evidence from building can send us back to planning and to data preparation

**Visual:** Lifecycle as a loop; highlighted box; thin back-arrows from Model Building to Model Planning and from Model Planning to Data Preparation.

**Speaker notes:** "The lifecycle is usually drawn as a line, but it is lived as a loop. Model Planning receives a business question and a prepared dataset, and it produces a plan: which models, which evaluation, which experiments. Remember the back-arrows; today's case shows them in action."

**Sources:** theory.

---

## Slide 3 — What is Model Planning?
**On-slide content**
> **What should we try, why, under what conditions — and how will we judge it?**
- Choose the analytical task and candidate techniques
- Fix baselines, representations and features
- Fix the evaluation: metrics, threshold, validation
- List the experiments that Model Building must run

**Visual:** The question in the centre, four spokes: Problem · Data · Environment · Evaluation.

**Speaker notes:** "Model Planning is a decision stage. Its output is not a model; it is a plan. Notice that three of the four spokes have nothing to do with algorithms: they are about the problem, the data and the environment."

**Sources:** theory.

---

## Slide 4 — The decisions inside Model Planning
**On-slide content**

| Problem | Data | Models | Environment | Evaluation |
|---|---|---|---|---|
| Task type (classification, regression, ranking, sequence labelling…) | Dataset size | Candidate families (classical, transformer, LLM, rules, hybrid) | Compute, latency | Metrics |
| Target definition | Class imbalance | Baselines | Deployment setting, privacy | Success threshold |
| Target structure: flat, hierarchical, ordinal, multi-label | Label noise | Statistical assumptions | Operational risk | Train / validation / test |
| Human-in-the-loop need | Features / representation | Interpretability | | Experiments to run |

**Visual:** Five coloured columns. These colours return on later slides.

**Speaker notes:** "This is the checklist. At the end I will map the HCAT project onto it, row by row. Keep two items in mind: *target structure* and *environment*. In HCAT, those two changed the plan more than any algorithm."

**Sources:** theory.

---

## Slide 5 — Planning vs Building
**On-slide content**

| Model Planning | Model Building |
|---|---|
| What should we try, and how will we judge it? | Implement, train, tune, evaluate |
| Candidates, baselines, metrics, risks | Fitted models, tuned parameters, results |
| Decisions | Evidence |

**Plan → Build → Evidence → Revised plan → Build**

**Visual:** Two lanes with a feedback arrow. Label: "HCAT went around this loop at least three times."

**Speaker notes:** "'Should we use a hierarchy?' is a planning question. 'Train XGBoost on branch two' is a building task. In real projects the boundary is blurred, and that is fine, as long as we notice that building produces evidence that rewrites the plan."

**Sources:** theory.

---

## Slide 6 — Context prunes before experiments do
**On-slide content**
- All candidate model families
  - ↓ data constraints
  - ↓ hardware constraints
  - ↓ connectivity and privacy constraints
  - ↓ operational risk
- = **the feasible candidate set**

**Visual:** An empty funnel, to be filled with HCAT on slide 12.

**Speaker notes:** "A funnel idea: the best model in a textbook is rarely the best model in a hospital. Before we compare algorithms, the environment removes many of them. I will show you the real funnel in a moment."

**Sources:** theory.

---

## Slide 7 — Roadmap
**On-slide content**
1. The plan we started with
2. The evidence that broke it
3. The revised plans
4. What we learned about planning
- Scope: **modelling decisions only** — not software architecture

**Visual:** Mini timeline: Plan v1 → evidence → Plan v2 → evidence → Plan v3.

**Speaker notes:** "I will use HCAT as evidence. Results appear only when they explain a decision."

---

# PART II — THE CASE SET-UP

## Slide 8 — HCAT: the problem and the operational goal
**On-slide content**
- Rassoul Azam Hospital (specialised cardiac care): patient complaints in Arabic
- Today: an officer assigns **nine HCAT labels** per complaint — slow and inconsistent
- Goal: **the AI suggests; the officer confirms** → faster, more consistent, routing and analytics
- HCAT = Healthcare Complaints Analysis Tool (international framework)

**Visual:** Complaint text → AI suggestions → officer confirms → routing and reports.

**Speaker notes:** "Just enough context. The hospital receives complaints; each must be classified into a taxonomy. The AI is designed as an assistant, never as the decision-maker. Remember that last sentence: it changes how we evaluate risk."

**Sources:** D05 §2–3, D03 §2, D03 §11, D11 §8.

---

## Slide 9 — The data we actually had
**On-slide content**
- **111 → ~500 → 476 clean** records (before the job → job start → cleaned)
- Split: **380 train / 96 test**
- Arabic (standard + dialects) with English clinical terms
- Patient text avg **332 characters** (max 1,457); two hospital texts avg 155 and 138
- Real data, labelled by officers; **no inter-annotator agreement**
- Reference study (JMIR): 1,816 records

**Visual:** Bar chart: 111, ~500, 476 versus 1,816. Next to it: a shortened Arabic complaint excerpt (no names).

**Speaker notes:** "The data was small, Arabic, long and noisy; it also grew while we worked. Dataset size will explain almost every later decision: no deep learning, impossible fine-grained labels, a retraining plan. Labels were assigned by officers, so we cannot even say how consistent they are."

**Sources:** OR-R p.2, *(author)*, D02 §3–7, D01 §13.

---

## Slide 10 — One problem or nine?
**On-slide content**

| From the patient text | From the hospital texts |
|---|---|
| Domain (3), Category (7), Sub-category (26), fine-grained (78) | Severity (3), Stage (6), Harm (5), Improvement-opportunity type (3) |

- Target types: **hierarchical** · **ordinal** (Severity, Harm) · **multi-stage but single-label** (Stage) · **rare-event** (Red Flag)
- *First plan:* nine independent classifiers

**Visual:** 2 × 9 matrix with icons for target type.

**Speaker notes:** "This is 'define the analytical problem'. It looks like one classification task, but it is nine targets from two texts, with different structures. Our first plan treated them as independent — hold that assumption, we will test it."

**Sources:** N4, N5 §1, D01.

---

## Slide 11 — Target structure and class balance
**On-slide content**
- Classes grow down the hierarchy: **3 → 7 → 26 → 78**
- 476 records / 78 classes ≈ **6 per class**; rule of thumb ≈ **50 per class**
- Dominant classes: MANAGEMENT **59.9%**, LOW severity **66.6%**, Ordinary complaint **92.6%** (Red Flag: **19** records)
- Harm skewed toward Severe/Death — cardiac patients

**Visual:** Class-size ladder (log scale) with a "50 per class" line; three distribution bars (Domain, Severity, Improvement type).

**Speaker notes:** "This slide predicts the failures. When 93% of cases are 'ordinary', a model that always says 'ordinary' looks excellent. And a 78-class target with six examples per class is infeasible by arithmetic, before we train anything."

**Sources:** D01 §5, §7, §9–10, D12 §2.3.

---

## Slide 12 — The real constraint funnel
**On-slide content**
- All candidates
  - ↓ **476 records** → no deep learning
  - ↓ **CPU only, no GPU** → no fine-tuning
  - ↓ **Limited RAM** (≈ 8 GB or less *(author, not measured)*) → no large model
  - ↓ **No internet; privacy / hospital IT policy** → no cloud API
  - ↓ **Latency target < 3 s per text**
  - ↓ **Human review required**
- = **Frozen pre-trained embeddings + lightweight classifiers (LR, RF, XGBoost)**

**Visual:** Funnel with ✖ cloud LLM, ✖ GPU fine-tuning, ✖ local LLM, ✔ embeddings + classical ML.

**Speaker notes:** "In HCAT the environment did most of the model selection. The brief required a fully offline system for privacy. The server has no GPU and little memory — we could not even load the larger Whisper speech model comfortably. RAM and latency figures in the documentation are estimates; they were not measured. What remains is a frozen embedding model with light classifiers."

**Sources:** OR-P §4, D03 §3, D08 §3–4, N15, *(author)*.

---

## Slide 13 — Candidate approaches and their status
**On-slide content**

| Family | Needs data | Compute | Offline OK | Status in HCAT |
|---|---|---|---|---|
| Cloud LLM, zero-shot | none | external | **No** | Literature reference only (JMIR) |
| Local LLM | none–few | large RAM/GPU | yes | **Never tested** (RAM) |
| Fine-tuned transformer (AraBERT / multi-head BERT) | high | GPU | yes | **Implemented, benchmarked — weak** |
| Frozen embeddings + LR/RF/XGBoost | low | CPU | yes | **Implemented — selected** |
| TF-IDF + LR | low | CPU | yes | **Benchmarked — weaker** |

**Visual:** Colour-coded matrix; status column in bold.

**Speaker notes:** "Notice the status words. We did not test an LLM — we ruled it out by constraint. The only LLM evidence is a published study. The fine-tuned transformer was actually built and performed poorly on our small data. This is the difference between 'considered', 'excluded' and 'benchmarked'."

**Sources:** OR-R, N4, N5, D03 §3, D08, D12 §9.5.

---

## Slide 14 — "Why not simply use an LLM?"
**On-slide content**

| Real planning (HCAT) | Unconstrained planning |
|---|---|
| No internet → no API | Zero-shot baseline first (GPT-4o on English GP data: Domain 79.4%, Category 69.8%) |
| Offline for privacy | Explanations and summaries |
| CPU only, limited RAM | Local 7–8B model on a GPU server |
| Retraining loop must be cheap | Fine-tune an Arabic BERT |
| Latency < 3 s | Still needed: own-data evaluation, human review |

**Visual:** Two-column comparison.

**Speaker notes:** "For HCAT the LLM is not a design choice; it is infeasible. Remove the constraints and it becomes the first baseline I would run — but even then I would still test it on our own data and keep a human in the loop. Planning is selecting from the feasible set, and comparing with what the infeasible set reports."

**Sources:** N4, D08 §3.1, D12 §9.5, *(author)*.

---

## Slide 15 — The model-planning decision table
**On-slide content**

| Question | Evidence from HCAT | Planning consequence |
|---|---|---|
| Cloud inference possible? | No internet, IT policy | No APIs; all weights local |
| GPU available? | No | No fine-tuning; frozen encoder |
| Memory for an LLM? | Limited; Whisper medium already hard | LLM excluded |
| Dataset large? | 380 training records | No deep learning |
| Text Arabic? | Standard + dialect + English terms | Multilingual encoder |
| Labels balanced? | 60–93% dominant classes | Macro-F1, class weights, per-class recall |
| Labels independent? | First assumed; later: no, tree-shaped | Stacked → hierarchical |
| Mistakes equally costly? | No: missed High Harm / Red Flag worst | Binary triage, human gates |
| Latency important? | < 3 s target | Embed once; store embeddings |
| Retrainable? | Data grows, offline | Stored embeddings + retraining engine |

**Visual:** Table with colour rail (data / environment / risk).

**Speaker notes:** "This is the general artefact: a question, the evidence, and the consequence. Which row do you think we discovered last? Independence — and it was the one with the biggest effect on performance."

**Sources:** D03 §3, D02, D08, N4, N6, N15, *(author)*.

---

## Slide 16 — Plan v1: the initial model plan
**On-slide content**
- Representation: sentence embeddings stored in a database
- Option A — one logistic-regression model per target
- Option B — two multi-head models (one per text, 4–5 heads)
- Option C — heavier: multi-head BERT
- Decisions to take: single vs multi-head · which algorithm · all texts or separate
- Hypothesis: a shared model can exploit correlations between labels

**Visual:** Decision tree of three choices.

**Speaker notes:** "The first plan was reasonable. Logistic regression on embeddings had worked on the earlier 111-record project. The multi-head idea came from a good intuition — labels are correlated — but with a wrong mechanism. We will see why."

**Sources:** N4 (first section), OR-R.

---

## Slide 17 — Plan v1: the evaluation plan
**On-slide content**
- **Success threshold: accuracy ≥ 0.8** — "acceptable, not outstanding", set with the JMIR study as reference
- Metrics planned per target: macro-F1, per-class F1, Cohen's κ; weighted κ for Severity; recall for moderate+ harm
- Split: random 80/20 — **same split for every model**
- Per-target questionnaire: definition · complexity · signal · distribution · metrics · actions · consequence if used as is

**Visual:** Evaluation card: Threshold · Metrics · Split · Reference.

**Speaker notes:** "This was a good plan on paper — especially the per-target questionnaire. But note what it did not contain: no cross-validation, no validation set, no majority-class baseline. I will come back to that."

**Sources:** N4, D02 §7, D04 §2.3.

---

# PART III — DEVELOPMENT WAVES

## Slide 18 — Timeline
**On-slide content**
Brief (Oct 2025) → Wave 0: 111 records → Wave 1: flat baselines (Nov 2025) → **Accuracy lies** → **Labels are dependent** → Plan v2 → **Taxonomy is a tree** → Target-specific plans → Production (26 Feb 2026, 18 models)

**Visual:** Horizontal timeline; lanes: Plan · Experiment · Problem · Insight · Revised plan.

**Speaker notes:** "A map for the next twelve slides. Each stop follows the same pattern: assumption, evidence, change."

**Sources:** OR-P, OR-R, N4–N10, D04 §8.

---

## Slide 19 — Decision Moment 1: the environment decides first
**On-slide content**
- **Assumption:** choose the best model for Arabic text classification
- **Evidence:** offline mandate · CPU only · limited RAM · Whisper medium hard to load
- **Why insufficient:** cloud and GPU options do not exist
- **New understanding:** feasible set = frozen encoder + light classifier
- **Decision (month 1):** pre-compute and store embeddings; build a retraining engine on top of them *(author)*
- **Result:** every later experiment was cheap; retraining at 1.5× / 2× / 2.5× data is possible

**Visual:** Decision card (the six-row format reused on slides 20–30).

**Speaker notes:** "This decision was taken in the first month, before any result. It was a prior shaped by the constraints, not a comparison. Honest caveat: we never measured an LLM against it. But it made everything else affordable: embeddings are computed once and every retraining reuses them."

**Sources:** *(author)*, OR-P §4, D02 §11, D03 §5.3, §10, D08.

---

## Slide 20 — Decision Moment 2: more complex ≠ better (Wave 0)
**On-slide content**
- 111 records; three candidates: multi-head AraBERT · TF-IDF + LR · SBERT embeddings + LR
- Domain accuracy: **0.61 · 0.39 · 0.61**; Category accuracy **0.48 · 0.44 · 0.48**
- Rare classes: F1 ≈ 0 for all
- **Decision:** embeddings + logistic regression as the baseline for everything
- Later: multi-head BERT fine-tuned on CPU → Domain F1 **0.27**

**Visual:** Grouped bars of Domain and Category accuracy for the three models; callout "Listening, Safety: F1 = 0".

**Speaker notes:** "With 111 records, the deep model did not beat a simple classifier on a good sentence embedding. The lesson is about representation: when data is tiny, a strong frozen representation beats training a large network. A caution for the questions: in that report two result tables are numerically identical, so I use it for the reasoning, not as proof."

**Sources:** OR-R pp. 5–9, N5 §3.

---

## Slide 21 — Wave 1: flat baselines on the full data
**On-slide content**

| Target | Accuracy | Macro-F1 |
|---|---|---|
| Domain (3) | 79.6% | 0.70 |
| Category (7) | 60.2% | 0.45 |
| Sub-category (27) | 45.2% | 0.22 |
| Classification_ar (78) | 31.2% | 0.15 |
| Stage (6) | 56.7% | 0.39 |
| Harm (6) | 47.3% | 0.24 |

Threshold: 0.8 accuracy. Only Domain is near it.

**Visual:** Horizontal accuracy bars with a vertical 0.8 line; macro-F1 as dots.

**Speaker notes:** "Plan v1 is not enough. Look at the gap between the bars and the dots — accuracy and macro-F1 diverge. That divergence is the next slide."

**Sources:** D04 §3.1, N4.

---

## Slide 22 — Decision Moment 3: the accuracy trap
**On-slide content**

| Target | Accuracy | Minority view |
|---|---|---|
| Improvement type (RF) | **98.9%** | macro-F1 0.50 — always "Ordinary" |
| Severity (RF) | **82.8%** | macro-F1 0.40 |
| Severity (ordinal LR) | 72–74% | LOW recall 1.00, MEDIUM 0, HIGH 0 |
| Harm, binary | **93.8%** | High-Harm found: **0 of 6** |

- **Decision:** macro-F1 primary; per-class recall for rare safety classes; class weights; human gates
- *Lesson:* the definition must be fixed in the plan — the headline "F1 0.62" for Severity matches **weighted** F1; per-class values (0.85, 0, 0) give macro ≈ **0.28** *(derived)*

**Visual:** Three panels, "98% accuracy / minority recall ≈ 0".

**Speaker notes:** "This is the core evaluation lesson. A model can be very accurate and useless. In Harm, the model missed every severe case in the test set. I should add a candid point: even our final report mixed weighted and macro averages. That is a planning failure we can name: fix the metric definition before running experiments."

**Sources:** D04 §3.1, §7.1, §7.3, D09 §8, D11 §9.1; derived.

---

## Slide 23 — The threshold missed: "are we blocked?"
**On-slide content**
- Options on the table: merge labels · synthesise data (×20) · get more data · an LLM-based solution · ship anyway?
- Meeting with the supervisor — decisions:
  1. Keep going with the ML models, **even the ones that won't work**
  2. Retrain with more data; consider synthesis later
  3. Test off-the-shelf NER and Faster-Whisper
  4. Launch-or-wait decision left open
  5. Self-retraining at 1.5× / 2× / 2.5× data
- Reframing: **each target is its own project**

**Visual:** Decision tree with the chosen path highlighted.

**Speaker notes:** "A threshold is a planning instrument. Missing it did not stop the project; it produced a strategy: accept the limits, plan for data growth, evaluate per target. The comparison with the literature helped morale: 60% on about 500 records is not bad when GPT-4o reaches about 70% on 1,816 — with the comparability caveat I will return to."

**Sources:** N4 "Analysis" and "Decision Taking", N5 §6.

---

## Slide 24 — Decision Moment 4: the labels are dependent
**On-slide content**
- **Assumption:** nine independent targets; parallel heads are enough
- **Evidence:** stacked Category **63.4%** (F1 0.485) vs LR **60.2%** (F1 0.448) once the Domain output is an input
- **Why insufficient:** parallel heads share an encoder but do not condition on one another
- **Understanding:** labels follow the annotator's order: Domain → Category → Sub-category → Severity → Stage → Harm
- **Decision:** drop parallel multi-head networks; use sequential models, different inputs per target

**Visual:** Dependency graph, with the stacked arrow highlighted.

**Speaker notes:** "A small anomaly changed the architecture. The stacked model did slightly better because it was placed after the strongest model, Domain. The gain was modest — plus 0.037 F1 on about 93 test records, with no significance test — so the value was the hypothesis, not the number."

**Sources:** N6, N6.2, D04 §5, D01 §12.

---

## Slide 25 — Plan v2: five chosen, three pruned
**On-slide content**

| Chosen to test | Pruned (considered, not built) |
|---|---|
| Logistic regression baseline | Parallel multi-head neural nets — ignore dependency, heavy |
| Stacked sequential models | Bayesian networks — unstable with ~500 records |
| XGBoost with injected features | Conditional random fields — too much maths and engineering |
| Ordinal LR (Severity only) | |
| Better embeddings: multilingual MPNet | |

**Visual:** Keep / Prune columns with reason chips.

**Speaker notes:** "Planning is pruning, and each pruned option has a reason. Bayesian networks and CRFs were only considered. Note the embedding choice: MPNet replaced AraBERT as embedder because it produces sentence-level vectors, handles Arabic–English mixing, and runs on CPU."

**Sources:** N6, N6.2, D03 §5.1.

---

## Slide 26 — Plan v2: a plan card per target
**On-slide content**

| Target | Input text | Model & extras | Note |
|---|---|---|---|
| Domain | Patient | LR / RF / XGB | "Good" |
| Category | Patient | + Domain prediction, class weights | "Needs data" |
| Sub-category | Patient | + Domain + Category; merge rare classes | "Needs merging" |
| Classification_ar | — | Do not train as 78 classes | "Will not work" |
| Severity | Patient + hospital | Ordinal LR; stacked | "Needs structure" |
| Stage | Hospital | + Domain + Category | "Needs redesign" |
| Harm | All | Binary High/Low + 7-class | "Hardest" |
| Improvement type | Patient + hospital | LR / RF / XGB | "Simple" |

**Visual:** Eight plan cards in a grid.

**Speaker notes:** "This is the most literal 'model plan' in the project: for each target, which text, which model, which extras, and an honest label. Use it as a template for your own plans."

**Sources:** N7, N6.2.

---

## Slide 27 — Decision Moment 5: it is a tree
**On-slide content**
- **Assumption:** stacking probabilities is the way to use the dependency
- **Evidence:** a 10-minute check — Domain 1 → Categories 5, 7 only; branches never overlap
- **Why insufficient:** stacking still lets the model consider impossible labels
- **Understanding:** split the data by branch; one model per branch (≈ 11 models to sub-category)
- **Decision:** hierarchical pipeline
- **Result:** Category MANAGEMENT branch **0.448 → 0.85** F1; Environment sub-category **95%** accuracy
- *Caution:* a 2-class branch (n = 61) — majority baseline ≈ 64% *(derived)*

**Visual:** Flat vs stacked vs hierarchical diagram plus bars 0.448 → 0.485 → 0.85 with an asterisk.

**Speaker notes:** "The biggest gain in the project did not come from a better algorithm but from respecting the structure of the labels. The author's own meta-question is the one I want you to keep: what quick test would have revealed this earlier? Answer: profile label co-occurrence before choosing any architecture. And an honest asterisk: the branch problem is smaller, so the number is not comparable with the 7-class flat model."

**Sources:** N8, N9, D04 §6, D09 §9, D12 §13; derived.

---

## Slide 28 — Decision Moment 6: each target needs its own problem definition
**On-slide content**

| Target | Text | Structure | Chosen design |
|---|---|---|---|
| Domain, Category, Sub-category | Patient | Tree | Hierarchical LR/XGB |
| Severity | Patient + hospital | **Ordinal**, context-dependent | Ordinal LR |
| Harm | Patient + hospital | Ordinal, rare classes | **Two-stage**: binary safety triage + fine-grained |
| Stage | Hospital | Multi-stage, single label | Projection features |
| Improvement type | Patient | Rare event | LR + human confirmation |

**Visual:** Same matrix with target-type icons.

**Speaker notes:** "Back to the checklist: target structure drives the model family. Severity is ordered, so predicting 'low' for a 'high' case should cost more than being one step off. Harm has rare classes, so we first ask the safe question — is it high or low? — and only then the fine-grained one."

**Sources:** D02 §12, N10, D03 §6.2–6.5.

---

## Slide 29 — Decision Moment 7: Stage — change the representation
**On-slide content**
- **Problem:** stage cues are diluted in one embedding; complaints span several stages but carry one label
- **Idea:** ≈ 15 concept vocabularies → centroid per concept → dot-product with each of ≤ 6 sentences → max → classify the 15 scores
- **Result:** interpretable; accuracy 45.6%, macro-F1 ≈ 0.25 (Care on the Ward 0.64; Discharge, Unspecified 0)
- **Status:** hint only, human review
- *Lesson:* define in the plan what success for a redesign means — and compare with a correct baseline

**Visual:** Pipeline: text → sentences → projections → 15 features → classifier.

**Speaker notes:** "Planning can change the representation, not only the classifier. The redesign was justified by the structure of the problem. It did not beat the flat baseline on the reported numbers, so it was deployed as a hint only. One planning lesson: the baseline that makes a redesign look necessary must itself be correct. In this project the first baseline used the wrong text; with hospital text the plain classifier scored about 0.46. That does not make the redesign wrong — it shows why baselines belong in the plan."

**Sources:** N10, D03 §6.3, D04 §7.2, XL (corrected baseline).

---

## Slide 30 — Decision Moment 8: drop a target
**On-slide content**
- Classification_ar (78) / _en (73): F1 **0.15** flat · **0.12** stacked · **0.14** production
- ≈ 6 records per class; ≈ 50 per class needed → **≈ 4,000 records**
- **Decision:** excluded from the AI pipeline; kept for manual use
- No architecture can create data

**Visual:** Records-per-class ladder with the 78-class point far below the line.

**Speaker notes:** "A good plan removes what cannot work. Early notes hoped synthetic data ×20 would rescue it; it was never done. By arithmetic, the fine-grained layer needs about eight times more data than we have."

**Sources:** D04 §9.1, D12 §2.3, N4.

---

## Slide 31 — Production snapshot and operating rules
**On-slide content**
- 18 production models (26 Feb 2026)
- Reliable: Category MANAGEMENT (F1 0.85), Category RELATIONAL (0.82), Improvement type on Ordinary complaints
- Moderate: Domain, Severity (LOW only), Category CLINICAL
- Weak: Stage (≈ 0.25), Harm High, fine-grained labels (not deployed)
- **Rules:** suggestions need confirmation · High severity, High Harm, Red Flag, Never Event cannot be committed automatically · Stage = hint · retraining may not lose more than 5 points of F1 per model
- *Note:* some "F1" values in this report are weighted averages

**Visual:** Traffic-light tier chart plus a human-gate flow.

**Speaker notes:** "The plan ended as a tiered, human-gated system rather than a single accuracy number. Different outputs get different trust levels. The asterisk refers to the metric issue on slide 22."

**Sources:** D04 §8, D09 §7, D11 §8, D10 §8.3.

---

# PART IV — RISKS AND EVALUATION

## Slide 32 — Risk register
**On-slide content**

| Risk | Why it matters | Detected by | Planning decision |
|---|---|---|---|
| Tiny dataset | No deep learning | 111 and 476 records | Frozen embeddings + classical ML |
| Severe imbalance | Misleading accuracy | Per-class reports | Macro-F1, class weights |
| Classes with few records | F1 = 0 | Per-class reports | Drop 78-class target |
| Dependent labels | Impossible combinations | Stacked > flat | Hierarchy |
| Subjective labels, no agreement | Ceiling on accuracy | Documentation | "Suggestions", human review |
| Multi-stage, single label | Label noise | Stage results | Projection features |
| 768 dimensions, ~380 records | Overfitting | BERT multi-head failure | LR baseline |
| Rare safety classes | Costliest misses | Harm 0/6 | Human gates |
| Hardware | LLM not loadable | Whisper medium | Exclude LLM / fine-tuning |
| Weak validation | Optimistic, noisy | *Found late* | See next slide |

**Visual:** Risk table colour-coded by severity.

**Speaker notes:** "Model Planning is largely risk management. Read the rows as chains: risk, why, how we found out, what we decided. Pick three to narrate: imbalance, dependent labels, and the last row, which is about our own plan."

**Sources:** D01 §14, D02 §13, D03 §13, D11 §9, D12 §2–3.

---

## Slide 33 — Evaluation planning: what we would plan differently
**On-slide content**

| What happened | What the plan should contain |
|---|---|
| One random 80/20 split; branches tested on 14–18 records | Stratified k-fold with confidence intervals |
| Same test metrics used to choose and to report | Separate validation set / nested validation |
| No majority-class baseline (85% vs ≈ 64%) *(derived)* | Always report the trivial baseline |
| κ planned, not computed for HCAT | Compute κ or drop it from the plan |
| Weighted vs macro F1 mixed | Fix metric definitions up front |
| Benchmark not like-for-like | Comparability checklist |
| Domain 82% (archive) vs 57% (a later run), unexplained | Keep splits and versions under control |

**Visual:** Two-column checklist, "What happened | What the plan should say".

**Speaker notes:** "This is the most honest slide. These are not criticisms of a failed project — it is a deployed system. They are the points where a more complete Model Planning would have produced more trustworthy evidence. If you are asked 'how reliable are these numbers?', this slide is the answer."

**Sources:** D02 §7, D04 §6.2, §8.1, §10, D09, D10 §8; derived.

---

# PART V — LESSONS

## Slide 34 — Real vs unconstrained planning
**On-slide content**

| Real plan | If constraints disappeared |
|---|---|
| Frozen MPNet + LR/XGBoost on CPU | Zero-shot LLM baseline first |
| No fine-tuning | Fine-tuned Arabic BERT (+5–15 F1 expected) |
| No synthetic data | Paraphrase / LLM-generated data |
| Black-box suggestions | LLM explanations and summaries |
| 476 records | ≥ 2,000 records for fine-grained labels, Severity MEDIUM/HIGH |

**What would not change:** the hierarchy, per-class metrics, human gates

**Visual:** Two pipelines side by side.

**Speaker notes:** "The test of a contextual plan: remove the context and see what changes. Model families change. Structural insights and risk discipline do not."

**Sources:** D12 §9, theory.

---

## Slide 35 — Planning as an iterative loop
**On-slide content**
Plan → Build → Evidence → Revised plan, with eight nodes:
1. Environment decides first
2. Simple beats complex
3. Accuracy lies
4. Labels are dependent
5. It is a tree
6. Each target is its own problem
7. Stage: change the representation
8. Drop what cannot work

**Visual:** Circular loop with eight numbered nodes.

**Speaker notes:** "One sentence per node. The shape matters more than any single node: every stop followed evidence from building."

---

## Slide 36 — Methodological lessons
**On-slide content**
1. Test label structure before choosing an architecture
2. Structure beats volume when structure exists
3. Ordered targets need ordered thinking
4. Check per-class recall for safety classes
5. Study one target at a time; don't over-generalise from one finding
6. Fix metric definitions and baselines early
7. Plan validation, not only models

**Visual:** Seven rule cards.

**Speaker notes:** "The first four come from the project's own lessons. The fifth is the author's self-critique: evaluating all models in parallel made one finding spill over into every target. The last two come from reviewing the evidence honestly."

**Sources:** D12 §13, N6.1, N8.

---

## Slide 37 — Theory ↔ HCAT
**On-slide content**

| Planning decision | HCAT example |
|---|---|
| Problem definition | Nine targets from two texts |
| Target structure | Tree + ordinal + rare-event |
| Representation | MPNet embeddings; text 1/2/3; projection scores |
| Candidate families | LLM, fine-tuned BERT, embeddings + ML, TF-IDF |
| Baselines | LR on embeddings; TF-IDF |
| Imbalance | Macro-F1, class weights |
| Validation | 380 / 96 split (single) |
| Metrics | Accuracy → macro-F1, per-class recall |
| Compute / latency / deployment | CPU, < 3 s, offline |
| Interpretability | LR; projection scores |
| Operational risk / human in the loop | Mandatory gates |
| LLM or classical? | Classical, by constraint |
| Experiments for Building | Next slide |

**Visual:** Two-column mapping table, using the slide-4 colours.

**Speaker notes:** "A quick tour. If a row surprises you, that was probably a place where the plan moved."

---

## Slide 38 — From Model Planning to Model Building
**On-slide content**

| # | Experiment | Input | Metric | Decision rule |
|---|---|---|---|---|
| E1 | LR baseline per target | Stored embeddings | Macro-F1 + per-class recall | Reference line |
| E2 | Text-source comparison | Text 1 / 2+3 / 1+2+3 | Macro-F1 | Best text per target |
| E3 | Flat vs stacked | + upstream prediction | Macro-F1 | Keep if gain exceeds noise |
| E4 | Hierarchical per branch | Branch subsets | Per-class F1 vs majority | Keep if beats baseline |
| E5 | Ordinal vs nominal Severity | text123 | Weighted κ, recall of HIGH | Keep if HIGH recall improves |
| E6 | Stage: projection vs LR | Sentences | Macro-F1 | Keep if it beats the *correct* baseline |
| E7 | Harm: binary vs 7-class | text123 | Recall of High Harm | Threshold for recall |
| E8 | Retrain at 1.5× data | New records | Regression gate | No model −5 pp |

Generic template: hypothesis · data · model · metric · threshold · stop rule

**Visual:** Experiment table, labelled "written in Planning, executed in Building".

**Speaker notes:** "A plan ends as an experiment list with decision rules. Planning writes this table; Building fills in the results column, and the results may rewrite the table."

---

## Slide 39 — Comparing with the literature
**On-slide content**

| | JMIR / Koh 2025 | HCAT Insight |
|---|---|---|
| Records | 1,816 | 476 |
| Language | English | Arabic |
| Setting | General practice | Cardiac hospital |
| Method | GPT-4o zero-shot | Trained LR/XGBoost, offline CPU |
| Test size | large | ~96 |
| Result | Domain 79.4% (κ 0.62), Category 69.8% (κ 0.57), Harm 74.2% (κ 0.16) | Domain ≈ 82% (archive); Category and Harm figures come from sub-problems |

**Use:** to calibrate a threshold — **not** to claim superiority.

**Visual:** Comparison table with a "comparable?" column.

**Speaker notes:** "Benchmarks help planning only if they are comparable. The strongest safe claim is: in a very different setting, a lightweight offline model reached a Domain accuracy in the same range as zero-shot GPT-4o. Beyond that, the comparison is context, not evidence."

**Sources:** N4, D04 §10, D09 §10; derived.

---

## Slide 40 — Take-aways
**On-slide content**
1. Constraints prune before experiments do
2. A plan defines success, baselines and validation — not only models
3. Structure in the targets mattered more than the algorithm
4. Metrics can lie — look at the minority class
5. The best theoretical model is not the best operational model
6. Plan → Build → Evidence → Re-plan

> More complex ≠ better. More structure = better.

**Speaker notes:** "To summarise: planning is contextual, iterative, evidence-driven and honest about risk. HCAT's plan changed because the evidence said so, and that is exactly what a plan is for. Thank you — questions?"

---

# APPENDIX

## A1 — Text source by target
| Target | Best text | Evidence |
|---|---|---|
| Domain, Category, Sub-category | Patient (text 1) | D02 §12 |
| Severity | Combined (1+2+3) | D02 §12, D03 §6.2 |
| Stage | Hospital (2+3) | D02 §12, N10 |
| Harm | Combined (1+2+3) | D02 §12, D03 §6.4 |
| Improvement type | Patient (D03) / combined (N7) | Sources differ |

## A2 — Phase 1 flat baselines (all targets)
| Target | LR acc / macro-F1 | RF acc / macro-F1 |
|---|---|---|
| Domain (3) | 79.6% / 0.696 | 74.2% / 0.503 |
| Category (7) | 60.2% / 0.448 | 56.9% / 0.241 |
| Sub-category (27+) | 45.2% / 0.220 | 44.1% / 0.174 |
| Classification_ar (78) | 31.2% / 0.152 | 25.8% / 0.083 |
| Severity (3) | 74.2% / 0.396 | 82.8% / 0.401 |
| Stage (6) | 56.7% / 0.390 | 47.8% / 0.230 |
| Harm (6) | 47.3% / 0.244 | 41.8% / 0.247 |
| Improvement type (2) | 97.8% / 0.744 | 98.9% / 0.497 |
Source: D04 §3.1. Earlier notes give Domain LR 81.7% / 0.722 on a slightly different data version.

## A3 — Stacked and hierarchical
| Stage | Result |
|---|---|
| Stacked Category (771 features) | 63.4% / 0.485 vs flat 60.2% / 0.448 |
| Stacked Classification_ar | 30.1% / 0.124 (no gain) |
| Hierarchical Domain (LR) | 82.0% / 0.722 (RF 72%/0.50, XGB 75%/0.61) |
| Category CLINICAL | LR 67% / 0.52 (n=18) |
| Category MANAGEMENT | RF 85% / 0.83 (n=61) |
| Category RELATIONAL | LR 64% / 0.60 (n=14) |
| Sub-category Environment | LR 95% / 0.88 (n=21) |
| Sub-category Quality of Care | LR 79% / 0.53 (n=14) |
| Sub-category Safety | LR 75% / 0.33 (n=4) |
| Sub-category Institutional Processes | LR 56% / 0.36 (n=39) |
Source: D04 §5–6.

## A4 — Production report, 26 Feb 2026 (18 models)
| Model | Accuracy | F1 (as reported) |
|---|---|---|
| Domain | 57.3% | 0.582 |
| Category — CLINICAL / MANAGEMENT / RELATIONAL | 58.3% / 85.5% / 82.4% | 0.586 / 0.850 / 0.819 |
| Sub-category (7 branches) | 0–85.7% | 0.00–0.84 |
| Harm binary | 93.8% | 0.907 |
| Harm ordinal High / Low | 66.7% / 40.4% | 0.533 / 0.282 |
| Severity | 72.6% | 0.637 |
| Improvement type | 96.7% | 0.951 |
| Classification_en (70 classes) | 14.6% | 0.143 (not deployed) |
Average F1 0.635. **Metric caveat:** recall equals accuracy in every row, and Severity/Harm values match weighted F1; Stage's ≈ 0.25 is macro. Branch names in this report may use a different numeric mapping. Source: D04 §8, D09 §7.

## A5 — JMIR comparison (full)
| Dimension | GPT-4o (JMIR) | HCAT Insight | Comparable? |
|---|---|---|---|
| Domain | 79.4% (κ 0.623) | 82% archive; 57.3% later run | Partly |
| Category | 69.8% (κ 0.571) | 85.5% on a 2-class branch | **No** |
| Severity | 53.9% (κ 0.226) | 72.6% (LOW only reliable) | **No** |
| Stage | 66.1% (κ 0.534) | ~45–57% | Partly |
| Harm | 74.2% (κ 0.162) | 93.4% binary (0 of 6 High Harm) | **No** |
Differences: language, taxonomy, hospital type, 1,816 vs ~96 test records, κ not computed for HCAT, no majority baselines. Source: D04 §10, D09 §10; assessment derived.

## A6 — Embedding alternatives
| Alternative | Reason not selected |
|---|---|
| AraBERT | Token-level; needs pooling; fine-tuning needs GPU (benchmarked early as multi-head) |
| mBERT | Weaker sentence representations |
| AraBART, CAMeL | Generative/seq2seq focus |
| OpenAI ada | Needs internet and an API key |
| Larger sentence transformers | RAM |
Selected: `paraphrase-multilingual-mpnet-base-v2` — sentence-level, Arabic and mixed text, 768 dimensions, CPU. Source: D03 §5.1, D02 §11.

## A7 — Abandoned approaches
| Approach | Reason | Source |
|---|---|---|
| Classification_ar / _en | ≈ 6 records per class | D04 §9.1 |
| Parallel multi-head networks | Independent heads; compute | N6.2, D04 §9.2 |
| Stacking into Classification_ar | No gain | D04 §9.3 |
| Bayesian networks | Unstable with ~500 records | N6.2 |
| CRFs | Excess maths and engineering | N6.2 |
| LLM on the VM | Never tried (RAM) | *(author)*, D08 |
| Synthetic augmentation | Never done | D12 §2.1 |

## A8 — Stage projection: the mathematics
- Each concept *m*: embed its vocabulary (full sentences), average → centroid; normalise → unit vector **u_m**
- Each complaint: split into ≤ 6 sentences; embed each: **s_j**
- Projection: **p(m, j) = s_j · u_m**; feature **f_m = max_j p(m, j)**
- ≈ 15 features → logistic regression → Stage
- Corrections noted in the project: standardise embeddings; build concepts from full sentences, not keyword bags
- Concepts (D04 table): arrival, administration, bed/room, billing, cleanliness, clinical delay, administrative delay, communication/scheduling, conflicting diagnosis, disagreement with discharge, doctor not following up, food, lab/imaging issues, location, clinical errors, staff/security behaviour
Source: N10, D03 §6.3, D04 §7.2.

## A9 — Severity and Harm detail
- Severity ordinal LR (n = 91): LOW P 0.74 R 1.00 F1 0.85; MEDIUM 0/0/0; HIGH 0/0/0
- Harm binary (n = 91): Low P 0.93 R 1.00; High 0/0/0 (6 cases) → accuracy 93.4%, macro-F1 ≈ 0.49 *(derived)*
- Harm ordinal: High-branch 66.7% / F1 0.533 (30 training records); Low-branch 40.4% / 0.282
Source: D04 §7, D09 §8.

## A10 — Dataset details
- Text fields: complaint text (avg 332 chars), immediate action (155), taken action (138); five embedding variants + six sentence embeddings (768-d)
- Cleaning: severity casing, trailing spaces, legacy harm labels, duplicate sub-category names; nulls retained and excluded per-target
- Missing: stage 8, harm 4, improvement type 16
Source: D02 §5–10, D01 §14.

## A11 — Deployment constraints detail (estimates)
- Offline single VM, Windows Server, no GPU, no internet
- RAM total ≈ 2–7 GB for all components (estimate); embedding model ≈ 400 MB
- Embedding ≈ 0.5–2 s per text; start-up 50–120 s
- Model updates by manual transfer; no model registry
- ML database ≈ 2–4 GB (stored embeddings)
Source: D08 §3–4, §11, §14.

## A12 — Evidence caveats handled in this deck
Dataset versions · Category count inconsistency · weighted vs macro F1 · selection on the test set · Domain 82% vs 57% · AraBERT status · stacking significance · benchmark comparability · RAM and latency not measured. See `MODEL_PLANNING_RESEARCH_DOSSIER.md` §12 and `MODEL_PLANNING_EVIDENCE_MAP.md` §7.
