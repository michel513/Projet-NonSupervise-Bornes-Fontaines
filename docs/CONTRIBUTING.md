# 🤝 Guide de contribution

Merci de votre intérêt pour ce projet ! Voici comment contribuer efficacement.

---

## 📋 Avant de contribuer

1. **Lisez le README** pour comprendre le projet
2. **Consultez les issues** existantes pour éviter les doublons
3. **Ouvrez une issue** avant de commencer un travail important

---

## 🐛 Signaler un bug

1. Vérifiez que le bug n'est pas déjà signalé dans les [issues](../../issues)
2. Ouvrez une nouvelle issue avec le template **Bug Report**
3. Incluez :
   - Version de Python et des librairies (`pip freeze`)
   - Étapes pour reproduire le bug
   - Comportement attendu vs observé
   - Message d'erreur complet (stack trace)

---

## 💡 Proposer une amélioration

Idées bienvenues, par exemple :
- Algorithmes de clustering alternatifs (DBSCAN, CAH, GMM)
- Visualisations supplémentaires
- Métriques d'évaluation complémentaires (Davies-Bouldin, Calinski-Harabasz)
- Meilleure gestion du chemin des données

Ouvrez une issue avec le template **Feature Request** avant de coder.

---

## 🔧 Soumettre du code (Pull Request)

### Setup développement

```bash
# Fork + clone
git clone https://github.com/<votre-username>/bornes-fontaines-clustering.git
cd bornes-fontaines-clustering

# Créer un environnement virtuel
python -m venv .venv
source .venv/bin/activate

# Installer les dépendances
pip install -r requirements.txt
```

### Conventions de code

- **Python** : suivre [PEP 8](https://pep8.org/)
- **Nommage** : variables en `snake_case`, constantes en `UPPER_CASE`
- **Commentaires** : en français (langue du projet)
- **Cellules Jupyter** : chaque cellule doit avoir un commentaire de titre (`# Qxx — ...`)

### Conventions de commit

Suivre le format [Conventional Commits](https://www.conventionalcommits.org/) :

```
feat: ajout de DBSCAN comme alternative à K-Means
fix: correction de la détection des valeurs manquantes
docs: mise à jour du dictionnaire des données
refactor: extraction de la fonction clean_numeric
test: ajout de tests sur le pipeline de nettoyage
```

### Process de Pull Request

1. Créez une branche depuis `main` :
   ```bash
   git checkout -b feat/mon-amelioration
   ```
2. Faites vos modifications
3. Testez que le notebook s'exécute sans erreur (Kernel → Restart & Run All)
4. Committez avec un message clair
5. Poussez et ouvrez une Pull Request vers `main`
6. Décrivez les changements et référencez l'issue associée

---

## 📁 Structure à respecter

- Le notebook principal reste `Projet_Bornes_Fontaines.ipynb`
- Les données nettoyées sont générées automatiquement par le notebook
- Ne committez pas de données brutes sensibles
- Mettez à jour `CHANGELOG.md` si vous modifiez le comportement

---

## ❓ Questions

Ouvrez une [Discussion](../../discussions) pour toute question générale.
