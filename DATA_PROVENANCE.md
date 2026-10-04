# Data provenance and attribution

## Training sources

The adapter was trained on printed Bengali word and block crops, sampled from
the following sources.

- **Mozhi-Bengali** — IIIT Hyderabad / NLTM. The official training split
  contains 80,113 word crops and is published under CC BY 4.0. Training used an
  18,000-crop sample drawn from that split, not the complete set. Evaluation
  used the separate official **test** split, which was held out.
  [Dataset page](https://ilocr.iiit.ac.in/dataset/4/) ·
  [paper/project page](https://cvit.iiit.ac.in/usodi/tdocrmil.php)
- **Synthetic Bengali pages** — locally generated pages; recorded sources
  include OFL fonts, CC BY-SA Wikisource text, and licensed news text.

Page-level sources used during the mixed-recipe experiments are recorded in
[`comparison.json`](comparison.json) provenance fields.

## Dataset permissions

The project records that the page-dataset owners permitted use for training,
and the project maintainer has confirmed permission to publicly distribute this
derived adapter. Original dataset files, crops, text, and per-example
evaluation outputs are not part of this repository and are not redistributed
here.

## Evaluation data

Evaluation uses the **official Mozhi-Bengali test split**, which is disjoint
from the training split sampled above. No test images, labels, per-example
predictions, or references are distributed in this repository; only aggregate
metrics and their SHA256 fingerprints are released, in [`results.json`](results.json),
[`comparison.json`](comparison.json), and [`SHA256SUMS`](SHA256SUMS).

## Base model

The adapter is derived from
[`datalab-to/surya-ocr-2`](https://huggingface.co/datalab-to/surya-ocr-2), base
revision `3b3d4cdf88d6928b0acdc75181b13206ea67c4a3`. The base model's modified
OpenRAIL-M license is included in [`LICENSE`](LICENSE). The base model license
governs this derivative and includes use restrictions, required attribution,
and share-alike terms.

This project is independent and is not endorsed by Datalab. See
[`NOTICE.md`](NOTICE.md).

## Benchmark environment

Package versions and cached weight hashes for every compared engine are
recorded in [`benchmark_environment.json`](benchmark_environment.json), so the
comparison can be reproduced against the same software stack.