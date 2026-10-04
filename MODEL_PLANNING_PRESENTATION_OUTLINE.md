# MODEL PLANNING — PRESENTATION OUTLINE (full version, not time-limited)

**Topic:** Task 7 — Model Planning, Data Analytics Lifecycle. **Case study:** HCAT Insight.
**Audience:** M2 Data Science students and professor.
**Central question:** *How do requirements, data, constraints, evaluation choices, model alternatives, experiments, risks and discoveries shape the model plan — before and during model building?*

Source keys are defined in `MODEL_PLANNING_RESEARCH_DOSSIER.md` (OR-P, OR-R, XL, N0–N16, D01–D12, PA). Tags: [A] file · [U] author's own statement · [B] my inference · [C] theory.
Wording rule on the slides: *considered / planned / implemented / benchmarked / deployed* are used strictly.

**Narrative spine (one sentence):** *We had a problem, data and constraints; we planned candidates and an evaluation; evidence broke the plan three times (accuracy lied, labels were dependent, the taxonomy was a tree); each time the plan changed; this is Model Planning as an iterative loop with Model Building.*

**Structure:** Part I Theory (S1–S7) · Part II Case set-up (S8–S17) · Part III Development waves (S18–S30) · Part IV Risks and evaluation (S31–S33) · Part V Lessons (S34–S40) · Appendix (A1–A12).

---

# PART I — THEORY

### Slide 1 — Title
- **Core message:** Model Planning is where a Data Science project decides what to try, why, and how it will judge it.
- **Content:** "Model Planning in the Data Analytics Lifecycle — lessons from a real offline hospital NLP project (HCAT Insight)". Name, course, date.
- **Visual:** Lifecycle ribbon faintly in the background with *Model Planning* highlighted.
- **Speaker explanation:** One sentence: "I will explain the theory briefly, then show how a real project's plan changed under evidence."
- **Sources:** —

### Slide 2 — Where Model Planning sits in the lifecycle
- **Core message:** Planning follows Discovery and Data Preparation and precedes Model Building, but loops with them.
- **Content:** Six phases: Discovery → Data Preparation → **Model Planning** → Model Building → Communicate Results → Operationalise. Inputs to planning: business objective, data understanding, constraints. Outputs: candidate models, evaluation plan, experiment list.
- **Visual:** Lifecycle diagram (circular/ribbon) with back-arrows from Building to Planning and from Planning to Data Preparation.
- **Speaker explanation:** The lifecycle is drawn as a line but practised as loops. Today's topic is the third box, but we will see it forced backwards (new data checks) and forwards (build experiments).
- **Sources:** [C]

### Slide 3 — Definition: what Model Planning means
- **Core message:** "What should we try, why, under what conditions, and how will we judge it?"
- **Content:** Model Planning = choose analytical technique and candidate models given the problem, the data and the environment; define baselines, features/representations, evaluation and validation; list the experiments for building.
- **Visual:** One question in the centre with four spokes: *Problem · Data · Environment · Evaluation*.
- **Speaker explanation:** Emphasise it is a decision stage, not an algorithm-selection stage. The deliverable is a plan, not a model.
- **Sources:** [C]; the question wording from the task brief.

### Slide 4 — The decisions inside Model Planning
- **Core message:** Planning is a bundle of ~15 decisions, many unrelated to the algorithm itself.
- **Content (grouped checklist):**
  1. *Problem:* ML task type, target definition, target structure (flat / hierarchical / ordinal / multi-label), human-in-the-loop need.
  2. *Data:* size, imbalance, label noise, text/feature representation, train/validation/test strategy.
  3. *Models:* candidate families, baselines, statistical assumptions, interpretability.
  4. *Environment:* compute, latency, deployment, privacy, operational risk.
  5. *Evaluation:* metrics, success threshold, experiments to run.
- **Visual:** Five-column checklist; later slides re-use the same five colours.
- **Speaker explanation:** This is the checklist I will map HCAT onto at the end (S37).
- **Sources:** [C]

### Slide 5 — Planning vs Building
- **Core message:** Planning decides *what and how to judge*; Building *implements, trains, tunes, evaluates* — and evidence from Building revises the plan.
- **Content:** Two-column table (Question asked · Typical outputs · Typical artefacts). Loop: Plan → Build → Evidence → Revised plan → Build.
- **Visual:** Two-lane diagram with a feedback arrow; label "HCAT lived this loop 3 times".
- **Speaker explanation:** Contrast "should we try a hierarchy?" (planning) with "train XGBoost on branch 2" (building). Real projects blur this; the evidence arrows are the interesting part.
- **Sources:** [C]

### Slide 6 — Planning inputs: why context outranks algorithms
- **Core message:** The best algorithm in theory is rarely the best operational choice; context prunes the candidate set first.
- **Content:** Constraint funnel concept: all model families → data constraints → hardware → connectivity/privacy → operational risk → feasible set.
- **Visual:** Empty funnel (to be filled with HCAT in S12).
- **Speaker explanation:** Introduce funnel in generic terms so students see it first as a method.
- **Sources:** [C]

### Slide 7 — Roadmap: how HCAT will be used
- **Core message:** HCAT is an illustration of planning, not the topic.
- **Content:** Four things we will see: (1) a plan, (2) evidence that broke it, (3) a revised plan, (4) lessons. Mention: *only the modelling story; no software architecture*.
- **Visual:** Mini-timeline: Plan v1 → evidence → Plan v2 → evidence → Plan v3.
- **Speaker explanation:** Set expectations: results appear only as evidence for decisions.
- **Sources:** —

---

# PART II — CASE SET-UP

### Slide 8 — The HCAT problem and the operational goal
- **Core message:** A hospital wants complaints classified into the HCAT taxonomy faster and more consistently — with a human still deciding.
- **Content:** Rassoul Azam Hospital (cardiac). Officers must assign nine HCAT labels per complaint (manual, slow, inconsistent) [D05 §2, D03 §2]. Goals: suggest labels at intake, enable routing, support analytics, scale. AI = **assistant**, not decision-maker [D03 §11].
- **Visual:** Complaint text → AI suggestions → officer confirms → routing/analytics.
- **Speaker explanation:** Keep to two sentences; the audience needs only enough to understand the ML problem. HCAT = Healthcare Complaints Analysis Tool (international framework).
- **Sources:** D05 §2–3, D03 §2, D11 §8

### Slide 9 — The data: what we actually had
- **Core message:** The data was small, Arabic, long, two-perspective, and grew while we worked.
- **Content:** Dataset evolution **111 (before the job) → ~500 (job start) → 476 cleaned** [OR-R; U; D02]. 380 train / 96 test. Arabic (MSA + dialects) with English clinical terms; patient text avg 332 chars; two hospital texts (155, 138). Real, non-synthetic; labels by officers; no inter-annotator agreement. Reference study had 1,816.
- **Visual:** Growth bar (111 → 500 → 476) beside the JMIR bar (1,816); a sample Arabic complaint (shortened, no patient names).
- **Speaker explanation:** Data size drives almost every planning decision downstream (no deep learning, many classes unlearnable, retraining plan).
- **Sources:** OR-R p.2, [U], D02 §3–7, D01 §13

### Slide 10 — The prediction problem is not one problem
- **Core message:** The task is nine targets from two texts, with different structures — not a single classifier.
- **Content:** Patient text → Domain, Category, Sub-category, fine-grained label. Hospital texts → Severity, Stage, Harm, Improvement type (Status dropped: constant) [N4, N5]. Types: nominal tree (Domain/Category/Sub), ordinal (Severity, Harm), multi-stage-but-single-label (Stage), rare-event (Improvement type).
- **Visual:** Two-text × nine-target matrix, with target type icons.
- **Speaker explanation:** "Define the analytical problem" in practice: our first plan treated these as nine independent classification problems — an assumption that will break (S22).
- **Sources:** N4 (first lines), N5 §1, D01

### Slide 11 — Target structure and class balance
- **Core message:** Classes explode in number down the hierarchy and are heavily imbalanced.
- **Content:** Classes: Domain 3 → Category 7 → Sub-category 26 → fine-grained 78 (73 EN). Records per class: 476/78 ≈ 6; rule of thumb ~50/class [D12]. Dominant classes: MANAGEMENT 59.9%, LOW severity 66.6%, Ordinary complaint 92.6% (Red Flag: 19 records) [D01]. Harm skewed to Severe/Death (cardiac population) [D01 §9].
- **Visual:** Class-size ladder (log scale) with the "50 per class" line; three distribution bars for Domain, Severity, Improvement type.
- **Speaker explanation:** This slide predicts the failures to come: a model can score high by predicting the majority; the 78-class target is infeasible by arithmetic.
- **Sources:** D01 §5, §7, §9–10, D12 §2.3. (Category counts omitted deliberately — see dossier C2.)

### Slide 12 — Real deployment constraints: the constraint funnel
- **Core message:** Environment eliminated most of the textbook candidates before any experiment.
- **Content:** Funnel stages: All candidates → *476 records* → *CPU only, no GPU* → *limited RAM (≈8 GB or less [U], not measured)* → *no internet, privacy mandated by hospital IT policy* → *latency target < 3 s* → *human review required* ⇒ **frozen pre-trained embeddings + lightweight classifiers**. Excluded: cloud APIs, GPU fine-tuning, large LLMs.
- **Visual:** The funnel from S6 filled in; with icons ✖ cloud LLM, ✖ fine-tuning, ✖ local LLM, ✔ embeddings + LR/RF/XGB.
- **Speaker explanation:** Quote the author: even loading Whisper medium was difficult, so a local LLM was never attempted [U]. RAM and latency are estimates in the documentation, not measurements [D08].
- **Sources:** OR-P §4, D03 §3, D08 §3–4, N15, [U]

### Slide 13 — Candidate approaches (the model-selection trade-off)
- **Core message:** Four families were realistic; constraints and data made one feasible.
- **Content table:** Rows: Cloud LLM (zero-shot) · Local LLM · Fine-tuned transformer (AraBERT / multi-head BERT) · Frozen embeddings + classical ML (LR/RF/XGB) · TF-IDF + LR. Columns: data need · compute · latency · offline? · interpretability · **status in HCAT** (*benchmark-reference only (JMIR)* / *never tested (RAM)* / *implemented & benchmarked — poor* / *implemented — selected* / *benchmarked — weaker*).
- **Visual:** Colour-coded trade-off matrix.
- **Speaker explanation:** Status labels matter: we did not test an LLM; we ruled it out by constraint. The only LLM evidence is a published benchmark.
- **Sources:** OR-R, N4, N5, D03 §3, D08, D12 §9.5

### Slide 14 — "Why not simply use an LLM?"
- **Core message:** In this environment an LLM is not a model choice, it is an impossibility; elsewhere it would be a strong candidate.
- **Content:** *Actual HCAT reasons* [A/U]: no internet (no API), offline for privacy, CPU-only, limited RAM (Whisper medium already hard to load), latency, 18-model retraining loop must be cheap. *What would change without the constraints* [C/A D12]: zero-shot classification (GPT-4o 79.4% Domain, 69.8% Category on English GP data), explanation generation, summaries, local 7–8B model on a GPU VM. *Still true even then:* human review, evaluation on own data, cost.
- **Visual:** Two-column "Real planning vs Unconstrained planning".
- **Speaker explanation:** Make the point that planning is a *feasible-set* exercise; JMIR is used as external reference only, because the models are not comparable (different language, taxonomy, size).
- **Sources:** N4, D08 §3.1, D12 §9.5, [U]

### Slide 15 — The model-planning decision table (evidence → consequence)
- **Core message:** Each planning question was answered by evidence from the project.
- **Content (matrix):**

| Question | Evidence from HCAT | Planning consequence |
|---|---|---|
| Cloud inference possible? | No internet; hospital IT policy | No APIs; all weights local |
| GPU available? | No | No fine-tuning; frozen encoder |
| Memory enough for an LLM? | RAM limited; Whisper medium hard [U] | LLM excluded |
| Is the dataset large? | 380 train records | No deep learning; classical ML |
| Is text Arabic? | MSA + dialect + English terms | Multilingual encoder, not English-only |
| Are labels balanced? | 60–93% dominant classes | Macro-F1, class weights, per-class recall |
| Are labels independent? | *Not at first known* → dependent, tree-shaped | Stacked → hierarchical |
| Are mistakes equally costly? | No: missed High Harm / Red Flag are worst | Binary triage, human gates |
| Is latency important? | < 3 s target, 0.5–2 s/embedding (est.) | Embed once; pre-computed storage |
| Must models be retrainable? | Data grows; offline | Stored embeddings + retraining engine |

- **Visual:** The table itself, three-colour left rail (data / environment / risk).
- **Speaker explanation:** This is the generalisable artefact; ask the audience which row they would have discovered last (hint: independence).
- **Sources:** D03 §3, D02, D08, N4, N6, N15, [U]

### Slide 16 — Plan v1: the initial model plan
- **Core message:** The first plan was reasonable and explicit.
- **Content:** Representation: text embeddings (SBERT early, MPNet later) stored in a database. Models: LR per target (single-head) vs one multi-head model per text (4–5 heads) vs RF; BERT multi-head as the heavy alternative. Inputs: choose text 1/2/3 per target. Question to answer: single vs multi-head; which algorithm; all texts or separate [N4]. Hypothesis: a shared model could exploit correlation between labels [N4].
- **Visual:** Decision tree of the three planned choices.
- **Speaker explanation:** Note the reasoning: LR on embeddings had "made the job" in the previous exam project; multi-head was proposed to use label correlation — to be refuted in S22.
- **Sources:** N4 first section, OR-R conclusion

### Slide 17 — Plan v1: evaluation plan
- **Core message:** Success was defined before results: ≥0.8 accuracy, per-class analysis, external benchmark.
- **Content:** Threshold ≥0.8 accuracy ("acceptable, not outstanding") justified by JMIR (GPT-4o ≈68.8% concordance) [N4]. Metrics planned per target: macro F1, Cohen's κ, per-class F1, quadratic weighted κ (Severity), recall for ≥ moderate harm [N4]. Split: random 80/20 with the same split for every model [D02]. Per-target questionnaire (definition, complexity, signal, distribution, metrics, actions, consequence-if-used-as-is) [N4].
- **Visual:** Evaluation-plan card: Threshold · Metrics · Split · Reference.
- **Speaker explanation:** Praise the questionnaire; say honestly that the plan *did not include* cross-validation, a validation set or majority-class baselines (S33).
- **Sources:** N4, D02 §7, D04 §2.3

---

# PART III — DEVELOPMENT WAVES

### Slide 18 — Timeline of waves
- **Core message:** Planning evolved through four waves and three discoveries.
- **Content:** Brief (Oct 2025) → Wave 0 (111 records) → Wave 1 (flat baselines, Nov 2025) → **Discovery 1: accuracy lies** → **Discovery 2: labels depend on each other** → Plan v2 → **Discovery 3: taxonomy is a tree** → Wave 3 target-specific plans → Production (26 Feb 2026, 18 models).
- **Visual:** Horizontal timeline with plan/experiment/problem/insight/revised-plan swim-lanes.
- **Speaker explanation:** Use as a map; return to it as a progress bar on later slides.
- **Sources:** OR-P, OR-R, N4–N10, D04 §8

### Slide 19 — Decision Moment 1: the environment decides first
- **Core message:** The earliest, biggest planning decision was taken before any result.
- **Content:** Initial assumption: pick the best model for Arabic classification. Evidence: offline mandate, CPU, limited RAM, even Whisper medium hard [U]. Why insufficient: GPU/cloud options unavailable. New understanding: feasible set = frozen encoder + light classifier. **Decision (month 1) [U]:** pre-compute and store embeddings; build a retraining engine that reuses them. Result: all later experiments cheap; retraining at 1.5×/2×/2.5× possible [D03 §10]. Caveat: a prior, not a measured comparison with an LLM.
- **Visual:** Single "decision card" in the standard 6-row format used for all decision moments.
- **Speaker explanation:** Introduce the card format: assumption → evidence → insufficiency → understanding → decision → result.
- **Sources:** [U], OR-P §4, D02 §11, D03 §5.3, §10, D08

### Slide 20 — Wave 0 (111 records): first model comparison
- **Core message:** On tiny data, representation mattered more than model depth.
- **Content:** Compared: multi-head AraBERT, TF-IDF + LR, SBERT embeddings + LR. Domain acc: 0.609 / 0.391 / 0.609; Category acc 0.478 for the first and third, 0.435 for TF-IDF. Rare classes got F1 ≈ 0. Choice: SBERT+LR (lighter, semantic, local). Decision Moment 2 summary: *more complex ≠ better*.
- **Visual:** Grouped bars (Domain acc, Category acc) × 3 models; a callout "Listening, Safety: 0 F1".
- **Speaker explanation:** Mention the report's own warning (111 records "very small"). Do not use the BERT-vs-SBERT equality as proof (the tables are identical; dossier C9); use only the rationale. Later multi-head BERT fine-tuned on CPU gave Domain F1 0.27 [N5].
- **Sources:** OR-R pp. 5–9, N5 §3

### Slide 21 — Wave 1: flat baselines on the full data
- **Core message:** Without structure, only the easy targets pass the threshold.
- **Content:** Flat LR macro-F1 / accuracy: Domain 0.70 / 79.6%; Category 0.45 / 60.2%; Sub-category 0.22 / 45.2%; Classification_ar 0.15 / 31.2%; Stage 0.39 / 56.7%; Harm 0.24 / 47.3%. Only Domain close to 0.8.
- **Visual:** Horizontal bars of accuracy with a vertical 0.8 line; macro-F1 shown as dots.
- **Speaker explanation:** This is the evidence that the plan v1 is not enough; show the gap between accuracy and macro-F1 and leave it for the next slide.
- **Sources:** D04 §3.1, N4

### Slide 22 — Decision Moment 3: the accuracy trap
- **Core message:** High accuracy can mean a useless model; planning must fix metrics before trusting results.
- **Content:** Improvement type: RF 98.9% accuracy, macro-F1 0.497 (predicts Ordinary always). Severity: RF 82.8% accuracy, F1 0.40; production ordinal model: LOW recall 1.00, MEDIUM/HIGH recall 0. Harm binary: 93.8% accuracy, 0 of 6 High-Harm found. Decision: macro-F1 primary, per-class recall for rare high-stakes classes, class weights, human gates for High severity/harm/Red Flag.
- **Visual:** "98% accuracy / minority recall ≈ 0" three-panel: accuracy bar vs minority-recall bar for the three targets.
- **Speaker explanation:** Also say (lesson, not blame): even our final report mixed definitions — headline "F1 0.62–0.64" for Severity equals weighted F1; the per-class table (0.85, 0, 0) gives macro ≈ 0.28 (recomputed [B]). Metric definition must be planned and fixed.
- **Sources:** D04 §3.1, §7.1, §7.3, D11 §9.1, D09 §8; [B] arithmetic

### Slide 23 — The threshold and the "are we blocked?" decision
- **Core message:** When results miss the plan's threshold, planning chooses between data, labels, models or scope.
- **Content:** Options listed in N4: merge tags; synthesise data (×20); get more data; search for an LLM-based solution; ship the UI anyway? Decision after the meeting with the supervisor [N4]: proceed with the ML models as they are (even the non-viable), accumulate data and retrain; test off-the-shelf NER and Faster-Whisper; leave "launch vs collect more" open; design retraining at 1.5×/2×/2.5×. Also the reframing: *each model is its own project* [N4].
- **Visual:** Decision tree with the chosen path highlighted.
- **Speaker explanation:** Show that the threshold was a planning instrument; missing it produced a strategy, not a stop. Benchmark argument: "60% on 500 records is solid when GPT-4o on 1,816 records reaches ~70%" (comparability caveat on S39).
- **Sources:** N4 "Analysis" and "Decision Taking", N5 §6

### Slide 24 — Decision Moment 4: the labels are dependent
- **Core message:** A small anomaly (stacked > LR) revealed the structure of the problem.
- **Content:** Initial assumption: nine independent targets; parallel heads suffice. Evidence: stacked Category beats LR (63.4% vs 60.2%; F1 0.485 vs 0.448) because it receives the Domain model's output. Why insufficient: parallel heads share an encoder but do not condition on each other. New understanding: labels form a dependency graph mirroring human annotation order (Domain→Category→Sub-category→Severity→Stage→Harm). Decision: reject parallel multi-head networks; use sequential/stacked models with a different input set per target.
- **Visual:** Dependency graph (D01 Fig. 4) with the stacked arrow highlighted.
- **Speaker explanation:** Emphasise that the gain was modest (+0.037 F1); the value was the hypothesis. Stress "don't over-claim significance" (no test was run).
- **Sources:** N6, N6.2, D04 §5, D01 §12

### Slide 25 — Plan v2: five chosen, three abandoned
- **Core message:** Planning is pruning: every excluded approach had a stated reason.
- **Content:** *Chosen (to test):* LR baseline; stacked sequential models; XGBoost with injected features; ordinal LR (Severity only); better embeddings (MPNet multilingual). *Abandoned (considered):* parallel multi-head NN (ignores dependency, heavy), Bayesian networks (unstable with ~500 records, complexity), CRF (excess maths/feature engineering). Embedding alternatives rejected with reasons (AraBERT, mBERT, AraBART/CAMeL, ada, larger transformers).
- **Visual:** Two-column "Keep / Prune" with reason chips; embedding alternatives in appendix.
- **Speaker explanation:** Point out the status words: Bayesian networks and CRFs were *considered*, never built.
- **Sources:** N6, N6.2, D03 §5.1

### Slide 26 — Plan v2: one-page plan per target
- **Core message:** A model plan is a table: for each target, input, model, features, metric, risk.
- **Content:** Domain: patient text, LR/RF/XGB. Category: + Domain prediction, class weights. Sub-category: + Domain+Category, merge rare classes. Classification_ar: do not train as 78-class. Severity: patient+hospital text, ordinal LR, stacked. Stage: hospital text. Harm: all texts, binary + 7-class. Improvement type: simple. Global: embeddings, folders per feature, large comparison table, pick best.
- **Visual:** Per-target plan cards (the template reused for the final design on S30).
- **Speaker explanation:** This is the most literal "model plan" in the repository; compare with a generic plan template.
- **Sources:** N7, N6.2

### Slide 27 — Decision Moment 5: it isn't just dependent, it is a tree
- **Core message:** Testing an assumption on the data in ten minutes led to a different architecture.
- **Content:** Initial assumption: stacking probabilities is how to use dependency. Evidence: a manual check showed non-overlapping branches (e.g. Domain 1 → Categories 5, 7 only). Why insufficient: stacking still allows impossible label pairs. New understanding: split by branch. Decision: hierarchical pipeline (~11 models up to sub-category). Result: Category MANAGEMENT branch 0.448 → 0.85 F1; Environment sub-category 95% accuracy. Caveat: the branch is a 2-class sub-problem (n=61), so the comparison with 7-class is not like-for-like (majority baseline ≈64% [B]).
- **Visual:** Flat vs stacked vs hierarchical diagram plus a bar (0.448 → 0.485 → 0.85) with an asterisk.
- **Speaker explanation:** Quote the author's meta-lesson: *what quick test would have shown this earlier?* — profile label co-occurrence before choosing any architecture [N8, D12 lesson 1].
- **Sources:** N8, N9, D04 §6, D09 §9, D12 §13; [B] baseline

### Slide 28 — Decision Moment 6: different targets need different problem definitions
- **Core message:** Text source, ordering and rarity determine the model family per target.
- **Content:** Text-source experiments: Domain/Category/Sub-category → patient text; Severity, Harm → combined text; Stage → hospital text. Severity: ordinal (3→0 worse than 3→2), context-dependent → ordinal LR on text123. Harm: rare classes unlearnable → binary safety triage + fine-grained report model. Improvement type: LR; rare-event.
- **Visual:** Matrix: target × (text, structure, model, why).
- **Speaker explanation:** Link back to S4 (target structure decision).
- **Sources:** D02 §12, N10, D03 §6.2–6.5

### Slide 29 — Decision Moment 7: Stage — when the representation is the problem
- **Core message:** Planning can change the *representation*, not just the classifier.
- **Content:** Problem: stage cues diluted in a single embedding; multi-stage complaints with single label. Idea: ≈15 concept vocabularies → centroid → dot-product projection per sentence (≤6) → max → classify 15 scores. Result: interpretable, but accuracy 45.6%, F1 ≈0.25 (Care on the Ward 0.64; Discharge, Unspecified 0). Status: hint only, human review. Lesson: define in the plan what would count as success for a redesign, and compare against a correct baseline.
- **Visual:** Pipeline sketch: text → sentences → projections on concept vectors → 15 features → classifier.
- **Speaker explanation:** Present as justified by the structure of the problem [U]; note the baseline used to motivate it was weak (0.189 F1 from an experiment later flagged), while the correct hospital-text LR scored ≈0.46 — hence "baseline decides whether redesign looks necessary" (optional, author's call).
- **Sources:** N10, D03 §6.3, D04 §7.2, XL (corrected baseline) [dossier C14]

### Slide 30 — Decision Moment 8: dropping a target from the plan
- **Core message:** Some targets cannot be modelled; an honest plan removes them.
- **Content:** Classification_ar (78) / _en (73): F1 0.15 flat, 0.12 stacked, 0.14 production; ~6 records per class; ≥~50/class would need ~4,000 records. Decision: exclude from the AI pipeline, keep manual; no architecture can create data.
- **Visual:** Records-per-class ladder with the 78-class point far below the line.
- **Speaker explanation:** Contrast with the earlier hope in N4 that synthetic data ×20 might rescue it; this was never done [D12].
- **Sources:** D04 §9.1, D12 §2.3, N4

### Slide 31 — Production snapshot and operating rules
- **Core message:** The plan ended as a tiered, human-gated system, not a single accuracy number.
- **Content:** 18 production models (26 Feb 2026); tiers: high (e.g. Category MANAGEMENT 0.85, Harm binary "0.91" weighted*), moderate (Domain, Severity, CLINICAL), low (Stage ≈0.25, Classification_en 0.14 not deployed). Rules: Domain/Category suggestions need confirmation; HIGH severity, High Harm, Red Flag, Never Event cannot be committed automatically; Stage = hint only. Retraining gate: no model −5 pp F1.
- **Visual:** Traffic-light tier chart plus a human-gates flow.
- **Speaker explanation:** The asterisk: some "F1" values in this report are weighted averages; refer back to S22.
- **Sources:** D04 §8, D09 §7, D11 §8, D10 §8.3

---

# PART IV — RISKS AND EVALUATION

### Slide 32 — Risk register
- **Core message:** Model Planning is largely risk management; each risk linked to a detection and a decision.
- **Content (Risk → why → detected by → decision):** tiny data → no deep learning; imbalance → macro-F1, weights; rare classes → drop 78-class; dependent labels → hierarchy; subjective labels/no IAA → suggestions only; multi-stage single-label → projection; high-dim embeddings vs 380 records → LR baseline; safety-critical rare classes → human gates; hardware → no LLM/fine-tune; weak validation → *not addressed* (S33).
- **Visual:** Four-column risk table with colour-coded severity.
- **Speaker explanation:** Choose three risks to narrate; leave the rest on screen.
- **Sources:** D01 §14, D02 §13, D03 §13, D11 §9, D12 §2–3

### Slide 33 — Evaluation planning: what we would plan differently
- **Core message:** The evaluation plan itself had gaps — a teachable section.
- **Content:** (1) one random 80/20 split; branch test sets of 14–18 records; no cross-validation or confidence intervals. (2) Same test metrics used to compare models and pick the best (no validation set). (3) No majority-class baselines (MANAGEMENT branch 85% vs ≈64% [B]). (4) Cohen's κ planned but not computed for HCAT. (5) Metric definitions drifted (weighted vs macro). (6) External benchmark not like-for-like (2-class branch vs 7-class; binary Harm vs 5-class; different language). (7) Domain 82% (archive) vs 57.3% (Feb 2026 run, not explained). Recommended plan: stratified k-fold, nested validation, baselines, fixed metric definitions, per-class recall for safety classes.
- **Visual:** "Planned vs what we'd plan now" two-column checklist.
- **Speaker explanation:** Present as lessons from a real project; this demonstrates maturity and addresses likely questions from the professor.
- **Sources:** D02 §7, D04 §6.2, §8.1, §10; D09; D10 §8; [B]

---

# PART V — LESSONS

### Slide 34 — Real vs unconstrained planning
- **Core message:** Remove the constraints and the plan changes — which proves the plan was contextual.
- **Content:** Real: frozen MPNet + LR/XGB/hierarchy on CPU. Unconstrained [C/D12]: zero-shot LLM baseline first; fine-tuned Arabic BERT on GPU (+5–15 F1 expected per D12); LLM explanations; synthetic data via paraphrase/LLM; ~2,000+ records for fine-grained and Severity MEDIUM/HIGH; multi-label Stage. What would *not* change: hierarchy, metrics discipline, human gates.
- **Visual:** Side-by-side pipelines.
- **Speaker explanation:** Distinguish what is constraint-driven from what is structure-driven.
- **Sources:** D12 §9, [C], [D]

### Slide 35 — Planning as an iterative loop
- **Core message:** HCAT lived Plan → Build → Evidence → Revised plan several times.
- **Content:** Loop diagram annotated with the decision moments: DM1 environment; DM2 simple beats complex; DM3 accuracy trap + threshold; DM4 dependency; DM5 tree; DM6 target-specific; DM7 stage representation; DM8 drop a target.
- **Visual:** Circular loop with eight numbered nodes.
- **Speaker explanation:** One sentence per node; this is the closing synthesis of Part III.
- **Sources:** dossier §8

### Slide 36 — Methodological lessons (what planning should include)
- **Core message:** Practical rules distilled from the project.
- **Content:** Test label dependency/structure before choosing architecture [D12 L1]. Structure beats volume when structure exists [D12 L2]. Ordinal targets need ordinal thinking [L3]. Check per-class recall for safety classes [L4]. Evaluate targets one at a time; avoid generalising from one finding [N6.1]. Fix metric definitions and baselines early. Plan validation, not just models.
- **Visual:** Seven "rules" cards.
- **Speaker explanation:** Combine HCAT's own lessons with the meta-lessons from this analysis.
- **Sources:** D12 §13, N6.1, N8

### Slide 37 — Theory ↔ HCAT mapping
- **Core message:** Every theoretical planning decision has a concrete HCAT instance.
- **Content table:** Problem definition → nine targets, two texts. Target structure → tree + ordinal + rare. Features/representation → MPNet embeddings, text 1/2/3, projection scores. Candidate families → LLM, BERT, embeddings+ML, TF-IDF. Baselines → LR on embeddings, TF-IDF. Imbalance → macro-F1, class weights. Split → 380/96. Metrics → accuracy → macro-F1/per-class. Compute/latency/deployment → CPU, <3 s, offline. Interpretability → LR, projection scores. Operational risk/human-in-loop → gates. LLM vs classical → LLM excluded. Experiments for Building → next slide.
- **Visual:** Two-column mapping table (the checklist from S4 reused).
- **Speaker explanation:** Quick tour; invite the audience to spot the row that surprised them.
- **Sources:** all above

### Slide 38 — From Model Planning to Model Building
- **Core message:** A plan ends in an experiment list with decision rules.
- **Content (HCAT-style experiment list):** E1 baseline LR on stored embeddings per target. E2 text-source comparison (1/23/123). E3 stacked vs flat. E4 hierarchical per branch. E5 ordinal LR for Severity. E6 Stage projection vs rule/LR. E7 binary vs 7-class Harm. E8 retrain at 1.5×. Each with: input, metric, success rule, risk. *Generic template:* hypothesis, data, model, metric, threshold, stop rule.
- **Visual:** Experiment table with "owner phase = Building".
- **Speaker explanation:** Shows the boundary: planning writes this table; building fills the results column.
- **Sources:** N6.2, N7, D04, [C], [D]

### Slide 39 — A word on comparing with the literature
- **Core message:** External benchmarks guide planning only if comparable.
- **Content:** JMIR/Koh 2025: 1,816 English GP complaints, GPT-4o zero-shot: Domain 79.4% (κ 0.623), Category 69.8% (κ 0.571), Severity 53.9%, Stage 66.1%, Harm 74.2% (κ 0.162). HCAT: Arabic, cardiac, 476 records, classical ML offline. Differences: language, taxonomy, test size (~96 vs 1,816), class structure; HCAT's Category and Harm figures come from sub-problems. Use: calibrate the threshold; not claim superiority.
- **Visual:** Comparison table with a "comparable?" column.
- **Speaker explanation:** A short critical-thinking slide aimed at the professor; the safest claim is "competitive on Domain in a very different setting".
- **Sources:** N4, D04 §10, D09 §10; [B]

### Slide 40 — Take-aways
- **Core message:** Planning is contextual, iterative, evidence-driven and honest about risk.
- **Content:** (1) Constraints prune before experiments do. (2) The plan must define success, baselines and validation. (3) Structure in the targets matters more than the algorithm. (4) Metrics can lie; per-class view. (5) The best theoretical model ≠ the best operational model. (6) Plan → Build → Evidence → Re-plan.
- **Visual:** Six icons; closing quote: "More complex ≠ better; more structure = better."
- **Speaker explanation:** End on the loop and invite questions.
- **Sources:** dossier §13

---

# APPENDIX (reference slides, not narrated)

| # | Title | Content | Source |
|---|-------|---------|--------|
| A1 | Full text-source table | Which text for which target; Excel/notes experiments; official findings | D02 §12, N4, N10 |
| A2 | Phase 1 flat results (all 8 targets × LR/RF) | Full accuracy and macro-F1 table | D04 §3.1 |
| A3 | Stacked and hierarchical results | Domain, Category per branch, Sub-category per branch with n | D04 §5–6 |
| A4 | Production report (26 Feb 2026) | 18-model table with tiers and notes on metric definitions | D04 §8, D09 §7 |
| A5 | JMIR comparison with caveats | Full table and comparability checklist | D04 §10, D09 §10 |
| A6 | Embedding alternatives | AraBERT, mBERT, AraBART/CAMeL, ada, larger ST — reasons | D03 §5.1 |
| A7 | Abandoned approaches | Reasons and evidence | N6.2, D04 §9 |
| A8 | Stage projection maths | Centroid, unit vector, projection, max aggregation, ≤6 sentences, 15 metrics list | N10, D03 §6.3, D04 §7.2 |
| A9 | Severity and Harm detail | Per-class tables, ordinal rationale, two-stage design, zero recall | D04 §7.1, §7.3 |
| A10 | Dataset details | Text lengths, language examples, preprocessing stages, label cleaning | D02 §5–10, D01 §14 |
| A11 | Deployment constraints detail | RAM/latency estimates, start-up, offline update process | D08 §3–4, §11 |
| A12 | Evidence caveats | Table of known inconsistencies handled (dossier §12) | dossier |

---

## Design notes [D]
- Every decision moment uses the same six-row card (assumption → evidence → insufficiency → understanding → decision → result) so the audience learns the pattern once.
- Reuse the five colours of S4 across the deck (problem/data/models/environment/evaluation).
- Do not show: Category count pies, software architecture, UI, patient names.
- When quoting numbers, state the source version (e.g. "archive hierarchical", "production Feb 2026") on the slide footer.
