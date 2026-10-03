# Data provenance and scope

## Training sources

The Round 2b run combined page and crop examples. Its training pool contained 3,101 unique page examples over four epochs and 18,000 sampled crops from the Mozhi-Bengali training split.

- **Mozhi-Bengali** — IIIT Hyderabad / NLTM; official training split contains 80,113 word crops and is published under CC BY 4.0. Round 2b used an 18,000-crop sample, not the complete split. [Dataset page](https://ilocr.iiit.ac.in/dataset/4/) · [paper/project page](https://cvit.iiit.ac.in/usodi/tdocrmil.php).
- **Synthetic Bengali pages** — locally generated pages; recorded sources include OFL fonts, CC BY-SA Wikisource text, and licensed news text.
- **Real page datasets** — `swapnil7777/bangla-ocr-dataset`, `shadid113/bangla-ocr-dataset`, `sahil74/bangla-ocr-1000`, and `arobin79/bangla-ocr-validation_data_printed`. Dataset source links: [Swapnil](https://huggingface.co/datasets/swapnil7777/bangla-ocr-dataset), [Shadid](https://huggingface.co/datasets/shadid113/bangla-ocr-dataset), [Sahil](https://huggingface.co/datasets/sahil74/bangla-ocr-1000), [Arobin](https://huggingface.co/datasets/arobin79/bangla-ocr-validation_data_printed).

The project records that the four page-dataset owners permitted use for training; the project maintainer has confirmed permission to publicly distribute this derived adapter. Original dataset files, crops, text, and per-example evaluation outputs are not part of this repository.

## Quality note

Post-run quality review identified that the Shadid page corpus was handwritten and contained annotation issues. It was removed from subsequent training/evaluation pools, but it was part of the historical Round 2b mixed training run. This is one reason this release is scoped to the measured crop result and should not be treated as a general page-OCR model. Later page-level results show the base Surya OCR 2 model performing better on historical REID pages.

## Base model

The adapter is derived from [`datalab-to/surya-ocr-2`](https://huggingface.co/datalab-to/surya-ocr-2), base revision `3b3d4cdf88d6928b0acdc75181b13206ea67c4a3`. The base model's modified OpenRAIL-M license is included in [`LICENSE`](LICENSE). The base model license governs this derivative and includes use restrictions, required attribution, and share-alike terms.
