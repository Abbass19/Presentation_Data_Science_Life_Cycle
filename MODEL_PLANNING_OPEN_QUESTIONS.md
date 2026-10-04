# MODEL PLANNING — OPEN QUESTIONS

Resolved items come from the author's answers (marked [U]); they are not stored in any repository file.

## Resolved

| # | Question | Answer [U] | Effect |
|---|----------|-----------|--------|
| R1 | Excel "Warning" cell | Not needed: the Excel is old work. | Official docs (D01–D12) are the reference. Excel/notes used only for "plan at that time". |
| R2 | Which embedding model is actually in use? | MPNet (author wrote "MPNF") | Production = `paraphrase-multilingual-mpnet-base-v2`. |
| R3 | Dataset/test-split conflict | Rely on official docs (476; 380/96) | Early numbers (n≈93) are labelled "early wave". |
| R4 | Meaning of Feb 2026 report metrics | Author doesn't remember. | Keep as an open item; present recomputed per-class evidence instead of claiming. |
| R5 | VM RAM/latency measured? | No, not measured. | Present as estimates. |
| R6 | LLMs tested? | No. RAM too limited ("eight [GB?] or less"); even Whisper medium was hard to load. | LLM = *excluded by constraint, never tested*. |
| R7 | When was the embedding-storage/retraining-engine decision taken? | Very early, first month of the work. | Becomes Decision Moment 1 (constraint-driven prior, not experimental). |

| R8 | Dataset sizes | 111 given before the job; full ~500 given at job start. | 476/485 treated as cleaned/archived versions. |
| R9 | Category counts | Counts keep changing; any snapshot is fine. | Avoid showing Category counts beside Domain counts. |
| R10 | Domain 57.3% vs 82% | Author doesn't remember. | Don't lean on the 57.3% figure. |
| R11 | Stage redesign | Author: it was justified. | Keep as design decision; baseline lesson is optional. |

## Still open (not blocking the outline)

1. **RAM figure.** "eight weeks of RAM or less" — I read this as "8 GB of RAM or less". Please confirm the number (even approximate) and the actual CPU core count if known.
2. **Whisper medium.** What exactly failed (could not load / too slow / crashed)? Which smaller Whisper variant did you end up with? (Useful as the only *empirical* hardware evidence.)
3. **Category distribution (D01).** The Category counts don't fit the Domain totals (e.g. MANAGEMENT 285 vs Environment+Institutional Processes = 76). Which category is actually the most frequent? (Notes and the old report suggest Institutional Processes.) Until confirmed, the Category pie chart will not be used.
4. **Production Domain 57.3% vs archive 82%.** Is there a known reason (new split, new data, a bug)? Which should the presentation call "the Domain model's performance"?
5. **Per-branch labels in the Feb 2026 report** (which numeric ID is which category). Needed only if we want to show per-branch bars.
6. **Weighted vs macro F1.** If you can regenerate the Feb 2026 metrics (even one model) with macro and weighted F1 side by side, that would turn my arithmetic (Severity macro ≈0.28 vs reported 0.62–0.64) into confirmed evidence.
7. **Dr. Hussein.** Role (supervisor / clinical lead?) and whether naming him on a slide is appropriate.
8. **Why 0.8 accuracy?** Was the threshold agreed with the hospital or chosen by you based on JMIR (N4 says justified by JMIR)?
9. **Note 13 ("three models… mix the 3 in a binary way… 500 records")** — what did "three models" mean and when was it written?
10. **Stage metrics.** Is the final production list 15 or 16 metrics, and was the strikethrough in note 10 "done" or "dropped"?
11. **The "991 records, balanced dataset"** used for the per-metric binary classifiers in D04 §7.2: what dataset is that?
12. **Presentation framing.** Are you comfortable showing the candid weaknesses (zero High-Harm recall, Stage below baseline, single validation split)? I plan to present them as planning lessons, not as failures of the project.
13. **Audience/time.** Language of the slides (English?), and the course's required structure for "Model Planning" (e.g. must mention specific textbook items like the EMC/Bill Franks lifecycle's model-planning tasks such as data exploration, variable selection, model selection, tools)?
