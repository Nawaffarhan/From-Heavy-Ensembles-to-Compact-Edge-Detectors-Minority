# From Heavy Ensembles to Compact Edge Detectors

Code and results for the article:

> **From Heavy Ensembles to Compact Edge Detectors: Minority-Aware Distillation for Intrusion Detection on DNP3 Traffic and PMU Measurements in IoT-Enabled Smart Grids**
> Suhaila Abuowaida et al. — submitted to *Computers* (MDPI), Special Issue "Wireless Sensing and IoT for Power System Monitoring, Protection and Operation".

The study compresses a heavy soft-voting ensemble (RF, ExtraTrees, XGBoost, LightGBM, MLP) into compact MLP and gradient-boosted-tree (GBDT) students for edge deployment at substations, and compares three training objectives: class-weighted cross-entropy (CE), standard knowledge distillation (KD) and minority-aware knowledge distillation (MA-KD).

## Repository layout

```
notebooks/
  01_dnp3_ensemble_kd.ipynb        # DNP3 network-layer experiments (6 attack families)
  02_msu_ornl_ensemble_kd.ipynb    # MSU/ORNL PMU physical-layer experiments (Attack vs. Natural)
scripts/
  make_figures.py                  # regenerates all paper figures from results/
results/
  dnp3/        results_main.csv, results_students_per_seed.csv, per_class.csv,
               split_diagnostic.csv, task_granularity_xgb.csv, shap_top20.csv, run_info.json
  msu_ornl/    same files (no granularity table)
figures/       paper figures (PNG, 400 dpi)
```

## Data

The datasets are not redistributed here. Attach them as Kaggle inputs:

| Dataset | Original source | Kaggle mirror used | Setting |
|---|---|---|---|
| DNP3 Intrusion Detection Dataset | IEEE Dataport, https://doi.org/10.21227/s7h0-b081 | `achyutdave/dnp3-intrusion-detection-dataset-final` | `PATH_INCLUDE="120_timeout"` (one flow-timeout export only) |
| MSU/ORNL Power System Attack Dataset | Mississippi State University / Oak Ridge National Laboratory | `bachirbarika/power-system` | binary label version (15 CSV files) |

## How to run

1. Create a Kaggle notebook, import a notebook from `notebooks/`, and add the matching dataset with **Add Data**.
2. **Run All** (CPU is sufficient; a GPU only speeds up the MLP models).
3. Results are written to `/kaggle/working/results_<dataset>/` and zipped to `results_<dataset>.zip`.

All settings are in the `CFG` dictionary in the first code cell. Key settings used in the paper:

| Setting | DNP3 | MSU/ORNL |
|---|---|---|
| Split | stratified 70/15/15 (`MAIN_SPLIT="random"`) | same |
| Labels | `TASK="family"` (6 classes) | binary (Attack / Natural) |
| MLP student | (64, 32) | (256, 128, 64) |
| GBDT student | 100 trees, 15 leaves | 500 trees, 127 leaves |
| MLP input scaling | signed log + standardisation | quantile → normal |
| KD temperature / α / MA-KD β | 4 / 0.5 / 0.5 | 4 / 0.5 / 0.5 |
| Seeds (students) | 42, 43, 44 | 42, 43, 44 |

To regenerate the figures from the saved results:

```bash
pip install -r requirements.txt
python scripts/make_figures.py
```

## Main results (test set)

| Dataset | Model | Accuracy (%) | Macro-F1 | Size | Latency (ms/record) |
|---|---|---|---|---|---|
| DNP3 | Ensemble teacher | 99.74 | 0.988 | 43.18 MB | 131.93 |
| DNP3 | GBDT student (CE) | 99.70 | 0.986 | 1.02 MB | 1.19 |
| DNP3 | MLP student (MA-KD) | 98.88 | 0.964 ± 0.007 | 32.7 KB | 0.136 |
| MSU/ORNL | Ensemble teacher | 92.40 | 0.905 | 808.60 MB | 213.29 |
| MSU/ORNL | GBDT student (CE) | 92.47 | 0.908 ± 0.001 | 6.48 MB | 1.35 |
| MSU/ORNL | MLP student (MA-KD) | 77.66 | 0.726 ± 0.004 | 297.3 KB | 0.205 |

Latency is the mean single-record `predict_proba` time on a Kaggle CPU; absolute values depend on hardware.

## Citation

If you use this code, please cite the article (citation details will be added after publication).

## License

MIT License — see `LICENSE`.
