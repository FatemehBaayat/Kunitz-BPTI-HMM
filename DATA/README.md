# Data

This folder contains raw and processed data files used in the project.

Large Swiss-Prot negative datasets are not included in the repository because of file size. They can be regenerated from UniProt/Swiss-Prot using the queries described in the notebook.

## Raw data
- `pdb_ids.txt`: list of selected PDB identifiers.
- `pdb_report.csv`: metadata downloaded from PDB.

## Processed data
- `pdb_seqs.txt`: chain-level sequences extracted from PDB entries.
- `pdb_id.rep`: representative sequences selected after MMseqs2 clustering.
- `chains_list.txt`: selected representative chains.
- `positive_kunitz.fasta`: Swiss-Prot proteins annotated with the Kunitz/BPTI domain.
- `positive_kunitz_independent.fasta`: positive benchmark after removing training-like sequences.
- `positive_training_like_removed.fasta`: removed positives similar to the training structures.
