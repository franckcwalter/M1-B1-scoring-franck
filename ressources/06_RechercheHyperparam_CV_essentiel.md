# Recherche d'hyperparamètres + validation croisée — Mini-cours ⭐ BONUS

> Brief associé : M1-B1 — **ressource bonus** (mission étoile / promo qui avance vite).
> Durée de lecture + pratique : ~35 min
> Pré-requis : [`03_RandomForest_hyperparams_essentiel.md`](./03_RandomForest_hyperparams_essentiel.md)
> lu et le réglage **manuel** de 2-3 jeux d'hyperparamètres déjà fait.

> 🧭 **Où ça se situe** — Ce mini-cours est **optionnel**. Le tronc M1-B1
> demande un réglage manuel (mini-cours 03). Ici on **automatise** la
> recherche et on la fait **proprement** (sans toucher au test). Le
> traitement approfondi de l'optimisation de performance, c'est **M6**
> (C8 N3) — ici on pose juste le réflexe et son coût.

## Pourquoi cette techno ?

Dans le mini-cours 03, tu as réglé RandomForest **à la main** : tu as testé
un jeu par défaut, un jeu `balanced`, un jeu « tuné ». Deux limites
apparaissent vite. D'abord ça **ne passe pas à l'échelle** : dès que tu as
4 hyperparamètres avec 4 valeurs chacun, tu ne vas pas tester 256
combinaisons à la main. Ensuite — plus grave — si tu compares tes jeux **sur
le test set**, tu es en train de **régler tes hyperparamètres sur les
données d'évaluation** : c'est une fuite, tes chiffres deviennent
optimistes et faux.

`GridSearchCV` et `RandomizedSearchCV` résolvent les deux d'un coup : ils
explorent **beaucoup** de combinaisons **automatiquement**, et ils évaluent
chaque combinaison par **validation croisée sur le train uniquement**. Le
test set reste vierge jusqu'au verdict final. C'est le geste standard pour
choisir des hyperparamètres sans se mentir.

> 🧠 **Le schéma mental à garder en tête** — tout le mini-cours tient dans
> ce flux. Le test n'est touché **qu'une seule fois, tout à la fin** :
>
> ```
> train ──▶ recherche + validation croisée  (on explore, on compare)
>                 │
>                 ▼
>            best_params_        (le meilleur jeu SELON la CV)
>                 │
>                 ▼
>        modèle final réentraîné sur tout le train
>                 │
>                 ▼
>            test  ◀── UNE seule fois, pour le verdict
> ```
>
> Et une phrase à retenir : **le tuning n'améliore pas le modèle « en
> soi »** — il **réduit la variance du choix des paramètres** sous
> contrainte des données dont tu disposes. Ce n'est pas de la magie, c'est
> de la sélection prudente.

**Alternatives à connaître :**

| Approche | Quand l'utiliser ? |
|---|---|
| **Réglage manuel** (mini-cours 03) | Premiers pas, 2-3 jeux, comprendre les leviers. Toujours commencer par là. |
| **`RandomizedSearchCV`** | **Le réflexe par défaut** dès que l'espace dépasse ~20 combinaisons. Meilleur ratio qualité/coût. |
| **`GridSearchCV`** | Espace **petit et discret** (< 30 combinaisons) qu'on veut balayer exhaustivement. |
| **Optimisation bayésienne** (Optuna, `skopt`) | Espaces énormes, entraînements coûteux. **Hors scope M1** — sur-ingénierie ici. À voir en culture M6. |

> ⚠️ **Quand NE PAS le faire.** La recherche d'hyperparamètres a un **coût
> CPU réel** (des dizaines à des centaines d'entraînements). Si ton
> RandomForest par défaut donne déjà un F1 satisfaisant, un tuning qui gagne
> +0,3 point de F1 pour 20 min de calcul **ne vaut souvent pas le coup** en
> production. Optimiser est une **décision économique**, pas un réflexe
> automatique. On y revient en fin de mini-cours.

## Concepts clés

- **Validation croisée k-fold** — Au lieu d'un seul split train/validation,
  on découpe le **train** en `k` plis (typiquement 5). On entraîne sur `k-1`
  plis, on valide sur le pli restant, et on **répète `k` fois** en changeant
  le pli de validation. Le score retenu est la **moyenne** des `k` scores.
  Avantage : l'estimation est **plus stable** et **n'utilise jamais le test**.
  Sur données déséquilibrées, on utilise **`StratifiedKFold`** pour garder la
  même proportion de défauts dans chaque pli.
  À retenir : la CV est **plus stable qu'un simple holdout** (elle moyenne
  `k` évaluations au lieu d'une), **mais elle garde une variance propre**.
  ⚠️ Le score CV reste donc **une estimation, pas la vérité** : il
  dépend du découpage des plis. Conséquence directe :
  **`best_params_` n'est pas « la » bonne combinaison absolue**, c'est la
  meilleure *selon ces folds-là*. Deux découpages différents peuvent
  désigner deux gagnants proches. On optimise donc **une estimation
  bruitée** — d'où l'intérêt de ne pas sur-interpréter un écart de 0,2 point.

- **`GridSearchCV`** — Balaye **toutes** les combinaisons d'une grille
  (produit cartésien). Attention à l'**explosion combinatoire** : 4
  hyperparamètres × 4 valeurs = 256 combinaisons, × 5 folds = **1280
  entraînements**. Exhaustif mais coûteux. **Il n'est pas « inutile »** pour
  autant : sur un **petit espace discret** (< 30 combinaisons), pour
  **déboguer** ou pour **analyser systématiquement** l'effet d'un
  hyperparamètre, la grille reste le bon outil. Il est juste **inadapté aux
  grands espaces**.

- **`RandomizedSearchCV`** — **Échantillonne** `n_iter` combinaisons au
  hasard dans des plages (ou des distributions) que tu définis. Avec
  `n_iter=30`, tu fais 30 combinaisons quel que soit le nombre
  d'hyperparamètres. **Résultat quasi équivalent au grid pour une fraction
  du coût** : c'est pourquoi c'est le réflexe par défaut (cf. Bergstra &
  Bengio 2012, en références). Pourquoi ça marche si bien ? Parce que les
  hyperparamètres **ne sont pas indépendants** (un `max_depth` élevé se
  compense par un `min_samples_leaf` élevé, `class_weight` interagit avec la
  métrique visée…) : le tirage aléatoire explore des **régions cohérentes**
  de l'espace au lieu de gaspiller des runs sur les nœuds artificiels d'une
  grille rigide.

- **`scoring` — tu optimises ce que tu déclares.** La recherche choisit les
  hyperparamètres qui maximisent **la métrique que tu lui passes**. Mais une
  métrique n'est jamais l'objectif : c'est un **proxy d'une décision
  métier**. `f1_macro` = « je traite les deux classes à égalité »,
  `roc_auc` = « je veux bien classer les risques », `accuracy` = « j'ignore
  le coût asymétrique d'un défaut raté ». Choisir la métrique, c'est déjà
  arbitrer côté client. En déséquilibre, `scoring="accuracy"` te donnera un
  modèle qui prédit « remboursé » tout le temps. **Passe `"f1_macro"` ou `"roc_auc"`** pour
  optimiser ce qui compte vraiment pour Pyrenex. Nuance utile :
  **`f1_macro`** traite les deux classes à **égalité stricte** (idéal quand
  la classe minoritaire « défaut » compte autant que l'autre) ; **`f1_weighted`**
  pondère par la fréquence (moins sévère sur la minoritaire) ; **`roc_auc`**
  juge le **classement** des probabilités et suppose donc des `predict_proba`
  à peu près exploitables. En doute pour Pyrenex : `f1_macro`.

- **Le coût comme critère de décision.** `best_score_` te donne le meilleur
  score CV, `best_params_` la combinaison gagnante. Mais la vraie question
  n'est pas « quel est le meilleur ? » — c'est **« le gain vs le modèle par
  défaut justifie-t-il le temps de calcul et la complexité ajoutée ? »**.
  Toujours reporter les deux dans `experiments.md` : score par défaut ET
  score tuné, avec le temps passé.

## Exemple minimal qui tourne

```python
# example_search.py — versions testées : python 3.11+, scikit-learn 1.5+
import time
from scipy.stats import randint
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import RandomizedSearchCV, StratifiedKFold

# Espace de recherche : des PLAGES, pas des listes figées
param_dist = {
    "n_estimators": randint(100, 400),
    "max_depth": randint(5, 20),
    "min_samples_leaf": randint(1, 20),
    "class_weight": ["balanced", None],
}

cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)

search = RandomizedSearchCV(
    estimator=RandomForestClassifier(random_state=42, n_jobs=-1),
    param_distributions=param_dist,
    n_iter=30,                 # 30 combinaisons échantillonnées
    scoring="f1_macro",        # on optimise la BONNE métrique
    cv=cv,
    random_state=42,
    n_jobs=-1,
    refit=True,                # réentraîne le meilleur modèle sur tout le train
)

t0 = time.time()
search.fit(X_train, y_train)   # ⚠️ TRAIN uniquement — le test reste vierge
elapsed = time.time() - t0

print(f"Meilleur F1 macro (CV) : {search.best_score_:.3f}")
print(f"Meilleurs params       : {search.best_params_}")
print(f"Temps                  : {elapsed:.1f}s pour {30 * 5} entraînements")

# Le meilleur modèle est prêt (refit=True) — on l'évalue UNE fois sur le test
from sklearn.metrics import f1_score
best = search.best_estimator_
print(f"F1 macro sur le TEST    : {f1_score(y_test, best.predict(X_test), average='macro'):.3f}")
```

Tu obtiens le meilleur jeu d'hyperparamètres **et** un score CV honnête,
sans avoir touché au test pour le réglage.

## Exercice guidé

À partir de ton squelette M1-B1 (données déjà préparées et splittées) :

1. **Grid exhaustif.** Lance un `GridSearchCV` sur une petite grille :
   `n_estimators ∈ {100, 200, 300}`, `max_depth ∈ {8, 12, 16}`,
   `class_weight ∈ {"balanced", None}`. Note le **nombre d'entraînements**
   (`len(cv) × combinaisons`) et le **temps**.
2. **Random search.** Lance le `RandomizedSearchCV` de l'exemple
   (`n_iter=30`) sur un espace **plus large** (plages continues). Note temps
   et `best_score_`.
3. **Compare.** Le random a exploré un espace plus vaste — pour un temps
   comparable ou inférieur, obtient-il un score aussi bon ?
4. **Le test de la décision.** Compare `best_score_` au F1 macro de ton
   **RandomForest par défaut** du mini-cours 03. **Écris dans `verdict.md` :
   le gain justifie-t-il de garder la recherche dans le pipeline de
   réentraînement ?**
5. **Mission étoile ⭐** : relance le random search avec `scoring="roc_auc"`
   au lieu de `"f1_macro"`. Les `best_params_` changent-ils ? Qu'est-ce que
   ça t'apprend sur « optimiser, oui, mais optimiser **quoi** » ?

**Solution attendue (point 3)** : le `RandomizedSearchCV` atteint
typiquement un score **à moins de 1 point** du grid exhaustif, tout en
ayant fait **beaucoup moins** d'entraînements et exploré des valeurs que la
grille ne contenait pas. L'argument central : **dès que l'espace
devient grand, random offre souvent le meilleur compromis qualité/coût**.
Attention à ne pas sur-généraliser :
sur un **petit espace discret**, le grid reste pertinent (exhaustif +
reproductible + lisible). La règle n'est pas « random > grid partout », mais
« random dès que l'espace grandit ».

**Solution attendue (point 5)** : les `best_params_` peuvent différer
(souvent `class_weight` bascule), parce que `roc_auc` et `f1_macro` ne
récompensent pas le même compromis précision/rappel. La leçon : le résultat
d'une recherche **dépend de la métrique cible** — la choisir est une
décision métier, pas un détail technique.

## Pièges fréquents

| Piège | Conséquence |
|---|---|
| `scoring="accuracy"` en déséquilibre | La recherche « gagne » avec un modèle qui ignore les défauts (F1 défaut ≈ 0) |
| `GridSearchCV` sur une grille trop large | Explosion combinatoire — la recherche tourne des dizaines de minutes |
| Faire `.fit()` de la recherche sur `X` complet (train+test) | Fuite majeure — le test n'est plus une évaluation honnête |
| Prétraiter (scaler/encoder) **avant** la CV | Fuite subtile — le préprocessing « voit » les plis de validation. Encapsuler dans un `Pipeline` |
| Oublier `StratifiedKFold` en déséquilibre | Des plis peuvent contenir trop peu de défauts — scores CV instables |
| `n_iter` trop petit (ex. 5) dans le random | Sous-échantillonnage de l'espace — on rate les bonnes zones |
| Reporter le score tuné **sans** le score par défaut | Impossible de juger si l'optimisation valait le coût |
| **« Plus je tune, meilleur ce sera »** (piège cognitif) | Faux : au-delà d'un point, gain marginal nul **et** sur-ajustement indirect à la CV. Savoir s'arrêter fait partie du métier |

### Symptôme → cause probable

| Symptôme | Cause probable |
|---|---|
| La recherche tourne pendant des heures | Grille trop large — passer en `RandomizedSearchCV`, réduire `n_iter` ou l'espace |
| `best_score_` (CV) très supérieur au score test | Sur-ajustement à la grille, ou fuite (préprocessing hors `Pipeline`) |
| Les `best_params_` sont sur une **borne** de la grille (ex. `max_depth=16`, le max) | La bonne valeur est probablement **au-delà** — élargir la plage |
| Aucun gain vs le modèle par défaut | Rendements décroissants — RF est robuste par défaut ; documenter que le tuning n'apporte rien ici |
| Score CV qui saute beaucoup d'un fold à l'autre | Déséquilibre + `KFold` simple — utiliser `StratifiedKFold`, vérifier la taille des plis |

## Pour aller plus loin

- Doc officielle : [scikit-learn — Tuning hyperparameters (Grid/Randomized Search)](https://scikit-learn.org/stable/modules/grid_search.html)
- Doc officielle : [scikit-learn — Cross-validation](https://scikit-learn.org/stable/modules/cross_validation.html)
- Article fondateur : Bergstra & Bengio, *Random Search for Hyper-Parameter Optimization*, JMLR 2012 — **pourquoi le random bat le grid**.
- Aurélien Géron, *ML avec scikit-Learn* ch. 2 — *Fine-tune your model* (Grid vs Randomized Search).
- 🔭 **Culture, pas M1** : optimisation bayésienne avec [Optuna](https://optuna.org/) — puissante mais **sur-dimensionnée** pour du tabulaire M1. On la situera en M6.
- 🔭 **Pont vers M6** : à force de comparer beaucoup de modèles sur les *mêmes* plis de CV, on finit par **sur-ajuster la validation elle-même** (le score CV devient optimiste). La parade — la *validation croisée imbriquée* (nested CV) — se verra en M6. En M1, il suffit de le savoir et de garder le test **vraiment** pour la fin.

## Vérification (checklist apprenant)

- [ ] Je sais redessiner le **schéma mental** (train → recherche+CV → best_params → modèle final → test **une seule fois**)
- [ ] Je sais expliquer **pourquoi on ne règle jamais les hyperparamètres sur le test**
- [ ] Je comprends la **validation croisée k-fold**, pourquoi `StratifiedKFold` en déséquilibre, et que le **score CV est lui-même une estimation bruitée**
- [ ] Je peux justifier **`RandomizedSearchCV` plutôt que `GridSearchCV`** (ratio qualité/coût) — **sans** en conclure que grid est inutile
- [ ] J'ai passé le **bon `scoring`** (`f1_macro` ou `roc_auc`, pas `accuracy`)
- [ ] J'ai comparé le modèle tuné au modèle **par défaut** et tranché dans `verdict.md` : **le gain valait-il le coût ?**
