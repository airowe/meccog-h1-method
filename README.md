# Method

MecCog Step 1 submission.

- **Team:** Groovy
- **Hypothesis:** H1, reduced endosome pH. "In some types of non-Alzheimer's, non-aged human brain cells, for example astrocytes, APOE4 causes reduced endosome pH compared with APOE3."
- **Submission date:** 2026-11-24 (work began 2026-11-23)

---

## 1. Who did the work

A person and an agent (`both`). One human team leader set the task, the scope and the rules, approved each bulk fetch, and holds the final hand-check. One orchestrating agent did the searching, screening, full-text retrieval, extraction, review and assembly.

Unlike this team's H2 submission, extraction was **not** handed to sub-agents: all 15 findings were written by the orchestrator directly from the full-text files, because the set is small. Sub-agents were used only for the blind review in section 5.

## 2. Model and version

- Orchestrator: `claude-opus-5-5` (Anthropic), run in Claude Code (desktop app).
- Blind review coders (3 groups): `claude-opus-5-5`, run as sub-agents of the orchestrator in the same session.

The H2 and H4 scripts this work reuses (`assemble.py`, `check_rows.py`, `phtest.py`, `hand_check_sample.py`, `pool_scan.py`) were written in earlier sessions by the same model.

## 3. How papers were found

All searching ran on 2026-11-23 and 2026-11-24. Every screened record is logged with a decision and a reason in `state/ledger.jsonl` in the work folder: **474 decisions over 259 unique records**.

**Round 1 sources:**

- **PubMed (NCBI E-utilities)**, 6 boolean queries, 96 records. The queries pair an APOE4 term with:
  1. endosome/lysosome/vesicle terms *and* pH/acidification terms
  2. the named pH regulators (NHE6, NHE9, SLC9A6, SLC9A9, V-ATPase, ATP6V*)
  3. the named acidity probes (LysoTracker, LysoSensor, pHrodo, pHluorin, acridine orange)
  4. a brain cell type *and* an endosome/lysosome term *and* an acidification term
  5. apoE isoform *and* endosome/lysosome *and* pH
  6. endosome/lysosome/autophagy dysfunction in astrocytes, microglia or iPSC cells
- **Europe PMC**, two full-text searches (447 and 354 records).
- **Forward citations** via the Europe PMC citations API for the key papers (180 records).

**Screening, round 1.** Titles were screened first, then the 221 survivors were run through a full-text pH test (`scripts/phtest.py`): it asks Europe PMC whether the paper's full text contains a pH-family term *and* both an APOE3 and an APOE4 term. Positive controls 29946028, 30375977 and 22383525 return 1; negative controls 42936906 and 42135730 return 0. 56 records hit; 13 reviews and 3 off-topic records were dropped; 40 went to full-text screen, of which 8 were kept.

**Round 1b (2026-11-24).** 709 records from the round 1 searches had only ever been screened on title. All 709 were put through the same pH test: 202 hit, 0 were missing from Europe PMC, and none were already in the ledger. Abstracts for all 202 were read: 191 excluded (reviews, conference abstract books, and papers where the pH hit is an incidental mention), 11 sent to full-text screen, 2 kept.

**What the pH test does not do.** It matches anywhere in the full text, including reference lists and buffer recipes, so most of its hits are incidental. Of the 11 round 1b papers read in full, 4 matched only on buffer pH or on "glial fibrillary **acidic** protein". The test is a cheap filter, not a finding.

**Searches that added nothing usable:**

- Pairing APOE4 with the named pH regulators (query 2) returned no paper measuring pH by genotype; the NHE6 literature is about expression and about rescue in cells that are not APOE-genotyped.
- The acidity-probe query (query 3) is dominated by pHrodo *cargo uptake* assays, which are a different readout (see section 6).
- Round 1b's 202 pH-test hits yielded 2 papers, a 1% hit rate; the remaining 198 were reviews, abstract books or incidental mentions.

**Ranking.** Papers are numbered P1, P2 and so on by their most relevant finding, then by mean finding relevance (`scripts/assemble.py`, tested in `scripts/test_assemble.py`). Relevance reflects how directly the paper measures **endosomal luminal pH** under APOE4 vs APOE3, and how close its system is to human brain cells.

## 4. How full text was obtained

Routes, in order:

1. Europe PMC `fullTextXML`
2. NCBI PMC efetch (`db=pmc`)
3. the PMC article HTML page, read in the team leader's Chrome

44 full texts were retrieved. Europe PMC returned a stub (about 150 bytes) for 8 of them, and NCBI served 7 of those 8. The eighth (18485101) is a publisher-forbids stub at NCBI too; it was read in Chrome on PMC and excluded on its content, so no row depends on it.

Five round 1 papers could only be read in Chrome; their text is not in `ft/`, they were screened by sentence filter only, and none were included. Figure panels were viewed in Chrome from the PMC CDN (`cdn.ncbi.nlm.nih.gov/pmc/blobs/...`); no images were saved.

One supplement was needed: Supplementary Figure 3 of 35031484. The Europe PMC `supplementaryFiles` endpoint returned a 176-byte zip whose single entry is 0 bytes, and a direct fetch of the PMC `bin/` URL returned a bot-check page. It was read by opening the .docx in Chrome on PMC and unzipping it in the page. The S3 legend is transcribed verbatim in `ft/supp/35031484/S3_legend.txt`.

**Nothing in this submission is behind a paywall we could not reach.** Every one of the 10 papers that contributes a row was read in full.

## 5. What a human checked

**At the time of submission the human hand-check is outstanding.** The sheet is machine-valid and agent-reviewed; a person has approved the wording changes in it but has not yet re-read the findings against the papers.

What has been checked, and by whom:

- **Automated, every row:** `scripts/check_rows.py` (field shapes, provenance codes, effect formats) and the official `validate_submission.py`. Both pass.
- **Agent blind review, 13 of 15 findings:** three sub-agent coders independently re-coded every row from the full-text files and the brief alone. No prompt carried our effect size, P value or n, and a planted value fires the leak detector. Result: 7 papers pass; one disagreement (42636814 Fig 5d) stands unresolved because the referee could not see the image. The review corrected one factual claim of ours about shared cell lines (section 6).
- **Human, so far:** the team leader approved the supplement fetch and the bulk full-text fetches, and approved five wording changes arising from the blind review. No value, P value, n, directness class, strength or against-flag was changed by that review.
- **Human, outstanding:** `H1_hand_check.xlsx` lists **all 15 findings** (not a sample, because the set is small enough to check whole) with per-row instructions. 42636814 Fig 5d is placed first, because it is the row that rests most heavily on reading a figure.
- **Not blind-reviewed:** the two papers added in round 1b (36825945 and 33450186) were extracted after the review ran. They are in the hand-check sheet but have not had a second agent read them.

## 6. Anything else

**The set is majority evidence AGAINST the hypothesis: 9 of 15 findings.** This is a real result, not a shortfall in searching. "Against" is applied mechanically and identically in every compartment: APOE4 pH equal to or higher than APOE3, or a non-significant difference. Lysosomal alkalinisation in APOE4 is therefore coded `against: true` at class 3, even though it is a well-replicated APOE4 phenotype, because H1 asks about *reduced* pH.

**The strongest direct evidence is split.** 29946028 (Prasad, PNAS 2018) reports APOE4 astrocyte endosomes *more acidic* with a ratiometric reporter, which is H1's exact claim; 36825945 (Somogyi, J Cell Sci 2023) reports *no difference* in endolysosomal pH in isogenic APOE4 iPSC neurons with an equally direct ratiometric method. Both are class 1. A reader should weigh them against each other rather than treating the set as one-directional.

**Three of the ten papers may share one cell model.** 32517777 and 42274838 both cite Morikawa et al. 2005 for their immortalised APOE3/APOE4 astrocytes; 29946028 cites Verghese et al. 2013 and a gift from the Holtzman lab. So the shared source is *confirmed* for two papers and *probable but not printed* for the third. Our first write-up stated all three shared the lines; the blind review caught that, and the correction is recorded. Treat those rows as fewer independent observations than the paper count suggests.

**Readouts deliberately excluded, with reasons:**

- **pHrodo cargo uptake and phagocytosis.** A pHrodo-labelled substrate reports uptake multiplied by acidity, and the authors read it as uptake. It is not a measurement of compartment pH. This excluded 38086389, 39612244 and 40419479 in round 1b, among others.
- **LysoTracker used as a label.** Where LysoTracker only marks lysosomes for colocalisation or counting, it is not an acidity readout (37455931, 18485101). Where it *is* read as acidity, the row is kept at class 3 strength 3-4, with a note that the signal also reflects organelle number and mass (40675218).
- **pH manipulated by drug with no genotype comparison of pH** (30594929, 20686698).

**Known weaknesses:**

- **Effect sizes are often read off figures.** 5 of 15 carry a leading `~` and `estimated_from_figure`. That convention is honest but imprecise, and those are the rows most worth hand-checking.
- **A pH difference cannot be expressed in the validator's effect formats.** The validator accepts a percentage, a fold change, `d=`/`r=` or `qualitative`. A difference in pH units is none of these, and converting it to a percentage would invent a statistic the paper never reported. Such rows are entered as `qualitative` with the printed value in the description ("Effect detail: 0.84 pH units").
- **41103078 has no P value** although significance stars are drawn: its legend key is printed garbled and defines no `**`. Entered as `N/A` with `ambiguous_panel` rather than guessed.
- **42636814 Fig 5d rests on our reading of an image.** Its only printed P value (0.044) belongs to a different comparison (the MβCD rescue), not to APOE4 vs APOE3. It is first in the hand-check.
- **33450186 is the weakest row in the set** (class 3, strength 5): it measures the mRNA of a V-ATPase accessory subunit, not pH. It is included because it is the only pH-regulator-expression row and it points in the direction H1 predicts, but the chain from mRNA to luminal pH is long and untested in that paper.
- **The compartment is often not the one H1 names.** H1 asks about *endosomes*. Several rows measure lysosomes or the whole endolysosomal compartment, which can move in the opposite direction in the same cells: 29946028 reports endosomes more acidic and lysosomes less acidic in the same APOE4 astrocytes. Each row's `system` and `location` state which compartment was actually measured.

**On the boundary of the hypothesis.** H1 specifies non-aged, non-AD, in-vivo human cells. Almost nothing in this literature satisfies that reading literally. Following the challenge rules (model-system findings are relevant and should be recorded precisely, with distance from the hypothesis left to a later stage), model systems were kept and the deviation named in the `system` column rather than used as grounds for exclusion. Relevance scores carry the discount.
