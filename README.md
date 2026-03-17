# Chemical Ontology Classification

Hackathon solution for hierarchical multi-label classification of molecules into 500 chemical ontology classes (ChEBI-style DAG).

## Problem

Given a molecule (as SMILES), predict which of 500 ontology classes it belongs to — a **multi-label hierarchical classification** problem on a DAG class graph.

- **Train:** 33,668 molecules, 500 classes
- **Test:** 11,223 molecules
- **Output:** binary predictions (0/1) for all 500 classes per molecule

## Scoring

- **Primary metric:** Macro-averaged F1 (mean F1 across all 500 classes)
- **Tiebreaker** (if F1 difference < 1%): graph consistency — penalizes predicting a child class without its parent
- **Public leaderboard:** 50% of test molecules
- **Final leaderboard:** 100% of test molecules

<img width="1161" height="635" alt="image" src="https://github.com/user-attachments/assets/8deae600-33fd-447e-9163-3ae92f0301e2" />
