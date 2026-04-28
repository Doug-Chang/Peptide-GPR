# Machine Learning-Driven Discovery of Highly Selective Antifungal α/β-Peptides via GPR

This repository is a fork of the supplemental code for:

Chang, D. H.**‡**, Richardson, J. D.**‡**, Lee, M.-R., Lynn, D. M., Palecek, S. P., & Van Lehn, R. C. Machine learning-driven discovery of highly selective antifungal peptides containing non-canonical β-amino acids. *Chemical Science* **16**, 5579–5594 (2025). https://doi.org/10.1039/D4SC06689H
**‡** These authors contributed equally.

**Changes from the original:** updated `requirements.txt` (added `openpyxl`, corrected `PIL` → `Pillow`); replaced hard-coded paths (all scripts now use `os.path.dirname(os.path.abspath(__file__))`); enabled pandas 3.0 compatibility (`DataFrame.append` → `pd.concat`, chained indexing); updated `RDKit.py` test descriptor output writing to `train_path` instead of `test_path`; aligned SMILES CSV column references (`Sequence`, `SMILES`) across `Make_Smiles.py`, `RDKit.py`, `Analysis.py`, `GPR.py`, and `Res_Dif.py`.

## Overview

- Active learning pipeline over 6 rounds using Gaussian Process Regression (GPR)
- Input: RDKit 2D molecular descriptors from peptide SMILES → Output: predicted HC10 and MIC across a 168,000-sequence test design space
- Descriptor selection via LASSO cross-validation each round; test space reduced to training descriptor range
- Reproduces figures and analysis from Chang, D. H.**‡**, Richardson, J. D.**‡**, et al. (2025)

## Quick start (Linux / WSL)

```bash
git clone https://github.com/Doug-Chang/Peptide-GPR.git
cd Peptide-GPR
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

**Git LFS** — all `.csv` files are LFS-tracked; pull the real data after cloning:

```bash
sudo apt install git-lfs && git lfs install && git lfs pull
```

**Arial font** — not installed by default on Linux; required to match paper figures:

```bash
sudo apt install ttf-mscorefonts-installer -y && fc-cache -fv
python -c "import matplotlib, shutil; shutil.rmtree(matplotlib.get_cachedir())"
```

Set `label`, `round_num`, and other parameters in the **USER INPUT** block of each script before running (see [Analysis pipeline](#analysis-pipeline)).

## Repository structure

```
Peptide-GPR/
├── Make_Smiles.py          # Generate SMILES strings for test design space
├── RDKit.py                # Compute 200 RDKit 2D descriptors (train or test)
├── GPR.py                  # LASSO descriptor selection + GPR training and prediction, round 6 data is already set
├── Analysis.py             # Combine HC10/MIC predictions; design space figures
├── Res_Dif.py              # Residue-difference histogram (test vs. training)
├── Res_NSD.py              # NSD per round for newly introduced residues
├── requirements.txt
├── Training/               # Experimental data, SMILES, descriptors, CV results, figures
└── Test Design Space/      # Test sequences, predictions, reduced test spaces, figures
```

## Analysis pipeline

Set parameters in each script's **USER INPUT** block. `GPR.py` must be run for each `label` (`HC10`, `MIC`) × `round_num` (`1st`–`6th`) combination (12 total) before downstream scripts.

| Step | Script | Output |
|------|--------|--------|
| 1 | `Make_Smiles.py` | `SMILES_aaB.csv` / `SMILES_aaBaaaB.csv` — skip if files exist |
| 2 | `RDKit.py` | `RDKit.csv` (train) and `RDKit_aaB/aaBaaaB.csv` (test) — skip if files exist |
| 3 | `GPR.py` | `HC10_Results_*_Round.csv` / `MIC_Results_*_Round.csv` per label × round |
| 4 | `Analysis.py` | Combined results CSV; log₂(HC10) vs. log₂(MIC) and StDev figures |
| 5 | `Res_Dif.py` | `Res Dif/*_Round.csv` — residues different from training per test sequence |
| 6 | `Res_NSD.py` | NSD box plots per residue across rounds 1–4 and hypothetical 4* |

## Key parameters (`GPR.py`)

| Parameter | Options | Notes |
|-----------|---------|-------|
| `label` | `'HC10'`, `'MIC'` | Target property |
| `round_num` | `'1st'`–`'6th'` | Prediction round |
| `update_desc` | `'yes'` / `'no'` | Re-run LASSO CV (`yes` for rounds 4–6, `no` for 1–3) |
| `desc_red` | `'yes'` / `'no'` | Restrict test space to training descriptor range (recommended: `yes`) |
| `select_desc` | `'auto'` / `'manual'` | GPR kernel/α gridsearch vs. manual hyperparameter entry |

## Requirements

```bash
pip install -r requirements.txt
```

`scikit-learn` · `pandas` · `numpy` · `matplotlib` · `Pillow` · `rdkit` · `openpyxl` · `ipykernel`

> **Note:** `GPR.py`, `Analysis.py`, `Res_Dif.py`, and `Res_NSD.py` call `plt.show()` and render figures interactively. 
