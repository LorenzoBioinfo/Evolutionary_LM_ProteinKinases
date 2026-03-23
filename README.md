# Evolutionary Language Model on Protein Kinases

A decoder-only transformer trained from scratch on protein kinase sequences 
to investigate whether a small-scale language model can capture evolutionary 
constraints without any biological supervision.

## Motivation

Large protein language models like ESM-2 (650M parameters, 250M sequences) 
have demonstrated impressive biological knowledge. But how much of this signal 
emerges at a much smaller scale? This project uses the kinase superfamily as a 
case study to answer: **what does a 1.2M parameter model learn about biology 
from 6,333 sequences?**

## Dataset

- **Source**: UniProt Swiss-Prot + TrEMBL (existence:1) via REST API
- **Domain**: PF00069 (protein kinase domain) from InterPro/Pfam
- **Sequences**: 6,333 after filtering (length 200–400 aa)
- **Organisms**: 155 unique species across the tree of life
- **Split**: 90% train / 10% validation (random, seed=42)

## Model Architecture

Decoder-only transformer (GPT-style):

| Component | Value |
|-----------|-------|
| Parameters | ~1.2M |
| Vocabulary | 24 tokens (20 amino acids + PAD, UNK, BOS, EOS) |
| Embedding dim | 128 |
| Attention heads | 8 |
| Layers | 6 |
| Context length | 402 tokens |
| Dropout | 0.1 |

Training details:
- Optimizer: AdamW (lr=3e-4, weight_decay=0.01)
- Scheduler: cosine decay with 200-step warmup
- Epochs: 40
- Batch size: 32
- Final val loss: 1.533 (vs baseline ln(24) = 3.178, -52%)

## Experiments

### Experiment 1 and 2 — Functional Position Recognition

We compare model probabilities at each alignment position against Shannon 
entropy conservation scores computed from the Pfam seed alignment (37 sequences).

Key finding: the model shows two distinct behaviors across the 16 highly 
conserved positions (H < 0.5):

- **Above chance (1/20 = 0.05)**: catalytic lysine (P=0.29), P-loop G2 (P=0.14), 
  catalytic loop Arg (~0.05) — positions with highly recognizable local context
- **Below chance**: DFG motif, HRD motif, Mg2+ coordination — positions in 
  structurally flexible regions where local context is variable

The model captures evolutionary constraints mediated by local sequence context, 
but not those in structurally flexible regions such as the activation loop.

### Experiment 3 — Drug Resistance Mutations *(coming soon)*

Log-likelihood ratio analysis of known resistance mutations (T315I in BCR-ABL, 
T790M in EGFR) vs wild-type. Are clinically selected resistance mutations 
evolutionarily surprising to the model?

### Experiment 4 — Kinase Family Organization *(coming soon)*

UMAP of last-layer representations colored by kinase family (KinBase 
classification). Does the model's internal space reflect biological taxonomy?

### Experiment 5 — Sequence Generation *(coming soon)*

Do generated sequences contain conserved kinase motifs (DFG, HRD, GXGXXG)?
Validation via InterProScan and amino acid composition analysis.

### Experiment 6 — Zero-shot Fitness Prediction *(coming soon)*

Correlation between model log-likelihood scores and experimentally measured 
fitness on ProteinGym benchmark. Comparison with ESM-1v baseline.

## Results Summary

| Experiment | Result |
|------------|--------|
| Val loss reduction vs baseline | -52% |
| Catalytic lysine probability | 0.29 (6× above chance) |
| DFG motif probability | <0.01 (5× below chance) |
| Conserved positions above chance | 3/16 |

## Repository Structure
```
Evolutionary_LM.ipynb          # Single notebook: data → training → experiments
README.md
```

## Requirements
```
torch
transformers
biopython
numpy
pandas
matplotlib
scipy
umap-learn
requests
```

Install with:
```bash
pip install torch biopython numpy pandas matplotlib scipy umap-learn requests
```

## Usage

Open `Evolutionary_LM.ipynb` in Google Colab (GPU recommended) and run cells 
sequentially. The notebook handles data download from UniProt automatically.

Mount Google Drive before running to persist checkpoints between sessions.

## Biological Context

Protein kinases are enzymes that transfer phosphate groups from ATP to substrate 
proteins, regulating key cellular processes including signal transduction, 
metabolism and cell division. With over 500 kinases in the human proteome and 
many serving as drug targets in oncology (imatinib, gefitinib, erlotinib), 
understanding their evolutionary constraints has direct pharmacological relevance.

## Limitations

- Small model (1.2M parameters) — cannot be compared directly to ESM-2
- Limited dataset (6,333 sequences) — larger models use 250M+ sequences
- No structure information — sequence-only approach
- Seed alignment limited to 37 sequences for conservation analysis

```

## License

MIT
