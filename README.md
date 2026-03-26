# EEG_ML_pipelines — Diagnostic d’Alzheimer à partir d’EEG (OpenNeuro ds004504)

Projet reproductible basé sur un notebook Jupyter pour extraire des caractéristiques EEG et entraîner des classifieurs afin de distinguer Healthy Control (HC), Mild Cognitive Impairment (MCI) et Alzheimer (AD).

Référence méthodologique (inspiration features): https://pmc.ncbi.nlm.nih.gov/articles/PMC11048688/

> Important
> - Ne modifiez pas le fichier du notebook: `notebooks\code.ipynb`.
> - L’installation se fait via `requirements.txt` et les données se téléchargent séparément.

## Table des matières
- [Installation](#installation)
- [Données (OpenNeuro ds004504)](#donnees-openneuro-ds004504)
- [Exécuter le notebook](#executer-le-notebook)
- [Extraction de caractéristiques](#extraction-de-caracteristiques-inspiree-de-pmc11048688)
- [Bonnes pratiques de reproductibilité](#bonnes-pratiques-de-reproductibilite)
- [Dépannage rapide](#depannage-rapide)
- [Licence et citation](#licence-et-citation)

## Installation

### Pré-requis
- Python 3.9 ou supérieur (3.11 recommandé)
- pip
- Optionnel: environnement virtuel

### Étapes (Windows, PowerShell)
1. Se placer à la racine du projet.
2. Créer et activer un environnement virtuel:
   ```powershell
   python -m venv venv
   .\venv\Scripts\Activate.ps1
   ```
3. Mettre à jour pip puis installer les dépendances:
   ```powershell
   pip install --upgrade pip
   pip install -r requirements.txt
   ```

Le fichier `requirements.txt` inclut notamment: `numpy`, `pandas`, `scipy`, `scikit-learn`, `xgboost`, `mne`, `mne-bids`, `antropy`, `pywavelets`, `matplotlib`, `seaborn`, `awscli`, `jupyter`, `ipykernel`.

## Données — OpenNeuro ds004504 (prétraité)

Le notebook détecte automatiquement la racine des données (data_root) parmi les chemins suivants, dans cet ordre:
- `..\data\ds004504`
- `./data/ds004504`

Il privilégie la lecture des fichiers EEGLAB prétraités (`*.set`) dans `derivatives/sub-XXX/eeg/`, avec repli automatique vers une lecture BIDS brute si disponible.

### Téléchargement via AWS S3 (sans signature)
```powershell
aws s3 sync --no-sign-request s3://openneuro.org/ds004504 data/ds004504/
```

### Organisation conseillée dans ce dépôt (données locales)
- Si vos données sont déjà installées en local sous `data\ds004504`, vérifiez que la hiérarchie ressemble à:
  ```text
  data\ds004504\derivatives\sub-001\eeg\sub-001_task-eyesclosed_eeg.set
  data\ds004504\derivatives\sub-002\eeg\sub-002_task-eyesclosed_eeg.set
  …
  ```


#### Remarques
- Le dataset ds004504 contient des enregistrements EEG (repos, yeux fermés) pour HC, MCI et AD. Voir la page OpenNeuro pour les détails.
- Les formats `.set`/`.fdt` (EEGLAB) et BIDS dérivés sont pris en charge par MNE/MNE-BIDS.

## Exécuter le notebook

1. Activer l’environnement virtuel si besoin:
   ```powershell
   .\venv\Scripts\Activate.ps1
   ```
2. Lancer Jupyter:
   ```powershell
   jupyter notebook
   ```
3. Ouvrir `notebooks\code.ipynb` et exécuter les cellules dans l’ordre.
4. Le notebook détecte automatiquement `data_root` (voir plus haut). Aucune jonction n’est nécessaire si vos données sont sous `data\ds004504`.
   - Au lancement, la première cellule affiche: «Racine des données détectée: …» et indique le type de lecture (EEGLAB prioritaire, BIDS en repli).
   - Les figures de la section 7 sont enregistrées automatiquement sous `results\figures` à la racine du projet (et non sous `notebooks\...`).

## Extraction de caractéristiques (inspirée de PMC11048688)

Le pipeline regroupe descripteurs temps, fréquence et temps-fréquence, via `mne`, `antropy`, `pywavelets` et `scipy`:
- Puissance par bande (PSD via Welch):
  - Delta (0.5–4 Hz), Theta (4–8), Alpha (8–13), Beta (13–30), Gamma basse (30–45).
  - Puissances absolues/relatives et ratios (p.ex. Theta/Alpha, (Theta+Delta)/(Alpha+Beta)).
- Entropies et complexité (`antropy`):
  - `spectral_entropy`, `permutation_entropy`, `sample_entropy`.
  - `petrosian_fd`, `higuchi_fd`; paramètres de Hjorth (activité, mobilité, complexité).
- Statistiques temporelles: moyenne, variance, skewness, kurtosis, RMS, ZCR.
- Temps-fréquence (`pywavelets`): énergie par sous-bandes d’ondelettes (p.ex. Daubechies).
- Agrégation par canaux/régions: moyennes par canal/région (frontal, temporal, pariétal, occipital); indices d’asymétrie gauche/droite si pertinent.
- Normalisation et sélection: standardisation (z-score); sélection optionnelle de features (importance XGBoost/RandomForest, ANOVA).
- Modélisation: classifieurs classiques (SVM, RandomForest, XGBoost) avec validation croisée stratifiée pour distinguer HC/MCI/AD.

Le papier PMC11048688 souligne l’intérêt des ratios de bandes et des entropies pour discriminer les classes. Le notebook suit ce principe avec les bibliothèques listées dans `requirements.txt`.

## Dépannage rapide

- Chemin de données: vérifiez le message imprimé «Racine des données détectée: …». Assurez-vous que:
  - `data\ds004504\derivatives\sub-XXX\eeg\*_eeg.set` existe, ou
  - la structure BIDS brute est présente si vous ne disposez pas des `.set`.
- ImportError (mne-bids, antropy, xgboost):
  ```powershell
  pip install -r requirements.txt
  ```
- AWS CLI manquant:
  ```powershell
  pip install awscli
  ```
  puis rouvrez/activez le terminal.



