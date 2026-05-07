# Project Workflow

This file describes the main workflow used in this project for building and evaluating a structure-based Kunitz/BPTI profile HMM.

## Main steps

1. Data acquisition from RCSB PDB
2. Sequence cleaning and filtering
3. Sequence clustering using MMseqs2
4. Representative PDB chain selection
5. PDB structure download
6. Single-chain PDB extraction
7. Structural alignment using mTM-align / TM-align
8. HMM construction
9. Model evaluation
10. Result visualization and consistency check
