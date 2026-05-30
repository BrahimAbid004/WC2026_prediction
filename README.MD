# 🏆 Prédiction et Simulation - Coupe du Monde de la FIFA 2026

## 📝 Présentation du Projet
Ce projet de Data Science a pour objectif de simuler l'intégralité de la Coupe du Monde de la FIFA 2026, de la phase de groupes jusqu'à la finale. Il prend en compte le nouveau format de la compétition incluant **48 équipes**, répartissant les nations dans 12 groupes et intégrant la règle complexe de qualification des "8 meilleurs troisièmes".

Pour garantir un réalisme maximal, le moteur de simulation repose sur la **Loi de Poisson**, permettant de prédire des scores exacts pour chaque match plutôt que de simples probabilités de victoire.

## 📊 Données et Critères Utilisés
Le modèle d'évaluation de la force des équipes s'appuie sur plusieurs jeux de données (fichiers CSV) :
1. **Classement FIFA :** Hiérarchie officielle mondiale.
2. **Valeur Marchande :** Qualité intrinsèque et financière de l'effectif de chaque pays.
3. **Dynamique Récente :** Résultats des matchs officiels depuis 2021 (Victoires, Nuls, Défaites).
4. **Solidité Défensive :** Nombre de buts encaissés depuis 2021, agissant comme bris d'égalité et pondération de la force défensive.
5. **Historique des Têtes-à-Têtes (H2H) :** Historique des confrontations directes sur les 10 dernières années, utilisé spécifiquement lors des phases à élimination directe (matchs couperets et tirs au but).

## 🛠️ Sommaire et Démarche Technique

Le développement complet est réalisé sur un Notebook Jupyter (`.ipynb`) structuré en 8 grandes étapes :

### 1. Introduction et Configuration du Projet
- Définition des règles du tournoi 2026.
- Importation des bibliothèques nécessaires (Pandas, NumPy, SciPy, etc.).

### 2. Collecte et Chargement des Données
- Importation des jeux de données bruts.
- Exploration initiale et vérification de la cohérence (noms de pays, formats).

### 3. Nettoyage et Préparation des Données (Data Preprocessing)
- Fusion des jeux de données en une matrice unique de 48 équipes.
- Standardisation des identifiants et gestion des valeurs manquantes.

### 4. Ingénierie des Caractéristiques (Feature Engineering)
- Normalisation des données pour créer des "Scores de Puissance".
- Intégration de la valeur marchande comme modificateur de talent.

### 5. Modélisation des Capacités d'Attaque et de Défense
- Calcul des *Expected Goals (xG)* potentiels par équipe.
- Pondération des forces offensives et défensives en croisant les résultats récents et le classement FIFA.

### 6. Construction du Moteur de Match (Algorithme de Poisson)
- Création de la fonction de simulation croisant l'attaque de l'équipe A et la défense de l'équipe B (et inversement).
- Utilisation de la distribution de Poisson pour générer une matrice de probabilités de scores exacts.
- Tirage au sort du résultat final basé sur ces probabilités pour conserver la "magie de la coupe".

### 7. Simulation de la Phase de Groupes
- Génération du calendrier des 72 matchs de poules.
- Application du moteur de match et calcul strict du classement (Points, Différence de buts, Buts marqués).
- Extraction automatique des 2 premiers de chaque groupe et calcul des 8 meilleurs troisièmes.

### 8. Simulation de la Phase à Élimination Directe (Knockouts)
- Génération de l'arbre final officiel (Bracket) à partir des 16èmes de finale.
- Intégration d'un module de "Prolongation / Tirs au but" excluant les matchs nuls, influencé par l'historique H2H et l'expérience de l'équipe.
- Boucle d'itération jusqu'au sacre final.

### 9. Analyse des Résultats et Visualisation
- Représentation graphique du parcours du vainqueur.
- Analyse des performances du modèle.

---
*Projet développé en Python - Brahim Abid