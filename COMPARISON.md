# Comparison with Bengali OCR baselines

These are measured local implementations on common inputs and a common scorer.
They are not numbers copied from model cards. Source hashes and exact edit
totals are in [`comparison.json`](comparison.json); package versions and cached
weight hashes are in [`benchmark_environment.json`](benchmark_environment.json).

## Mozhi-Bengali word crops

Same 9,233 scorable official test items and 46,849 normalized reference
characters. Lower is better.

| Model / implementation | Micro-CER | Micro-WER |
|---|---:|---:|
| **This adapter** | **0.499%** | **1.254%** |
| bbOCR / APSIS-Net (`apsisocr` 0.0.7) | 1.279% | 2.358% |
| Surya OCR 2 base, Transformers | 10.032% | 20.950% |
| Surya OCR 2 official pipeline, llamacpp | 13.422% | 26.973% |
| Tesseract 5.5.3, `ben` | 19.471% | 41.071% |
| EasyOCR 1.7.2, `bn` | 42.919% | 69.043% |

The adapter's measured crop CER is lower than the archived APSIS-Net run, and
an order of magnitude below every other engine measured here. This claim is
limited to this evaluation set and implementation: the engines have different
training data, and competitor training overlap with Mozhi-Bengali is unknown.
No general state-of-the-art claim is made.

## What each number measures

These are delivered pipelines, not isolated recognizers, and the difference
matters when reading the table:

- **bbOCR / APSIS-Net** receives RGB word crops directly. The repository
  directs current users to [Chitrolipi](https://github.com/BengaliAI/chitrolipi);
  this comparison covers the recorded APSIS-Net implementation and does not
  evaluate Chitrolipi.
- **EasyOCR** uses `readtext`, including its detector, so missed detections
  affect its score.
- **Tesseract** uses OEM 1 / PSM 8 with the recorded `ben.traineddata` hash.
- **Surya OCR 2 official pipeline** uses a different backend and preprocessing
  from the Transformers adapter harness.
- **This adapter** runs through the Transformers + PEFT harness with greedy
  decoding and a 64-token cap.

All crop runs are rescored with identical references and normalization.

## Reproduction

The public package supplies aggregate evidence and code. Saved source
predictions and dataset files must be supplied locally; they are not
redistributed. Commands assume placeholder paths replaced with local paths.

Run each baseline in a separate compatible environment. APSIS-Net was run with
`apsisocr==0.0.7`, `onnxruntime==1.19.2`, `fastdeploy-python==1.0.7`,
`numpy==1.23.0`, and `bnunicodenormalizer==0.1.7`. EasyOCR used `easyocr==1.7.2`.
Tesseract used version 5.5.3 with the recorded `ben.traineddata` hash.

```bash
python scripts/baselines/run_bbocr.py \
  --items /path/to/private/mozhi.jsonl --split mozhi --device cpu \
  --timeout 300 --output /path/to/private/hypotheses_bbocr.jsonl
python scripts/baselines/run_easyocr.py \
  --items /path/to/private/mozhi.jsonl --split mozhi --device cuda \
  --timeout 180 --output /path/to/private/hypotheses_easyocr.jsonl
python scripts/baselines/run_tesseract.py \
  --items /path/to/private/mozhi.jsonl --split mozhi \
  --tesseract-bin /path/to/tesseract --tessdata-dir /path/to/tessdata \
  --lang ben --oem 1 --psm 8 --timeout 120 \
  --output /path/to/private/hypotheses_tesseract.jsonl
```

Rebuild the comparison without re-running the models (requires `rapidfuzz`):

```bash
python scripts/compare_saved.py \
  --runs-dir /path/to/archived/external-eval/runs/full \
  --finetune-dir /path/to/archived/adapter \
  --mozhi-dir /path/to/mozhi/test/test \
  --output /path/to/aggregate-output
```

The scorer checks the item set, reference identities, and any archived
per-example edit counts. JSON/CSV values are fractions; multiply by 100 to
display percentages. The published SHA256 fingerprints identify the exact saved
prediction files and frozen references used.

## Upstream references

[bbOCR](https://github.com/BengaliAI/bbocr) ·
[APSIS-Net](https://github.com/mnansary/apsisnet) ·
[EasyOCR](https://github.com/JaidedAI/EasyOCR) ·
[Tesseract](https://github.com/tesseract-ocr/tesseract) ·
[Surya OCR 2](https://huggingface.co/datalab-to/surya-ocr-2)