# Workflow

This project follows a structure-based bioinformatics workflow to build and evaluate a profile Hidden Markov Model (HMM) for detecting BPTI/Kunitz-type protease inhibitor domains.

The workflow was designed to address the main project aim: building a profile HMM for the Kunitz-type protease inhibitor domain starting from available structural information and using the model to annotate Kunitz domains in Swiss-Prot proteins.

---

## 1. PDB Structure Retrieval

The workflow starts from experimentally resolved protein structures. Kunitz/BPTI-related structures were retrieved from the Protein Data Bank (PDB).

The goal of this step was to collect structural examples of the Kunitz domain that could be used to build a structure-guided seed alignment.

---

## 2. Chain-Level Sequence Extraction

PDB entries can contain multiple chains, and not all chains necessarily correspond to the Kunitz/BPTI domain.

For this reason, sequences were extracted at the chain level instead of only at the PDB-entry level. This avoids mixing the Kunitz inhibitor chain with unrelated chains from protein complexes.

---

## 3. Length-Based Filtering

Kunitz/BPTI domains are short domains, typically around 50–60 amino acids.

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
