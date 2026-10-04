# Evaluation results

## Primary result

The adapter scores **0.499477% micro-CER** and **1.253585% micro-WER** on 9,233
scorable items from the official Mozhi-Bengali test split, against
**10.032231% CER** and **20.949750% WER** for the base model on the same items
under the same local evaluation harness — a **20.1x reduction in character
error rate**.

The summary reports **234 character edits over 46,849 reference characters**.

The source test split lists 10,113 items. The harness excludes 880
punctuation-only references because they contain no letter, mark, or number
tokens and are not scorable by the project's benchmark metric. No test images,
labels, identifiers, or per-example outputs are included in this repository.

## Results

| Model / implementation | Micro-CER | Micro-WER | Scorable examples |
|---|---:|---:|---:|
| **This adapter** | **0.499477%** | **1.253585%** | 9,233 |
| Surya OCR 2 base, zero-shot | 10.032231% | 20.949750% | 9,233 |
| bbOCR / APSIS-Net (`apsisocr` 0.0.7) | 1.279% | 2.358% | 9,233 |
| Surya OCR 2 official pipeline, llamacpp | 13.422% | 26.973% | 9,233 |
| Tesseract 5.5.3, `ben` | 19.471% | 41.071% | 9,233 |
| EasyOCR 1.7.2, `bn` | 42.919% | 69.043% | 9,233 |

Exact values are in [`results.json`](results.json) and [`comparison.json`](comparison.json);
[`results.csv`](results.csv) and [`comparison.csv`](comparison.csv) provide
spreadsheet-friendly tables.

## Protocol

- Task: printed Bengali word/block-crop OCR.
- Prompt: `OCR this block image to HTML.`
- Greedy generation: `do_sample=False`, `max_new_tokens=64`.
- HTML markup is stripped from model output before scoring.
- Text normalization: Unicode NFC, curly single quotes folded to straight
  apostrophes, and whitespace collapsed.
- CER and WER are micro-averages across all scorable references. CER uses
  character-level Levenshtein edits; WER tokenizes Unicode letters, marks, and
  numbers before word-level Levenshtein scoring.

## Scope of the comparison

Every row above was produced by a local implementation on the same 9,233
scorable official test crops, the same normalized references, and the same
scorer. None of the figures are copied from model cards.

The engines are compared as delivered pipelines rather than as isolated
recognizers: APSIS-Net receives RGB word crops directly, EasyOCR uses
`readtext` including its detector so missed detections affect the score, and
Tesseract uses OEM 1 / PSM 8. The official Surya pipeline runs a different
backend and preprocessing from the Transformers adapter harness. Adapter runs
use greedy decoding with a 64-token cap.

Because the compared systems have different training data and competitor
overlap with Mozhi-Bengali is unknown, this is a like-for-like comparison on
this evaluation set, not a general state-of-the-art claim.

These are saved results from the project's evaluation run. The comparison
establishes performance on this crop benchmark and does not establish
page-level performance.