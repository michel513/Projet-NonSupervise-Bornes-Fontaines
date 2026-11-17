# 🔬 Méthodologie — Profils de Fonctionnement des Bornes Fontaines

Ce document détaille les choix méthodologiques du projet, les alternatives envisagées et leur justification.

---

## 1. Définition de l'unité d'analyse

**Choix :** La borne-semaine (une borne observée pendant 7 jours).

**Justification :** Cette granularité capture les variations de comportement d'une même borne dans le temps et permet d'inclure des bornes dans plusieurs semaines d'observation. Elle est cohérente avec le dictionnaire des données fourni.

**Variables exclues du clustering :** `id_observation`, `commune`, `date_releve` — ces variables identifient et contextualisent, elles ne constituent pas des attributs de fonctionnement.

---

## 2. Pipeline de nettoyage (Phase 1)

### 2.1 Stratégie d'import

Le fichier brut est chargé **intégralement en texte** (`dtype=str`, `keep_default_na=False`) afin de :
- Préserver toutes les formes d'absence sans transformation automatique
- Détecter les 5 encodages de manquants avant conversion
- Identifier les suffixes d'unité éventuels (m3, h, min, bar…)

### 2.2 Détection des manquants

5 formes d'absence identifiées : `""`, `NA`, `?`, `non renseigne`, `NaN`.  
Un zéro (`0`) est une **mesure valide** (borne fermée, pas d'attente…) et n'est jamais traité comme manquant.

### 2.3 Conversion numérique (4 étapes)

1. Remplacement des formes d'absence par `NaN`
2. Retrait des suffixes d'unité via regex (`m3`, `h`, `min`, `bar`, `passages`…)
3. Remplacement de la virgule décimale par un point
4. Conversion en `float` avec `pd.to_numeric(errors='coerce')`

Un journal de conversion documente les comptages à chaque étape.

### 2.4 Vérification des plages techniques

Chaque variable numérique est confrontée aux plages définies dans le dictionnaire :

| Variable | Min | Max |
|---|---|---|
| `volume_m3` | 0 | 5 000 |
| `passages` | 0 | 30 000 |
| `attente_min` | 0 | 240 |
| `ouverture_h` | 0 | 168 |
| `pression_bar` | 0 | 15 |
| `interruption_h` | 0 | 168 |
| `stockage_m3` | 0 | 1 000 |

Les valeurs hors plage sont remplacées par `NaN` (non supprimées).

**Contrainte métier** : `interruption_h ≤ ouverture_h` — toute violation remplace `interruption_h` par `NaN`.

### 2.5 Stratégie d'imputation

**Choix retenu : médiane**

| Critère | Médiane ✅ | Moyenne ❌ |
|---|---|---|
| Robustesse aux outliers | ✅ Oui | ❌ Sensible |
| Conservation de la distribution | Partielle | Plus biaisée |
| Interprétabilité | Simple | Simple |

L'imputation par la **moyenne** est testée en Q19 (impact du nettoyage) pour quantifier la sensibilité de la partition à ce choix.

### 2.6 Règles de nettoyage (journalisées)

| # | Règle | Observations touchées |
|---|---|---|
| 1 | Dé-doublonnage strict | 19 |
| 2 | Harmonisation `commune` | 76 |
| 3 | Hors plage technique → NaN | 21 |
| 4 | Contrainte métier → NaN | 0 |
| 5 | Imputation médiane | 208 |
| 6 | Suppression > 50% NaN | 0 |

---

## 3. Standardisation

**Méthode :** `StandardScaler` (z-score : µ=0, σ=1 par variable)

**Justification :** Les 7 variables ont des unités et des ordres de grandeur très différents (m³ vs heures vs passages). Sans standardisation, les variables à grande variance (ex: `passages`) domineraient la distance euclidienne utilisée par K-Means.

**Alternative envisagée :** MinMaxScaler (normalisation [0,1]) — non retenue car sensible aux valeurs extrêmes qui risquent de comprimer les observations normales.

---

## 4. Choix de l'algorithme de clustering

**Algorithme retenu : K-Means**

| Critère | K-Means | DBSCAN | CAH |
|---|---|---|---|
| Complexité | O(nkd) | O(n²) | O(n² log n) |
| Forme des groupes | Convexes | Quelconques | Quelconques |
| Paramètre à fixer | K | ε, MinPts | Seuil de coupure |
| Interprétabilité | ✅ Élevée | Moyenne | Élevée |
| Adapté à n ≈ 1000 | ✅ Oui | Possible | Possible |

K-Means est retenu pour sa **lisibilité des profils de groupe** (centres interprétables dans les unités originales) et sa **rapidité** permettant d'explorer K=2→8 × 5 graines = 35 runs.

**Paramètres :** `n_init=10` (10 initialisations aléatoires par run), `random_state` variable selon la graine.

---

## 5. Choix du nombre de clusters K

**Méthodes utilisées :**

1. **Courbe du coude** (inertie intra-cluster) — recherche du "coude" dans la diminution
2. **Score de silhouette** — maximisé à K=4 (0.611), bon à K=3 (0.561)
3. **Interprétabilité métier** — K=3 retenu pour sa lisibilité opérationnelle

**Résultats des 35 runs :**

| K | Silhouette moy. | Inertie moy. | Décision |
|---|---|---|---|
| 2 | 0.427 | 3 906 | Trop peu distinctif |
| **3** | **0.561** | **1 911** | **✅ Retenu** |
| 4 | 0.611 | 956 | Silhouette max mais K=3 plus lisible |
| 5 | 0.511 | 891 | Instable selon les graines |
| 6–8 | < 0.40 | — | Dégradation de la séparation |

**Justification du choix K=3 vs K=4 :** Malgré une silhouette légèrement inférieure à K=4, K=3 offre des profils plus interprétables et des tailles de groupes plus équilibrées, ce qui est préférable pour un usage opérationnel.

---

## 6. Validation & robustesse

### 6.1 Stabilité inter-graines

5 graines testées : `[42, 0, 1, 7, 123]`  
L'ARI (Adjusted Rand Index) entre toutes les paires de graines pour K=3 est calculé.  
Un ARI > 0.9 indique une partition très stable (robuste à l'initialisation).

### 6.2 Silhouette individuelle

La silhouette de chaque observation est calculée avec `silhouette_samples`. Les 5 observations les plus ambiguës (silhouette la plus basse) sont documentées avec deux interprétations possibles.

### 6.3 Études d'ablation (ablation studies)

| Test | Variable retirée | Métrique comparée |
|---|---|---|
| Q14 | `stockage_m3` | ARI avant/après |
| Q15 | `pression_bar` | ARI + observations qui changent |

### 6.4 Sensibilité au nettoyage

Q19 compare les partitions obtenues avec :
- Imputation par la **médiane** (choix retenu)
- Imputation par la **moyenne** (alternative)

L'ARI quantifie l'impact du choix d'imputation sur la partition finale.

---

## 7. Interprétation des groupes

Les centres de clusters sont **projetés dans les unités originales** (via inversion de la standardisation) pour l'interprétation métier.

**Variables dérivées calculées pour enrichir l'interprétation :**
- `vol_par_passage` = volume / passages
- `intensite_vol_h` = volume / heures d'ouverture
- `intensite_pass_h` = passages / heures d'ouverture

**Visualisation :** Coordonnées parallèles (normalisées min-max) pour comparer les 3 profils sur toutes les dimensions simultanément.

---

## 8. Reproductibilité

Toutes les sources d'aléatoire sont contrôlées :
- `RANDOM_STATE = 42` (graine principale)
- `SEEDS = [42, 0, 1, 7, 123]` (graines de robustesse)
- `n_init=10` pour K-Means (réduction de la sensibilité à l'initialisation)

Le notebook est **auto-suffisant** : toutes les transformations sont appliquées dans l'ordre, sans modification manuelle des CSV.

---

## 9. Limites reconnues

### 9.1 Données fictives
Les données sont entièrement simulées. Les profils identifiés ne correspondent à aucune réalité observée. Les plages techniques ne constituent pas des normes validées.

### 9.2 Hypothèses K-Means
- Suppose des groupes **convexes** (sphériques dans l'espace standardisé)
- Sensible aux **outliers** (bien que l'imputation médiane limite ce risque)
- Le poids **égal** de toutes les variables n'est pas nécessairement justifié du point de vue métier

### 9.3 Généralisation
La typologie est spécifique aux 4 zones et à la période 2025–2026. Elle ne peut pas être extrapolée sans validation.
