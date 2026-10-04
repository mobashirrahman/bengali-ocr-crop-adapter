# Bengali OCR Crop Adapter

A LoRA adapter for **printed Bengali word and block-crop OCR**, fine-tuned from [Datalab's Surya OCR 2](https://huggingface.co/datalab-to/surya-ocr-2).

> **0.499% micro-CER** and **1.254% micro-WER** on 9,233 scorable test crops —
> **20x lower character error than the base model** under the same evaluation
> harness, and lower than the archived APSIS-Net/bbOCR implementation.

| Model / implementation | Micro-CER | Micro-WER |
|---|---:|---:|
| **This adapter** | **0.499%** | **1.254%** |
| bbOCR / APSIS-Net (`apsisocr` 0.0.7) | 1.279% | 2.358% |
| Surya OCR 2 base, Transformers | 10.032% | 20.950% |
| Surya OCR 2 official pipeline, llamacpp | 13.422% | 26.973% |
| Tesseract 5.5.3, `ben` | 19.471% | 41.071% |
| EasyOCR 1.7.2, `bn` | 42.919% | 69.043% |

Lower is better. All rows are measured locally on the same 9,233 official
Mozhi-Bengali test crops with the same references, normalization, and scorer —
not numbers copied from model cards. Full precision values and protocol are in
[`RESULTS.md`](RESULTS.md); per-engine provenance is in
[`COMPARISON.md`](COMPARISON.md).

## Scope

This is a **crop** adapter: printed Bengali word and block crops, which is what
it was trained and measured on. It is the right tool for word-level datasets,
cropped scanned text, and OCR post-processing pipelines that segment first.

The model emits HTML-wrapped transcription (typically `<h2>…</h2>`); strip the
markup if you need plain text. For full-page OCR, use the base
[Surya OCR 2](https://huggingface.co/datalab-to/surya-ocr-2) model, which is
what it was fine-tuned from.

## Contents

Adapter weights and config, aggregate results, evaluation provenance, and
comparison evidence. The base model, training data, test images, and
per-example predictions are not included.

```
adapter_model.safetensors   LoRA weights (PEFT)
adapter_config.json         adapter configuration
results.json / results.csv  crop evaluation, full precision
comparison.json / .csv      multi-engine comparison
SHA256SUMS                  fingerprints for the released files
```

## Load the adapter

Install compatible versions of `torch`, `transformers`, `peft`, and `pillow`.
The base model requires a recent Transformers release that supports its
Qwen3.5 architecture; see the [upstream model card](https://huggingface.co/datalab-to/surya-ocr-2)
for current runtime requirements.

```bash
git clone https://github.com/mobashirrahman/bengali-ocr-crop-adapter.git
```

```python
import torch
from PIL import Image
from peft import PeftModel
from transformers import AutoModelForImageTextToText, AutoProcessor

BASE_ID = "datalab-to/surya-ocr-2"
REVISION = "3b3d4cdf88d6928b0acdc75181b13206ea67c4a3"
ADAPTER_PATH = "./bengali-ocr-crop-adapter"

device = "cuda" if torch.cuda.is_available() else "cpu"
dtype = torch.float16 if device == "cuda" else torch.float32

processor = AutoProcessor.from_pretrained(BASE_ID, revision=REVISION)
base = AutoModelForImageTextToText.from_pretrained(
    BASE_ID, revision=REVISION, dtype=dtype
).to(device)
model = PeftModel.from_pretrained(base, ADAPTER_PATH).to(device)

image = Image.open("crop.png").convert("RGB")
prompt = "OCR this block image to HTML."
inputs = processor(text=prompt, images=[image], return_tensors="pt").to(device, dtype)

with torch.no_grad():
    out = model.generate(**inputs, max_new_tokens=64, do_sample=False)

text = processor.batch_decode(out, skip_special_tokens=True)[0]
```

## Results

| Evaluation | n | Micro-CER | Micro-WER |
|---|---:|---:|---:|
| **This adapter** | 9,233 | **0.499%** | **1.254%** |
| Surya OCR 2 base, zero-shot | 9,233 | 10.032% | 20.950% |

In total the adapter makes **234 character edits over 46,849 reference characters**.
Exact values are in [`results.json`](results.json);
[`RESULTS.md`](RESULTS.md) documents the full protocol.

## Protocol

- Task: printed Bengali word/block-crop OCR.
- Prompt: `OCR this block image to HTML.`
- Greedy generation: `do_sample=False`, `max_new_tokens=64`.
- HTML markup stripped from model output before scoring.
- Text normalization: Unicode NFC, curly single quotes folded to straight
  apostrophes, whitespace collapsed.
- CER/WER are micro-averages across all scorable references. CER uses
  character-level Levenshtein edits; WER tokenizes Unicode letters, marks, and
  numbers before word-level Levenshtein scoring.
- The source test split lists 10,113 items. The harness excludes 880
  punctuation-only references that contain no letter, mark, or number token and
  are therefore not scorable.

All comparisons are within one harness and one test split. Different engines
have different training data and unknown competitor overlap with Mozhi-Bengali,
so this is a like-for-like pipeline comparison on this evaluation set rather
than a general state-of-the-art claim.

## Data and licensing

Training sources, dataset provenance, and attribution are documented in
[`DATA_PROVENANCE.md`](DATA_PROVENANCE.md). The adapter is a derivative of
Surya OCR 2; the upstream modified OpenRAIL-M license governs it, including use
restrictions, required attribution, and share-alike terms. See
[`LICENSE`](LICENSE) and [`NOTICE.md`](NOTICE.md).

## Citation

See [`CITATION.cff`](CITATION.cff). Cite this project and the upstream
[Surya OCR 2](https://huggingface.co/datalab-to/surya-ocr-2) model.