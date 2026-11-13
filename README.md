# 🩺 Medical Knowledge Assistant using RAG

A Retrieval-Augmented Generation (RAG) system that gives doctors fast, source-cited answers to clinical questions, grounded in the Merck Manual of Diagnosis & Therapy instead of the model's memory.

## 📌 Business Problem

Healthcare professionals deal with information overload: thousands of pages of references, and little time to search them, especially in emergencies. A general-purpose LLM can answer quickly, but it can't point to a trusted source and may hallucinate, which is a serious risk in medicine.

This project builds an assistant for clinicians that:
- Answers only from a trusted medical manual
- Cites the chapter and pages behind each answer
- Says "Sorry, this is out of my knowledge base" when the manual doesn't cover the question

## 🗂️ Data

**The Merck Manual of Diagnosis & Therapy (19th Edition)**
- 4,114 pages across 23 sections, loaded from a PDF
- Split into **8,156 chunks** (512 tokens each, 20-token overlap)

> The PDF is not included in this repo. Add your own copy as `medical_diagnosis_manual.pdf` to run the notebook.

## 🏗️ Architecture

```
Merck Manual PDF
      ↓
PyMuPDF loader → Recursive text splitter (512 tokens, 20 overlap)
      ↓
gte-large embeddings (1024-dim) → ChromaDB vector store
      ↓
User question → Retriever (top-k similarity search)
      ↓
System prompt + retrieved context + question → Mistral-7B-Instruct
      ↓
Grounded answer with source citation
```

## 🛠️ Tech Stack

- **LLM:** Mistral-7B-Instruct-v0.2 (GGUF, Q6_K) running locally via `llama-cpp-python`
- **Embeddings:** `thenlper/gte-large` (Sentence Transformers)
- **Vector DB:** ChromaDB
- **Orchestration:** LangChain (PDF loading, chunking, retrieval)
- **PDF parsing:** PyMuPDF
- **Environment:** Jupyter Notebook with GPU

## 🔍 Approach

1. **Baseline LLM:** asked five clinical questions with no context
2. **Prompt engineering:** added a medical system prompt to improve reliability
3. **RAG pipeline:** chunked and embedded the manual, stored it in ChromaDB and wired up a retriever
4. **Strict prompting:** told the model to use only the retrieved context, cite sources and refuse out-of-scope questions
5. **Parameter tuning:** tested temperature, top-p, top-k, max tokens and number of retrieved chunks (k)
6. **Evaluation:** used LLM-as-a-judge to score **groundedness** and **relevance** on a 1 to 5 scale

## 📊 Results

**Baseline vs RAG:** the plain LLM gave generic textbook answers with no sources. The RAG version returned answers drawn from the Merck Manual with chapter and page citations, e.g. for appendicitis:

> *Appendicitis is typically treated via surgical removal, either open or laparoscopic appendectomy.*
> *Source: The Merck Manual of Diagnosis & Therapy, 19th Edition, Chapter 11. Acute Abdomen & Surgical Gastroenterology, pages 163-164.*

**Evaluation highlights:**
- Strong groundedness on most queries, with answers staying within the retrieved context (appendicitis and hair loss scored 5/5 or "followed completely")
- Retrieval quality was the main weak spot (see limitations)

## 💡 Key Takeaways

- Grounding answers in a trusted source with citations makes the output far more usable for clinicians than a standalone LLM.
- Strict prompt rules (context-only answers, mandatory citations, a refusal fallback) are what keep hallucination in check.
- Retrieval quality limits answer quality. If the wrong chunks come back, even a well-prompted model gives the wrong answer.

## ⚠️ Limitations

- **Retrieval errors:** for the question about a fractured leg during a hiking trip, the retriever pulled frostbite and cold-injury passages, so the answer partly mixed in rewarming advice. This is a good example of why retrieval has to be evaluated, not just generation.
- **Self-evaluation bias:** the same Mistral model judged its own answers, so scores are likely optimistic. A stronger or independent judge would be more reliable.
- **Chunking:** 20-token overlap is small for sequential medical protocols. Higher overlap or section-aware chunking would help.
- **Not a clinical tool:** this is a prototype and shouldn't be used for real patient decisions.

## 🚀 Next Steps

- Switch to a medical-domain embedding model (e.g. BioBERT or PubMedBERT-based) for better retrieval
- Add re-ranking and larger chunk overlap
- Keep the knowledge base current with newer guidelines and research
- Add a doctor feedback loop and support for other specialties and languages

## 📁 Repository Structure

```
├── Medical_Knowledge_Assistant_using_RAG.ipynb
├── medical_diagnosis_manual.pdf   # add your own copy
└── README.md
```

## 👤 Author

**Karan**
[LinkedIn](https://www.linkedin.com/in/karanmeher/) · [GitHub](https://github.com/karan-meher)
