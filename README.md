# Steam : Qu'est-ce qui fait la popularité d'un jeu ? Etude réalisée pour Ubisoft

Certification CDSD RNCP35288 - Bloc 1 : analyse exploratoire, descriptive et inférentielle de données.


## Contexte
Analyse exploratoire de **55 690 jeux** disponibles sur Steam, réalisée avec **PySpark sur Databricks**, pour aider Ubisoft à préparer le lancement d'un nouveau jeu : comprendre l'écosystème Steam et les facteurs qui influencent la popularité d'un jeu.


## Données

- Source : `s3://full-stack-bigdata-datasets/Big_Data/Project_Steam/steam_game_output.json` (bucket Jedha).
- Format : JSON semi-structuré (une ligne par jeu, champs imbriqués dans `data`).
- Volume : 55 691 lignes lues, 55 690 jeux conservés (1 entrée de type `hardware` écartée).
- Période : sorties du 30/06/1997 au 11/11/2022.


## Notebook publié

Lien du notebook publié sur Databricks : ⚠️ La publication n'étant plus disponible sur Databricks, il est demandé de réaliser des captures d'écran des sorties de cellules de code ⚠️  
Celles-ci sont sauvergardées dans le notebook. 


## Structure du notebook

| Section | Contenu |
|---|---|
| 0. Configuration | Imports, constantes (seuil de 300 jeux par genre, genres exclus) |
| 1. ETL | Extraction (JSON), transformation (aplatissement, nettoyage, variables dérivées), chargement Delta, contrôle qualité |
| 2. Analyse macro | Éditeurs, jeux les mieux notés, sorties par année et Covid, prix et promotions, langues, âge requis |
| 3. Genres | Représentation, notes pondérées, genres favoris des éditeurs, revenu estimé |
| 4. Plateformes | Disponibilité Windows/Mac/Linux, genres × plateformes |
| 5. Facteurs de popularité | régression linéaire multiple |
| 6. Conclusions | Constats, recommandations pour Ubisoft, limites |


## Choix méthodologiques

- **Notes** : `rating_pct` vaut `null` quand un jeu n'a aucun avis (et non 0 %). Le classement des meilleurs jeux ne garde que les jeux d'au moins 500 avis. Les notes par genre sont pondérées : somme des avis positifs divisée par somme des avis.
- **Genres** : « Early Access » et « Free to Play » (statuts) et les genres logiciels (Utilities, Education...) sont écartés. Seuls les genres d'au moins 300 jeux sont classés.
- **Éditeurs** : les entités Ubisoft sont regroupées sous un seul nom. Un éditeur déclaré avec un co-éditeur (« A, B ») est compté comme une entité à part.
- **Revenu estimé** : `milieu de la tranche de propriétaires × prix de lancement`, résumé par la médiane par jeu. C'est un ordre de grandeur.
- **Popularité** : mesurée par le nombre de propriétaires estimé, en échelle logarithmique.


## Principaux résultats

**Marché**
- 71 % des jeux sont indépendants. Entre 6 000 et 8 800 jeux sortent chaque année depuis 2017.
- Pas de choc Covid sur les sorties : 6 949 en 2019 (-9 %), 8 287 en 2020 (+19 %), 8 805 en 2021 (+6 %). L'année 2022 est incomplète (données jusqu'au 11/11/2022).
- Ubisoft est le 7e éditeur avec 135 jeux (75 d'action, 49 d'aventure, 23 de stratégie).

**Offre**
- 14 % de jeux gratuits, prix médian d'un jeu payant à 5,99 $, 78 % des jeux à moins de 10 $ (gratuits inclus), 5,6 % au-dessus de 20 $. 4,5 % des jeux sont en promotion à la date de collecte.
- L'anglais couvre 99 % des jeux, puis l'allemand (25 %), le français (24 %) et le russe (23 %). Un jeu compte en moyenne 3,6 langues.
- 1,2 % des jeux sont déconseillés aux moins de 16 ans.
- Plateformes : 74 % des jeux sont disponibles sur Windows seul, 22,9 % sur Mac et 15,2 % sur Linux. Les genres Indie et Strategy sont les plus portés.

**Genres**
- Meilleurs ratios d'avis positifs pondérés : Indie (88,5 %), Casual (86,7 %), Simulation (86,6 %). Les MMO sont les moins bien notés (73,1 %).
- Revenu estimé médian par jeu : RPG en tête (99 900 $), puis Simulation et Strategy (89 900 $). Casual est le plus bas (49 900 $).

**Facteurs de popularité**
- Régression sur 55 468 jeux (R² = 0,26). Toutes choses égales par ailleurs, l'audience est multipliée par :
  - 2,64 pour un jeu gratuit
  - 1,32 pour un RPG, 1,15 pour la stratégie, 1,14 pour l'action
  - 1,12 par plateforme supplémentaire, 1,05 par langue supplémentaire
  - 0,86 pour un jeu Indie ou Casual

**Recommandations pour Ubisoft** : viser une composante RPG, stratégie ou action, localiser largement et viser plusieurs plateformes, arbitrer entre gratuité (audience) et prix premium (au-dessus de 20 $), sans compter sur une période de sortie particulière.


## Limites

- Les propriétaires sont connus par tranches larges ; le revenu estimé est un ordre de grandeur.
- Les prix et remises sont un instantané à la date de collecte.
- Les co-éditeurs ne sont pas séparés dans le classement des éditeurs.
- La régression mesure des associations, pas des relations causales. Un jeu récent a moins de propriétaires simplement parce qu'il a eu moins de temps pour en accumuler.


## Structure du repo

cdsd_bloc2_projet_steam/
├── notebooks/projet_steam.ipynb 
├── README.md  
└── requirements.txt


## Prérequis et exécution

- Environnement : Databricks, PySpark.
- Bibliothèques : `plotly` et `statsmodels` (statsmodels installée par la première cellule), `scipy`, `pandas`, `numpy`.
- Exécuter le notebook de haut en bas. Il écrit une table Delta `steam_games_clean` (cellule 1.3 facultative). Le reste du notebook utilise le DataFrame `df` pour éviter les éventuels problèmes de droits d'écriture manquants.
