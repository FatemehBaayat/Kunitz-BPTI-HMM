# Workflow

> **Project goal:** Build and evaluate a structure-guided profile Hidden Markov Model (HMM) for detecting BPTI/Kunitz-type protease inhibitor domains.

This project follows a structure-based bioinformatics workflow to build and evaluate a profile HMM for the Kunitz/BPTI domain. The model is trained from structurally resolved representative domains and evaluated on Swiss-Prot benchmark datasets.

The workflow addresses the main project aim: to build a Kunitz/BPTI profile HMM starting from structural information and then use it to annotate Kunitz domains in Swiss-Prot proteins.

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
- [HMM Profile Inspection](#11-hmm-profile-inspection)
- [HMM Logo Comparison](#12-hmm-logo-comparison)

### 3. Swiss-Prot Benchmarking
- [Swiss-Prot Benchmark Dataset](#13-swiss-prot-benchmark-dataset)
- [Removal of Training-Like Positives](#14-removal-of-training-like-positives)
- [HMM Search](#15-hmm-search)
- [No-Hit Handling](#16-no-hit-handling)

### 4. Model Evaluation
- [Threshold Optimization](#17-threshold-optimization)
- [Performance Evaluation](#18-performance-evaluation)
- [Benchmark-Specific Threshold Comparison](#19-benchmark-specific-threshold-comparison)
- [False Positive and False Negative Analysis](#20-false-positive-and-false-negative-analysis)

### 5. Biological Interpretation
- [Final Interpretation](#21-final-interpretation)

---

## Stage 1 — Structure-Based Dataset Preparation

## 1. PDB Structure Retrieval

The workflow starts from experimentally resolved protein structures. Kunitz/BPTI-related structures were retrieved from the Protein Data Bank (PDB).

The goal of this step was to collect structural examples of the Kunitz/BPTI domain that could be used to build a structure-guided seed alignment.

The PDB query was based on:

```text
Pfam ID: PF00014
Resolution: ≤ 3.5 Å
Polymer entity sequence length: 45–80 amino acids
```

The length range was selected because Kunitz/BPTI domains are short domains, usually around 50–60 residues.

Generated / input files:

```text
DATA/raw/pdb_ids.txt
DATA/raw/pdb_report.csv
```

---

## 2. Chain-Level Sequence Extraction

PDB entries can contain multiple chains, and not all chains necessarily correspond to the Kunitz/BPTI domain.

For this reason, sequences were extracted at the chain level instead of only at the PDB-entry level. This avoids mixing the Kunitz inhibitor chain with unrelated chains from protein complexes.

---

## 3. Length-Based Filtering

Kunitz/BPTI domains are short domains, typically around 50–60 amino acids.

A length-based filter was applied to remove chains that were too short or too long to represent a typical Kunitz/BPTI domain.

Generated file:

```text
DATA/processed/pdb_seqs.txt
```

---

## 4. Sequence Clustering

Redundant sequences were clustered using **MMseqs2**.

The final clustering parameters were:

```text
90% sequence identity
90% coverage
```

The goal of clustering was to reduce redundancy and avoid over-representation of nearly identical structures in the seed alignment, while still retaining enough representative structures for multiple structural alignment.

Generated file:

```text
DATA/processed/pdb_cluster_rep_seq.fasta
```

---

## 5. Representative Structure Selection

One representative chain was selected from each MMseqs2 cluster.

The representative chains were used as the structural seed set for the multiple structural alignment.

Generated file:

```text
DATA/processed/pdb_id.rep
```

---

## 6. PDB Chain Extraction

The selected representative chains were extracted from their corresponding PDB files.

The extracted chain-specific structure files were used as input for structural alignment.

Generated files:

```text
DATA/processed/chains_list.txt
RESULTS/structures/chains/
RESULTS/structures/pdbs/
```

---

## Stage 2 — Structural Alignment and HMM Construction

## 7. Multiple Structural Alignment

The representative structures were aligned using a multiple structural alignment workflow.

The structural alignment was then converted into a structure-informed sequence alignment for the Kunitz/BPTI domain.

Generated file:

```text
RESULTS/alignments/alignment.fasta
```

---

## 8. Structural Quality Control

Pairwise RMSD and TM-score values were calculated to evaluate the structural consistency of the selected representative chains.

Heatmaps and line plots were generated to inspect structural relationships among the selected representatives and to identify potential outliers.

Generated files:

```text
RESULTS/figures/pairwise_rmsd_heatmap.png
RESULTS/figures/pairwise_tm_score_heatmap.png
RESULTS/figures/mean_rmsd_lineplot.png
RESULTS/figures/mean_tm_score_lineplot.png
```

---

## 9. Outlier Removal and Refined Alignment

Structural outliers were evaluated using the pairwise RMSD and TM-score analyses.

The strongest structural outlier was:

```text
2KAI_A
```

This structure showed high mean RMSD and low mean TM-score compared with the remaining representative structures. Therefore, only `2KAI_A` was removed before rebuilding the final structural alignment.

Generated files:

```text
DATA/processed/chains_list_no_outliers.txt
RESULTS/alignments/alignment_no_outliers.fasta
RESULTS/alignments/alignment_clean_trimmed.fasta
```

---

## 10. Profile HMM Construction

A profile HMM was built from the final cleaned and trimmed structure-based alignment using HMMER `hmmbuild`.

Command:

```bash
hmmbuild --amino -n Kunitz_BPTI kunitz.hmm alignment_clean_trimmed.fasta
```

The final model was saved as:

```text
RESULTS/hmm/kunitz.hmm
```

---

## 11. HMM Profile Inspection

The final `kunitz.hmm` file was inspected at the text-file level to verify the internal HMMER profile structure.

The HMM file contains:

- model metadata such as `NAME`, `LENG`, `ALPH`, and `NSEQ`
- amino-acid emission scores
- transition scores between match, insert, and delete states

This confirms that the HMM is a position-specific probabilistic model rather than only a sequence alignment.

---

## 12. HMM Logo Comparison

A sequence logo was generated from the final profile HMM and compared with the curated Pfam PF00014 Kunitz/BPTI logo.

The final HMM logo preserves the main conserved cysteine pattern expected for the Kunitz/BPTI fold. Minor differences from the curated Pfam logo are expected because this custom HMM was trained on a smaller structure-based representative dataset, while Pfam is based on a larger curated family alignment.

Generated file:

```text
RESULTS/figures/kunitz_logo_comparison.png
```

---

## Stage 3 — Swiss-Prot Benchmarking

## 13. Swiss-Prot Benchmark Dataset

Swiss-Prot proteins were used to evaluate the predictive performance of the trained HMM.

Two main benchmark groups were prepared:

- **Positive set:** reviewed Swiss-Prot proteins annotated with Pfam `PF00014`
- **Negative set:** reviewed Swiss-Prot proteins without Pfam `PF00014`

The positive set was used to evaluate whether the model can correctly detect known Kunitz-containing proteins, while the negative set was used to estimate the false positive rate.

Input / generated files:

```text
DATA/raw/positive_kunitz.fasta
DATA/raw/negative_kunitz.fasta
```

---

## 14. Removal of Training-Like Positives

To avoid evaluating the model on proteins already represented in the training structures, training-like Swiss-Prot positive sequences were identified and removed.

This step makes the benchmark more independent from the structural seed set used to train the HMM.

Generated files:

```text
DATA/processed/training_representatives_ungapped.fasta
DATA/processed/positive_kunitz_independent.fasta
DATA/processed/training_like_positive.ids
```

---

## 15. HMM Search

The trained HMM was searched against the positive and negative Swiss-Prot benchmark datasets using HMMER `hmmsearch`.

The search was performed using fixed database-size correction and disabled heuristic filters:

```bash
hmmsearch -Z 1000 --max --tblout output.tbl kunitz.hmm input.fasta
```

The `--tblout` output was parsed for downstream prediction and performance evaluation. The best domain E-value was used as the classification score.

---

## 16. No-Hit Handling

Some benchmark proteins did not return any HMMER hit.

To ensure that all proteins were included in the final evaluation, sequences with no HMMER hit were reintroduced into the prediction table and assigned a high default E-value.

This prevented no-hit proteins from being accidentally excluded from the confusion matrix and performance calculations.

---

## Stage 4 — Model Evaluation

## 17. Threshold Optimization

Different E-value thresholds were tested to convert HMMER scores into binary Kunitz/non-Kunitz predictions.

The Matthews Correlation Coefficient (MCC) was used as the main threshold-selection metric because it is informative for binary classification, especially when positive and negative classes are unbalanced.

Generated files:

```text
RESULTS/tables/threshold_optimization_results_hard.tsv
RESULTS/tables/threshold_optimization_results_random_full.tsv
```

---

## 18. Performance Evaluation

The model was evaluated using confusion matrices and standard classification metrics.

The following metrics were calculated:

- Accuracy
- Sensitivity / Recall
- Specificity
- Precision
- F1-score
- Matthews Correlation Coefficient (MCC)

Two benchmark settings were evaluated:

1. **Hard balanced benchmark**  
   A balanced benchmark with the same number of positive and negative sequences.

2. **Random/full Swiss-Prot benchmark**  
   A highly imbalanced benchmark containing the full or large Swiss-Prot negative set. This setting better represents a real large-scale annotation scenario.

---

## 19. Benchmark-Specific Threshold Comparison

Two benchmark-specific thresholds were reported:

```text
Hard balanced benchmark threshold: 1e-1
Random/full Swiss-Prot benchmark threshold: 1e-6
```

The hard balanced benchmark selected a more relaxed threshold because the positive and negative classes were balanced.

The random/full Swiss-Prot benchmark selected a stricter threshold because the negative set was much larger. In a large imbalanced dataset, even a small false positive rate can generate many false positive predictions.

For this reason, the final operational threshold was selected from the random/full Swiss-Prot benchmark:

```text
Final operational threshold: 1e-6
```

Generated files:

```text
RESULTS/tables/benchmark_threshold_comparison.tsv
RESULTS/tables/final_metrics_benchmark_specific_thresholds.tsv
RESULTS/tables/final_metrics_random_full_threshold.tsv
RESULTS/tables/final_confusion_matrix_random_full_threshold.tsv
```

---

## 20. False Positive and False Negative Analysis

False positive and false negative predictions were inspected using the final operational threshold.

False positives correspond to non-Kunitz proteins predicted as Kunitz by the model.

False negatives correspond to annotated Kunitz proteins missed by the model.

False negatives may correspond to:

- divergent Kunitz/BPTI domains
- incomplete or atypical domain sequences
- weak HMMER matches above the selected E-value threshold
- proteins whose Kunitz-like regions differ from the structurally resolved domains used to train the model

Generated files:

```text
RESULTS/tables/final_false_positives_random_full_threshold.tsv
RESULTS/tables/final_false_negatives_random_full_threshold.tsv
```

---

## Stage 5 — Biological Interpretation

## 21. Final Interpretation

The final model was interpreted in relation to known structural features of Kunitz/BPTI domains.

Special attention was given to conserved cysteine residues, because Kunitz/BPTI domains are stabilized by disulfide bonds formed between conserved cysteines.

The HMM/logo comparison was used to visually inspect whether the generated model captured conserved sequence features of the Kunitz/BPTI domain.

The final evaluation used the random/full Swiss-Prot benchmark and the final operational threshold of `1e-6`, because this benchmark better reflects realistic large-scale annotation where negative proteins greatly outnumber positive Kunitz/BPTI proteins.
