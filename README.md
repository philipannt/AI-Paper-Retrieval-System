# 🤖 AI Paper Retrieval System

This repository contains a **hybrid content retrieval system** for **AI research papers**, combining **Word2Vec semantic embeddings** and **LDA topic modeling** to support **semantic + topic-based search** at the paragraph level.

---

## 🧠 Overview

Traditional keyword search often fails to capture *semantic meaning* or *latent topics* in AI research literature.  
This system bridges that gap by representing each paragraph as a **hybrid vector** of:
- **Word2Vec** (semantic similarity)
- **LDA** (topic distribution)

Queries are converted into the same hybrid space, and results are ranked by **hybrid cosine similarity**.

---

## 🧩 System Architecture

```
TEI XML Papers
     ↓
MySQL Database
     ↓
Text Chunking & Tokenization
     ↓
Word2Vec + LDA Training
     ↓
Hybrid Vector Generation
     ↓
Semantic & Topic-based Retrieval
```

---

## ⚙️ Features

✅ Parse and extract structured content from **TEI XML AI papers**  
✅ Store metadata, authors, and paragraph chunks in **MySQL**  
✅ Train **Word2Vec** (semantic) and **LDA** (topic) models  
✅ Build **hybrid representations** for each text chunk  
✅ Query by natural language → ranked hybrid similarity results  
✅ Evaluate retrieval quality (Precision@K, Recall@K, MAP, MRR, NDCG)  
✅ Easy to extend with **transformer-based embeddings (e.g., BERT)**  

---

## 🧰 Tech Stack

| Component | Technology |
|------------|-------------|
| **Language** | Python 3.9+ |
| **Database** | MySQL 8.0 |
| **Libraries** | `gensim`, `nltk`, `sqlalchemy`, `pandas`, `numpy`, `sklearn`, `tqdm`, `python-dotenv`, `lxml` |

---

## 🧪 How to Use

1. **Clone the repository**
   ```bash
   git clone https://github.com/philippannt/ai-paper-retrieval-system.git
   cd ai-paper-retrieval-system
   ```

2. **Configure database in `.env`**
   ```bash
   MYSQL_HOST=localhost
   MYSQL_PORT=3306
   MYSQL_USER=root
   MYSQL_PASSWORD=yourpassword
   MYSQL_DB=papers
   ```

3. **Prepare your dataset**
   - Place TEI XML files of AI papers in the `data-full/` directory.

4. **Run the notebook**
   - Open `cursor_seg.ipynb` and execute step-by-step:
     - Parse TEI XML → insert to MySQL  
     - Train Word2Vec + LDA models  
     - Generate hybrid chunk vectors  
     - Query using natural language (e.g., “transformer model optimization”)

---

## 📈 Example Queries

```python
query_papers_hybrid("deep reinforcement learning", top_k=10, alpha=0.6)
query_papers_hybrid("attention mechanism in transformers", top_k=5)
```

---

## 🧮 Evaluation

Use the provided test JSON (`models/sample_test_set.json`) or create your own:
```json
{
  "test_queries": [
    {
      "query": "neural network optimization",
      "relevant_paper_ids": [1, 2],
      "relevant_chunk_ids": [10, 11, 12]
    }
  ]
}
```

Evaluate:
```python
evaluate_system("models/sample_test_set.json", top_k=10, alpha=0.5)
```

---

## 🚀 Future Enhancements

- Integrate **Sentence-BERT** or **OpenAI Embeddings**  
- Implement **ANN (FAISS/HNSW)** for large-scale retrieval  
- Support **Vietnamese / multilingual** papers  
- Build a **Streamlit Web UI** for live search  

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

---

## 👨‍💻 Author

**PhilippanNT**  
Educational Researcher & AI Enthusiast  
💡 *Developing intelligent retrieval systems for academic knowledge.*
