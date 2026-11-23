# mlops-ml-project — baseline ML reproductible

Baseline de classification sur le dataset **Iris** (150 fleurs, 4 mesures, 3 espèces),
intégré à scikit-learn : aucune donnée externe n'est nécessaire.
Le projet lit sa configuration YAML, entraîne un pipeline
(prétraitement + régression logistique), produit des artefacts traçables
et documente sa procédure de reproduction.

## Structure

```
mlops-ml-project/
├── config/train.yaml        # toute la configuration (données, split, modèle, sorties)
├── src/                     # code réutilisable, importable et testable
│   ├── data.py              #   chargement des données (iris ou csv)
│   ├── features.py          #   prétraitement : imputation, standardisation, clipping
│   └── model.py             #   construction du modèle
├── scripts/                 # points d'entrée exécutables
│   ├── train.py             #   entraînement + artefacts
│   └── evaluate.py          #   rapport de classification
├── tests/                   # tests pytest
├── notebooks/               # exploration (non versionné tant qu'il est vide)
├── data/                    # données réelles (non versionnées)
└── artifacts/               # sorties générées (non versionnées)
```

## Installation

Python 3.10 ou plus récent.

```bash
python -m venv .venv
source .venv/bin/activate          # Windows : .venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

Avec [uv](https://docs.astral.sh/uv/) (plus rapide, réutilise le cache local) :

```bash
uv venv --python 3.11 .venv
source .venv/bin/activate
uv pip install -r requirements.txt
```

> Si le chemin du projet est très long (plus de 127 caractères jusqu'à `.venv/bin/python`)
> ou contient un `&`, le script `pip` du venv peut être inutilisable :
> utilisez alors `python -m pip install -r requirements.txt`.

## Entraînement

```bash
python scripts/train.py
```

Sortie attendue (configuration de la release v0.1.0) :

```
Train OK: {'accuracy': 1.0, 'f1_macro': 1.0}
```

## Évaluation

```bash
python scripts/evaluate.py
```

Produit `artifacts/report.json` (précision, rappel et F1 par classe).
⚠️ Ce script évalue actuellement sur les 150 fleurs, jeu d'entraînement compris :
son accuracy est optimiste. La métrique de référence est celle de `metrics.json`,
calculée sur le jeu de test.

## Tests

```bash
python -m pytest -q
```

Sortie attendue : `3 passed`.

## Configuration

Tout le comportement est piloté par `config/train.yaml` ; changer un hyperparamètre
ne demande aucune modification du code.

| Clé | Valeur | Rôle |
|---|---|---|
| `data.kind` | `"iris"` | source des données : `iris` (baseline) ou `csv` (dataset réel) |
| `data.path` | `"data/dataset.csv"` | chemin du CSV si `kind: "csv"` |
| `data.target` | `"target"` | colonne cible du CSV |
| `split.test_size` | `0.2` | part du jeu de test (30 fleurs) |
| `split.random_state` | `42` | graine du découpage (reproductibilité) |
| `model.name` | `"logistic_regression"` | seul modèle supporté par la baseline |
| `model.max_iter` | `3000` | itérations maximales du solveur |
| `model.C` | `10` | inverse de la régularisation (défaut : `1.0`) |
| `artifacts_dir` | `"artifacts"` | dossier des sorties |

## Artefacts (générés dans `artifacts/`, non versionnés)

- `model.joblib` : pipeline entraîné (prétraitement + modèle)
- `metrics.json` : accuracy et f1_macro sur le jeu de test
- `confusion_matrix.png` : matrice de confusion
- `report.json` : rapport de classification détaillé

Les artefacts de la version `v0.1.0` (`metrics.json`, `model.joblib`) sont attachés
à la release GitHub correspondante.

## Résultats de référence

Mesurés sur le jeu de test (30 fleurs, `random_state: 42`) :

| Version | Configuration | accuracy | f1_macro |
|---|---|---|---|
| baseline initiale | `C = 1.0` | 0.933 | 0.933 |
| + clipping | `C = 1.0` | 0.933 | 0.933 |
| **v0.1.0** | `C = 10`, `max_iter = 3000` | **1.0** | **1.0** |

Un score de 1.0 sur 30 fleurs reste fragile (2 fleurs d'écart avec la baseline) :
une validation croisée est recommandée avant de figer `C = 10`.

## Workflow Git

- `main` : versions stables, taggées (`v0.1.0`, …) — on n'y commite jamais directement
- `dev` : branche d'intégration
- `feature/...` : une branche par évolution, fusionnée dans `dev` via Pull Request
  (modèle `.github/pull_request_template.md` : What / Why / How to test / Proof)
- Messages de commit au format [Conventional Commits](https://www.conventionalcommits.org/) :
  `feat:`, `fix:`, `chore:`, `docs:`, `test:`, `refactor:`…

## Retrouver une version

```bash
git checkout v0.1.0          # code + configuration exacts de la release
python scripts/train.py      # régénère le modèle (ou télécharger model.joblib de la release)
```
