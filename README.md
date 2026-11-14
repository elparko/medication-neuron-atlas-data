# Medication neuron atlas: data

Data files for the medication neuron atlas, a map of what each feed-forward neuron in the open
language model Qwen3-8B responds to in medication text. The atlas browser on pereste.com reads
these files directly.

This describes how a language model reads text. It is not guidance for patient care, and it
says nothing about whether the model gives safe or correct medication advice.

## What is here

- `meta.json`: the model, the corpus, definitions, and run-wide numbers.
- `index.json`, `names.json`, `featured.json`: lists used for browsing.
- `u/`: one file per unit with a card: the 30,529 selected units, plus 502 near-silent units whose early cards are kept here but not shown in the browser (31,031 files).
- `s/`: compact summaries of all 884,736 units, in blocks of 512.
- `d/`, `c/`, `q/`: drug, drug-class and search lookups.

A unit is the high or the low values of one neuron. Each unit's file gives how often it fires
by label section and by kind of token, its strongest contexts, a written description, and the
scores of that description against a string-matching baseline.

## How it was made

- Model: Qwen/Qwen3-8B, 36 layers of 12,288 feed-forward neurons.
- Text: about 8 million tokens from 2,400 FDA prescription drug labels (openFDA) and about
  2 million tokens of general educational web text (FineWeb-Edu) for contrast, 10,002,874 tokens
  in all. The 2,400 labels span 474 drug classes, and 1,158 of them carry one.
- Descriptions were written by Claude Sonnet from each unit's strongest contexts and scored by
  Claude Haiku on held-out contexts.

## Known limits

- One general-purpose model. The text is drug label text with general web text for contrast;
  it includes no patient messages or prescriptions.
- A description covers a unit's strongest firing. It says little about its everyday firing.
- Many units were selected for a label section or drug class that their strongest contexts do
  not match. Each unit's file records whether its selection reason and its description agree.
- Drug-class links for small classes reflect the wording of a few labels, not pharmacology.

## Sources

- Drug label text: openFDA drug labeling.
- General text: short excerpts from the FineWeb-Edu dataset.
- Model: Qwen3-8B.
