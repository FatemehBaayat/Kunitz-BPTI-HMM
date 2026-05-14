# Workflow

> **Structure-based pipeline for building and evaluating a Kunitz/BPTI profile HMM**

This document summarizes the full analysis pipeline used to construct a profile Hidden Markov Model (HMM) for the BPTI/Kunitz domain and evaluate its ability to detect Kunitz-containing proteins in Swiss-Prot.

---

## Quick Navigation

### 1. Structure-Based Dataset Preparation
- [PDB Structure Retrieval](#1-pdb-structure-retrieval)
- [Chain-Level Sequence Extraction](#2-chain-level-sequence-extraction)
- [Length-Based Filtering](#3-length-based-filtering)
- [Sequence Clustering](#4-sequence-clustering)
- [Representative Structure Selection](#5-representative-structure-selection)
- [PDB Chain Extraction](#6-pdb-chain-extraction)

### 2. Structural Alignment and HMM Construction
- [Multiple Structural Alignment](#7-multiple-structural-alignment)
- [Structural Quality Control](#8-structural-quality-control)
- [Outlier Removal and Refined Alignment](#9-outlier-removal-and-refined-alignment)
- [Profile HMM Construction](#10-profile-hmm-construction)

### 3. Swiss-Prot Benchmarking
- [Swiss-Prot Benchmark Dataset](#11-swiss-prot-benchmark-dataset)
- [Removal of Training-Like Positives](#12-removal-of-training-like-positives)
- [HMM Search](#13-hmm-search)
- [No-Hit Handling](#14-no-hit-handling)

### 4. Model Evaluation
- [Threshold Optimization](#15-threshold-optimization)
- [Performance Evaluation](#16-performance-evaluation)
- [Hard-Negative Benchmark](#17-hard-negative-benchmark)
- [Cross-Validation](#18-cross-validation)
- [False Positive and False Negative Analysis](#19-false-positive-and-false-negative-analysis)

### 5. Biological Interpretation
- [Final Interpretation](#20-final-interpretation)

---
# Workflow

> **Project goal:** Build and evaluate a structure-guided profile Hidden Markov Model (HMM) for detecting BPTI/Kunitz-type protease inhibitor domains.

This project follows a structure-based bioinformatics workflow to build and evaluate a profile Hidden Markov Model (HMM) for detecting BPTI/Kunitz-type protease inhibitor domains.

The workflow was designed to address the main project aim: building a profile HMM for the Kunitz-type protease inhibitor domain starting from available structural information and using the model to annotate Kunitz domains in Swiss-Prot proteins.


---

## Stage 1 — Structure-Based Dataset Preparation

## 1. PDB Structure Retrieval

The workflow starts from experimentally resolved protein structures. Kunitz/BPTI-related structures were retrieved from the Protein Data Bank (PDB).

The goal of this step was to collect structural examples of the Kunitz domain that could be used to build a structure-guided seed alignment.


---

## 2. Chain-Level Sequence Extraction

PDB entries can contain multiple chains, and not all chains necessarily correspond to the Kunitz/BPTI domain.

For this reason, sequences were extracted at the chain level instead of only at the PDB-entry level. This avoids mixing the Kunitz inhibitor chain with unrelated chains from protein complexes.


---

## 3. Length-Based Filtering

Kunitz/BPTI domains are short domains, typically around 50-60 amino acids.

A length-based filter was applied to remove chains that were too short or too long to represent a typical Kunitz domain.


---

## 4. Sequence Clustering

Redundant sequences were clustered using MMseqs2.

The goal of clustering was to reduce redundancy and avoid over-representation of nearly identical structures in the seed alignment.


---

## 5. Representative Structure Selection

One representative chain was selected from each cluster.

The representative chains were used as the structural seed set for the multiple structural alignment.


---

## 6. PDB Chain Extraction

The selected representative chains were extracted from their corresponding PDB files.

The extracted chain-specific structure files were used as input for structural alignment.


---

## Stage 2 — Structural Alignment and HMM Construction

## 7. Multiple Structural Alignment

The representative structures were aligned using a structural alignment workflow.

The structural alignment was then used to generate a structure-informed sequence alignment for the Kunitz/BPTI domain.


---

## 8. Structural Quality Control

Pairwise RMSD and TM-score values were calculated to evaluate the structural consistency of the selected chains.

Heatmaps were generated to inspect the structural relationships among the selected representatives and identify potential outliers.


---

## 9. Outlier Removal and Refined Alignment

Structural outliers were removed when necessary.

After outlier removal, a refined alignment was generated and used as the final input for profile HMM construction.


---

## 10. Profile HMM Construction

A profile HMM was built from the refined alignment using HMMER `hmmbuild`.

The final model was saved as:

```text
RESULTS/hmm/kunitz.hmm
```


---

## Stage 3 — Swiss-Prot Benchmarking

## 11. Swiss-Prot Benchmark Dataset

Swiss-Prot proteins were used to evaluate the predictive performance of the trained HMM.

Two main benchmark datasets were prepared:

- **Positive set:** Swiss-Prot proteins annotated with the BPTI/Kunitz domain.
- **Negative set:** Swiss-Prot proteins without the BPTI/Kunitz domain annotation.

The positive set was used to evaluate whether the model can correctly detect known Kunitz-containing proteins, while the negative set was used to estimate the false positive rate.


---

## 12. Removal of Training-Like Positives

To avoid evaluating the model on proteins already represented in the training structures, training-like Swiss-Prot positive sequences were identified and removed.

This step makes the benchmark more independent from the structural seed set used to train the HMM.

The remaining sequences were used as the independent positive benchmark.

Generated files:

```text
DATA/processed/positive_kunitz_independent.fasta
DATA/processed/positive_training_like_removed.fasta
```


---

## 13. HMM Search

The trained HMM was searched against the positive and negative Swiss-Prot benchmark datasets using HMMER `hmmsearch`.

The search was performed using fixed database-size correction and disabled heuristic filters:

```bash
hmmsearch -Z 1000 --max --tblout output.tbl kunitz.hmm input.fasta
```

The `--tblout` output was used for downstream parsing and performance evaluation.


---

## 14. No-Hit Handling

Some benchmark proteins did not return any HMMER hit.

To ensure that all proteins were included in the final evaluation, sequences with no HMMER hit were reintroduced into the prediction table and assigned a high default E-value.

This prevented no-hit proteins from being accidentally excluded from the confusion matrix and performance calculations.


---

## Stage 4 — Model Evaluation

## 15. Threshold Optimization

Different E-value thresholds were tested to convert HMMER scores into binary predictions.

The Matthews Correlation Coefficient (MCC) was used to identify thresholds that best balanced:

- True positives
- True negatives
- False positives
- False negatives

MCC was used because it is informative for binary classification, especially when positive and negative classes are unbalanced.


---

## 16. Performance Evaluation

The model was evaluated using a confusion matrix and standard classification metrics.

The following metrics were calculated:

- Accuracy
- Sensitivity / Recall
- Specificity
- Precision
- F1-score
- Matthews Correlation Coefficient (MCC)

These metrics were used to evaluate both the standard Swiss-Prot benchmark and the hard-negative benchmark.


---

## 17. Hard-Negative Benchmark

In addition to the standard Swiss-Prot benchmark, a hard-negative benchmark was created.

Hard negatives were selected among non-Kunitz proteins with Kunitz-like properties, such as:

- Cysteine-rich composition
- Domain-like sequence length
- Absence of BPTI/Kunitz annotation

This benchmark was used as an additional robustness analysis to test whether the model remains specific against proteins that are more similar to Kunitz domains than random Swiss-Prot negatives.


---

## 18. Cross-Validation

An iterative cross-validation strategy was used to evaluate the robustness of threshold selection and model performance.

The benchmark data were repeatedly split into training and test subsets. For each iteration, the optimal threshold was selected on one subset and evaluated on the other.

Performance metrics were then averaged across iterations.

Generated files:

```text
RESULTS/tables/iterative_cross_validation_results_hard.tsv
RESULTS/tables/iterative_cross_validation_summary_hard.tsv
```


---

## 19. False Positive and False Negative Analysis

False positive and false negative predictions were inspected to better understand the limitations of the model.

False negatives may correspond to:

- Divergent Kunitz domains
- Incomplete domain annotations
- Weak HMMER matches above the selected E-value threshold
- Proteins whose Kunitz-like regions differ from the structurally resolved domains used to train the model

This step helps interpret model errors rather than only reporting global performance metrics.


---

## Stage 5 — Biological Interpretation

## 20. Final Interpretation

The final model was interpreted in relation to known structural features of Kunitz/BPTI domains.

Special attention was given to conserved cysteine residues, because Kunitz/BPTI domains are stabilized by disulfide bonds formed between conserved cysteines.

The HMM/logo comparison was used to visually inspect whether the generated model captured conserved sequence features of the Kunitz domain.
