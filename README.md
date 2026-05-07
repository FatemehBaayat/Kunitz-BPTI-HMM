# Kunitz-BPTI-HMM
Structure-based workflow for building and evaluating a Kunitz/BPTI profile HMM.
# Kunitz-BPTI-HMM

> Structure-based workflow for building and evaluating a profile HMM for Kunitz/BPTI domain detection.

## Overview

This project builds and evaluates a structure-based profile HMM for detecting **Kunitz/BPTI domains**.

The workflow starts from PDB-derived Kunitz/BPTI sequences, removes redundant entries, extracts representative PDB chains, performs structural alignment, and prepares the final alignment for HMM construction and evaluation.

## Repository Structure

| Folder / File | Description |
|---|---|
| `data/raw_data/` | Original input files, such as the RCSB CSV report |
| `data/raw_pdbs/` | Full PDB structures downloaded from RCSB |
| `data/pdbs/` | Cleaned single-chain PDB files |
| `data/processed_data/` | Intermediate files from filtering, clustering, and ID extraction |
| `data/datasets/` | Positive/negative datasets and evaluation data |
| `data/tmalign_results/` | TM-align or mTM-align output files |
| `data/visualization/` | Plots, matrices, and visual analysis results |
| `scripts/` | Pipeline scripts |
| `notebooks/` | Jupyter or Colab notebooks |
| `results/` | Final outputs such as alignment, HMM model, and evaluation results |
| `WORKFLOW.md` | Step-by-step explanation of the project workflow |

## Main Tools

- RCSB PDB
- MMseqs2
- TM-align / mTM-align
- HMMER
- Python

## Workflow

The complete step-by-step workflow is described in [`WORKFLOW.md`](WORKFLOW.md).
