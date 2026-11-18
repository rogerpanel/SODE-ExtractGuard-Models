# SODE-Guard: supplementary material (anonymised for review)

Code, configurations and every result file behind the tables and figures of
*Stochastic ODE-Guard: Robust and Certified Neural SDE Intrusion Detection*.

## Contents

| Path | What it is |
|---|---|
| `src/models/` | SODE-Guard (`sode_guard.py`), the deployed-model wrapper with post-hoc bias (`pipeline.py`), drift and diffusion networks |
| `src/sde/` | Euler--Maruyama integrator and virtual Brownian tree |
| `src/training/imbalance.py` | Module M1: logit-adjusted, focal and class-balanced losses, post-hoc logit adjustment |
| `src/training/robust.py` | Module M2: EOT-matched adversarial training (N-FGSM, PGD, TRADES); module M3: noise prefix with consistency regularisation |
| `src/certify/smoothing_sde.py` | Proposition 1: smoothing certificate for a random base classifier |
| `src/certify/mean_certificate.py`, `path_flip.py` | Theorem A with Corollary 1, Proposition 2 |
| `src/attacks/eot_autoattack.py` | PGD, EOT-PGD, APGD-CE/DLR with EOT, expected-margin attack, Square |
| `src/attacks/constrained.py` | feature-constrained attacks (future-work extension) |
| `src/evaluation/stats.py`, `operations.py` | CIs, paired bootstrap, Holm; temperature scaling, conformal sets and alarms |
| `scripts/v2_run.py` | all stages: `prepare`, `train`, `attack`, `certify`, `calibrate`, `constrained`, `aggregate` |
| `scripts/v2_driver.py` | resumable, time-budgeted multi-seed driver |
| `notebooks/v2_rerun_colab.ipynb` | Colab notebook that runs everything end to end |
| `results/v2/` | summaries and per-seed result files (see below) |
| `tests/` | unit tests (`pytest tests`) |

## Where each number in the paper comes from

* Tables 2, 4, 5 and Figures 2a and 3: `results/v2/runs/v2/summary.md` and
  `results/v2/runs/v2/seed_*/results.json` (`attack` entries; masking sweeps in seed 0).
* Table 3 and Figure 2b: the same files (`certify` entries).
* Table 6: `results/v2/runs/v2_fullcheck/` (full-strength re-attack, seed 0) and
  `results/v2/runs/v2_classtime/` (per-class chronological split, three seeds).
* Per-flow predictions used by the paired-bootstrap contrasts: `preds_attack_*.npz` and `preds_cert_*.npz`.
* After `prepare`, `python scripts/v2_run.py aggregate --out <run folder>` recomputes every
  summary from the per-seed files; the paired-bootstrap contrasts need the prepared splits.

## Reproducing

On Colab (GPU runtime):
1. Upload this zip through the Files panel.
2. Open `notebooks/v2_rerun_colab.ipynb` and place the dataset CSV files in `DATA_DIR`. Dataset links are withheld for review.
3. Choose *Runtime → Run all*.

The run is resumable and mirrors its results to Google Drive.

Locally:

```bash
pip install -r requirements.txt lightgbm xgboost
python scripts/v2_run.py prepare --csv "DATA/Containers_Dataset.csv" --label-col Label --drop-cols "Src Port" --out runs/v2
python scripts/v2_driver.py --out runs/v2 --seeds 0 1 2 3 4 --deadline $(( $(date +%s) + 86400 )) \
    --extras fullcheck ablations classtime smoothing calibrate constrained
```

Hardware: one GPU per run. A SODE-Guard model trains in about 6–13 minutes on an A100 or T4. The five-seed core with all extras takes about 15–20 GPU hours on a T4.
