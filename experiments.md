# Expériences — M1-B1 Pyrenex Crédit (Lending Club)

> Trace tes runs au fur et à mesure. Format imposé : un bloc par run, avec
> date, modèle, hyperparams, métriques **test interne uniquement**, verdict.
> Commit à chaque run final (pas à chaque essai jetable).
>
> ⚠️ **Règle d'or — comparabilité.** Le holdout **n'apparaît jamais** dans les
> blocs `exp_NNN`. Il sort **une seule fois**, pour le modèle retenu, dans
> la section finale en bas de fichier. Cf. mini-cours 04.

---

## exp_001 — RF par défaut

- **Date** : 2026-07-07 10:30
- **Modèle** : RandomForestClassifier (sklearn 1.5.1)
- **Dataset** : lending_club_train.csv (sha256 d2da093bee40024b196e73a0d2d763193782f947e3d60552a3d7bbad0bd944e3), n=24000 (train 19200 / test 4800)
- **Split** : test_size=0.2, stratify=y, random_state=42
- **Hyperparamètres** : `n_estimators=100`, `n_jobs=-1`, `random_state=42` (reste par défaut)
- **Pré-traitement** : OneHotEncoder + StandardScaler (Pipeline scikit-learn)
- **Métriques (test interne)** :
  - F1 macro : 0.5142
  - F1 défaut : 0.1289
  - ROC-AUC : 0.7134
  - Recall défaut : 0.0725
  - Precision défaut : 0.58
- **Temps d'entraînement** : 0.55 s
- **Verdict** : Recall très bas (n'attrape que 7% des défauts, rate 93%), F1 défaut très bas (car le recall est bas); ROC-AUC ok donc bien classés, mais problème de seuil

---

## exp_002 — RF class_weight='balanced'

- **Date** : 2026-07-07 10:45
- **Modèle** : RandomForestClassifier (sklearn 1.5.1)
- **Dataset** : lending_club_train.csv (sha256 d2da093bee40024b196e73a0d2d763193782f947e3d60552a3d7bbad0bd944e3), n=24000 (train 19200 / test 4800)
- **Split** : test_size=0.2, stratify=y, random_state=42
- **Hyperparamètres** : `n_estimators=200`, `max_depth=10`, `min_samples_leaf=10`, `class_weight='balanced'`, `n_jobs=-1`, `random_state=42`
- **Pré-traitement** : OneHotEncoder + StandardScaler (Pipeline scikit-learn)
- **Métriques (test interne)** :
  - F1 macro : 0.6137
  - F1 défaut : 0.4362
  - ROC-AUC : 0.7444
  - Recall défaut : 0.6410
  - Precision défaut : 0.33
- **Temps d'entraînement** : 0.59 s
- **Verdict** : en ajoutant un class_weight balanced, on attrape désormais 64% des défauts (recall défaut de 0.6410), mais la précision chute de 58% à 33% (davantage de fausses alertes); c'est déjà plus intéressant pour le métier pour qui il est coûteux de manquer un défaut ;  

---

## exp_003 — RF class_weight={0:1, 1:8}

- **Date** : 2026-07-07 11:00
- **Modèle** : RandomForestClassifier (sklearn 1.5.1)
- **Dataset** : lending_club_train.csv (sha256 d2da093bee40024b196e73a0d2d763193782f947e3d60552a3d7bbad0bd944e3), n=24000 (train 19200 / test 4800)
- **Split** : test_size=0.2, stratify=y, random_state=42
- **Hyperparamètres** : `n_estimators=200`, `max_depth=10`, `min_samples_leaf=10`, `class_weight={0:1, 1:8}`, `n_jobs=-1`, `random_state=42`
- **Pré-traitement** : OneHotEncoder + StandardScaler (Pipeline scikit-learn)
- **Métriques (test interne)** :
  - F1 macro : 0.5120
  - F1 défaut : 0.4045
  - ROC-AUC : 0.7409
  - Recall défaut : 0.8573
  - Precision défaut : 0.2647
- **Temps d'entraînement** : 0.59 s
- **Verdict** : j'ai rendu plus coûteux de rater un défaut en jouant sur le class weight, toutes choses égales par ailleurs; le recall a bien augmenté jusqu'à 85%, mais la précision a encore chuté à 1/4 ; donc le F1 macro a beaucoup baissé, donc c'était trop agressif, sauf si Pyrenex a une aversion au risque très forte, quitte à rater des bons clients  

---

## exp_004 — RF balanced, plus d'arbres + plus profond

- **Date** : 2026-07-07 11:15
- **Modèle** : RandomForestClassifier (sklearn 1.5.1)
- **Dataset** : lending_club_train.csv (sha256 d2da093bee40024b196e73a0d2d763193782f947e3d60552a3d7bbad0bd944e3), n=24000 (train 19200 / test 4800)
- **Split** : test_size=0.2, stratify=y, random_state=42
- **Hyperparamètres** : `n_estimators=400`, `max_depth=20`, `min_samples_leaf=10`, `class_weight='balanced'`, `n_jobs=-1`, `random_state=42`
- **Pré-traitement** : OneHotEncoder + StandardScaler (Pipeline scikit-learn)
- **Métriques (test interne)** :
  - F1 macro : 0.6249
  - F1 défaut : 0.4170
  - ROC-AUC : 0.7400
  - Recall défaut : 0.5051
  - Precision défaut : 0.3551
- **Temps d'entraînement** : 1.40 s
- **Verdict** : on a remis balanced weights, mais on a fait des arbres plus profonds ; F1 défaut a baissé car le recall défaut a baissé (moins de défauts attrappés) même si la précision a augmenté de 2 points; ROC-AUX identique ; pas intéressant car le recall a baissé et c'est ça qui nous intéresse 

---

## exp_005 — RF balanced, moins d'arbres + moins profond

- **Date** : 2026-07-07 11:20
- **Modèle** : RandomForestClassifier (sklearn 1.5.1)
- **Dataset** : lending_club_train.csv (sha256 d2da093bee40024b196e73a0d2d763193782f947e3d60552a3d7bbad0bd944e3), n=24000 (train 19200 / test 4800)
- **Split** : test_size=0.2, stratify=y, random_state=42
- **Hyperparamètres** : `n_estimators=100`, `max_depth=6`, `min_samples_leaf=10`, `class_weight='balanced'`, `n_jobs=-1`, `random_state=42`
- **Pré-traitement** : OneHotEncoder + StandardScaler (Pipeline scikit-learn)
- **Métriques (test interne)** :
  - F1 macro : 0.6039
  - F1 défaut : 0.4377
  - ROC-AUC : 0.7428
  - Recall défaut : 0.6908
  - Precision défaut : 0.3204
- **Temps d'entraînement** : 0.24 s
- **Verdict** : balanced weights, arbres peu profonds : augmentation du recall par rapport à exp_002 de 64% à 69%, avec un petite baisse de la précision; le F1 reste stable;

---

## exp_006 — RF balanced, profond (max_depth=20) + min_samples_leaf=50

- **Date** : 2026-07-07 11:50
- **Modèle** : RandomForestClassifier (sklearn 1.5.1)
- **Dataset** : lending_club_train.csv (sha256 d2da093bee40024b196e73a0d2d763193782f947e3d60552a3d7bbad0bd944e3), n=24000 (train 19200 / test 4800)
- **Split** : test_size=0.2, stratify=y, random_state=42
- **Hyperparamètres** : `n_estimators=400`, `max_depth=20`, `min_samples_leaf=50`, `class_weight='balanced'`, `n_jobs=-1`, `random_state=42`
- **Pré-traitement** : OneHotEncoder + StandardScaler (Pipeline scikit-learn)
- **Métriques (test interne)** :
  - F1 macro : 0.6033
  - F1 défaut : 0.4325
  - ROC-AUC : 0.7442
  - Recall défaut : 0.6693
  - Precision défaut : 0.3195
- **Temps d'entraînement** : 1.03 s
- **Verdict** : on part d'un arbre profond dont on pose une taille minimale aux feuilles, avec min_samples_leaf pour voir si ça change quelque chose ; recall élevé (66%) mais moins bon que l'005 (69%); F1 défaut équivalent à celui des arbres peu profonds (43%) ; équivalent à 005 mais entrainement plus long (1s vs .25s)

---

## 🏁 Évaluation finale sur holdout (modèle retenu)

> **À remplir une seule fois**, à la tâche 5 du brief, **après** avoir choisi
> ton modèle retenu parmi les `exp_NNN` ci-dessus. Le holdout n'est consulté
> qu'ici.

- **Date** : 2026-07-07 14:00
- **Expérience retenue** : exp_005 (RF balanced court)
- **Modèle persisté** : `models/pyrenex_risk_v2.joblib`
- **Données holdout** : `data/lending_club_holdout.csv` (sha256 b5ca9339a6ddc4303b73e7b7529329de44e1bcfe72371639eb3d4a8a6209fc77, n=6000)
- **Métriques** :
  - F1 macro : 0.5994
  - F1 défaut : 0.4299
  - ROC-AUC : 0.7307
  - Recall défaut : 0.6745
  - Precision défaut : 0.3155
- **Matrice de confusion** :

|  | Pred Fully Paid | Pred Charged Off |
|---|---|---|
| **Vrai Fully Paid** | 3283 | 1614 |
| **Vrai Charged Off** | 359 | 744 |

- **Comparaison baseline 2017** : (cf. `verdict.md`)