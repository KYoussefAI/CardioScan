# Publication and recorded demo

## GitHub About

Description: **End-to-end 12-lead ECG classification system using PyTorch, PTB-XL, FastAPI and React.**

Topics: `ecg`, `deep-learning`, `pytorch`, `fastapi`, `react`, `ptb-xl`, `medical-ai`, `healthcare-ai`, `time-series`, `mlops`.

Use a 1280×640 social preview crop of the waveform workstation, with the title
“CardioScan · 12-lead ECG research workspace”. Keep “Synthetic demo signal” visible
if using the built-in preview. If adding a performance badge, identify it explicitly
as held-out PTB-XL v1.0.3 benchmark performance.
Screenshot path: `docs/images/workstation.png`; optional recording: `docs/demo.gif`.

## Before recording

- Use the published frozen `artifacts/ptbxl-resnet1d-v1` bundle; do not retrain or retune it for the demo.
- Point `MODEL_DIR` at that entire run directory and restart the backend.
- Check `/health`: `model_loaded=true`, expected version, `demo_mode=false`.
- Use a de-identified exported recording with permitted attribution, or clearly label the synthetic demo. Never display identifying data.
- Ensure `/api/model` reports the active artifact's fold-10 metrics and `synthetic=false`.

## Suggested 90-second sequence

1. Home: explain the research purpose and five independent superclasses.
2. Workstation: load a signal; show all leads, time/mV axes and quality checks.
3. Run classification; show all five scores and each threshold. Explain NORM conflicts if present.
4. Performance: show actual fold-10 results and the 1–8 / 9 / 10 protocol. Do not infer clinical effectiveness.
5. Optional account: explicitly save a de-identified Case ID; show owner-only history and delete the demo result.
6. Close on the architecture and reproducibility commands. Keep the research disclaimer visible.

For a missing-model demonstration, use an empty `MODEL_DIR` and retain the explicit
fail-closed state. Do not enable a random model or label smoke metrics as results.

## Publication language

“Built a reproducible ECG classification research prototype with patient-separated
PTB-XL folds, validation-only thresholds, shared preprocessing, artifact-checked
inference and a twelve-lead waveform workstation. The frozen model reached 0.9230
macro AUROC (95% patient-bootstrap CI 0.9155–0.9297) on 2,158 held-out PTB-XL
v1.0.3 fold-10 ECGs.” Keep the retrospective-dataset and non-clinical limitations
beside this result.
