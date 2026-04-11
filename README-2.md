# Evaluation of AI Health Advisors: Alignment of Global Health Chatbots with Kazakhstani Women's Health Clinical Protocols

**Author:** Akhmetzhan Dilnaz  
**Degree:** Master of Science in Data Science  
**Institution:** Nazarbayev University  
**Supervisor:** Lisa Chalaguine  

---

## Overview

This repository contains all notebooks used to produce the results of the thesis. The study evaluates four AI health advisory systems — ChatGPT-4o, Gemini-2.5-Flash, Perplexity, and MedGemma-27B — against official Kazakhstani OB-GYN clinical protocols across 15 clinical questions in both English and Russian.

The pipeline is divided into two phases:

- **Phase I** — Corpus construction and structural validation of 84 Kazakhstani OB-GYN clinical protocols
- **Phase II** — AI chatbot response collection and multi-dimensional alignment evaluation

---

## Repository Structure

```
├── Phase I — Corpus Preparation
│   ├── protocol_preprocessing.ipynb
│   ├── embeddings_english.ipynb
│   ├── embeddings_russian.ipynb
│   └── corpus_validation.ipynb
│
├── Phase II — Data Collection
│   ├── question_design.ipynb
│   ├── chatgpt_collection.ipynb
│   ├── gemini_collection.ipynb
│   ├── medgemma_collection.ipynb
│   └── perplexity_collection.ipynb   ← collected manually via web interface
│
├── Phase II — Evaluation
│   └── chatbot_evaluation.ipynb
│
└── README.md
```

---

## Notebook Descriptions

### Phase I — Corpus Preparation

#### `protocol_preprocessing.ipynb`
Converts all 84 OB-GYN clinical protocols from PDF to structured Markdown using Docling layout-aware parsing. Performs OCR fallback for scanned pages, structural cleaning (removes citations, metadata, headers/footers), section segmentation, and Russian-to-English translation using LLaMA-3 8B Instruct (4-bit quantized, temperature=0). Also runs biomedical NER using `d4data/biomedical-ner-all` (BioBERT-based) to extract drugs, diseases, procedures, and therapies.

**Inputs:** 84 PDF protocol files from the Ministry of Health of Kazakhstan  
**Outputs:** `en_aux_v1_fixed/` (English Markdown corpus), `ner/en_biobert_v1/` (NER entity sets per protocol), `ner_summary.csv`

---

#### `embeddings_english.ipynb`
Generates section-level embeddings for the English protocol corpus using sentence-transformers. Runs KMeans clustering (k=12) and computes silhouette scores to validate thematic coherence. Produces t-SNE coordinates for visualisation.

**Inputs:** English Markdown corpus from `protocol_preprocessing.ipynb`  
**Outputs:** `en_corpus_validation_summary.json`, `clusters_sections.csv`, `en_tsne_coords.npy`, `en_cluster_summary.csv`

---

#### `embeddings_russian.ipynb`
Mirrors `embeddings_english.ipynb` for the Russian protocol corpus using multilingual MPNet embeddings to enable cross-lingual structural comparison.

**Inputs:** Russian Markdown corpus  
**Outputs:** `ru_corpus_validation_summary.json`, `ru_clusters_sections.csv`, `ru_tsne_coords.npy`, `ru_cluster_summary.csv`

---

#### `corpus_validation.ipynb`
Applies Top2Vec topic modelling to both English and Russian corpora jointly. Computes cross-lingual topic alignment scores (cosine similarity ~0.95) to confirm that translation preserved thematic structure. Also used for final topic anchor selection to guide clinical question design.

**Inputs:** Section metadata CSVs from embedding notebooks  
**Outputs:** `top2vec_ru_en_topic_alignment.csv`, `top2vec_summary.json`, `top2vec_topics_EN.csv`

---

### Phase II — Data Collection

#### `question_design.ipynb`
Designs and validates the 15 clinical evaluation questions through stratified purposive sampling across 5 clinical strata (Diagnosis & Criteria, Pharmacotherapy, Surgical & Emergency, Monitoring & Screening, Infection & Sepsis). Questions are grounded in specific protocol sections and validated for coverage and clinical specificity.

**Inputs:** `top2vec_topics_EN.csv`, English protocol Markdown files  
**Outputs:** `final_questions.json` (15 questions with EN/RU versions, stratum labels, target protocol mappings)

---

#### `chatgpt_collection.ipynb`
Queries ChatGPT-4o (GPT-4o, temperature=0) with all 15 questions in both English and Russian using the OpenAI API. Includes resume support for interrupted runs.

**Inputs:** `final_questions.json`  
**Outputs:** `chatgpt_responses.json` (30 responses)  
**Requirements:** OpenAI API key, CPU runtime

---

#### `gemini_collection.ipynb`
Queries Gemini-2.5-Flash via the Google GenAI API for all 15 questions × 2 languages.

**Inputs:** `final_questions.json`  
**Outputs:** `gemini_responses.json` (30 responses)  
**Requirements:** Google AI API key, CPU runtime

---

#### `medgemma_collection.ipynb`
Queries MedGemma-27B (instruction-tuned, 4-bit quantized via bitsandbytes) locally via HuggingFace Transformers for all 15 questions × 2 languages.

**Inputs:** `final_questions.json`  
**Outputs:** `medgemma_responses.json` (30 responses)  
**Requirements:** HuggingFace token, T4 GPU runtime, accepted model licence at huggingface.co/google/medgemma-27b-it

---

> **Note on Perplexity:** Perplexity Medical responses were collected manually via the web interface (perplexity.ai) and saved directly to `perplexity_responses.json`. No API collection notebook exists for this model.

---

### Phase II — Evaluation

#### `chatbot_evaluation.ipynb`
The main evaluation notebook. Merges all four bot response files into `responses_raw.json` (120 responses: 4 bots × 15 questions × EN + RU) and runs the full multi-dimensional alignment pipeline. Must be run **top to bottom** — later sections depend on variables produced by earlier ones.

**Inputs:** `responses_raw.json`, `final_questions.json`, NER outputs from `ner/en_biobert_v1/`, English protocol Markdown files  
**Outputs:** All CSVs and figures listed below

| Section | Analysis | Output files |
|---------|----------|--------------|
| 5 | NER + entity-level metrics (Table 4.1) | `alignment_entity_results.csv` |
| 6 | Semantic similarity — Table 4.2 | `alignment_semantic_results.csv` |
| 7 | Cross-lingual stability — Table 4.3 | `cross_lingual_stability.csv` |
| 8 | Statistical significance (Kruskal-Wallis, Mann-Whitney) | `statistical_tests.csv` |
| 9 | Figures 4.1–4.4 | PNG figures |
| 11.1 | Protocol adherence scoring | `protocol_adherence.csv` |
| 11.2 | Guideline source detection (INTL vs KZ) | `guideline_sources.csv` |
| 11.3 | Numeric value comparison | — |
| 11.4 | Critical omissions | `critical_omissions.csv` |
| 12.1 | Drug order alignment (Kendall-tau) | `drug_order_alignment.csv` |
| 12.2 | Localization gap (KZ-specific drugs) | `localization_gap.csv` |
| 12.3 | Clinical density | `clinical_density.csv` |
| 13.1 | Composite stratum radar chart | PNG figure |
| 13.2 | Hedging & confidence calibration | `hedging_analysis.csv` |
| 13.3 | Patient safety risk profile (12 flags) | `safety_flags.csv` |
| 14.1 | Language compliance (RU responses) | `language_compliance.csv` |
| 14.2–14.5 | Russian mirrors of Sections 11–13 | CSV outputs |
| 14.6 | EN vs RU composite delta | PNG figure |

**Requirements:** GPU recommended for NER (CPU works but is slow), all Phase I outputs must be present on Google Drive

---

## Data

All data files are stored on Google Drive at:
```
/content/drive/MyDrive/medical_protocols/
```

The 84 source protocols are official clinical guidelines published by the Ministry of Health of the Republic of Kazakhstan. They are publicly available at the official Ministry of Health portal. No patient data of any kind was used in this study.

---

## How to Run

1. Upload all notebooks to Google Colab
2. Mount Google Drive in each notebook (first code cell)
3. Run Phase I notebooks in order: `protocol_preprocessing` → `embeddings_english` → `embeddings_russian` → `corpus_validation`
4. Run `question_design.ipynb` to produce `final_questions.json`
5. Run each collection notebook (`chatgpt_collection`, `gemini_collection`, `medgemma_collection`) and add Perplexity responses manually
6. Merge all response files into `responses_raw.json`
7. Run `chatbot_evaluation.ipynb` **top to bottom** — do not skip cells or restart mid-run

---

## Dependencies

All notebooks install their own dependencies in the first cell via `pip install`. Key libraries:

- `sentence-transformers` — semantic embeddings
- `transformers`, `accelerate`, `bitsandbytes` — MedGemma inference
- `scikit-learn`, `scipy` — clustering and statistical tests
- `seaborn`, `matplotlib` — visualisation
- `openai` — ChatGPT-4o API
- `google-genai` — Gemini API
- `top2vec` — topic modelling
- `docling` — PDF parsing (Phase I)

---

## Notes

- All API calls use `temperature=0` for deterministic, reproducible outputs
- The evaluation notebook computes embeddings on-the-fly; do not load pre-computed Phase I embeddings into the Phase II pipeline as they are incompatible
- Figures are saved at 300 DPI to `plots/` subdirectory on Google Drive
- The unified colour scheme for all figures uses: ChatGPT-4o = `#2E75B6`, Gemini-2.5-Flash = `#ED7D31`, Perplexity = `#70AD47`, MedGemma-27B = `#7030A0`
