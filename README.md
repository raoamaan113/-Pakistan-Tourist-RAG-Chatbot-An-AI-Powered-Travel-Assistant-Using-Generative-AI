# 🇵🇰 Pakistan Tourist RAG Chatbot

A multilingual **Retrieval-Augmented Generation (RAG)** chatbot for exploring tourist destinations, hotels, restaurants, food, weather, and other travel-related information across Pakistan.

The project combines **multilingual embeddings, FAISS vector search, BM25 keyword retrieval, category-aware routing, MMR re-ranking, and a local LLM** to produce grounded tourism answers in **English, Urdu, and Roman Urdu**.

---

## 📌 Project Overview

This project was developed as a Generative AI project and follows a complete RAG pipeline:

**Dataset → Cleaning → Structured Chunking → Embeddings → FAISS → Query Understanding → Hybrid Retrieval → Re-ranking → LLM Generation → Gradio Interface**

The chatbot is designed to answer tourism questions using the retrieved knowledge base rather than relying only on the language model's general knowledge.

### Project Information

- **Project:** Pakistan Tourist RAG Chatbot
- **Phase:** Generative AI Project — Phase 02
- **Instructor:** Dr. Tafseer Ahmed
- **Members:** Rao Muhammad Amaan, Hamza Nisar
- **Primary notebook:** `Pakistan_Tourist_Chatbot_(Final_Submission)(1).ipynb`

---

## ✨ Key Features

### 🌍 Multilingual Queries

The retrieval pipeline supports:

- English
- Urdu script
- Roman Urdu

Examples:

```text
Best places to visit in Lahore

لاہور میں گھومنے کی بہترین جگہیں کون سی ہیں؟

Lahore mein ghumne ki jagah batao
```

Roman Urdu normalization handles common spelling variations such as:

```text
khana / khanay / khaana / khane
zaiqa / zaika / zaeqa
rehayish / rehayesh / rehaish
```

---

### 🔎 Hybrid Retrieval

The chatbot uses both semantic and lexical retrieval:

```text
Final Score = 0.6 × FAISS Similarity + 0.4 × BM25 Score
```

#### FAISS

Used for semantic similarity between the user query and destination embeddings.

#### BM25

Helps improve exact keyword and place-name matching, especially for queries such as:

```text
Badshahi Mosque Lahore
Lahore Fort Shahi Qila
Ansoo Lake Kaghan
Mohenjo Daro Sindh
```

---

### 🧠 Category-Aware Retrieval

The system identifies the likely category of a query before retrieval.

Supported categories include tourism-related categories such as:

- Hotel
- Restaurant
- Food
- Tourist Attraction
- Weather
- Historical places
- and other categories present in the dataset

This helps prevent category mixing. For example:

```text
"Best restaurants in Lahore"
```

should prioritize restaurant information instead of returning hotel records simply because both contain the word "Lahore".

---

### 🎯 MMR Re-ranking

Maximal Marginal Relevance (**MMR**) is used to improve result diversity.

Instead of returning several nearly identical chunks, the system attempts to return useful and diverse results.

The retrieval function uses:

```python
mmr_lambda = 0.7
```

---

### 🤖 Local LLM Generation

The primary generation model is:

```text
Qwen/Qwen2-1.5B-Instruct
```

The notebook also compares alternative generation models including:

- `google/gemma-2-2b-it`
- `google/mt5-small`

The final chatbot uses a context-grounded system prompt:

> Answer only using the provided context and reply in the same language as the question.

This is intended to reduce unsupported answers and hallucinations.

---

## 🏗️ System Architecture

```text
                     ┌─────────────────────┐
                     │   User Question     │
                     └──────────┬──────────┘
                                │
                                ▼
                  ┌──────────────────────────┐
                  │ Query Preprocessing      │
                  │ English / Urdu / Roman   │
                  │ Urdu Normalization       │
                  └────────────┬─────────────┘
                               │
                               ▼
                  ┌──────────────────────────┐
                  │    Intent Detection      │
                  │ Category + Location +    │
                  │ Tags / Filters            │
                  └────────────┬─────────────┘
                               │
                    ┌──────────┴──────────┐
                    ▼                     ▼
             ┌──────────────┐      ┌──────────────┐
             │    FAISS     │      │     BM25     │
             │  Semantic    │      │   Keyword    │
             │  Retrieval   │      │  Retrieval   │
             └──────┬───────┘      └──────┬───────┘
                    └──────────┬──────────┘
                               ▼
                  ┌──────────────────────────┐
                  │    Hybrid Scoring        │
                  │ 0.6 FAISS + 0.4 BM25     │
                  └────────────┬─────────────┘
                               │
                               ▼
                  ┌──────────────────────────┐
                  │      MMR Re-ranking      │
                  │   Diverse Top-K Results  │
                  └────────────┬─────────────┘
                               │
                               ▼
                  ┌──────────────────────────┐
                  │ Context Construction     │
                  │ Destination + Location   │
                  │ Season + Description     │
                  │ Tags                     │
                  └────────────┬─────────────┘
                               │
                               ▼
                  ┌──────────────────────────┐
                  │ Qwen2-1.5B-Instruct      │
                  │ Local LLM Generation     │
                  └────────────┬─────────────┘
                               │
                               ▼
                  ┌──────────────────────────┐
                  │   Final Tourism Answer   │
                  │   + Retrieved Sources    │
                  └──────────────────────────┘
```

---

## 📊 Dataset Processing

The notebook loads:

```text
Pakistan_Tourist_Dataset 2.xlsx
```

The preprocessing pipeline:

1. Removes rows without a title.
2. Removes unwanted rows containing photo credits, URLs, or social-media references.
3. Removes duplicate destinations based on title.
4. Resets the dataframe index.
5. Fills missing metadata fields.
6. Builds one structured chunk per destination.

Each chunk follows a consistent structure:

```text
[CATEGORY]
[TITLE]
[CITY]
[PROVINCE]
[DESCRIPTION]
[TAGS]
```

Keeping the fields structured helps the embedding model distinguish information such as category, location, and tags.

---

## 🧬 Embedding Model

The primary embedding model is:

```text
intfloat/multilingual-e5-large
```

The notebook uses normalized embeddings and FAISS Inner Product similarity.

The embedding dimension used by the notebook is:

```text
1024
```

Additional retrieval models were also tested:

```text
sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2
BAAI/bge-small-en-v1.5
intfloat/multilingual-e5-small
```

---

## 🗄️ Vector Database

The project uses **FAISS** for vector similarity search.

The main index is:

```text
faiss_index.bin
```

Destination metadata/chunks are stored in:

```text
chunks_metadata.pkl
```

The notebook also saves:

```text
embeddings.npy
```

---

## 🤖 Generation Models Compared

The notebook evaluates multiple local generation models:

| Model | Purpose |
|---|---|
| `Qwen/Qwen2-1.5B-Instruct` | Primary/final generation model |
| `google/gemma-2-2b-it` | Generation comparison |
| `google/mt5-small` | Lightweight multilingual alternative |

The final application uses Qwen:

```python
MODEL_NAME = "Qwen/Qwen2-1.5B-Instruct"
```

No external LLM API is required for the local generation pipeline.

---

## 🧪 Prompt Engineering

Four prompt styles are explored:

1. **Standard**
2. **Enthusiastic**
3. **Formal**
4. **Concise**

The prompts are tested across:

- English
- Urdu
- Roman Urdu

The final prompt instructs the model to:

- Use the retrieved context.
- Reply in the user's language.
- Mention relevant places and locations.
- Mention visiting seasons where available.
- State when the context does not contain enough information.

---

## 📈 Evaluation

The project includes both qualitative and retrieval-focused quantitative evaluation.

### Qualitative Evaluation

The notebook checks:

- Answer relevance
- Answer coherence
- Context adherence
- Possible hallucination
- Helpfulness and specificity

Example evaluation queries include:

```text
What are the best places for hiking in Pakistan?

مری میں کہاں ٹھہرنا چاہیے؟

Tell me about the food culture in Karachi.
```

### Quantitative Evaluation

The notebook implements:

- Precision@3
- Recall@3
- F1@3
- MRR@3
- Precision@5
- Recall@5
- F1@5
- MRR@5

The ground-truth evaluation set contains queries in:

- English
- Urdu
- Roman Urdu

> The evaluation scores are generated by the notebook at runtime. The README does not hard-code scores so that the reported results always correspond to the current dataset and index.

---

## 💬 Example Queries

### English

```text
What are the best historical places to visit in Lahore?
```

### Urdu

```text
مری میں کچھ ہوٹلوں کی تجاویز دیں
```

### Roman Urdu

```text
Lahore mein ghumne ki jagah batao
```

Other examples:

```text
Best hotels in Murree

What street food should I try in Karachi?

Pakistan mein mashoor qillay

Karachi mein khanay

Best places to visit in Swat

Historical forts in Punjab
```

---

## 🖥️ Gradio Interface

The project includes a Gradio-based web interface.

The interface provides:

- Chat-style interaction
- User and assistant messages
- Retrieved source display
- Destination category
- City and province
- Custom CSS styling

The interface is designed to make the RAG system accessible through a browser rather than only through a notebook input loop.

---

## 🚀 Running the Notebook

### 1. Clone the Repository

```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPOSITORY.git
cd YOUR_REPOSITORY
```

### 2. Install Dependencies

```bash
pip install transformers sentence-transformers faiss-cpu torch accelerate bitsandbytes pandas numpy rank-bm25 openpyxl gradio
```

### 3. Prepare the Dataset

Place:

```text
Pakistan_Tourist_Dataset 2.xlsx
```

in the project working directory.

### 4. Run the Notebook

Open:

```text
Pakistan_Tourist_Chatbot_(Final_Submission)(1).ipynb
```

using Google Colab or Jupyter Notebook.

### 5. Execute the Cells

Run the notebook sequentially so that:

```text
Dataset
   ↓
Cleaning
   ↓
Chunks
   ↓
Embeddings
   ↓
FAISS
   ↓
BM25
   ↓
Retrieval
   ↓
LLM
   ↓
Gradio
```

are initialized in the correct order.

---

## 📦 Deployment

The notebook contains a Docker-based deployment pipeline for **Google Cloud Run**.

Deployment artifacts include:

```text
requirements.txt
app.py
Dockerfile
faiss_index.bin
chunks_metadata.pkl
```

### Docker Build

```bash
gcloud builds submit --tag gcr.io/YOUR_PROJECT_ID/pakistan-chatbot .
```

### Deploy to Cloud Run

```bash
gcloud run deploy pakistan-chatbot \
    --image gcr.io/YOUR_PROJECT_ID/pakistan-chatbot \
    --platform managed \
    --region YOUR_REGION \
    --allow-unauthenticated \
    --memory 4Gi \
    --cpu 2 \
    --timeout 300 \
    --port 7860
```

Replace:

```text
YOUR_PROJECT_ID
YOUR_REGION
```

with your Google Cloud configuration.

> Running a local LLM such as Qwen2-1.5B on Cloud Run can require substantial memory and CPU. Resource settings may need adjustment depending on the deployment environment.

---

## 📁 Suggested GitHub Repository Structure

```text
Pakistan-Tourist-RAG-Chatbot/
│
├── README.md
├── Pakistan_Tourist_Chatbot_(Final_Submission)(1).ipynb
│
├── app.py
├── requirements.txt
├── Dockerfile
│
├── faiss_index.bin
├── chunks_metadata.pkl
│
├── data/
│   └── Pakistan_Tourist_Dataset 2.xlsx
│
└── assets/
    └── screenshots/
```

### Important

Large model files, datasets, and generated artifacts may be better stored using Git LFS, cloud storage, or release artifacts rather than normal Git commits.

---

## 🔐 Privacy & API Requirements

The final local generation pipeline does not require an OpenAI API key.

The project uses Hugging Face models through the `transformers` and `sentence-transformers` libraries.

Some model-loading experiments, particularly Gemma, may require Hugging Face authentication depending on the model's access requirements.

---

## 🔮 Future Improvements

Potential improvements identified in the project include:

- Expand the multilingual evaluation dataset.
- Add more tourism destinations and categories.
- Improve Roman Urdu normalization.
- Add stronger exact-name retrieval.
- Compare additional multilingual embedding models.
- Improve generation evaluation using a larger labeled benchmark.
- Add automated faithfulness/hallucination evaluation.
- Add conversational memory.
- Add filtering by province, city, category, and season.
- Add map-based destination discovery.
- Add recommendation ranking based on user preferences.
- Optimize deployment for CPU inference.
- Consider category-specific FAISS indexes as the dataset grows substantially.

---

## 🛠️ Technology Stack

| Technology | Role |
|---|---|
| Python | Core development |
| Pandas | Data processing |
| NumPy | Numerical operations |
| Sentence Transformers | Text embeddings |
| `multilingual-e5-large` | Primary embedding model |
| FAISS | Vector similarity search |
| BM25 | Keyword retrieval |
| Transformers | Local LLM inference |
| Qwen2-1.5B-Instruct | Primary generation model |
| Gradio | Web interface |
| PyTorch | Model inference |
| Docker | Containerization |
| Google Cloud Run | Deployment target |

---

## 🎓 Learning Outcomes

This project demonstrates practical experience with:

- Retrieval-Augmented Generation (RAG)
- Vector databases
- Semantic search
- Hybrid retrieval
- BM25
- FAISS
- Multilingual NLP
- Roman Urdu normalization
- Prompt engineering
- Local LLM inference
- Model comparison
- Information retrieval evaluation
- MMR re-ranking
- Gradio application development
- Docker deployment
- Google Cloud Run

---

## 👨‍💻 Contributors

### Rao Muhammad Amaan
Computer Science Student  
Generative AI / Machine Learning / RAG

### Hamza Nisar
Project Member

---

## 📚 Project Status

**Status:** Final Submission / Academic Project

The notebook contains the complete experimental workflow from dataset preparation through retrieval, generation, evaluation, interface development, and deployment preparation.

---

## ⭐ Acknowledgement

This project was developed as part of the Generative AI coursework under the guidance of:

**Dr. Tafseer Ahmed**

---

## 📄 License

This repository is intended primarily for academic and educational purposes.

Before redistributing the dataset or model artifacts, verify the licensing terms of the original data and third-party models.
