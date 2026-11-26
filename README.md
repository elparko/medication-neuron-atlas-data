# Medication neuron atlas: data

Data files for the medication neuron atlas, a map of what each feed-forward neuron in the open
language model Qwen3-8B responds to in medication text. The atlas browser on pereste.com reads
these files directly.

This describes how a language model reads text. It is not guidance for patient care, and it
says nothing about whether the model gives safe or correct medication advice.

## What is here

- `meta.json`: the model, the corpus, definitions, and run-wide numbers.
- `index.json`, `names.json`, `featured.json`: lists used for browsing.
- `u/`: one file per unit with a card: the 30,529 selected units (`flagged: true`), plus 502 near-silent units (`near_silent: true`, `flagged: false`) whose early cards are kept here for auditability but not shown in the browser (31,031 files, stored at `u/<layer>/<pole_id>.json`).
- `s/`: compact summaries of all 884,736 units, in blocks of 512.
- `d/`, `c/`, `q/`: drug, drug-class and search lookups.

A unit is the high or the low values of one neuron. Each unit's file gives how often it fires
by label section and by kind of token, its strongest contexts, a written description, and the
scores of that description against a string-matching baseline.

## Recommended filtering for downstream research

Units were selected on corpus-wide everyday firing rates, whereas cards were written from each
unit's 20 strongest contexts. When selecting subsets of `u/` for downstream analysis, apply the
curated fields recorded in each unit's JSON (`u/<layer>/<id>.json`):

1. **Exclude near-silent units**: filter `flagged == true` and `near_silent == false` (leaves the 30,529 active selected units; excludes the 502 early-batch cards on units with peak activation `< 0.05`).
2. **Require strongest-context agreement (`check.strict_*`)**: a selection reason without its strict mark describes everyday firing enrichment only. To restrict to units whose 20 strongest contexts agree with their selection reason:
   - `check.strict_token == 1` (2,494 of 9,821 token-type units: rule-based tagger marks `>= 5` of the 20 strongest contexts as a selected token type).
   - `check.strict_section == 1` (2,278 of 8,894 section units: `>= 10` of the 20 strongest contexts lie in a selected label section).
   - `check.strict_class == 1` (249 of 4,139 drug-class units: `>= 5` of the 20 strongest contexts lie in labels of the class across `>= 2` distinct labels).
   - `check.strict_medication == 1` (8,865 of 10,000 medication-text units: `<= 10` of the 20 strongest contexts lie in web text).
   - `check.agrees_with_a_reason == 1` (13,103 of 30,529 flagged units agree with at least one selection reason).
3. **Require preference for medication labels over web text (`selectivity >= 1.0`) and minimum class size**: because the initial section and drug-class rules compared label sections or labels only against other label text, 3,662 section-selected and 2,843 class-selected units fire more often in general web text than in drug labels (`selectivity < 1.0`). For section- or class-focused analyses, require `selectivity >= 1.0` (`rate_label >= rate_general`) and, for pharmacologic classes, `check.class_size_tokens >= 20000` (`meta.json` `class_min_tokens`, met by 52 of 150 classes; retains 5,232 of 8,894 section units and 523 of 4,139 class units, or 5,715 of 12,558 units selected on either).
4. **Check card reliability and repeated boilerplate**: `scores.reliability_tier == "high"` (9,746 units) marks cards that found a clear pattern (`card.no_clear_pattern == false`), sit in the top three-fifths of their layer by peak activation, agree with a selection reason, and carry no repeated-text mark (`check.repeated_sentence == 0` and `check.nearby_wording_repeated == 0`).

## How it was made

- Model: Qwen/Qwen3-8B, 36 layers of 12,288 feed-forward neurons.
- Text: about 8 million tokens from 2,400 FDA prescription drug labels (openFDA) and about
  2 million tokens of general educational web text (FineWeb-Edu) for contrast, 10,002,874 tokens
  in all. The 2,400 labels span 474 drug classes, and 1,158 of them carry one.
- Descriptions were written by Claude Sonnet from each unit's strongest contexts and scored by
  Claude Haiku on held-out contexts.

## Known limits and data notes

- One general-purpose model. The text is drug label text with general web text for contrast;
  it includes no patient messages or prescriptions.
- A description covers a unit's strongest firing. It says little about its everyday firing.
- Many units were selected for a label section or drug class that their strongest contexts do
  not match. Each unit's file records whether its selection reason and its description agree
  (see `check.strict_*` and `selectivity >= 1.0` above).
- Drug-class links for small classes reflect the wording of a few labels, not pharmacology.
- **`firing_split` on `full_104` refill units**: in this static export (`2026-11-13`),
  `firing_split` is `null` for the 905 refill units of batch `full_104` (recorded in
  `values_extra.npy` after near-silent units were excluded) because the initial `mna.atlas_site`
  builder iterated only over `a.vcol` (`values_poles.json`) and omitted `a.ecol`
  (`values_extra_poles.json`). This bug has been fixed in `src/mna/atlas_site.py`
  (`firing_split_rows`) for subsequent site database builds; all card scores and decile hit
  records (`scores.score_top`, `scores.score`, `scores.by_decile`) for those 905 units are
  present and unaffected.

## Sources and license

See [`LICENSE`](./LICENSE) for full terms and attributions:

- **Original dataset metadata, statistics, and written descriptions**: licensed by Pereste Health, Inc. under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
- **General web text excerpts**: derived from [FineWeb-Edu](https://huggingface.co/datasets/HuggingFaceFW/fineweb-edu) (Penedo et al., NeurIPS 2024 Datasets and Benchmarks Track), licensed under [ODC-By 1.0](https://opendatacommons.org/licenses/by/1-0/) and subject to Common Crawl's Terms of Use.
- **Drug label text excerpts**: United States FDA Structured Product Labeling from [openFDA](https://open.fda.gov/apis/drug/label/) (Kass-Hout et al., *JAMIA* 2016; public domain / CC0 1.0; FDA endorsement not implied).
- **Model**: Qwen/Qwen3-8B (Apache License 2.0).
