# leakcheck

**Screening reported classification results for the resampling-before-splitting artefact**

`leakcheck` estimates whether a published classification result resembles the statistical pattern produced, in a controlled corpus, by applying a synthetic over-sampler *before* splitting a dataset into training and test partitions rather than to the training partition alone.

It takes only quantities that papers commonly report — the headline minority-class F1, the imbalance ratio, the number of minority cases, the number of features and the model family — and returns a model-estimated screening probability together with a short written explanation. The output is a **triage signal**, not a causal finding of leakage.

Runtime dependency: **NumPy only**. The trained model ships as JSON and is evaluated by a pure-NumPy tree walker, so scores do not change when scikit-learn does.

- GitHub: <https://github.com/occbuu/leakcheck>
- Zenodo: <https://doi.org/10.5281/zenodo.22196497>
- Support: [occbuu@gmail.com](mailto:occbuu@gmail.com)

## Code metadata

This table matches Table 1 of the accompanying Original Software Publication.

| Metadata | Description |
|---|---|
| Current code version | v1.0.0 |
| Permanent link to code/repository | Zenodo: [10.5281/zenodo.22196497](https://doi.org/10.5281/zenodo.22196497); GitHub: [occbuu/leakcheck](https://github.com/occbuu/leakcheck) |
| Legal code license | BSD 3-Clause (`LICENSE.txt` / `LICENCE.txt`) |
| Code versioning system used | git |
| Software code languages, tools | Python, NumPy; JSON model representation |
| Compilation requirements | Python ≥ 3.9 and NumPy ≥ 1.21 for inference; no separate compilation step and no scikit-learn dependency at run time. Optional: pandas (`pip install 'leakcheck[table]'`) for CSV batch screening. Training/export extras are listed in `Code/requirements.txt`. |
| Link to developer documentation/manual | This README |
| Support email for questions | [occbuu@gmail.com](mailto:occbuu@gmail.com) |

Keywords: data leakage; class imbalance; SMOTE; peer review; research software; reproducibility

## Purpose

Applying SMOTE or a related over-sampler to a whole dataset and only then splitting it is a documented form of data leakage. Reviewers and editors often cannot run an author’s code. `leakcheck` turns the general warning into a question about the case in hand: given what this paper reports, how unusual is this number relative to a controlled corpus?

Intended uses:

1. **Peer review.** Decide whether to ask where the resampler sits in the pipeline. `explain()` prints a paragraph suitable for a review report.
2. **Self-audit.** Check one’s own results before submission.
3. **Meta-research.** Screen a table of extracted published results to prioritise cases for methodological review. Screening flags alone do not estimate leakage prevalence.

## Install and test

```bash
git clone https://github.com/occbuu/leakcheck.git
cd leakcheck
pip install -e ".[dev]"
pytest -q
```

`pythonpath = ["src"]` is set in `pyproject.toml`, so `python -m pytest -q` also works without an editable install.

Inference-only install (NumPy):

```bash
pip install -e .
```

Batch CSV screening additionally needs pandas:

```bash
pip install -e ".[table]"
```

Confirm the shipped JSON model against its regression fixtures:

```python
import leakcheck
assert leakcheck.selftest()
```

## Use

### Python

```python
import leakcheck

r = leakcheck.check(
    f1=0.92, imbalance_ratio=5.2, n_minority=237,
    n_features=44, clf="RandomForest",
)
print(r.probability, r.verdict)
print(r.explain())
```

This is the IBM HR worked example from the paper (237 leavers among 1,470 employees, imbalance 5.2 to 1, 44 encoded features). The screen places a minority-class F1 of 0.92 in the “likely leaked” band. The appropriate response is to ask where the resampler sits relative to the split, not to infer that a paper is wrong.

### Command line

```bash
leakcheck --f1 0.92 --ir 5.2 --n-minority 237 --n-features 44 --clf RandomForest
```

```bash
leakcheck --file results.csv          # columns: f1, imbalance_ratio, n_minority, n_features [, clf]
leakcheck --info                      # print model provenance
```

### Batch API

```python
import leakcheck

table = leakcheck.check_many([
    dict(f1=0.92, imbalance_ratio=5.2, n_minority=237, n_features=44, clf="RandomForest"),
    dict(f1=0.50, imbalance_ratio=5.2, n_minority=237, n_features=44, clf="LogReg"),
])
```

Returns a pandas DataFrame if pandas is installed, otherwise a list of `Result` objects.

## Inputs

| Argument | Meaning |
|---|---|
| `f1` | Reported **minority-class** F1 (0–1) |
| `imbalance_ratio` | Majority cases / minority cases in the dataset as analysed |
| `n_minority` | Number of minority cases **before** any resampling |
| `n_features` | Number of predictors entering the model, after encoding |
| `clf` | Model family, in whatever spelling the paper used |

Unrecognised classifier names fall back to the tree-ensemble family (`RandForest`), with that fallback documented in the result rather than raising. Around thirty spellings map onto the seven families in the training corpus: 5NN, DecTree, GradBoost, LogReg, RandForest, SVM-RBF, XGBoost.

Inputs outside the fitted ranges still return a probability, but the result carries an explicit **extrapolation** note.

## Outputs

| Probability | Verdict |
|---|---|
| < 0.35 | unremarkable |
| 0.35–0.65 | worth asking about |
| ≥ 0.65 | likely leaked |

A high probability is a prompt to ask where the resampler sits relative to the split. It is **not** a finding of error or misconduct. Several non-leakage explanations can produce the same signature, including a macro-averaged F1 reported as a minority-class F1, unusual preprocessing, or a genuinely separable dataset.

## Screening model

The screen is a gradient-boosted classifier trained on **22,400** labelled results from **41** public datasets (11,200 matched resample-then-split vs train-only pairs). Leave-one-dataset-out validation: **AUC 0.795**, accuracy 0.697 at the natural threshold, **Brier score 0.197**.

The fitted domain (inputs outside it are flagged as extrapolations):

| Item | Value |
|---|---|
| Trees | 100 |
| Imbalance-ratio range | 1.02–109.9 |
| Minority-count range | 30–1813 |
| Feature-count range | 3–180 |
| Classifier families | 5NN, DecTree, GradBoost, LogReg, RandForest, SVM-RBF, XGBoost |
| Resamplers in the corpus | ADASYN, Borderline, RandomOS, SMOTE |
| Package version | 1.0.0 |

Dataset inventory: `data/processed/manifest.csv` (six employee-attrition datasets plus 35 Penn Machine Learning Benchmarks). Processed tables are the `.npz` files in `data/processed/`. The complete record of every model fit reported in the study is `results/raw_results.csv`; a machine-readable summary is `results/summary.json`.

The shipped estimator is `src/leakcheck/model.json`. A stored fixture set checks that the NumPy evaluator reproduces the original scikit-learn predictions to within 10⁻⁹.

## Retrain the shipped model

```bash
cd Code
pip install -r requirements.txt
python 04_export_model.py
```

That refits on `results/raw_results.csv` and writes `src/leakcheck/model.json`. Exploratory notebook: `Code/analysis.ipynb`. Diagnostic figure: `figures/fig05_diagnostic.png`.

## Repository layout

```
src/leakcheck/     package (JSON model + NumPy walker + CLI)
tests/             pytest suite
Code/              training/export scripts and analysis notebook
data/processed/    41-dataset corpus and manifest
results/           raw fit record and summary.json
figures/           diagnostic figure used in the paper
LICENSE.txt        BSD 3-Clause (SoftwareX filename)
LICENCE.txt        same licence, British spelling of the filename
```

## Citation

If you use this software, please cite the Original Software Publication (see `CITATION.cff`):

Le Ngoc, H., & Huynh Ngoc Thanh, T. (2026). *leakcheck: screening reported classification results for the resampling-before-splitting artefact* (v1.0.0). Zenodo. https://doi.org/10.5281/zenodo.22196497

The corpus on which the screening model was trained is described in the companion methodological study, released alongside this repository.

## Authors

- **Hieu Le Ngoc** (corresponding) — Faculty of Information Technology 2, Posts and Telecommunications Institute of Technology. ORCID: [0000-0002-1133-1433](https://orcid.org/0000-0002-1133-1433). Email: [lnhieu@ptit.edu.vn](mailto:lnhieu@ptit.edu.vn)
- **Trung Huynh Ngoc Thanh** — Faculty of Data Science, University of Finance – Marketing. ORCID: [0009-0001-5785-6630](https://orcid.org/0009-0001-5785-6630). Email: [hnttrung@ufm.edu.vn](mailto:hnttrung@ufm.edu.vn)

Questions about the software: [occbuu@gmail.com](mailto:occbuu@gmail.com)

## Funding

This research is funded by University of Finance – Marketing, HCMC Vietnam. This study is supported by Posts and Telecommunications Institute of Technology (PTIT), Vietnam.

## Licence

BSD 3-Clause. See [`LICENSE.txt`](LICENSE.txt) (also [`LICENCE.txt`](LICENCE.txt) and [`LICENSE`](LICENSE)).
