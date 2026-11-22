# TreeVision — type de couverture forestière à partir de variables cartographiques

Projet d'école (2025). Classer chaque parcelle de 30 × 30 m de la forêt nationale de Roosevelt (Colorado) dans l'un des 7 types de couverture forestière, à partir de variables cartographiques seules (altitude, pente, distances, ombrage, type de sol). Quatre modèles comparés sur le même jeu de test, avec un fort déséquilibre entre classes.

## Résultats

Jeu de test : 116 203 parcelles (20 % du jeu, tirées au hasard, jamais vues pendant l'entraînement ni le réglage). Chiffres lus dans les sorties de [`model.ipynb`](model.ipynb).

| Modèle | Exactitude | F1 macro | Rappel macro |
|---|---|---|---|
| Régression logistique (SMOTE) | 0,59 | 0,51 | 0,71 |
| Random Forest (SMOTE) | 0,80 | 0,72 | 0,74 |
| **XGBoost (SMOTE)** | **0,82** | **0,78** | 0,83 |
| Réseau de neurones dense (poids de classes) | **0,82** | 0,77 | **0,90** |

Ce que ces chiffres montrent :

- **XGBoost et le réseau de neurones font jeu égal** en exactitude (0,818 contre 0,818). XGBoost a le meilleur F1 macro ; le réseau a le meilleur rappel macro, c'est-à-dire qu'il retrouve mieux les classes rares, au prix de plus de fausses alertes sur ces classes.
- Le déséquilibre est fort : la classe la plus fréquente compte environ 100 fois plus de parcelles que la plus rare. Le F1 macro, qui donne le même poids à chaque classe, est donc plus parlant que l'exactitude.
- La régression logistique reste loin derrière : les frontières entre types de forêt ne sont pas linéaires dans ces variables.

![Répartition des classes](https://github.com/user-attachments/assets/ce2b50c6-76f9-4663-bb21-57b0dec52446)

## Méthode

- **Préparation** ([`cleaning.ipynb`](cleaning.ipynb)) : découpage 80 / 20 avant toute transformation, puis transformation Yeo-Johnson et mise à l'échelle robuste ajustées sur l'entraînement seul et appliquées au test.
- **Modèles classiques** : SMOTE placé dans le pipeline, donc appliqué seulement aux plis d'entraînement de la validation croisée (5 plis stratifiés) ; hyperparamètres choisis par recherche aléatoire.
- **Réseau de neurones** : 3 couches denses (512, 256, 128) avec normalisation et dropout, poids de classes inversement proportionnels aux effectifs, arrêt anticipé sur un jeu de validation tiré de l'entraînement.
- **Suivi** : paramètres et métriques enregistrés dans MLflow.

## Limites

- Recherche d'hyperparamètres courte (3 tirages par modèle) : les écarts entre modèles peuvent en partie venir du réglage.
- Un seul découpage train / test, sans intervalle de confiance : l'égalité entre XGBoost et le réseau ne peut pas être départagée.
- Les 54 variables sont des variables cartographiques ; aucune image n'est utilisée malgré le nom du projet.
- La métrique « f1_score » affichée pendant l'entraînement du réseau est une version binaire inadaptée au cas à 7 classes ; les chiffres du tableau viennent du rapport de classification de scikit-learn sur le jeu de test.

## Données

- **Source** : jeu *Covertype* (Blackard et Dean), issu du Resource Information System de la région 2 du US Forest Service, disponible sur le [UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/31/covertype).
- **Fichier** : `cover_data.csv`, 581 012 parcelles, 54 variables explicatives et la classe.

| Variable | Unité | Description |
|---|---|---|
| Altitude | m | Altitude de la parcelle |
| Orientation | degrés (azimut) | Direction de la pente |
| Pente | degrés | Inclinaison |
| Distances horizontale et verticale à l'eau | m | Jusqu'au plan d'eau le plus proche |
| Distance aux routes | m | Jusqu'à la route la plus proche |
| Ombrage à 9 h, midi et 15 h | indice 0-255 | Au solstice d'été |
| Distance aux points de départ d'incendie | m | Jusqu'au plus proche |
| Zone sauvage | 4 colonnes binaires | Zone de la forêt |
| Type de sol | 40 colonnes binaires | Classification des sols |
| Classe (variable prévue) | 1 à 7 | Type de couverture forestière |

## Reproduire

```bash
git clone https://github.com/FlavioDurante/TreeVision.git
cd TreeVision
pip install -r requirements.txt
mlflow ui --port 5000          # dans un autre terminal
jupyter lab                    # cleaning.ipynb (crée train_data.csv et test_data.csv), puis model.ipynb
```

## Licence

Code sous licence MIT, voir [`LICENSE`](LICENSE). Données : UCI Machine Learning Repository, licence CC BY 4.0.
