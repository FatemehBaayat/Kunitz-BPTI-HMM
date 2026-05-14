# Kunitz-BPTI-HMM

Structure-based workflow for building and evaluating a profile Hidden Markov Model for detecting BPTI/Kunitz-type protease inhibitor domains in Swiss-Prot proteins.

---

## Overview

This project builds and evaluates a profile Hidden Markov Model (HMM) for detecting BPTI/Kunitz-type protease inhibitor domains.

The workflow starts from structurally resolved Kunitz/BPTI domains from the Protein Data Bank (PDB), builds a structure-guided seed alignment, trains a profile HMM using HMMER, and evaluates the model on Swiss-Prot benchmark proteins.

The project follows a structure-based strategy for Kunitz domain detection and evaluates the model using threshold optimization, confusion matrices, cross-validation, and false positive / false negative analysis.

---

## Biological Background

Kunitz domains are protease inhibitor domains found in proteins that inhibit protein-degrading enzymes.

A classical example is bovine pancreatic trypsin inhibitor (BPTI), also known as aprotinin. Kunitz/BPTI domains are short alpha/beta domains, usually around 50–60 amino acids long. Their fold is stabilized by conserved cysteine residues that form disulfide bonds.

---

## Aim

The main aims of this project are:

1. Build a profile HMM for the BPTI/Kunitz domain starting from available structural information.
2. Use the trained model to detect Kunitz domains in Swiss-Prot proteins.
3. Evaluate model performance using benchmark datasets, threshold optimization, confusion matrices, and classification metrics.
4. Analyze false positive and false negative predictions.

---

## Workflow

The full workflow is described in [`WORKFLOW.md`](WORKFLOW.md).

### Workflow Summary

```text
PDB structures
      ↓
Chain-level sequence extraction
      ↓
Length filtering and MMseqs2 clustering
      ↓
Representative chain selection
      ↓
Multiple structural alignment
      ↓
Structural QC and outlier removal
      ↓
Refined alignment
      ↓
Profile HMM construction
      ↓
Swiss-Prot benchmark search
      ↓
Threshold optimization and evaluation
      ↓
False positive / false negative analysis
      ↓
Biological interpretation
```

---

## Repository Structure

```text
Kunitz-BPTI-HMM/
│
├── DATA/
│   ├── raw/
│   │   ├── pdb_ids.txt
│   │   └── pdb_report.csv
│   │
│   └── processed/
│       ├── pdb_seqs.txt
│       ├── pdb_id.rep
│       ├── chains_list.txt
│       ├── positive_kunitz.fasta
│       ├── positive_kunitz_independent.fasta
│       └── positive_training_like_removed.fasta
│
├── NOTEBOOK/
│   └── Kunitz_BPTI_HMM_workflow.ipynb
│
├── RESULTS/
│   ├── alignments/
│   ├── hmm/
│   ├── tables/
│   ├── figures/
│   └── structures/
│
├── README.md
├── WORKFLOW.md
└── .gitignore
```

---

## Data Sources

This project uses data from:

- **Protein Data Bank (PDB)**  
  Structurally resolved Kunitz/BPTI-related protein structures.

- **UniProt / Swiss-Prot**  
  Manually curated protein sequences used for benchmark construction.

- **Pfam PF00014**  
  BPTI/Kunitz domain annotation used to define positive benchmark proteins.

- **HMMER output files**  
  Profile HMM search results used for prediction and evaluation.

- **Structural alignment outputs**  
  RMSD and TM-score matrices used for structural quality control.

Large Swiss-Prot negative FASTA files are not included in this repository because of file size. They can be regenerated using the queries and workflow described in the notebook.

---

## Methods

### 1. Structure Selection

Kunitz-related PDB entries were collected and parsed at the chain level.

Because PDB entries can contain multiple chains, chain-level extraction was used to avoid including unrelated chains from protein complexes.

---

### 2. Length Filtering

Chain sequences were filtered based on the expected length of Kunitz/BPTI domains.

This step reduced unrelated chains before clustering and structural alignment.

---

### 3. Sequence Clustering

Redundant sequences were clustered using **MMseqs2**.

One representative chain was selected from each cluster.

---

### 4. Structural Alignment

Representative structures were aligned using a multiple structural alignment workflow.

Pairwise **RMSD** and **TM-score** values were calculated to assess structural consistency.

---

### 5. Outlier Removal

Potential structural outliers were identified using RMSD and TM-score matrices.

After removing outliers, a refined alignment was generated.

---

### 6. HMM Construction

A profile HMM was built from the refined alignment using HMMER:

```bash
hmmbuild kunitz.hmm alignment_clean_trimmed.fasta
```

The final HMM model is stored in:

```text
RESULTS/hmm/kunitz.hmm
```

---

### 7. Benchmark Construction

Swiss-Prot proteins were divided into:

- **Positive benchmark:** proteins annotated with the BPTI/Kunitz domain.
- **Negative benchmark:** proteins without the BPTI/Kunitz domain annotation.

Training-like positive sequences were removed to obtain an independent positive benchmark.

---

### 8. HMM Search

The trained HMM was searched against benchmark proteins using:

```bash
hmmsearch -Z 1000 --max --tblout output.tbl kunitz.hmm input.fasta
```

The `--tblout` output was parsed for downstream prediction and performance evaluation.

---

### 9. No-Hit Handling

Some benchmark proteins did not return any HMMER hit.

To keep all benchmark proteins in the final evaluation, no-hit sequences were reintroduced into the prediction table and assigned a high default E-value.

---

### 10. Performance Evaluation

Predictions were evaluated using:

- Accuracy
- Sensitivity / Recall
- Specificity
- Precision
- F1-score
- Matthews Correlation Coefficient (MCC)

Threshold optimization was performed by testing different E-value thresholds and selecting thresholds based on MCC.

---

## Results

The model achieved strong performance in detecting Kunitz/BPTI domains in Swiss-Prot proteins.

The final evaluation included:

- Standard Swiss-Prot benchmark
- Hard-negative benchmark
- Threshold optimization
- Confusion matrix analysis
- False positive / false negative inspection
- Iterative cross-validation analysis

Main output files are stored in:

```text
RESULTS/tables/
```

The final HMM model is stored in:

```text
RESULTS/hmm/kunitz.hmm
```

The refined alignment is stored in:

```text
RESULTS/alignments/alignment_clean_trimmed.fasta
```

Structural quality control figures are stored in:

```text
RESULTS/figures/
```

---

## Key Outputs

| Output | Description |
|---|---|
| `alignment_clean_trimmed.fasta` | Refined structure-guided sequence alignment |
| `kunitz.hmm` | Final profile HMM model |
| `threshold_optimization_results_hard.tsv` | Threshold optimization results for the hard-negative benchmark |
| `threshold_optimization_results_random.tsv` | Threshold optimization results for the random/full Swiss-Prot benchmark |
| `final_metrics_benchmark_specific_thresholds.tsv` | Final benchmark-specific performance metrics |
| `final_confusion_matrix_hard_optimized_threshold.tsv` | Confusion matrix for the hard benchmark at the MCC-optimized threshold |
| `final_confusion_matrix_hard_conservative_threshold.tsv` | Confusion matrix for the hard benchmark at the conservative threshold |
| `iterative_cross_validation_results_hard.tsv` | Detailed iterative cross-validation results |
| `iterative_cross_validation_summary_hard.tsv` | Summary of iterative cross-validation performance |
| `pairwise_rmsd_heatmap.png` | RMSD structural quality control heatmap |
| `pairwise_tm_score_heatmap.png` | TM-score structural quality control heatmap |
| `kunitz_logo_comparison.png` | Comparison between generated and curated Kunitz logos |

---

## Hard-Negative Benchmark

In addition to the standard Swiss-Prot benchmark, a hard-negative benchmark was created.

Hard negatives were selected among non-Kunitz proteins with Kunitz-like properties, such as:

- Cysteine-rich composition
- Domain-like sequence length
- Absence of BPTI/Kunitz annotation

This benchmark was used as an additional robustness analysis to test whether the model remains specific against proteins that are more similar to Kunitz domains than random Swiss-Prot negatives.

---

## Cross-Validation

An iterative cross-validation strategy was used to evaluate the robustness of threshold selection and model performance.

The benchmark data were repeatedly split into training and test subsets. For each iteration, the optimal threshold was selected on one subset and evaluated on the other.

Performance metrics were then averaged across iterations.

Generated files:

```text
RESULTS/tables/iterative_cross_validation_results_hard.tsv
RESULTS/tables/iterative_cross_validation_summary_hard.tsv
```

---

## False Positive and False Negative Analysis

False positive and false negative predictions were inspected to better understand the limitations of the model.

False negatives may correspond to:

- Divergent Kunitz domains
- Incomplete domain annotations
- Weak HMMER matches above the selected E-value threshold
- Proteins whose Kunitz-like regions differ from the structurally resolved domains used to train the model

This analysis helps interpret model errors rather than only reporting global performance metrics.

---

## How to Run

The complete workflow is provided in the notebook:

```text
NOTEBOOK/Kunitz_BPTI_HMM_workflow.ipynb
```

Required tools include:

- Python
- Biopython
- pandas
- NumPy
- matplotlib
- HMMER
- BLAST+
- MMseqs2
- TM-align / mTM-align

Recommended environment setup:

```bash
conda create -n kunitz-hmm python=3.10
conda activate kunitz-hmm
conda install -c conda-forge -c bioconda biopython pandas numpy matplotlib hmmer blast mmseqs2
```

Then open and run the notebook.

---

## Notes

The standard benchmark follows the Swiss-Prot positive/negative validation strategy.

The hard-negative benchmark was added as an additional robustness analysis to test whether the model remains specific against cysteine-rich non-Kunitz proteins.

The negative Swiss-Prot FASTA file is not included because of file size, but it can be regenerated using the notebook.

---

## References

- Protein Data Bank (PDB)
- UniProt / Swiss-Prot
- Pfam PF00014: BPTI/Kunitz domain
- HMMER
- MMseqs2
- TM-align / mTM-align
