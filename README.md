# CRISPR Screen Follow-Up: TXNDC15 in ER Protein Quality Control

A computational follow-up to a CRISPR screen that identified **TXNDC15** as a novel hit alongside known ERAD components **SYVN1** (HRD1) and **MARCHF6**. https://www.biorxiv.org/content/10.64898/2026.04.01.715963v1.full This notebook asks a retrospective question:

> Could AI protein models have predicted TXNDC15's functional relationship to MARCHF6 and SYVN1?

## What the notebook does

1. **ESM2 sequence embeddings** — Fetches sequences for TXNDC15, SYVN1, MARCHF6, and known ERAD context proteins (SEL1L, DERL1, UBE2J1, OS9) from UniProt, then computes cosine similarity between ESM2 embeddings to test whether the hit clusters with known ERAD machinery.
2. **ESMFold structure prediction** — Predicts the structure of TXNDC15's thioredoxin-fold domain (residues 200–300) via NVIDIA's BioNeMo API, with pLDDT-based confidence analysis.
3. **DiffDock virtual screen** — Docks a panel of mechanism-diverse compounds against the predicted TXNDC15 structure:
   - Proteasome inhibitors (Bortezomib, MG-132)
   - E3 ligase modulators (Lenalidomide, Thalidomide)
   - ERAD / p97 inhibitors (Eeyarestatin, NMS-873)
   - Thioredoxin system inhibitor (Disulfiram)
4. **Fragment refinement** — Compares docking against residues 150–300 vs. the tighter 200–300 core to localize the putative binding pocket.

## Key findings

- TXNDC15's 200–300 thioredoxin core gives **mean pLDDT 81.7** (vs. 69.1 for 150–300), confirming the core fold is well-predicted.
- Top docking hits against the core: **Lenalidomide** (−0.155), **Thalidomide** (−0.841), **Bortezomib** (−0.878).
- 4 of 7 compounds improved with the tighter 200–300 fragment, suggesting the binding pocket lives in the thioredoxin core specifically.

## Stack

`transformers` (ESM2) · `torch` · NVIDIA BioNeMo (ESMFold, DiffDock) · `RDKit` · `py3Dmol` · `scikit-learn` · `scipy`

Designed to run in Google Colab with Drive mounted at `/content/drive/MyDrive/bionemo_project/`.

## Setup

You'll need an NVIDIA BioNeMo API key from [build.nvidia.com](https://build.nvidia.com). Set it as an environment variable rather than hardcoding it in the notebook:

```python
import os
NVIDIA_API_KEY = os.environ["NVIDIA_API_KEY"]
```

Or use a prompt in Colab:

```python
from getpass import getpass
NVIDIA_API_KEY = getpass("NVIDIA API key: ")
```

## Outputs

- `txndc15_structure.pdb` — ESMFold structure (residues 150–300)
- `txndc15_core.pdb` — ESMFold structure (residues 200–300, higher pLDDT)
- `erad_esm2_clustering.png` — Similarity heatmap + dendrogram
- `txndc15_erad_screen.csv` — DiffDock scores, 150–300 fragment
- `txndc15_core_screen.csv` — DiffDock scores, 200–300 core, with deltas vs. the longer fragment

## Interpretation

ESM2 cosine similarity narrows candidate interactors from the full proteome to a ranked shortlist — but the TXNDC15 ↔ MARCHF6 interaction is biochemical, not sequence-driven, so Co-IP mass spec remains necessary for confirmation. The value of the AI stack is in **prioritizing hypotheses**, not replacing the experiment.
