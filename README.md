# dosTres-secstruct 🧩

**dosTres-secstruct** is part of the wider **dosTres** initiative for full scientific paper processing.  
This module focuses on **section detection, normalization, and hierarchy reconstruction**.

---

## 🔹 Motivation

Scientific articles are often parsed into flat lists of text blocks.  
However, section names are inconsistent (“Methods”, “Materials and Methods”),  
and hierarchy (subsections like *2.1 Cell culture*) is often missing or ambiguous.

Our goal is to **reconstruct a clean, hierarchical tree of section structure** with normalized canonical labels.

---

## 🔹 Problem Breakdown

- **Input:** a flat list of `(section_name, section_content)` items (often with missing/dirty numbering).  
- **Challenges:**
  - No consistent section numbering across papers.
  - Section names vary in specificity (“Methods” vs. “Immunocytochemistry”).
  - Some papers are well-structured, others are not.

---

## 🔹 Tasks

1. **Section Name Normalization**  
   Map raw section headers to a **canonical taxonomy**  
   (e.g., Introduction, Methods, Results, Discussion, Conclusion, References).

2. **Hierarchy Prediction**  
   From a flat list of sections, predict a **tree structure** where each section has:
   - a **parent** (or ROOT if top-level),
   - a **depth level**.

---

## 🔹 Dataset

We use the [SciLake Fulltext Corpus](https://huggingface.co/datasets/SIRIS-Lab/scilake-fulltext-corpus):  
- 1k curated CC-BY / Public Domain papers from diverse domains.  
- Parsed by section with metadata (`section_name`, `section_num`, `section_content`).

---

## 🔹 Approach (Baseline)

- **Step 1:** Preprocess data → extract hierarchy from `section_num` when available.  
- **Step 2:** Train a **classifier** (DistilBERT / SciBERT) to map section headers to canonical labels.  
- **Step 3:** Train a **hierarchy predictor**:
  - Baseline: rule-based from `section_num`.
  - ML: pairwise parent classification or GNN over section graph.  
- **Step 4:** Optionally unify into a **multi-task model** with shared encoder.

---

## 🔹 Example Output

Input (flat):  
```
["Introduction", "Cell culture", "Western blot", "Results", "Discussion"]
```

Output (tree):  
```
1. Introduction
2. Methods
  2.1 Cell culture
  2.2 Western blot
3. Results
4. Discussion
```
---

## 🔹 Approaches to Test

- **Rule-Based Baselines**
  - Use `section_num` (when available) to reconstruct hierarchy (1 → parent ROOT, 1.2 → parent 1, etc.).
  - Map `section_name` to canonical taxonomy with regex/keyword rules (e.g., “Materials and Methods” → **Methods**).

- **Supervised Section Name Normalization**
  - Train a classifier (DistilBERT, SciBERT) on section names (and optionally content).
  - Input: `[CLS] section_name [SEP] section_content`.
  - Output: canonical label (Introduction, Methods, Results, …).

- **Pairwise Parent Classification**
  - For each candidate pair (section j, section i with j<i), predict if j is the parent of i.
  - Use BERT with pairwise encoding: `[CLS] name_i [SEP] name_j [SEP] …`.
  - Select the highest-scoring parent per child.

- **Multi-Task Model**
  - Shared encoder with two heads:
    - Head 1: section label classification.
    - Head 2: parent pointer (predict parent section).
  - Joint training improves generalization.

- **Depth Prediction with CRF**
  - Predict **depth level** for each section in the sequence (0 = top-level, 1 = subsection, …).
  - Use CRF or structured decoding to enforce valid depth transitions.
  - Tree reconstructed deterministically from depth sequence.

- **Graph Neural Network (GNN)**
  - Build a graph: nodes = sections, edges = order, similarity, candidate parent links.
  - Use GAT/GraphSAGE to propagate context between nodes.
  - Node head → section label; Edge head → parent probability.

- **Sequence Labeling (BERT encoder)**
  - Treat the list of section names as a sequence.
  - Predict canonical label for each name in context of others (bidirectional attention).

- **Seq2Seq Generation (T5 / FLAN-T5)**
  - Input: flat list of raw section titles.
  - Output: normalized + structured hierarchy (as a string or serialized JSON).
  - More flexible but harder to control structure.

---

## 🔹 Project Status

- ✅ Data preprocessing scripts  
- 🚧 Baseline classifier (name normalization)  
- 🚧 GNN model for hierarchy prediction  

---

## 🔹 Installation

```bash
git clone https://github.com/your-org/dosTres-secstruct.git
cd dosTres-secstruct
pip install -e .
```

