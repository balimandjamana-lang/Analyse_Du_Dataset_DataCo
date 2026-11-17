# Analyse Du Dataset DataCo

Analyse complète du dataset **DataCo Supply Chain** couvrant les quatre niveaux de l'analytique : descriptive, diagnostique, prédictive et prescriptive. KPI, causes des retards de livraison, prévisions et recommandations (Power BI, DAX, Python).

## Démarche analytique

| Analyse | Question | Exemple dans le projet | Outils |
|---|---|---|---|
| **Descriptive** | Que s'est-il passé ? | KPI, revenu par région, tendance mensuelle, % de commandes livrées à temps | Mesures DAX, page 1 du rapport Power BI |
| **Diagnostique** | Pourquoi ? | Pourquoi tant de retards ? Analyse par mode d'expédition, région, catégorie, écart entre délai prévu et délai réel | Arborescence de décomposition et Influenceurs clés (Power BI) |
| **Prédictive** | Que va-t-il se passer ? | Prévision du revenu et des commandes ; probabilité qu'une commande soit livrée en retard | Prévision intégrée aux graphiques en courbe, Python |
| **Prescriptive** | Que faire ? | Quel mode d'expédition choisir par région ? Quel délai prévu annoncer ? | Recommandations chiffrées, paramètres What-if |

## Outils utilisés

Power BI, DAX, Python (Jupyter Notebook) avec `pandas`, `statsmodels` et `scikit-learn`.

## Méthodes du notebook

Le notebook Python couvre la partie prédictive du projet avec deux approches distinctes : une méthode statistique de séries temporelles pour les prévisions, et un modèle de machine learning pour le risque de retard.

### 1. Prévision du revenu et des commandes

- **Méthode** : lissage exponentiel (ETS), avec ou sans tendance amortie et saisonnalité annuelle.
- **Point de comparaison** : un modèle *seasonal naive* (valeur du même mois l'année précédente).
- **Sélection du modèle** : le meilleur modèle est choisi par un test sur les 6 derniers mois, selon le MAPE.
- **Bibliothèque** : `statsmodels`

### 2. Prédiction du risque de retard

- **Méthode** : classification par gradient boosting (`HistGradientBoostingClassifier`), avec encodage des variables texte via `OrdinalEncoder`.
- **Bibliothèque** : `scikit-learn`

### 3. Évaluation du modèle de risque de retard

- **Test temporel** : évaluation sur les 20 % de commandes les plus récentes.
- **Comparaison** : face à une règle simple basée sur le mode d'expédition.
- **Interprétation** : importance des variables par permutation.
- **Validation croisée** : `GroupKFold` par commande, utilisée pour le fichier final.
- **Bibliothèque** : `scikit-learn`

### Résumé

| Partie | Méthode | Bibliothèque |
|---|---|---|
| Prévision du revenu et des commandes | Lissage exponentiel (ETS) + baseline seasonal naive, choix par MAPE | statsmodels |
| Risque de retard | Gradient boosting (HistGradientBoostingClassifier) | scikit-learn |
| Évaluation du risque de retard | Test sur les 20 % les plus récents, baseline par mode d'expédition, importance par permutation, GroupKFold | scikit-learn |

> **Note** : la prévision n'utilise pas scikit-learn. Le lissage exponentiel est une méthode statistique de séries temporelles, et non du machine learning au sens strict. Seul le modèle de risque de retard relève du machine learning.

## Résultats

### Contrôle de cohérence avec Power BI

| Indicateur | Notebook | Power BI |
|---|---|---|
| Total des commandes | 65 752 | 65 752 |
| Revenu | 36 784 735 $ | 36 784 735 $ |
| Livraisons en retard | 57,3 % | 57,3 % |

Période : janvier 2015 à janvier 2018 (37 mois, 180 519 lignes de commande).

### Prévision du revenu et des commandes

Erreur (MAPE, %) sur les 6 derniers mois :

| Modèle | Commandes | Revenu |
|---|---|---|
| ETS tendance amortie | 11,8 | 67,7 |
| ETS tendance amortie + saisonnalité | 11,0 | 68,0 |
| ETS saisonnalité seule | **10,8** | 68,0 |
| Seasonal naive | 11,2 | **67,3** |

Modèles retenus : ETS saisonnalité seule pour les commandes, seasonal naive pour le revenu.

Prévision de février à juillet 2018 : environ 1 960 à 2 110 commandes par mois et 1,0 à 1,1 M$ de revenu mensuel.

> Voir la section *Limites* : le MAPE du revenu est élevé, la prévision de revenu est indicative.

### Risque de retard de livraison

Test sur les 20 % de commandes les plus récentes (34 554 lignes) :

| Modèle | Accuracy | ROC AUC | Precision | Recall | F1 |
|---|---|---|---|---|---|
| Toujours « en retard » | 0,573 | 0,500 | 0,573 | 1,000 | 0,728 |
| Règle : mode d'expédition seul | 0,696 | 0,744 | 0,887 | 0,538 | 0,670 |
| **Gradient boosting** | **0,723** | **0,778** | 0,889 | 0,591 | 0,710 |

- AUC hors échantillon par validation croisée (5 plis, GroupKFold) : **0,771**.
- Le mode d'expédition est de loin la variable la plus influente, suivi de l'heure de commande.
- Les commandes classées « risque élevé » sont en retard dans **89,6 %** des cas, contre 38,1 % pour le risque faible.

![Importance des variables](images/feature_importance.png)

## Limites et pistes d'amélioration

### Limites

**Prévision du revenu**
- L'erreur (MAPE) du revenu reste très élevée (environ 68 % pour tous les modèles testés) : aucune méthode n'a donc réellement battu les autres, et la prévision de revenu est à considérer comme indicative.
- Le revenu mensuel chute fortement de novembre 2017 à janvier 2018 (de 1,07 M$ en octobre à 0,33 M$ en janvier), alors que le nombre de commandes reste stable (environ 2 100 par mois). Les modèles n'anticipent pas cette rupture.
- Le modèle retenu pour le revenu (*seasonal naive*) reprend simplement les valeurs du même mois de 2017. Les prévisions de février à juillet 2018 (environ 1 M$ par mois) ignorent donc la baisse récente et sont probablement trop optimistes.
- La série ne compte que 37 mois (3 cycles saisonniers environ) et le choix du modèle repose sur un test de 6 mois seulement, ce qui rend la sélection fragile.

**Risque de retard**
- Le mode d'expédition explique l'essentiel de la prédiction (baisse d'AUC de 0,216 quand il est mélangé). Les taux de retard par mode sont très tranchés (First Class : 100 %, Second Class : 80 %, Standard Class : 40 %), ce qui ressemble davantage à une règle propre au jeu de données qu'à un comportement logistique réel.
- Le gain du gradient boosting sur la règle simple reste modeste (AUC 0,778 contre 0,744). Le modèle capte surtout l'effet du mode d'expédition et, dans une moindre mesure, de l'heure de commande.
- Le rappel est de 0,59 : environ 4 commandes en retard sur 10 ne sont pas détectées.
- Les bandes de risque « Faible » (38,1 % de retards) et « Moyen » (39,6 %) se distinguent à peine ; seule la bande « Élevé » (89,6 %) est réellement discriminante.
- Le jeu de données DataCo est public et ne reflète pas nécessairement une chaîne logistique réelle : les conclusions ne sont pas à généraliser telles quelles.

### Pistes d'amélioration

- Investiguer la chute du revenu fin 2017 – début 2018 (changement de mix produit, remises, anomalie de données) avant de produire une prévision de revenu.
- Prévoir le revenu à partir des commandes et du panier moyen plutôt que directement, puis comparer.
- Tester d'autres modèles de séries temporelles (SARIMA, Prophet) et une validation en fenêtre glissante plutôt qu'un seul test sur 6 mois.
- Ajouter des variables logistiques (entrepôt, transporteur, distance, ville de destination) pour dépasser l'effet du mode d'expédition.
- Ajuster le seuil de décision selon le coût d'un retard non détecté, au lieu du seuil par défaut, pour améliorer le rappel.
- Calibrer les probabilités prédites pour obtenir des bandes de risque plus fiables.

## Contenu du dépôt

- `DataCo_Predictive_Analysis.ipynb` : analyse, prévisions et modèle de risque de retard en Python
- `DataCo_Supply_Chain_Management.pbix` : rapport Power BI (si inclus)
- `data/` : jeux de données
- `images/` : captures d'écran et graphiques
- `requirements.txt` : bibliothèques Python nécessaires

## Installation

```bash
pip install -r requirements.txt
jupyter notebook
```

## Auteur

Mandjamana BALI – [LinkedIn](www.linkedin.com/in/mandjamana-bali-b0090242b)
