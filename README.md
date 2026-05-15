# Kunitz-BPTI-HMM

Structure-based workflow for building and evaluating a profile Hidden Markov Model (HMM) for detecting BPTI/Kunitz-type protease inhibitor domains in Swiss-Prot proteins.

---

## Overview

This project builds and evaluates a profile Hidden Markov Model for detecting Kunitz/BPTI-type protease inhibitor domains.

The workflow starts from structurally resolved Kunitz/BPTI domains retrieved from the Protein Data Bank (PDB). Representative structures are selected after sequence filtering and redundancy reduction, aligned using a multiple structural alignment approach, and converted into a structure-guided sequence alignment. This final alignment is used to build a profile HMM with HMMER.

The model is then evaluated on Swiss-Prot benchmark datasets using `hmmsearch`, threshold optimization, confusion matrices, classification metrics, and false positive / false negative analysis.

---

## Biological Background

Kunitz/BPTI domains are short protease inhibitor domains, usually around 50–60 amino acids long. A classical example is bovine pancreatic trypsin inhibitor (BPTI), also known as aprotinin.

The Kunitz/BPTI fold is stabilized by conserved cysteine residues that form disulfide bonds. These conserved cysteines are expected to appear clearly in the final HMM logo and are important indicators that the model captures the main structural features of the domain.

---

## Aim

The main aims of this project are:

1. Build a profile HMM for the Kunitz/BPTI domain using structural information from PDB.
2. Use the trained model to detect Kunitz/BPTI domains in Swiss-Prot proteins.
3. Evaluate model performance using benchmark datasets, threshold optimization, confusion matrices, and classification metrics.
4. Analyze false positive and false negative predictions.
5. Compare the generated HMM logo with the curated Pfam PF00014 Kunitz/BPTI logo.

---

## Workflow

The full workflow is described in [`WORKFLOW.md`](WORKFLOW.md).

### Workflow Summary

```text
PDB structures
      ↓
Chain-level sequence extraction
      ↓
Length filtering
      ↓
MMseqs2 clustering at 90% identity and 90% coverage
      ↓
Representative chain selection
      ↓
Multiple structural alignment
      ↓
RMSD and TM-score structural QC
      ↓
Removal of main structural outlier 2KAI_A
      ↓
Refined structure-based sequence alignment
      ↓
Profile HMM construction with HMMER
      ↓
Swiss-Prot benchmark search with hmmsearch
      ↓
Threshold optimization
      ↓
Final evaluation on random/full Swiss-Prot benchmark
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
│   │   ├── pdb_report.csv
│   │   ├── positive_kunitz.fasta
│   │   └── negative_kunitz.fasta
│   │
│   └── processed/
│       ├── pdb_seqs.txt
│       ├── pdb_cluster_rep_seq.fasta
│       ├── pdb_id.rep
│       ├── chains_list.txt
│       ├── chains_list_no_outliers.txt
│       ├── training_representatives_ungapped.fasta
│       ├── positive_kunitz_independent.fasta
│       └── training_like_positive.ids
│
├── NOTEBOOK/
│   └── Kunitz_BPTI_HMM_workflow.ipynb
│
├── RESULTS/
│   ├── alignments/
│   │   ├── alignment.fasta
│   │   ├── alignment_no_outliers.fasta
│   │   └── alignment_clean_trimmed.fasta
│   │
│   ├── hmm/
│   │   └── kunitz.hmm
│   │
│   ├── tables/
│   │   ├── threshold_optimization_results_hard.tsv
│   │   ├── threshold_optimization_results_random_full.tsv
│   │   ├── benchmark_threshold_comparison.tsv
│   │   ├── final_metrics_benchmark_specific_thresholds.tsv
│   │   ├── final_metrics_random_full_threshold.tsv
│   │   ├── final_confusion_matrix_random_full_threshold.tsv
│   │   ├── final_false_positives_random_full_threshold.tsv
│   │   └── final_false_negatives_random_full_threshold.tsv
│   │
│   ├── figures/
│   │   ├── pairwise_rmsd_heatmap.png
│   │   ├── pairwise_tm_score_heatmap.png
│   │   ├── mean_rmsd_lineplot.png
│   │   ├── mean_tm_score_lineplot.png
│   │   └── kunitz_logo_comparison.png
│   │
│   └── structures/
│       ├── chains/
│       └── pdbs/
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
  Reviewed protein sequences used for positive and negative benchmark construction.

- **Pfam PF00014**  
  BPTI/Kunitz domain annotation used to define positive benchmark proteins.

- **HMMER output files**  
  Profile HMM search results used for prediction and performance evaluation.

- **Structural alignment outputs**  
  RMSD and TM-score matrices used for structural quality control.

Large Swiss-Prot negative FASTA files may be excluded from the repository because of file size. They can be regenerated using the workflow described in the notebook.

---

## Methods

### 1. Structure Selection

Kunitz/BPTI-related structures were retrieved from the RCSB Protein Data Bank using the Advanced Search interface.

The query was based on:

- **Pfam ID:** `PF00014`
- **Experimental resolution:** `≤ 3.5 Å`
- **Polymer entity sequence length:** `45 ≤ length ≤ 80 amino acids`

The length range was selected because Kunitz/BPTI domains are short domains, typically around 50–60 residues.

The PDB results were exported using:

```text
Tabular Reports → Custom Report
```

The selected fields included:

- Entry ID
- Entity ID
- Auth Asym ID
- Polymer entity sequence

Because PDB entries may contain multiple chains, chain-level parsing was performed to avoid including unrelated chains from protein complexes.

---

### 2. Length Filtering

Extracted chain sequences were filtered according to the expected length of Kunitz/BPTI domains.

This step removed clearly unrelated chains before clustering and structural alignment.

---

### 3. Redundancy Reduction with MMseqs2

Redundant sequences were clustered using **MMseqs2**.

The final clustering parameters were:

```text
90% sequence identity
90% coverage
```

One representative chain was selected from each cluster. The 90% / 90% threshold was selected as a compromise between reducing redundancy and retaining enough representative structures for multiple structural alignment.

---

### 4. Structural Alignment

Representative PDB chains were downloaded and extracted.

A multiple structural alignment was performed using the selected representative structures. The resulting structural alignment was converted into a structure-based sequence alignment for HMM construction.

---

### 5. Structural Quality Control

Pairwise **RMSD** and **TM-score** matrices were calculated to assess structural consistency among representatives.

The RMSD and TM-score heatmaps showed that most representative structures formed a consistent Kunitz/BPTI structural cluster. The strongest structural outlier was:

```text
2KAI_A
```

This structure showed high mean RMSD and low mean TM-score compared with the remaining representatives. Therefore, only `2KAI_A` was removed before rebuilding the final structural alignment.

---

### 6. HMM Construction

The cleaned and trimmed structure-based alignment was used to build a profile HMM with HMMER:

```bash
hmmbuild --amino -n Kunitz_BPTI kunitz.hmm alignment_clean_trimmed.fasta
```

The final HMM model is stored in:

```text
RESULTS/hmm/kunitz.hmm
```

---

### 7. HMM Profile Inspection

The final `kunitz.hmm` file was inspected to verify the internal HMMER profile structure.

The HMM file contains:

- model metadata, such as `NAME`, `LENG`, `ALPH`, and `NSEQ`
- amino-acid emission scores
- transition scores between match, insert, and delete states

This confirms that the HMM is a position-specific probabilistic model rather than only a sequence alignment.

---

### 8. HMM Logo Analysis

A sequence logo was generated from the final profile HMM and compared with the curated Pfam PF00014 Kunitz/BPTI logo.

The final HMM logo preserved the main conserved cysteine pattern expected for Kunitz/BPTI domains. Minor differences compared with the Pfam logo are expected because the custom HMM was trained on a smaller structure-based representative dataset, while Pfam is based on a larger curated family alignment.

---

### 9. Benchmark Construction

Swiss-Prot benchmark proteins were divided into:

- **Positive benchmark:** reviewed proteins annotated with Pfam `PF00014`
- **Negative benchmark:** reviewed proteins without Pfam `PF00014`

Training-like positive sequences were removed from the positive benchmark to make the evaluation more independent from the PDB-derived training representatives.

---

### 10. HMM Search

The trained HMM was searched against benchmark proteins using:

```bash
hmmsearch -Z 1000 --max --tblout output.tbl kunitz.hmm input.fasta
```

The `--tblout` output was parsed for downstream prediction and performance evaluation.

The best domain E-value was used as the classification score.

---

### 11. No-Hit Handling

Some negative benchmark proteins did not return any HMMER hit.

To keep all benchmark proteins in the final evaluation, no-hit sequences were reintroduced into the prediction table and assigned a high default E-value.

---

### 12. Performance Evaluation

Predictions were evaluated using:

- Accuracy
- Sensitivity / Recall
- Specificity
- Precision
- F1-score
- Matthews Correlation Coefficient (MCC)

MCC was used as the main metric for threshold selection because it is more informative than accuracy, especially when class distributions are imbalanced.

Two benchmark settings were evaluated:

1. **Hard balanced benchmark**  
   A balanced benchmark with the same number of positive and negative sequences.

2. **Random/full Swiss-Prot benchmark**  
   A highly imbalanced benchmark containing all or a large set of Swiss-Prot negative sequences, better representing a real large-scale annotation scenario.

---

## Results

### Benchmark-Specific Thresholds

Two benchmark-specific thresholds were evaluated:

| Benchmark | Optimized threshold | Purpose |
|---|---:|---|
| Hard balanced benchmark | `1e-1` | Tests discrimination under balanced conditions |
| Random/full Swiss-Prot benchmark | `1e-6` | Final operational threshold for large-scale annotation |

The hard balanced benchmark selected a more relaxed threshold because the positive and negative classes were balanced.

The random/full Swiss-Prot benchmark selected a stricter threshold because the negative set was much larger. In a large imbalanced dataset, even a small false positive rate can generate many false positive predictions. Therefore, the `1e-6` threshold was selected as the final operational threshold.

---

### Final Evaluation

The final evaluation was performed on the random/full Swiss-Prot benchmark using:

```text
Final operational threshold = 1e-6
```

This benchmark better represents a realistic annotation scenario because the number of negative sequences is much larger than the number of positive Kunitz/BPTI sequences.

Final output files are stored in:

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
| `alignment_clean_trimmed.fasta` | Final cleaned and trimmed structure-guided sequence alignment |
| `kunitz.hmm` | Final Kunitz/BPTI profile HMM model |
| `threshold_optimization_results_hard.tsv` | Threshold optimization results for the hard balanced benchmark |
| `threshold_optimization_results_random_full.tsv` | Threshold optimization results for the random/full Swiss-Prot benchmark |
| `benchmark_threshold_comparison.tsv` | Comparison of hard-optimized and random/full-optimized thresholds |
| `final_metrics_benchmark_specific_thresholds.tsv` | Benchmark-specific performance comparison |
| `final_metrics_random_full_threshold.tsv` | Final metrics using the final operational threshold |
| `final_confusion_matrix_random_full_threshold.tsv` | Final confusion matrix on the random/full benchmark |
| `final_false_positives_random_full_threshold.tsv` | Final false positive predictions at the final threshold |
| `final_false_negatives_random_full_threshold.tsv` | Final false negative predictions at the final threshold |
| `pairwise_rmsd_heatmap.png` | RMSD structural quality control heatmap |
| `pairwise_tm_score_heatmap.png` | TM-score structural quality control heatmap |
| `mean_rmsd_lineplot.png` | Mean RMSD per representative structure |
| `mean_tm_score_lineplot.png` | Mean TM-score per representative structure |
| `kunitz_logo_comparison.png` | Comparison between generated HMM logo and curated Pfam PF00014 logo |

---

## False Positive and False Negative Analysis

False positive and false negative predictions were inspected using the final operational threshold.

False positives correspond to non-Kunitz proteins predicted as Kunitz by the model.

False negatives correspond to annotated Kunitz proteins missed by the model.

False negatives may correspond to:

- divergent Kunitz/BPTI domains
- incomplete or atypical domain sequences
- weak HMMER matches above the selected E-value threshold
- proteins whose Kunitz-like regions differ from the structurally resolved domains used to train the model

This analysis helps interpret model limitations rather than only reporting global performance metrics.

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
- MMseqs2
- TM-align / mTM-align

Recommended environment setup:

```bash
conda create -n kunitz-hmm python=3.10
conda activate kunitz-hmm
conda install -c conda-forge -c bioconda biopython pandas numpy matplotlib hmmer mmseqs2
```

Then open and run the notebook.

---

## Notes

The negative Swiss-Prot FASTA file may be excluded because of file size, but it can be regenerated using the workflow described in the notebook.

The final threshold used for large-scale annotation is:

```text
1e-6
```

This threshold was selected from the random/full Swiss-Prot benchmark because it provided strong MCC while keeping the number of false positives very low.

---

## References

- Protein Data Bank (PDB)
- UniProt / Swiss-Prot
- Pfam PF00014: BPTI/Kunitz domain
- HMMER
- MMseqs2
- TM-align / mTM-align
