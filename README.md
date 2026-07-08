# M1-B1 — Réentraînement du modèle de scoring Pyrenex Crédit

Réentraînement et challenge du modèle de scoring crédit `pyrenex-risk-v1` (baseline 2017)
sur un nouveau dataset Lending Club, aboutissant à `pyrenex_risk_v2`.

---

## 🚀 Démarrage (4 commandes)

```bash
# 0. Clone le repo
git clone git@github.com:franckcwalter/M1-B1-scoring-franck.git
cd M1-B1-scoring-franck

# 1. Environnement virtuel
python -m venv .venv && source .venv/bin/activate     # Linux/macOS
# .venv\Scripts\activate                              # Windows

# 2. Dépendances
pip install -r requirements.txt

# 3. Vérification
python src/train.py --help     # → doit afficher l'usage du script
```

---

## 📁 Structure du repo

```
M1-B1-scoring-franck/
├── data/
│   ├── lending_club_train.csv           # 24 000 lignes (train + test interne)
│   └── lending_club_holdout.csv         # 6 000 lignes (évaluation finale)
├── notebooks/
│   └── M1-B1_franck.ipynb               # démarche complète (EDA → verdict)
├── src/
│   ├── preprocess.py                    # transformations reproductibles (Pipeline)
│   ├── train.py                         # entraînement + benchmark des configs
│   └── evaluate.py                      # métriques sur holdout
├── models/
│   ├── pyrenex_risk_v2.joblib           # 🎯 modèle retenu (Pipeline complet)
│   └── pyrenex_risk_v2.json             # métadonnées + metrics_holdout
├── ressources/                          # mini-cours d'appui
├── contract_test.py                     # valide shapes/classes/probas du .joblib
├── experiments.md                       # trace des runs (test interne)
├── verdict.md                           # note de synthèse pour le métier
├── requirements.txt
└── README.md
```

---

## 📓 Notebook (`notebooks/M1-B1_franck.ipynb`)

Déroulé de la démarche, une section par tâche :

- **0. Setup** — imports et chargement de l'environnement.
- **1. Comprendre la baseline** — rappel des métriques de `pyrenex-risk-v1` (2017) à battre.
- **2. EDA** — exploration du nouveau dataset et de ce qui a changé depuis 2017.
- **3. Préparation et split** — split stratifié `random_state=42`, holdout mis de côté.
- **4. Entraînement + benchmark** — comparaison des configs d'hyperparamètres et récap des runs.
- **4bis. Stabilité du score** — vérification du plancher de bruit en faisant varier le `random_state`.
- **5. Évaluation sur holdout** — mesure finale du modèle retenu (exp_005), consultée une seule fois.
- **6. Verdict** — synthèse et recommandation.

Chaque run est tracé dans [`experiments.md`](./experiments.md) (métriques **test interne** ;
le holdout n'y apparaît qu'une fois, en bas de fichier, pour le modèle retenu).

---

## 📦 Rendu

- **Modèle livré** : [`models/pyrenex_risk_v2.joblib`](./models/) — Pipeline scikit-learn complet
  (preprocessing + RandomForest), accompagné de [`pyrenex_risk_v2.json`](./models/) qui porte
  les métadonnées et les `metrics_holdout`. Configuration retenue : `exp_005`
  (RandomForest `class_weight='balanced'`, `max_depth=6`).
- **Verdict** : [`verdict.md`](./verdict.md) — comparaison chiffrée à la baseline 2017, trade-off
  expliqué au métier et recommandation.