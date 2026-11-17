# Changelog

Toutes les modifications notables de ce projet sont documentées ici.

Le format suit [Keep a Changelog](https://keepachangelog.com/fr/1.0.0/),  
et ce projet adhère à la [gestion sémantique de version](https://semver.org/lang/fr/).

---

## [1.0.0] — 2026-11-17

### Ajouté
- Pipeline complet en 24 questions réparties en 4 phases
- Phase 1 : Diagnostic et préparation des données (Q01–Q08)
  - Import brut en texte intégral avec `dtype=str`
  - Détection de 5 formes de valeurs manquantes
  - Conversion numérique contrôlée (suffixes, virgules, types)
  - Vérification des plages techniques et de la contrainte métier (`interruption_h ≤ ouverture_h`)
  - 6 règles de nettoyage journalisées dans `journal_preparation.csv`
- Phase 2 : Expérimentation K-Means (Q09–Q16)
  - Standardisation Z-score (7 variables)
  - K-Means K=2→8, 5 graines aléatoires = 35 runs
  - Courbe du coude + score de silhouette
  - ACP 2D pour visualisation des clusters
  - Variables dérivées : `vol_par_passage`, `intensite_vol_h`, `intensite_pass_h`
  - 2 études d'ablation (`stockage_m3`, `pression_bar`)
  - Identification de 2 bornes prioritaires pour visite terrain
- Phase 3 : Évaluation & robustesse (Q17–Q20)
  - Matrice ARI inter-graines (stabilité)
  - Silhouette individuelle par observation
  - Comparaison médiane vs moyenne (impact du nettoyage)
  - Documentation des 5 cas difficiles
- Phase 4 : Interprétation & reproductibilité (Q21–Q24)
  - Synthèse en 3 profils nommés
  - Limites formalisées (données fictives, hypothèses, généralisation)
  - Informations de reproductibilité (versions, graines)
- Fichiers produits : `donnees_nettoyees.csv`, `journal_preparation.csv`, `tableau_experiences.csv`
- Documentation complète : `README.md`, `DATA_DICTIONARY.md`, `METHODOLOGY.md`, `CONTRIBUTING.md`

### Résultats
- **K retenu : 3** (silhouette = 0.561, ARI inter-graines = 1.000)
- 3 profils identifiés : volume modéré/forte attente, fort volume/faible attente, usage intense/haute pression

---

## [À venir]

### En cours de réflexion
- Algorithmes alternatifs : DBSCAN, Clustering Agglomératif Hiérarchique (CAH), Gaussian Mixture Models
- Métriques complémentaires : Davies-Bouldin, Calinski-Harabasz
- Interface de configuration du chemin des données (`.env` ou `config.yaml`)
- Tests unitaires pour le pipeline de nettoyage
- Support des données réelles (anonymisées)
