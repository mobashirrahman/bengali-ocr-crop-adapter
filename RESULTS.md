# Evaluation results

## Primary result

Round 2b scores **0.499477% micro-CER** and **1.253585% micro-WER** on 9,233 scorable items from the official Mozhi-Bengali test split. The base model scores 10.032231% CER and 20.949750% WER on the same items with the same local evaluation harness.

The source test split lists 10,113 items. The harness excludes 880 punctuation-only references because they contain no letter, mark, or number tokens and are not scorable by the project's benchmark metric. No test images, labels, identifiers, or per-example outputs are included in this repository.

## Run comparison

| Run | Micro-CER | Micro-WER | Scorable examples |
|---|---:|---:|---:|
| Surya OCR 2 base, zero-shot | 10.032231% | 20.949750% | 9,233 |
| Round 1 pilot | 1.103545% | 2.910868% | 9,233 |
| Round 1 full-train | 0.894363% | 1.954743% | 9,233 |
| Round 2 | 0.642490% | 1.476681% | 9,233 |
| **Round 2b** | **0.499477%** | **1.253585%** | **9,233** |
| Round 3 | 0.646759% | 1.519176% | 9,233 |

The exact values are in [`results.json`](results.json); [`results.csv`](results.csv) provides a spreadsheet-friendly table.

## Protocol

- Task: printed Bengali word/block-crop OCR.
- Prompt: `OCR this block image to HTML.`
- Greedy generation: `do_sample=False`, `max_new_tokens=64`.
- HTML markup is stripped from model output before scoring.
- Text normalization: Unicode NFC, curly single quotes folded to straight apostrophes, and whitespace collapsed.
- CER and WER are micro-averages across all scorable references. CER uses character-level Levenshtein edits; WER tokenizes Unicode letters, marks, and numbers before word-level Levenshtein scoring.
- The summary reports 234 character edits over 46,849 reference characters for Round 2b.

These are saved results from the project's evaluation run. The comparison is within one harness and test split; it does not establish page-level performance or performance on unrelated Bengali material.
