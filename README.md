# Analyse Du Dataset DataCo

Projet d'analyse de données du dataset **DataCo Supply Chain**, couvrant les quatre niveaux de l'analytique : descriptive, diagnostique, prédictive et prescriptive. KPI, causes des retards de livraison, prévisions et recommandations (Power BI, DAX, Python).

## Démarche analytique

| Analyse | Question | Exemple dans le projet | Outils |
|---|---|---|---|
| **Descriptive** | Que s'est-il passé ? | KPI, revenu par région, tendance mensuelle, % de commandes livrées à temps | Mesures DAX, page 1 du rapport Power BI |
| **Diagnostique** | Pourquoi ? | Pourquoi tant de retards ? Analyse par mode d'expédition, région, catégorie, écart entre délai prévu et délai réel | Arborescence de décomposition et Influenceurs clés (Power BI) |
| **Prédictive** | Que va-t-il se passer ? | Prévision du revenu et des commandes ; probabilité qu'une commande soit livrée en retard | Prévision intégrée aux graphiques en courbe, Python |
| **Prescriptive** | Que faire ? | Quel mode d'expédition choisir par région ? Quel délai prévu annoncer ? | Recommandations chiffrées, paramètres What-if |

## Outils utilisés
Power BI, DAX, Python (Jupyter Notebook)

## Contenu du dépôt
- `BI file` : analyse et tableau de bord
- `notebook.ipynb` : analyse et modèle prédictif en Python
- `data/` : jeux de données
- `README.md` : présentation du projet

## Installation
pip install -r requirements.txt
jupyter notebook

## Auteur
Mandjamana BALI
