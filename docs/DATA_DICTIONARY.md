# 📖 Dictionnaire des données — Bornes Fontaines

Ce document décrit toutes les variables présentes dans les fichiers de données du projet.

---

## Fichier source : `mesures_brutes.csv`

Chaque ligne représente **une borne fontaine observée pendant une semaine** (unité d'analyse : *borne-semaine*).

### Variables clés (identifiants & contexte)

> Ces variables **ne sont pas utilisées** pour calculer les distances en clustering. Elles servent à identifier et contextualiser les observations.

| Colonne | Type | Description | Exemple |
|---|---|---|---|
| `id_observation` | `str` | Identifiant unique de la borne-semaine | `P01_00396` |
| `commune` | `str` | Zone géographique d'appartenance | `Zone_A`, `Zone_B`, `Zone_C`, `Zone_D` |
| `date_releve` | `str` | Date de début de la semaine d'observation | `2025-05-16` ou `16/05/2025` |

### Variables numériques candidates au clustering

| Colonne | Unité | Plage technique | Description |
|---|---|---|---|
| `volume_m3` | m³ | [0 ; 5 000] | Volume total d'eau distribuée sur la semaine |
| `passages` | — | [0 ; 30 000] | Nombre total de passages d'usagers |
| `attente_min` | minutes | [0 ; 240] | Temps d'attente moyen à la borne |
| `ouverture_h` | heures | [0 ; 168] | Durée totale d'ouverture effective (max = 168 h = 1 semaine) |
| `pression_bar` | bar | [0 ; 15] | Pression de service mesurée |
| `interruption_h` | heures | [0 ; 168] | Durée totale d'interruption de service |
| `stockage_m3` | m³ | [0 ; 1 000] | Volume disponible en réservoir de stockage |

**Contrainte métier :** `interruption_h` ≤ `ouverture_h`  
(une borne ne peut pas être interrompue plus longtemps qu'elle n'est ouverte)

---

## Fichier produit : `donnees_nettoyees.csv`

Résultat du pipeline de nettoyage appliqué au fichier brut. Contient les mêmes colonnes originales, plus les colonnes dérivées suivantes.

### Variables dérivées

| Colonne | Unité | Formule | Description |
|---|---|---|---|
| `cluster` | entier (0, 1, 2) | K-Means K=3 | Groupe d'appartenance final |
| `vol_par_passage` | m³/passage | `volume_m3 / passages` | Volume moyen consommé par passage (NaN si passages=0) |
| `intensite_vol_h` | m³/h | `volume_m3 / ouverture_h` | Intensité volumique par heure d'ouverture |
| `intensite_pass_h` | passages/h | `passages / ouverture_h` | Intensité de fréquentation par heure d'ouverture |
| `stabilite` | [0 ; 1] | Score de Jaccard inter-graines | Stabilité de l'affectation de cluster (1 = très stable) |
| `silhouette_indiv` | [-1 ; 1] | `silhouette_samples` | Score de silhouette individuel de l'observation |

---

## Fichier produit : `journal_preparation.csv`

Traçabilité des règles de nettoyage appliquées au jeu de données.

| Colonne | Description |
|---|---|
| `regle` | Nom de la règle appliquée |
| `colonnes` | Colonne(s) concernée(s) |
| `effectif_touche` | Nombre d'observations modifiées |
| `justification` | Explication du choix |

---

## Fichier produit : `tableau_experiences.csv`

Résultats de tous les runs K-Means (K = 2 à 8, 5 graines aléatoires = 35 lignes).

| Colonne | Description |
|---|---|
| `K` | Nombre de clusters testés |
| `graine` | Valeur de `random_state` utilisée |
| `inertie` | Inertie intra-cluster (à minimiser) |
| `silhouette` | Score de silhouette global (à maximiser, ∈ [-1, 1]) |
| `tailles` | Liste des effectifs par cluster |

---

## Formes de valeurs manquantes détectées

Le fichier brut contient 5 encodages différents pour l'absence de donnée :

| Encodage | Description |
|---|---|
| `""` (chaîne vide) | Cellule vide |
| `NA` | Forme textuelle standard |
| `?` | Valeur inconnue |
| `non renseigne` | Libellé manuel |
| `NaN` | Forme Python/pandas exportée |

> Tous ces encodages sont normalisés en `NaN` lors de l'étape Q04 du pipeline.

---

## Formats de dates détectés

| Format | Exemple | Traitement |
|---|---|---|
| ISO 8601 | `2025-05-16` | Parsé avec `format='mixed'` |
| Format français | `16/05/2025` | Parsé avec `format='%d/%m/%Y'` |

> Les dates ne sont pas utilisées dans le clustering — elles servent uniquement à contextualiser les observations.
