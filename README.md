# NeuroCoBot — source implementation

Prediction-error-gated recurrent motion forecasting for human–robot collaboration.

## Scope and provenance

This source-only release contains the model, dataset importers, training and metric reporting utilities. Generated demonstration data, associated checkpoints, figures and numerical results from the supplied archive are excluded. No original dataset recordings or verified experimental results are included. The numerical claims in the manuscript are not validated by this release. The hybrid, masking and horizon analyses are not included as validated real-data implementations.

## Setup

```bash
python -m pip install -r requirements.txt
export PYTHONPATH="$PWD/src"
pytest -q
```

## Data preparation

Obtain the original releases listed in `docs/REAL_DATASETS.md`. For D1–D3, export trial-level skeleton arrays using the original source tools and create a manifest with verified participant identities, labels and partitions; `data/manifest_example.csv` illustrates the schema only. Verify units, root joint, sampling rate and split membership before execution. The adapters default to 16 observed and 6 forecast steps, matching the manuscript; these defaults were adjusted from the supplied archive. Strides remain as implemented and must be audited. Example commands (paths are user-supplied):

```bash
python scripts/prepare_manifest.py --root data/raw/D1 --manifest data/raw/D1/manifest.csv --dataset D1 --out data/D1.npz --history 16 --horizon 6 --root-joint 0
python scripts/prepare_bath.py --root data/raw/Bath --out data/D4.npz --history 16 --horizon 6
python scripts/prepare_hrc_forecast.py --root data/raw/HRC --out data/D5.npz --history 16 --horizon 6
bash scripts/run_experiments.sh data/D1.npz D1
```

The D1 root-joint index in the example must be checked against the source. Repeat for D2–D3 with the appropriate source-specific configuration. The HRC importer uses trajectory filenames as group identifiers, not verified participant IDs; participant-level claims and conformal calibration require a validated participant mapping. The training code separates selection and calibration groups within validation; verify that this matches the intended evaluation protocol. Source provenance is currently marked external_unverified by importers and must be independently checked before publication. A filename is not evidence of provenance.

## Reporting

The runner executes four variants with five seeds. Reports contain run-level metrics and are generated from new runs only. Do not use the unverified outputs as evidence of completed public-dataset experiments without auditing source provenance and protocol. Unit tests use small constructed arrays to check implementation behavior, not scientific performance. The optional controller is not a safety-certified controller.

## Integrated publisher-data entry point

```bash
export PYTHONPATH="$PWD/src"
python -m neurocobot.sources download D2 --root data/downloads/D2
# Lists available publisher files; repeat with --files followed by selected names.
python -m neurocobot.sources download D1 --root data/downloads/D1
# Extract publisher archives and follow source export instructions before preparation.
python -m neurocobot.sources run D1 --root data/raw/D1 --manifest data/raw/D1/manifest.csv --root-joint 0 --out data/D1.npz
python -m neurocobot.sources run D4 --root data/raw/Bath --out data/D4.npz
python -m neurocobot.sources run D5 --root data/raw/HRC --out data/D5.npz
```

D2 and D3 use the same manifest interface as D1. Root joint 0 is an example, not a verified anatomical index. D4 archive must be downloaded from its DOI landing page. D5 uses the original folder split; verify participant mapping separately. The download API has not been exercised against the live archives in this environment. Each preparation creates a SHA-256 provenance sidecar; local file hashes establish identity, not authenticity. No existing manuscript metrics are imported. The updated calibration routine reserves groups only when a finite bound is feasible and does not claim calibration for D5 unseen subjects or deterministic models.
