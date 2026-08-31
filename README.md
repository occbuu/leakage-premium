# leakcheck

Screen a reported classification result for the signature of resampling applied
**before** the train/test split.

Runtime dependency: NumPy only. The trained model ships as JSON and is evaluated
by a pure-NumPy tree walker.

## Install and test

```bash
pip install -e ".[dev]"
pytest -q
```

`pythonpath = ["src"]` is set in `pyproject.toml`, so `python -m pytest -q`
also works without an editable install.

## Use

```python
import leakcheck

r = leakcheck.check(f1=0.92, imbalance_ratio=5.2, n_minority=237,
                    n_features=44, clf="RandomForest")
r.probability
r.verdict
print(r.explain())
```

```bash
leakcheck --f1 0.92 --ir 5.2 --n-minority 237 --n-features 44 --clf RandomForest
```

## Retrain the shipped model

```bash
cd Code
pip install -r requirements.txt
python 04_export_model.py
```

That writes `src/leakcheck/model.json`. Notebook: `Code/analysis.ipynb`.

## GitHub + Zenodo DOI

```bash
git init
git add -A
git commit -m "Initial public release"
git branch -M main
git remote add origin https://github.com/<you>/<repo>.git
git push -u origin main
```

Then zenodo.org → GitHub → enable this repository → GitHub Release `v1.0.0`.
Put the DOI in the paper and in `CITATION.cff`.

## License

BSD 3-Clause. See `LICENSE`.
