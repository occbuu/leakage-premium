# Training / export

`04_export_model.py` refits the screening model on `results/raw_results.csv`
and writes `src/leakcheck/model.json` at the repository root.

```bash
pip install -r requirements.txt
python 04_export_model.py
```
