# Verdict — Modèle de scoring Pyrenex Crédit v2

> Document destiné à Sophie Léger (Lead Data, Pyrenex Crédit).
> 1 page max.

## Contexte

Le client utilisait un modèle qui commençait à faire de plus en plus de mauvaises prédictions suite à l'évolution des caractéristiques de sa base client. Il s'agissait de réentrainer un nouveau modèle avec des données client à jour, pour améliorer les prédictions.  

## Démarche

Le fichier client à jour a tout d'abord été scindé en deux, entre les données d'entrainement et les données de holdout (6000 lignes) qui ont servi au test final.  
Les données d'entraînement étaient composées de 24000 lignes, splitées en 19200 pour l'entrainement propre et 4800 pour les tests internes.  
Six configurations d'hyperparamètres ont été testées, par tâtonnement et aiguillés par les résultats des premiers tests, et consignés dans un fichier experiments.md.  
Les métriques retenues pour évaluer les entrainements ont été principalement le recall défaut et le F1 défaut : pour le client il est important de rater le moins de défauts possibles, tout en ne sacrifiant pas la précision pour ne pas refuser trop de clients solvalbles.

## Verdict chiffré

| Métrique | Baseline 2017 (Pyrenex-risk-v1) | Modèle retenu (v2) | Variation |
|---|---|---|---|
| F1 macro (holdout) | 0.5018 | 0.5994 | +0.0976 |
| F1 défaut | 0.086 | 0.4299 | +0.34 |
| ROC-AUC | 0.7296 | 0.7307 | +0.0011 |
| Recall défaut | 0.05 | 0.6745 | +0.62 |
| Precision défaut | 0.61 | 0.3155 | −0.29 |

**Configuration retenue** (exp_005) : RandomForestClassifier — `n_estimators=100`, `max_depth=6`, `min_samples_leaf=10`, `class_weight='balanced'`, `random_state=42`

Ce qu'on observe : 
- augmentation limitée du F1 macro de 9% 
- augmentation forte du F1 défaut 
- augmentation très signifcative du recall défaut 
- division par 2 de la précision  
- ROC-AUC stable 

## Trade-off explicité au métier

Le nouveau modèle détecte beaucoup plus de mauvais payeurs, seuls 33% ne sont pas détectés (précédemmnet 95% ne l'étaient pas).  
En revache, la précision a été divisée par deux, donc la majorité des alertes (2 sur 3) sont désormais des fausses alertes.  


## Précautions avant mise en production

- Vérifier que le **schéma d'entrée** en production correspond exactement
  au schéma d'entraînement (cf. `pyrenex_risk_v2.json` → `feature_columns`)
- Re-évaluer le **seuil de décision** (0.5 par défaut) avec l'équipe
  métier — un seuil 0.3 ou 0.4 peut être plus adapté selon l'appétence au risque
- Mettre en place un **monitoring** dès le déploiement (cf. M5/M6)
- Surveiller les **variables sensibles** identifiées (FICO, état US,
  revenu) — risque de disparate impact à auditer (M2/M7)

## Recommandation

✅ Remplacer Pyrenex-risk-v1 par Pyrenex-risk-v2.  
Malgré son imprécision, le nouveau modèle est beaucoup plus performant à détecter les possibles défauts.  
Pour le client, ne pas détecter un mauvais payeur est ce qu'il y a de plus coûteux, donc il semble que le nouveau modèle soit dans son intérêt.


---

*Signé : Franck Walter, FastIA, le 2026-07-07
