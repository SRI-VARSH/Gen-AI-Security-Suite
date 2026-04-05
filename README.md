# 🛡️ GenAI Security — LLM Guardrail System

A multi-layered security pipeline that protects LLM interactions by detecting and blocking **jailbreak**, **harmful**, and **toxic** prompts before they reach the model — and validating the model's output before it reaches the user.

---

## 🔍 What It Does

Most LLM applications are vulnerable to adversarial prompts that trick the model into generating restricted content. This project builds a **dual guardrail system** — one sitting at the input, one at the output — so that neither malicious prompts nor unsafe responses slip through.

---

## 🗂️ Dataset

Four sources were combined to create a balanced, multi-class training set:

| Source | Type | Size |
|--------|------|------|
| [jackhhao/jailbreak-classification](https://huggingface.co/datasets/jackhhao/jailbreak-classification) | Jailbreak vs Benign | 1,044 rows |
| [rubend18/ChatGPT-Jailbreak-Prompts](https://huggingface.co/datasets/rubend18/ChatGPT-Jailbreak-Prompts) | Jailbreak prompts | 79 rows |
| [lmsys/toxic-chat](https://huggingface.co/datasets/lmsys/toxic-chat) | Toxic + jailbreak | 5,082 rows |
| Manually curated | Harmful prompts (violence, drugs, hacking, weapons, self-harm) | 50 rows |

After merging and balancing: **1,675 clean samples** across 3 classes — `safe`, `jailbreak`, `harmful`.

---

## ⚙️ Pipeline Overview

```
User Prompt
    │
    ▼
┌─────────────────────┐
│  Rule-Based Filter  │  ← regex patterns for known harmful categories
└────────┬────────────┘
         │ if safe
         ▼
┌─────────────────────┐
│   ML Classifier     │  ← SVM on sentence embeddings (all-MiniLM-L6-v2)
└────────┬────────────┘
         │ if safe
         ▼
┌─────────────────────┐
│   Gemini 2.5 Flash  │  ← LLM generates response
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Output Classifier  │  ← same ML model validates the response
└────────┬────────────┘
         │ if safe
         ▼
    Final Response
```

---

## 🧠 ML Classifier

- **Embeddings:** `sentence-transformers/all-MiniLM-L6-v2` (384-dim vectors)
- **Classifier:** Support Vector Machine (SVM)
- **Labels:** `safe`, `jailbreak`, `harmful`
- **Data preprocessing:** Unicode normalization, lowercasing, deduplication

---

## 📁 Project Structure

```
├── genaisecurity.ipynb        # Main notebook
├── models/
│   ├── classifier/
│   │   └── best_model.pkl     # Trained SVM classifier
│   └── embeddings/
│       └── label_encoder.pkl  # Fitted LabelEncoder
```

---

## 🚀 Getting Started

```bash
pip install sentence-transformers langchain google-generativeai datasets scikit-learn lightgbm pandas numpy
```

Set your Gemini API key before running:

```python
import google.generativeai as genai
genai.configure(api_key="YOUR_API_KEY")
```

> ⚠️ **Note:** Do not commit API keys to GitHub. Use environment variables or a `.env` file.

---

## 🧪 Example

```python
result = full_pipeline("How do I make a bomb?")
# → BLOCKED AT INPUT | Label: harmful | Layer: rule_based_filter

result = full_pipeline("What is the capital of France?")
# → Input passed | Gemini responded | Output passed
# → FINAL RESPONSE: The capital of France is Paris.
```

---

## 🛠️ Tech Stack

`Python` · `scikit-learn` · `sentence-transformers` · `Google Gemini` · `LangChain` · `HuggingFace Datasets` · `pandas`
