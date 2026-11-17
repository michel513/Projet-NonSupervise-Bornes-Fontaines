# 🚰 Profils de Fonctionnement des Bornes Fontaines

> **Apprentissage non supervisé · K-Means · ACP**  
> Projet de Data Science — Clustering descriptif de bornes d'eau publiques

[![Python](https://img.shields.io/badge/Python-3.11+-blue.svg)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange.svg)](https://jupyter.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![scikit-learn](https://img.shields.io/badge/scikit--learn-1.x-f7931e.svg)](https://scikit-learn.org/)

---

## 📋 Présentation

Ce projet construit une **typologie descriptive des bornes fontaines** à partir de mesures hebdomadaires collectées dans 4 zones géographiques (Zone_A, Zone_B, Zone_C, Zone_D).

**Mission :** Identifier des profils de fonctionnement distincts en combinant :
- 🔵 **Intensité de service** — volume distribué, passages
- 🟢 **Disponibilité** — heures d'ouverture, interruptions
- 🟡 **Confort d'accès** — temps d'attente, pression, stockage

**Méthode :** Pipeline complet de Data Science non supervisé — nettoyage rigoureux → standardisation → K-Means → validation par silhouette → interprétation métier.

---

## 🗂️ Structure du projet

```
Projet_NonSupervise_Bornes_Fontaines/
│
├── 📓 Projet_Bornes_Fontaines.ipynb        # Notebook principal (source)
├── 📓 Projet_Bornes_Fontaines_executed.ipynb  # Notebook avec sorties
│
├── 📊 donnees_nettoyees.csv                # Données après pipeline de nettoyage
├── 📋 journal_preparation.csv              # Traçabilité des règles de nettoyage
├── 📈 tableau_experiences.csv              # Résultats K-Means K=2→8, 5 graines
│
├── 📄 requirements.txt                     # Dépendances Python
├── 📖 README.md                            # Ce fichier
├── 📖 METHODOLOGY.md                       # Détail des choix méthodologiques
├── 📖 DATA_DICTIONARY.md                   # Dictionnaire des variables
└── 📄 CHANGELOG.md                         # Historique des versions
```

---

## 🚀 Démarrage rapide

### Prérequis

- Python ≥ 3.11
- pip ou conda

### Installation

```bash
# Cloner le dépôt
git clone https://github.com/<votre-username>/bornes-fontaines-clustering.git
cd bornes-fontaines-clustering

# Créer un environnement virtuel (recommandé)
python -m venv .venv
source .venv/bin/activate        # Linux/macOS
.venv\Scripts\activate           # Windows

# Installer les dépendances
pip install -r requirements.txt
```

### Lancer le notebook

```bash
jupyter lab Projet_Bornes_Fontaines.ipynb
```

> **Note :** Le notebook utilise `glob` pour localiser le fichier de données brutes `mesures_brutes.csv`. Placez ce fichier dans un sous-dossier `donnees/` ou adaptez le chemin dans la cellule Q01.

---

## 📊 Résultats clés

| Métrique | Valeur |
|---|---|
| Observations analysées | ~965 borne-semaines |
| Nombre de clusters retenu | **K = 3** |
| Score de silhouette | **0.561** |
| Stabilité inter-graines (ARI moyen) | **1.000** (très stable) |
| Variables de clustering | 7 (standardisées) |

### Les 3 profils identifiés

| Groupe | Nom descriptif | Caractéristiques principales |
|---|---|---|
| **Cluster 0** | Volume modéré / forte attente | Attente ↑, interruptions ↑, pression ↓ |
| **Cluster 1** | Fort volume / faible attente | Volume ↑, passages ↑, disponibilité ↑ |
| **Cluster 2** | Usage intense / haute pression | Volume ↑↑, passages ↑↑, stockage ↑ |

---

## 🔬 Pipeline d'analyse

```
Phase 1 — Préparation des données (Q01–Q08)
  ├─ Q01 Définir l'unité d'analyse (borne-semaine)
  ├─ Q02 Import brut en texte intégral
  ├─ Q03 Audit des doublons et clés
  ├─ Q04 Détection des 5 formes de valeurs manquantes
  ├─ Q05 Conversion contrôlée (suffixes, virgules, types)
  ├─ Q06 Vérification des plages techniques & contraintes métier
  ├─ Q07 Application des 6 règles de nettoyage (journalisées)
  └─ Q08 Statistiques descriptives de référence

Phase 2 — Expérimentation (Q09–Q16)
  ├─ Q09 Standardisation & tableau d'expériences (K=2→8, 5 graines)
  ├─ Q10 K-Means + courbe du coude + ACP 2D
  ├─ Q11 Étude : volume par passage
  ├─ Q12 Étude : volumes totaux vs intensités normalisées (ARI)
  ├─ Q13 Noms descriptifs des groupes
  ├─ Q14 Rôle de stockage_m3 (ablation study)
  ├─ Q15 Retrait de pression_bar (ablation study)
  └─ Q16 Identification de 2 visites techniques prioritaires

Phase 3 — Évaluation & robustesse (Q17–Q20)
  ├─ Q17 Métriques d'évaluation (silhouette, inertie, tailles)
  ├─ Q18 Stabilité inter-graines (matrice ARI)
  ├─ Q19 Impact du choix d'imputation (médiane vs moyenne)
  └─ Q20 Documentation des 5 cas difficiles (silhouette individuelle)

Phase 4 — Interprétation & reproductibilité (Q21–Q24)
  ├─ Q21 Synthèse : lier résultats et décision opérationnelle
  ├─ Q22 Limites (données fictives, hypothèses K-Means)
  ├─ Q23 Reproductibilité (versions, graines, chemins)
  └─ Q24 Défense de la conclusion (restitution 10 min)
```

---

## 🧹 Règles de nettoyage appliquées

| # | Règle | Observations touchées | Justification |
|---|---|---|---|
| 1 | Dé-doublonnage strict | 19 | Lignes strictement identiques |
| 2 | Harmonisation `commune` | 76 | Espaces/casse hétérogènes |
| 3 | Hors plage technique → NaN | 21 | Valeurs dépassant le dictionnaire |
| 4 | Contrainte `interruption > ouverture` → NaN | 0 | Violation de la contrainte métier |
| 5 | Imputation médiane | 208 | Robustesse aux valeurs extrêmes |
| 6 | Suppression lignes > 50% NaN | 0 | Aucune ligne n'était éligible |

---

## ⚙️ Variables du clustering

| Variable | Unité | Description |
|---|---|---|
| `volume_m3` | m³ | Volume total distribué sur la semaine |
| `passages` | — | Nombre de passages d'usagers |
| `attente_min` | minutes | Temps d'attente moyen |
| `ouverture_h` | heures | Durée d'ouverture effective |
| `pression_bar` | bar | Pression de service |
| `interruption_h` | heures | Durée totale d'interruption |
| `stockage_m3` | m³ | Volume en stock disponible |

> Toutes les variables sont **standardisées** (µ=0, σ=1) avant le clustering.

---

## ⚠️ Limites

1. **Données fictives** — Les données sont simulées. La typologie ne peut pas être utilisée pour des décisions opérationnelles sans validation sur données réelles.
2. **Hypothèses K-Means** — Suppose des groupes convexes et de variance homogène. L'imputation par médiane biaise les distributions vers le centre.
3. **Généralisation limitée** — La typologie est spécifique aux 4 zones et à la période observée (2025–2026).

---

## 📦 Dépendances

```
numpy
pandas
matplotlib
seaborn
scikit-learn
scipy
jupyterlab
```

Voir [`requirements.txt`](requirements.txt) pour l'installation.

---

## 📄 Licence

Ce projet est distribué sous licence [MIT](LICENSE). Vous êtes libre de l'utiliser, le modifier et le redistribuer avec attribution.

---

## 🤝 Contribution

Les contributions sont les bienvenues ! Consultez [`CONTRIBUTING.md`](CONTRIBUTING.md) pour les guidelines.

1. Forkez le dépôt
2. Créez une branche (`git checkout -b feature/ma-fonctionnalite`)
3. Committez vos changements (`git commit -m 'feat: ajout de ...'`)
4. Poussez la branche (`git push origin feature/ma-fonctionnalite`)
5. Ouvrez une Pull Request

---

*Projet réalisé dans le cadre d'un cours d'apprentissage non supervisé.*
