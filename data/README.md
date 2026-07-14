## Data

The raw Jigsaw dataset is not included in this repository due to file size.

Download `train.csv` from Kaggle:
https://www.kaggle.com/competitions/jigsaw-toxic-comment-classification-challenge/data

Place it in this folder, then run `notebooks/EDA_and_data_sampling.ipynb` to generate:
- `stress_test_eval_set.csv` — 8,000-row evaluation set
- `few_shot_pool_fixed.csv` — 151k-row RAG retrieval pool
