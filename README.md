# Surya OCR 2 Bengali Crop Adapter — Round 2b

A LoRA adapter for printed Bengali word and block-crop OCR, fine-tuned from [Datalab's Surya OCR 2](https://huggingface.co/datalab-to/surya-ocr-2). Round 2b is the best crop result among the saved project runs on the Mozhi-Bengali test split.

> **Result:** 0.499% micro-CER and 1.254% micro-WER on 9,233 scorable test crops, compared with 10.032% CER / 20.950% WER for the base model under the same evaluation harness.

This repository contains the adapter weights and config, aggregate results, and provenance notes. It does not include the base model, training data, test images, or per-example predictions.

## Results

All rows use the same 9,233 scorable examples, prompt, greedy decoding, and scorer. Full precision values and protocol are in [`results.json`](results.json) and [`RESULTS.md`](RESULTS.md).

| Model | Micro-CER | Micro-WER |
|---|---:|---:|
| Surya OCR 2 base, zero-shot | 10.032% | 20.950% |
| Round 1 pilot | 1.104% | 2.911% |
| Round 1 full-train | 0.894% | 1.955% |
| Round 2 | 0.642% | 1.477% |
| **Round 2b — this adapter** | **0.499%** | **1.254%** |
| Round 3 | 0.647% | 1.519% |

Comparison with Bengali OCR baselines is in [`COMPARISON.md`](COMPARISON.md). The [Round 3 modern page adapter](https://github.com/mobashirrahman/surya-ocr-2-bengali-round3) is published separately.

## Intended use and limits

Use this adapter for **printed Bengali word or block crops**. The model emits an HTML-wrapped transcription (typically `<h2>…</h2>`); strip markup if plain text is needed.

Round 2b was trained with a mixed page-and-crop recipe, and is published because it had the strongest measured crop result. It is not a general page-OCR replacement: later page-level evaluation found the base model stronger on historical pages. One page corpus in the run was later removed from subsequent experiments after annotation-quality review; see [`DATA_PROVENANCE.md`](DATA_PROVENANCE.md). For full pages, use the base Surya OCR 2 model unless you have evaluated this adapter on your target material.

## Load the adapter

Install compatible versions of `torch`, `transformers`, `peft`, and `pillow`. The base model requires a recent Transformers release that supports its Qwen3.5 architecture; see the [upstream model card](https://huggingface.co/datalab-to/surya-ocr-2) for current runtime requirements.

Clone this GitHub repository, then use its local directory with PEFT:

```bash
git clone https://github.com/mobashirrahman/surya-ocr-2-bengali-round2b.git
```

```python
import torch
from PIL import Image
from peft import PeftModel
from transformers import AutoModelForImageTextToText, AutoProcessor

BASE_ID = "datalab-to/surya-ocr-2"
BASE_REVISION = "3b3d4cdf88d6928b0acdc75181b13206ea67c4a3"
ADAPTER_PATH = "./surya-ocr-2-bengali-round2b"

device = "cuda" if torch.cuda.is_available() else "cpu"
dtype = torch.float16 if device == "cuda" else torch.float32
processor = AutoProcessor.from_pretrained(BASE_ID, revision=BASE_REVISION)
base = AutoModelForImageTextToText.from_pretrained(
    BASE_ID, revision=BASE_REVISION, dtype=dtype
).to(device)
model = PeftModel.from_pretrained(base, ADAPTER_PATH).eval()

image = Image.open("crop.png").convert("RGB")
messages = [{"role": "user", "content": [
    {"type": "image", "image": image},
    {"type": "text", "text": "OCR this block image to HTML."},
]}]
inputs = processor.apply_chat_template(
    messages, tokenize=True, add_generation_prompt=True,
    return_tensors="pt", return_dict=True,
)
inputs = {key: value.to(device) for key, value in inputs.items()}
with torch.no_grad():
    output = model.generate(**inputs, max_new_tokens=64, do_sample=False)
text = processor.batch_decode(
    output[:, inputs["input_ids"].shape[1]:], skip_special_tokens=True
)[0]
print(text)
```

The base revision is pinned in the example. The original adapter config did not pin a base revision; the run's local Hugging Face cache pointed to this commit.

## Training summary

- Base: `datalab-to/surya-ocr-2`, cached revision `3b3d4cdf88d6928b0acdc75181b13206ea67c4a3`.
- PEFT LoRA: rank 8, alpha 16, dropout 0.05; 6.69M trainable parameters (~1.0%).
- Mixed training pool: 3,101 page examples over four epochs plus a subsample of 18,000 Mozhi-Bengali train crops; 30,404 interleaved training examples total. Round 2b resumed from step 22,600 and saved the final adapter after the run completed.
- Training used FP16 and AdamW with learning rate `1e-4` and effective batch size 8.

See [`DATA_PROVENANCE.md`](DATA_PROVENANCE.md) for source attribution and licensing. No source images or transcriptions are redistributed here.

## License and attribution

The adapter is a derivative of Surya OCR 2. It is distributed under the same modified OpenRAIL-M license supplied by the upstream model; read [`LICENSE`](LICENSE) before use. The license includes use restrictions, attribution, and share-alike obligations. This project is independent and is not endorsed by Datalab.

Mozhi-Bengali is attributed to IIIT Hyderabad / NLTM under CC BY 4.0. Source owners permitted public distribution of this adapter, as confirmed by the project maintainer; their datasets remain under their own terms and are not included here.
