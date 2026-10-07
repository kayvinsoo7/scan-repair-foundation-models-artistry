# Scan Repair for Foundation Models: artifacts

**Saved base-model predictions, correction results and timings** for the AccML 2027 paper *Fix What the Model Leaves
Behind: Fast Subgroup Repair for Tree Ensembles and Tabular Foundation Models*. These cover TabPFN-2.5, TabICLv2,
XGBoost and CatBoost on a synthetic planted-subgroup benchmark and on Credit Default.

The code lives in the companion repository
[`scan-repair-foundation-models`](https://github.com/<GITHUB-USER>/scan-repair-foundation-models). Clone both side by
side and run `make paper-inputs` there: every table, figure and number of the paper is rebuilt from the files below on
a CPU, in minutes.

<!-- After the first Zenodo release: [![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.YYYYYYY.svg)](https://doi.org/10.5281/zenodo.YYYYYYY) -->

## Contents (about 80 MB)

```
logits/                          frozen base-model predictions (stage 1; GPU for the foundation models)
  tabpfn25_ladder/               TabPFN-2.5 at contexts 500 ... 10,000 and full; 3 datasets x 10 seeds (240 files)
  xgboost/  catboost/            tuned trees, full training split (30 files each) + tuned hyperparameters (tuned_*.json)
  tabicl/                        TabICLv2, full training split as context, T4 GPU (30 files)
  tabicl_a100/                   the same run repeated on an A100 (GPU-consistency check)
  discovery/{xgboost,catboost,tabpfn,tabicl}/   discovery-power experiment, n = 19,200 (40 files each)
corrections/                     every post-hoc correction applied to those logits (stage 2; CPU)
  <same folders>/posthoc.csv     one row per (dataset, seed, context, correction): test log-loss, Brier, AUROC, ECE,
                                 number of rules, audit result, planted-subgroup bias, oracle gain, Jaccard, seconds
  <same folders>/rows/*.npz      per-row repaired test logits of every correction (used by the attribution analysis)
  tabpfn25_ladder/report/        context-ladder report tables (T0-T10: gains, equivalent contexts, attribution, costs)
timing/
  timing_cpu_2core.csv           TabPFN-2.5 seconds per 1,000 predictions by context, 2-core x86-64 CPU
  timing_gpu_t4.csv              the same on a free-tier Colab T4 GPU
systems/                         implementation experiments (Sec. VI) and in-text checks
  cost_breakdown.csv, cost_scaling.csv     profile of the original vs compiled scorer
  cost4_scaling.csv, cost4_breakdown.csv   three implementations x six problem sizes; profiled parts of one fit
  cost4_realdata_equivalence.csv           identical repairs on Credit Default (original vs batched)
  fidelity.json, fidelity_screen.csv       proxy screen and logit-histogram (B = 64) fidelity checks
  scorer_calls.json                        exact-scorer calls in one default fit (64,813)
  discovery_oracle_gain.csv                likelihood gain of the true subgroup on discovery rows, per base model
  credit_baseline.csv, credit_xgb_params.json   original study's Credit Default tree baseline
SHA256SUMS                       checksums of every file
```

### File formats

- **`logits/**/<dataset>_seed<seed>_ctx<context>.npz`** (NumPy): `zd`, `zs`, `zt` are the logits on discovery,
  selection and test rows; `yd`, `ys`, `yt` are the labels; `ctx_idx` gives the training rows used as context (or
  training set); then `context_rows`, `fit_seconds`, `predict_seconds`, `rows_predicted`, and `info` (JSON: model,
  version, device). Rows and splits are regenerated deterministically by `experiments/ladder_data.py` in the code
  repository, which checks that the labels match.
- **Datasets:** `synth8` is the planted subgroup (odds ratio 8), `synth1` has no planted effect (odds ratio 1), and
  `credit` is Credit Default. `synth8x4d4` / `synth8x4d1` are the discovery experiment with 9,600 / 2,400 discovery rows
  (same base model, selection and test rows).
- **Seeds:** 1–10 (synthetic) and 42001–42010 (Credit Default).

## How these files were produced

| Folder | Script in the code repository | Hardware |
|---|---|---|
| `logits/tabpfn25_ladder`, `logits/tabicl`, `logits/discovery/{tabpfn,tabicl}` | `base_tabpfn_ladder.py`, `base_tabicl.py`, `base_fm_discovery.py` (`notebooks/foundation_model_logits_colab.ipynb`) | Colab T4 GPU; `tabpfn` 9.1.0, `tabicl` 2.2.0 |
| `logits/{xgboost,catboost}`, `logits/discovery/{xgboost,catboost}` | `base_trees.py` | 2-core CPU |
| `corrections/**` | `corrections.py`, `ladder_report.py` | 2-core CPU |
| `timing/` | `timing_tabpfn.py` (CPU: random-weight checkpoint of the same architecture, `make_timing_checkpoint.py`) | CPU / T4 |
| `systems/` | `systems_*.py`, `discovery_oracle_gain.py`, `credit_baseline.py` | 2-core CPU |

The correction results are deterministic given the logits: re-running `corrections.py` on these logits reproduces
`posthoc.csv` to machine precision (`make corrections` in the code repository checks this row by row).

## Licence

**CC BY 4.0** (see `LICENSE`). The predictions are derived from the UCI Default of Credit Card Clients dataset (Yeh,
2009; CC BY 4.0; https://doi.org/10.24432/C55S3H) and from synthetic data. No model weights are included.

## Citing

See `CITATION.cff`. After archiving on Zenodo, cite the DOI of the release you used.
