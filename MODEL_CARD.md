# CardioScan model card

**Model family:** ECGResNet1D-v1. **Version:** ptbxl-resnet1d-v1. **Status:** frozen and evaluated.

## Intended use

Research on PTB-XL diagnostic superclass classification, ML engineering demonstrations and de-identified educational waveform exploration. Users should understand multilabel classification and dataset limitations.

## Out-of-scope use

Clinical diagnosis, emergency triage, treatment selection, patient reassurance, autonomous screening deployment and any claim of regulatory approval. This is not a medical device.

## Training data

PTB-XL v1.0.3, 100 Hz twelve-lead, ten-second recordings. Folds 1–8 train (17,084 records), fold 9 validates (2,146), and fold 10 is the held-out test (2,158). Exact patients, class counts and IDs are in `split_audit.json`; 411 records without mapped diagnostic superclasses were excluded. Labels use diagnostic SCP-code presence and the supplied superclass mapping. Original release data were collected in historical clinical settings; the distribution does not represent every population or device.

## Method

15-sample stem, four downsampling residual stages with two kernel-7 blocks each and channels 64/128/256/512, BatchNorm/ReLU, adaptive pooling, dropout and five logits. Weighted BCE uses training-only positive weights. Per-lead mean/std are fitted on training data only. No filtering, sampling reweighting or focal loss is used. The seed-42 run used AdamW, batch size 64, initial learning rate 0.001, weight decay 0.0001, dropout 0.3 and full-precision CUDA on a GTX 1660 Ti. It stopped after epoch 22 and selected epoch 12 by validation macro AUROC.

Validation macro AUROC chooses the checkpoint, scheduler and stopping point. Per-class F1 thresholds are selected on fold 9 using the frozen best checkpoint. No calibration is fitted. Validation is reused for selection and thresholding; validation performance is not an unbiased final estimate.

## Evaluation procedure and metrics

The separate evaluation command opens fold 10 only after all model decisions are frozen. It uses saved normalization and thresholds without fitting. Macro/micro/per-class AUROC and average precision, F1, precision, sensitivity, specificity, support and confusion matrices are exported. Optional AUROC intervals use a patient-cluster percentile bootstrap, conditional on the fitted model.

The single frozen evaluation covered 2,158 ECGs from 1,877 patients in fold 10. Macro AUROC was **0.9230** (95% patient-cluster bootstrap CI 0.9155–0.9297), macro AUPRC 0.8171, macro F1 0.7417 and micro F1 0.7707. Per-class AUROC was 0.9232 CD, 0.9062 HYP, 0.9102 MI, 0.9428 NORM and 0.9327 STTC. See `metrics.json` for thresholds, AUPRC, precision, sensitivity, specificity, confusion matrices and all confidence intervals.

These results condition on one fitted model and one public retrospective dataset. They are not external, prospective, subgroup or clinical validation. Development/test duplicate waveform hashes and patient separation were checked. A changed model requires a new artifact; these test results must not become tuning feedback.

## Input requirements

Physical millivolts, numeric `(1000, 12)`, 100 Hz, ten seconds. Order: I, II, III, aVR, aVL, aVF, V1, V2, V3, V4, V5, V6. Finite values are mandatory. No silent resampling, reordering, reshaping or unit conversion. Basic serving checks reject flat leads (<1e-5 mV standard deviation) and amplitudes outside ±100 mV. These limits are engineering guardrails, not validated clinical signal-quality criteria. Training/evaluation retain finite labeled dataset signals, including natural quality variation. Benchmark results are not restricted to cases passing the serving quality gate.

## Output interpretation

Fixed order: CD, HYP, MI, NORM, STTC. Five independent sigmoid model scores and validation-selected thresholds; a positive flag is a threshold comparison. Scores are not calibrated disease probabilities, and no class is a definitive clinical finding. NORM and abnormal labels can both cross thresholds; the UI warns and preserves the outputs. Absence of a positive flag does not establish a normal heart or exclude disease.

## Limitations, bias and domain shift

Potential label noise, historical acquisition bias, class imbalance, repeated-patient correlation and differences in age, sex, devices, clinical prevalence and setting. No subgroup fairness, prospective effectiveness or external transportability claim is made. The superclass task does not cover all clinically relevant ECG conditions. Synthetic demos are illustrations with no diagnostic ground truth.

## Serving and privacy

Missing, corrupt, incompatible, nonfinite or synthetic artifacts cannot produce API predictions. CPU is the default device. Artifact checks validate structure and hash consistency, not authenticity; trust the source. Guest analyses are never stored. Signed-in storage is explicit and contains result summaries and optional de-identified Case IDs, not raw signals. Authentication is a portfolio baseline; production operations need independent hardening and review. No HIPAA/GDPR compliance claim is made.

## Provenance and attribution

Artifacts record architecture, class/lead order, normalization, validation thresholds, dataset version, seed, training date, best epoch, git commit/dirty status, software and data/checkpoint hashes. The inference checkpoint SHA-256 is `2e660482295f66c0b42e8f20cbd47c18ad6d5931f4b933a597488efd3f6ead85`; training used clean source commit `57e8e2f986d8d0722468ef796f8d3d6b2f565ded`. See [README citations](README.md#21-dataset-citation--license) and [PTB-XL on PhysioNet](https://physionet.org/content/ptb-xl/1.0.3/), CC BY 4.0.

**Research prototype — not a medical device and not intended for clinical diagnosis or emergency decision-making.** Performance on PTB-XL does not establish clinical effectiveness.
