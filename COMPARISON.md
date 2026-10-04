# Comparison with Bengali OCR baselines

These are measured local implementations on common inputs and a common scorer. They are not numbers copied from model cards. Source hashes and exact edit totals are in `comparison.json`; package versions and cached weight hashes are in `benchmark_environment.json`.

## Mozhi-Bengali word crops

Same 9,233 scorable official test items and 46,849 normalized reference characters. Lower is better.

| Model / implementation | Micro-CER | Micro-WER |
|---|---:|---:|
| **Round 2b** | **0.499%** | **1.254%** |
| Round 3 | 0.647% | 1.519% |
| bbOCR / APSIS-Net (`apsisocr` 0.0.7) | 1.279% | 2.358% |
| Surya OCR 2 base, Transformers | 10.032% | 20.950% |
| Surya OCR 2 official pipeline, llamacpp | 13.422% | 26.973% |
| Tesseract 5.5.3, `ben` | 19.471% | 41.071% |
| EasyOCR 1.7.2, `bn` | 42.919% | 69.043% |

Round 2b and Round 3 have lower measured crop CER than the archived APSIS-Net run. This claim is limited to this evaluation set and implementation. The models have different training data, and competitor training overlap with Mozhi is unknown. No state-of-the-art claim is made.

APSIS-Net receives RGB word crops directly. EasyOCR uses `readtext`, including its detector, so missed detections affect its score; this is a pipeline comparison rather than an isolated recognizer comparison. Tesseract uses OEM 1 / PSM 8. The official Surya pipeline uses a different backend and preprocessing from the Transformers adapter harness. All crop runs are rescored with identical references and normalization; adapter runs use greedy decoding with a 64-token cap.

## Modern printed pages after exact image overlap removal

The original 90-page Swapnil slice had **38 exact image matches in Round 3's Sahil training source**. Modern52 excludes those images using SHA256 identity alone. Scores below use the same remaining 52 pages and references. Round 2b is excluded from this primary table because it had trained on the earlier Swapnil pool, including these pages.

| Model / implementation | Micro-CER | Micro-WER | Recorded failures |
|---|---:|---:|---:|
| Round 3 | 7.486% | 8.713% | 0 |
| Surya OCR 2 base, Transformers + adaptive | 10.783% | 12.461% | 2 |
| Tesseract ben | 11.708% | 15.790% | 0 |
| EasyOCR Bengali | 14.918% | 19.631% | 0 |
| bbOCR / APSIS-Net | 15.965% | 24.101% | 0 |

The conventional baselines were run on all 90 original images on 2026-10-04, then scored on Modern52. The Surya results are rescored saved native-prompt runs. Original 90-page metrics are preserved as diagnostics in `comparison.json`. Exact image matches were removed against the staged page training sources; near-duplicates, edition overlap, and competitor training contamination remain unaudited. References may be partial. The adaptive decoder was developed using saved page outputs, and failed runs remain in the metric. See the [Round 3 evaluation protocol](https://github.com/mobashirrahman/surya-ocr-2-bengali-round3/blob/main/RESULTS.md) for failure details.

Most of Round 3's CER gain over the base comes from the base's two decoder failures. On the 50 pages where both returned without exceptions, Round 3 scores 7.359% CER versus 7.482% for base. The primary table retains failures; the 50-page comparison is a sensitivity analysis rather than a replacement score.

## Historical REID2019 pages

Same fixed 51-page selection. The unavailable image is counted as an empty prediction for every implementation.

| Model / implementation | Micro-CER | Micro-WER | Recorded failures |
|---|---:|---:|---:|
| Surya OCR 2 official pipeline, llamacpp | **15.714%** | **28.816%** | 1 |
| Surya OCR 2 base, Transformers + adaptive | 18.401% | 30.268% | 1 |
| Round 2b + adaptive | 28.291% | 39.187% | 1 |
| EasyOCR Bengali | 28.704% | 60.243% | 1 |
| Tesseract `ben` | 30.663% | 61.512% | 1 |
| bbOCR / APSIS-Net pipeline | 32.914% | 55.774% | 1 |
| Round 3 + adaptive | 33.110% | 41.063% | 5 |

These are end-to-end page pipelines, including detection, reading order, and decoding. Surya Transformers uses RGB downscaling to 1,568 pixels and the corrected native HTML prompt with an adaptive token budget; other engines use their native preprocessing. APSIS-Net's page wrapper uses the documented nearest-line assignment fix when upstream detection leaves words without an assigned line. Thus this table compares delivered system output rather than holding every internal component constant. Round 3 has a historical-page regression and more decoder failures than the base model.

## Additional candidates and diagnostic results

The archived third-party [`Sarjinkhan2003/bengali-crnn-easyocr`](https://huggingface.co/Sarjinkhan2003/bengali-crnn-easyocr) integration scored 94.017% crop CER and 89.891% historical-page CER. It is retained in `comparison.json` as **diagnostic_unvalidated**, excluded from the main ranking. Its integration, vocabulary mapping, and distribution transfer were not revalidated; the result should not be used to characterize the model's intrinsic capability.

The separate [`Sarjinkhan2003/bengali-ocr-recognition`](https://huggingface.co/Sarjinkhan2003/bengali-ocr-recognition) checkpoint has not been benchmarked here. Its self-reported metrics use a different evaluation and are not inserted into these tables. The [`sakib04/Bangla-OCR-TrOCR`](https://huggingface.co/sakib04/Bangla-OCR-TrOCR/tree/e0cce6038459c705df6da17a0f4329f11304e3b7) repository contains a model card but no checkpoint files at the revision inspected on 2026-10-04, so no local score is available.

Upstream references: [bbOCR](https://github.com/BengaliAI/bbocr), [APSIS-Net](https://github.com/mnansary/apsisnet), [EasyOCR](https://github.com/JaidedAI/EasyOCR), [Tesseract](https://github.com/tesseract-ocr/tesseract), [Surya OCR 2](https://huggingface.co/datalab-to/surya-ocr-2). The bbOCR repository directs current users to [Chitrolipi](https://github.com/BengaliAI/chitrolipi); this comparison covers the recorded APSIS-Net implementation and does not evaluate Chitrolipi.

## Reproduction

The public package supplies aggregate evidence and code. The saved source predictions and dataset files must be supplied locally; they are not redistributed. Commands assume execution from the Round 3 repository directory and placeholder paths replaced with local paths.

Audit the original 90-page manifest against all staged page training sources. The manifest is a private JSON list of `{id, image, reference}` records; each training directory contains `page_gt.txt` and `images/`:

```bash
python scripts/audit_split.py \
  --modern-manifest /path/to/private/modern90.json \
  --train-page-dir /path/to/training/swapnil/page \
  --train-page-dir /path/to/training/sahil/page \
  --train-page-dir /path/to/training/bn_synth_surya/page \
  --output /path/to/private/audit
```

Run each baseline in a separate compatible environment. APSIS-Net was run with `apsisocr==0.0.7`, `onnxruntime==1.19.2`, `fastdeploy-python==1.0.7`, `numpy==1.23.0`, and `bnunicodenormalizer==0.1.7`. EasyOCR used `easyocr==1.7.2`. Tesseract used version 5.5.3 with the recorded `ben.traineddata` hash. The supplied runner files are exact copies of those used for the modern comparison:

```bash
python scripts/baselines/run_bbocr.py \
  --items /path/to/private/modern90.json --split modern90 --device cpu \
  --timeout 300 --output /path/to/private/hypotheses_bbocr_modern90.jsonl
python scripts/baselines/run_easyocr.py \
  --items /path/to/private/modern90.json --split modern90 --device cuda \
  --timeout 180 --output /path/to/private/hypotheses_easyocr_modern90.jsonl
python scripts/baselines/run_tesseract.py \
  --items /path/to/private/modern90.json --split modern90 \
  --tesseract-bin /path/to/tesseract --tessdata-dir /path/to/tessdata \
  --lang ben --oem 1 --psm-reid 3 --timeout 120 \
  --output /path/to/private/hypotheses_tesseract_modern90.jsonl
```

Rebuild the comparison without running models (requires `rapidfuzz`):

```bash
python scripts/compare_saved.py \
  --runs-dir /path/to/archived/external-eval/runs/full \
  --finetune-dir /path/to/archived/surya2_ft \
  --mozhi-dir /path/to/mozhi/test/test \
  --reid-dir /path/to/extracted/reid \
  --modern-manifest /path/to/private/modern90.json \
  --modern-clean-manifest /path/to/private/audit/modern_clean.json \
  --modern-runs-dir /path/to/private \
  --output /path/to/aggregate-output
```

The scorer checks the 9,233/51/90/52 item sets, reference identities, and any archived per-example edit counts. JSON/CSV values are fractions; multiply by 100 to display percentages. The published SHA256 fingerprints identify the exact saved prediction files and frozen references used.
