# Credit Card Fraud Detection

Cost-sensitive fraud detection for card transactions. This is the code of my
Bachelor's thesis in Telecommunications Engineering at the Universitat
Politècnica de València (UPV): *Fraud Detection in FinTech Transactions using
Artificial Intelligence* ([thesis, PDF, in Spanish](https://marcosibanezfandos.github.io/Memoria_Entrega_Final_MarcosIbanez.pdf)).

Only 0.17 % of the transactions in the dataset are fraudulent, so accuracy says
almost nothing. The models here are judged by what their mistakes **cost**: a
missed fraud costs its amount, a false alarm costs the review and the friction
it causes. Probability calibration and threshold optimisation turn that cost
function into a decision rule, and a Streamlit app lets you explore the
trade-off interactively.

![Python](https://img.shields.io/badge/Python-3-3776ab?style=flat-square&logo=python&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-ML-f7931e?style=flat-square&logo=scikitlearn&logoColor=white)
![XGBoost](https://img.shields.io/badge/XGBoost-gradient%20boosting-189fdd?style=flat-square)
![LightGBM](https://img.shields.io/badge/LightGBM-gradient%20boosting-9acd32?style=flat-square)
![Streamlit](https://img.shields.io/badge/Streamlit-app-ff4b4b?style=flat-square&logo=streamlit&logoColor=white)

## Results

Stratified hold-out test set: 85,443 transactions, 148 of them fraudulent.

**Decision threshold tuned for F1**

| Model | Precision | Recall | F1 | ROC AUC | PR AUC |
|---|---:|---:|---:|---:|---:|
| Random Forest (baseline) | 0.930 | 0.804 | **0.862** | 0.931 | 0.823 |
| Random Forest (tuned) | 0.891 | 0.770 | 0.826 | 0.974 | 0.821 |
| XGBoost | 0.918 | 0.757 | 0.830 | **0.977** | **0.826** |
| LightGBM | 0.956 | 0.736 | 0.832 | 0.936 | 0.794 |
| Logistic regression | 0.869 | 0.764 | 0.813 | 0.967 | 0.697 |

**Decision threshold tuned for expected cost**

When the threshold minimises the cost model of the thesis instead of F1, the
ranking changes: the tuned Random Forest has the lowest total cost, catching
136 of the 148 frauds. Recall goes up at the price of more transactions sent to
review, which is exactly the trade-off a fraud team makes.

| Model | Recall | Frauds missed | Relative cost |
|---|---:|---:|---:|
| Random Forest (tuned) | 0.919 | 12 | **1.00** |
| LightGBM | 0.878 | 18 | 1.18 |
| XGBoost | 0.939 | 9 | 1.20 |
| Logistic regression | 0.899 | 15 | 1.46 |
| Random Forest (baseline) | 0.865 | 20 | 2.01 |

The full tables are in [`comparar/`](comparar) and [`comparar_coste/`](comparar_coste),
and the plots (confusion matrices, ROC and PR curves, gain charts, learning
curves, feature importances) in [`imagenes/`](imagenes).

## How it works

- **Data**: the public [Credit Card Fraud Detection](https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud)
  dataset (284,807 transactions by European cardholders, PCA-anonymised features).
- **Models**: logistic regression, Random Forest (baseline and tuned), XGBoost
  and LightGBM, trained on a stratified 70/30 split.
- **Class imbalance**: class weighting instead of oversampling, so the test set
  keeps the real fraud rate.
- **Calibration**: predicted scores are calibrated so they can be read as
  probabilities and compared against costs.
- **Threshold optimisation**: the operating point is chosen twice, once for F1
  and once for minimum expected cost, to show how the business goal changes the
  "best" model.

## Repository layout

```text
app_streamlit.py         Streamlit app to explore models, thresholds and costs
core.py                  Data loading and shared helpers
train_*.py               One training script per model
compare_models.py        Metric comparison at the F1-optimal threshold
compare_models_cost.py   Cost-sensitive comparison
modelos/                 Trained models (.pkl, stored with Git LFS)
comparar/, comparar_coste/, imagenes/   Results and plots
docs/seguridad.md        Security rules for changes to this repository
```

## Getting started

```bash
git clone https://github.com/MarcosIbanezFandos/Credit-Card-Fraud-Detection.git
cd Credit-Card-Fraud-Detection
git lfs install && git lfs pull        # pre-trained models

python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

Download `creditcard.csv` from Kaggle (it is not redistributed here) and place
it in `data/creditcard.csv`.

```bash
streamlit run app_streamlit.py         # interactive app on http://localhost:8501
python compare_models.py               # F1 comparison
python compare_models_cost.py          # cost comparison
python train_rf_optimized.py           # retrain a model (one script per model)
```

There is no hosted demo: the app runs locally.

## Author

Marcos Ibáñez Fandos · [Portfolio](https://marcosibanezfandos.github.io) · [LinkedIn](https://www.linkedin.com/in/marcos-ibanez/)
