# v1.0.0 — Initial public release

First release of **leakcheck**: a Python package that screens a reported classification result for the signature of synthetic over-sampling applied *before* the train/test split.

This is the version archived on Zenodo ([10.5281/zenodo.22196497](https://doi.org/10.5281/zenodo.22196497)) and intended for SoftwareX review. Source: <https://github.com/occbuu/leakcheck>.

## Install

```bash
pip install -e ".[dev]"
python -m pytest -q
```

Runtime dependency: **NumPy only**. The trained model ships as `src/leakcheck/model.json` and is evaluated by a pure-NumPy tree walker, so scores do not change when scikit-learn does.

## Use

```python
import leakcheck

r = leakcheck.check(
    f1=0.92, imbalance_ratio=5.2, n_minority=237,
    n_features=44, clf="RandomForest",
)
print(r.probability, r.verdict)
print(r.explain())
```

```bash
leakcheck --f1 0.92 --ir 5.2 --n-minority 237 --n-features 44 --clf RandomForest
```

## What's in this version

- Screening model fitted on 22,400 labelled leaked/honest pairs from 41 datasets, validated leave-one-dataset-out (**AUC 0.795**, Brier score **0.197**).
- CLI (`leakcheck`), batch screening (`check_many`), and `selftest()` fixtures that match scikit-learn to 1e-9.
- `paper.md` / `paper.bib` for the software paper.
- `Code/04_export_model.py` to refit and rewrite `src/leakcheck/model.json` from `results/raw_results.csv`.
- `Code/analysis.ipynb` and `figures/fig05_diagnostic.png`.

## Verdicts

| Probability | Label |
|---|---|
| < 0.35 | unremarkable |
| 0.35–0.65 | worth asking about |
| ≥ 0.65 | likely leaked |

A high probability is a prompt to ask where the resampler sits relative to the split. It is not a finding of error or misconduct.

## Requirements

Python ≥ 3.9; `numpy>=1.21`. Optional: `pandas` (`pip install 'leakcheck[table]'`).

## Licence

BSD 3-Clause. See `LICENSE.txt` (also `LICENCE.txt`).
