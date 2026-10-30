# Verification record — 30 October 2026

These checks validate the implementation and record the frozen PTB-XL experiment.
Benchmark performance is not evidence of clinical effectiveness.

| Check | Result |
|---|---|
| Python ML/API suite | 43 passed; no warnings on the final run |
| Synthetic WFDB training smoke | Two epochs, best checkpoint, stored normalization and validation thresholds generated |
| Synthetic held-out evaluation | Metrics, patient-bootstrap intervals and all five requested plots generated |
| Real PTB-XL training | 17,084 records, seed 42, full-precision CUDA; stopped at epoch 22, best epoch 12 |
| Frozen fold-10 evaluation | 2,158 records / 1,877 patients; macro AUROC 0.9230 (95% CI 0.9155–0.9297) |
| Interrupted training | Resume reproduces uninterrupted model weights and thresholds on the deterministic fixture |
| Interrupted evaluation | Metadata finalization recovers without recomputing test predictions |
| JSON / CSV / NPY | Validation and prediction routes exercised with synthetic fixtures |
| Live API without a model | Health 503, prediction 503, metadata 200, upload validation 200 |
| Valid test checkpoint | Successful health/inference checked inside isolated FastAPI TestClient fixtures |
| Anonymous history | No persistence; authenticated storage requires explicit opt-in |
| History ownership | Cross-account access/deletion blocked |
| Frontend | Clean `npm ci`, ESLint and production build pass |
| Browser | Desktop and 390px mobile Playwright checks pass, including reduced motion and missing-model state |
| Dependency checks | npm audit: 0 vulnerabilities; Python installed dependency constraints compatible |
| Repository | `git diff --check` and Python compile checks pass |

## Real metadata audit

Official PTB-XL v1.0.3 metadata and the complete 100 Hz waveform tree were
downloaded from PhysioNet. Canonical superclass mapping and patient/fold checks
yielded:

| Partition | Records | Patients |
|---|---:|---:|
| Train, folds 1–8 | 17,084 | 14,823 |
| Validation, fold 9 | 2,146 | 1,917 |
| Held-out test, fold 10 | 2,158 | 1,877 |

411 records without mapped diagnostic superclasses were excluded. The run audit,
record IDs and class counts are published in `artifacts/ptbxl-resnet1d-v1/split_audit.json`.
Raw dataset files remain excluded from the repository.

## Definitive training environment

The default network has 8,739,973 parameters. Training used Python 3.12.10,
PyTorch 2.14.1+cu130 and a 6 GB NVIDIA GTX 1660 Ti, with deterministic algorithms,
batch size 64 and full precision. After the first epoch, epochs took approximately
18.1–18.5 seconds. A preliminary AMP attempt produced a non-finite first-epoch
loss on this machine and was discarded before a checkpoint existed; the frozen
run therefore used `--no-amp`.

## Practical limits

- Docker was unavailable; Dockerfiles and Compose files were reviewed, but images
  were not built or run locally. Equivalent Python and frontend checks passed locally.
- Calibration, external validation, subgroup analysis, prospective validation and
  clinical validation remain unperformed. Calibration is explicitly `none`.
- Browser WFDB pair upload is deferred. WFDB training ingestion is implemented.

## Repeat checks

From the repository root with the Python 3.12 environment selected:

```powershell
python -m pytest -q
python -m scripts.smoke --output artifacts/another-smoke-run
python -m training.audit --data-dir data/ptb-xl/1.0.3 --check-files
npm.cmd --prefix frontend ci
npm.cmd --prefix frontend run lint
npm.cmd --prefix frontend run build
```

On a fresh clone, the file preflight reports missing waveforms until `records100`
is installed. Browser missing-model checks require the backend running with an
empty `MODEL_DIR`; in
`frontend/`, set `$env:PW_CHANNEL='msedge'` on Windows or install Playwright
Chromium, then run `npx.cmd playwright test`.
