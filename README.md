# 🏆 Simulation Coupe du Monde 2026
> Modèle de prédiction probabiliste basé sur des données réelles — **Monte Carlo · Poisson · 10 000 simulations**

---

## Aperçu

Ce projet simule l'intégralité de la Coupe du Monde 2026 (48 équipes, format FIFA officiel) à partir de données footballistiques réelles collectées depuis 2021. Il combine un **modèle de score composite multi-critères** et une **simulation Monte Carlo** pour estimer les probabilités de victoire de chaque sélection.

Le tournoi complet est simulé — phase de groupes, Round of 32, quarts de finale, demi-finales, **match pour la 3ᵉ place** et grande finale au MetLife Stadium (19 juillet 2026).

---

## Résultats principaux

Sur 10 000 simulations, les équipes les plus souvent couronnées :

| Rang | Équipe | % Champion | % Top 4 |
|------|--------|-----------|---------|
| 1 | 🇪🇸 Espagne | ~11.5% | ~26.9% |
| 2 | 🇦🇷 Argentine | ~11.4% | ~25.1% |
| 3 | 🇫🇷 France | ~9.6% | ~23.6% |
| 4 | 🏴󠁧󠁢󠁥󠁮󠁧󠁿 Angleterre | ~8.3% | ~22.8% |
| 5 | 🇵🇹 Portugal | ~6.6% | ~20.0% |
| 6 | 🇧🇷 Brésil | ~5.9% | ~19.3% |
| 7 | 🇲🇦 Maroc | ~5.5% | ~18.5% |
| 8 | 🇳🇱 Pays-Bas | ~5.2% | ~17.8% |

> Les résultats sont stables sur deux séries indépendantes de 10 000 simulations (variation < 0.5 point), ce qui confirme la convergence statistique du modèle.

---

## Architecture du projet

```
cdm2026/
├── cdm26_vfinale.ipynb          # Notebook principal
├── data/
│   ├── classement_fifa.csv      # Classement FIFA officiel
│   ├── valeurs_marchandes.csv   # Valeurs de marché (en €)
│   ├── resultats_depuis_2021.csv  # Historique V/N/D depuis 2021
│   ├── meilleure_defense.csv    # Buts encaissés depuis 2021
│   └── buts_marques_depuis_2021.csv  # Buts marqués depuis 2021
└── README.md
```

---

## Méthodologie

### 1. Sources de données

Cinq fichiers CSV alimentent le modèle, tous basés sur des données officielles post-2021 :

| Fichier | Description | Usage |
|---------|-------------|-------|
| `classement_fifa.csv` | Rang FIFA officiel | Signal de long terme |
| `valeurs_marchandes.csv` | Valeur de marché totale (€) | Qualité du effectif |
| `resultats_depuis_2021.csv` | Victoires / Nuls / Défaites | Performance récente |
| `meilleure_defense.csv` | Buts encaissés depuis 2021 | Solidité défensive |
| `buts_marques_depuis_2021.csv` | Buts marqués depuis 2021 | Puissance offensive |

> **Pourquoi 2021 ?** On exclut les données pré-pandémie (2020 très perturbé) tout en conservant un historique suffisant pour lisser les performances.

---

### 2. Construction des scores composites

Chaque indicateur est normalisé en `[0, 1]` via Min-Max (avec inversion pour les métriques « moins c'est mieux » comme le classement FIFA et les buts encaissés).

**Puissance d'attaque :**

$$\text{puissance\_attaque} = 0.4 \times \text{buts\_norme} + 0.3 \times \text{taux\_victoires} + 0.2 \times \text{valeur\_log\_norme} + 0.1 \times \text{score\_fifa\_norme}$$

**Solidité défensive :**

$$\text{puissance\_defense} = 0.5 \times \text{solidite\_defensive} + 0.3 \times \text{taux\_invincibilite} + 0.2 \times \text{score\_fifa\_norme}$$

> La valeur marchande est transformée en `log₁₀` avant normalisation pour compresser l'écart entre les grandes nations (France ~1.2 Md€) et les petites sélections (Haïti ~10 M€).

---

### 3. Modèle de simulation — Loi de Poisson

Le nombre de buts de chaque équipe est modélisé par une **loi de Poisson** :

$$P(X = k) = \frac{e^{-\lambda} \cdot \lambda^k}{k!}$$

Le paramètre λ (nombre moyen de buts attendus) est calculé ainsi :

$$\lambda_A = c \times \text{att}[A] \times (1 - \text{def}[B])$$

avec `c = 5.0` en phase de groupes et `c = 6.0` en phase éliminatoire (les équipes s'ouvrent davantage en K.O.).

Les scores des deux équipes sont tirés **indépendamment**. En cas d'égalité en K.O., le vainqueur est désigné par tirage aléatoire (modélisation des prolongations et tirs au but).

---

### 4. Format du tournoi

Le format officiel de la CdM 2026 est intégralement reproduit :

- **Phase de groupes :** 12 groupes de 4 équipes, tous contre tous (72 matchs). Classement : points → différence de buts → buts marqués.
- **Qualification :** Top 2 de chaque groupe (24 équipes) + 8 meilleurs 3ᵉ = **32 qualifiés**.
- **Phase éliminatoire :** Round of 32 → Quarts → Demi-finales → **Match pour la 3ᵉ place** → Finale.
- **Contrainte de tirage :** deux équipes du même groupe ne peuvent pas s'affronter au Round of 32.

---

### 5. Monte Carlo

La fonction `simuler_un_tournoi()` encapsule l'intégralité du tournoi et retourne `{champion, finaliste, troisieme, quatrieme}`. Elle est exécutée **10 000 fois** et les fréquences de chaque résultat convergent vers les probabilités estimées.

**Optimisation clé :** les lookups `df_master[df_master['team']==...]` (1.5 ms/appel) ont été remplacés par des dictionnaires Python pré-calculés (0.002 ms/appel), soit un gain de **760×** — ramenant les 10 000 simulations de ~25 minutes à ~2 minutes.

---

## Visualisations

Le notebook produit 6 graphiques complémentaires :

| # | Type | Description |
|---|------|-------------|
| 1 | Scatter plot | Carte des forces Attaque vs Défense (48 équipes) |
| 2 | Barres empilées | Top 20 équipes par score global |
| 3 | Barres horizontales | Probabilités de remporter le titre (top 15) |
| 4 | Barres groupées | Probabilités par position : Champion / Finaliste / 3ᵉ place |
| 5 | Heatmap | Vue matricielle Top 15 × 4 positions |
| 6 | Radar chart | Profils multidimensionnels des 6 grands favoris |

---

## Installation et utilisation

### Prérequis

```bash
pip install pandas numpy matplotlib seaborn
```

### Lancement

```bash
jupyter notebook cdm26_vfinale.ipynb
```

Exécuter les cellules dans l'ordre. Les fichiers CSV doivent être dans le même répertoire que le notebook.

### Sur Google Colab

```python
from google.colab import files
uploaded = files.upload()  # Sélectionner les 5 fichiers CSV
```

Le reste du notebook s'exécute sans modification.

---

## Limites et perspectives

**Limites actuelles :**
- Les pondérations des scores composites sont fixées manuellement, sans calibration sur résultats historiques
- L'égalité en K.O. est résolue par un tirage 50/50 (pas de modélisation fine des tirs au but)
- Le modèle ne tient pas compte de la forme récente (5 derniers matchs), des blessures ni du contexte géographique (avantage relatif USA/Canada/Mexique)

**Améliorations possibles :**
- Calibration des poids par régression logistique sur les CdM 2010–2022
- Pondération temporelle des résultats (matchs récents > matchs anciens)
- Modélisation des tirs au but (historique par équipe)
- Interface interactive (Streamlit ou Dash) pour explorer les résultats en temps réel

---

## Conclusion

Ce projet démontre qu'un modèle relativement simple — cinq sources de données, deux scores composites, une loi de Poisson — produit des prédictions cohérentes avec la hiérarchie footballistique mondiale et stables sur plusieurs séries de simulation. L'Espagne et l'Argentine ressortent systématiquement comme co-favorites (~11% chacune), le Maroc confirme sa montée en puissance comme meilleure équipe africaine hors élite historique, et la structure en trois vitesses du football mondial (duo de tête / challengers / reste du monde) est clairement capturée par le modèle.

---

## Auteur

Projet réalisé dans le cadre d'une démarche personnelle d'application de la data science au football.

*Données sources : FIFA, Transfermarkt, résultats officiels des sélections nationales (2021–2025)*
