# RAG Pipeline: Cybercrime Act Prosecutions in Kenya

A Retrieval-Augmented Generation (RAG) question-answering system for exploring **prosecutions and constitutional challenges under Kenya's Computer Misuse and Cybercrimes Act, 2018 (Cap. 79C)**.

The system retrieves relevant passages from a curated collection of Kenyan legal documents and uses them as the evidence provided to a Large Language Model (LLM). The system is designed to answer questions grounded in the collected documents and provide source information for the retrieved evidence.

---

## Project Overview

Legal documents can be lengthy and difficult to search manually. This project demonstrates how Retrieval-Augmented Generation can be used to make a focused collection of Kenyan cybercrime-related legal documents easier to query.

The system follows a complete RAG pipeline:

```text
Legal Documents
      ↓
Document Collection
      ↓
Text Extraction
      ↓
Text Cleaning
      ↓
Chunking
      ↓
Embedding Generation
      ↓
ChromaDB Vector Store
      ↓
Semantic Retrieval
      ↓
LLM + Grounded Prompt
      ↓
Answer + Sources
```

The system is **document-grounded**: it is instructed to answer using the retrieved context rather than relying on unsupported information from the language model.

---

## Research Topic

**Topic:** Prosecutions and constitutional challenges under Kenya's Computer Misuse and Cybercrimes Act, 2018.

The dataset focuses on legal documents relevant to the prosecution, interpretation, and constitutional challenges associated with Kenya's cybercrime legislation.

### Example Questions

The system should be able to answer questions such as:

1. What constitutional issues have been raised in challenges involving the Computer Misuse and Cybercrimes Act?
2. What offence or provision of the Act was involved in a particular prosecution?
3. What did the court decide in a specific cybercrime-related case?

### Out-of-Scope Example

The system should not answer questions unrelated to the collected legal documents, such as:

> What is the current price of Bitcoin in Kenya?

If the required information cannot be found in the retrieved documents, the system should explicitly abstain rather than invent an answer.

---

## Objectives

The project aims to:

* Build a complete Retrieval-Augmented Generation pipeline.
* Collect and process a focused legal document collection.
* Extract text from PDF documents.
* Clean and normalize extracted text.
* Compare different chunking strategies.
* Generate semantic embeddings for document chunks.
* Store embeddings locally using an open-source vector database.
* Retrieve relevant legal passages for user questions.
* Integrate an LLM for grounded question answering.
* Provide source citations for generated answers.
* Implement relevance thresholds and abstention.
* Evaluate retrieval and answer quality using a predefined test set.

---

## Dataset

### Primary Source

**Kenya Law**

The documents were collected from publicly available legal materials relevant to the project topic.

The dataset consists of legal documents concerning prosecutions and constitutional challenges under Kenya's Computer Misuse and Cybercrimes Act, 2018.

### Source and Terms

The documents are sourced from publicly accessible Kenya Law materials. The project records the source URL, access date, document type, file size, and applicable source/terms information in the collection log.

The project uses the documents for educational and research purposes and does not claim ownership of the original legal materials.

### Dataset Structure

```text
data/
├── raw/
│   └── *.pdf
│
└── clean/
    └── cleaned documents
```

The notebook also generates collection statistics and a collection log documenting the source of each document.

---

## RAG Architecture

The system consists of the following stages.

### 1. Document Collection

Legal documents are downloaded and stored locally in `data/raw/`.

Each document is logged with:

* Filename
* Source URL
* Date accessed
* File type
* File size
* Licence/terms information

### 2. Text Extraction

Text is extracted from the collected PDF documents.

The pipeline records documents where extraction fails or produces empty text so that extraction problems do not silently enter the dataset.

### 3. Text Cleaning

Extracted text is normalized to improve retrieval quality.

The cleaning stage addresses issues such as:

* Unicode normalization
* Excess whitespace
* Repeated headers and footers
* Unnecessary formatting artefacts
* Duplicate text

Important legal structure is preserved where it may affect meaning.

### 4. Chunking

Documents are divided into smaller retrievable units.

Two chunking approaches are compared before selecting the final strategy.

Each chunk contains metadata such as:

```text
filename
chunk_id
page_number / position
```

This allows retrieved information to be traced back to the original document.

### 5. Embeddings

Each chunk is converted into a numerical vector using a sentence-transformer embedding model.

The embedding represents the semantic meaning of the chunk and allows the system to find passages that are semantically related to a user's question.

### 6. Vector Database

The embeddings and associated metadata are stored locally using **ChromaDB**.

The database is persisted to disk so that the embeddings do not need to be regenerated every time the notebook is executed.

### 7. Retrieval

When a user submits a question:

```text
User Question
      ↓
Question Embedding
      ↓
Similarity Search
      ↓
Top-k Relevant Chunks
```

The retrieved chunks are then passed to the LLM as contextual evidence.

### 8. Generation

The LLM receives:

* The user's question
* Retrieved legal context
* Grounding instructions
* Abstention rules
* Citation requirements
* Prompt-injection defenses

The model is instructed not to fabricate information when the answer cannot be supported by the retrieved documents.

---

## Technologies Used

| Component         | Technology                      |
| ----------------- | ------------------------------- |
| Language          | Python                          |
| Environment       | Jupyter Notebook / Google Colab |
| Document Format   | PDF                             |
| Text Extraction   | Python PDF processing tools     |
| Embedding Model   | `all-MiniLM-L6-v2`              |
| Vector Database   | ChromaDB                        |
| Similarity Search | Cosine similarity               |
| LLM               | Groq API                        |
| Data Processing   | Python                          |
| Evaluation        | Custom 10-question test set     |

API credentials are supplied through environment variables rather than being stored directly in the notebook.

---

## Prompting Strategy

The final RAG prompt incorporates multiple techniques to improve factual grounding:

* **Role prompting** — establishes the role of the legal-document QA system.
* **Clear task and constraints** — defines what the model is expected to do.
* **Delimited context** — separates retrieved documents from instructions.
* **Grounding and abstention** — requires the model to decline when evidence is insufficient.
* **Few-shot examples** — demonstrates expected answering and refusal behaviour.
* **Structured output** — requires an answer with source information.
* **Conflict handling** — instructs the model to identify conflicting or incomplete evidence.
* **Injection defense** — treats retrieved document content as data rather than instructions.

A baseline prompt is also compared against the final prompt to assess improvements in grounding and citation behaviour.

---

## Guardrails

The system includes several safeguards:

### Relevance Threshold

If retrieved documents do not meet the minimum similarity threshold, the system declines to provide an answer based on insufficient evidence.

### Abstention

The system is instructed to respond that the information is not available in the provided documents instead of generating unsupported claims.

### Source Display

Each response is accompanied by information about the retrieved evidence, including:

* Filename
* Page or chunk ID
* Relevant snippet
* Similarity score

### Error Handling

The interactive interface handles:

* Empty questions
* Missing context
* Retrieval failures
* API errors
* API timeouts
* Invalid input

### Legal Disclaimer

This project is an educational research system and is **not a substitute for professional legal advice**. Retrieved information should be checked against the original legal documents and authoritative sources before being relied upon for legal decisions.

---

## Evaluation

The system is evaluated using **10 questions prepared before evaluation**:

* 6 single-document factual questions
* 2 multi-document synthesis questions
* 2 out-of-scope questions

Evaluation considers:

* Whether the correct chunk was retrieved
* Whether the answer is correct
* Whether the answer is faithful to the retrieved sources
* Whether the system correctly abstains from out-of-scope questions

The complete evaluation table is included in the notebook.

---

## Repository Structure

```text
cybercrime-act-prosecutions-kenya-rag/
│
├── Njoroge_Dorcas_RAG_Assignment.ipynb
│
├── data/
│   ├── raw/
│   │   └── legal PDF documents
│   │
│   └── clean/
│       └── cleaned text
│
├── chroma_db/
│   └── persisted vector database
│
├── README.md
│
├── requirements.txt
│
└── .gitignore
```

The `.gitignore` file should prevent sensitive credentials and unnecessary generated files from being committed.

---

## Running the Project

### 1. Clone the Repository

```bash
git clone https://github.com/<your-username>/cybercrime-act-prosecutions-kenya-rag.git
cd cybercrime-act-prosecutions-kenya-rag
```

### 2. Create a Virtual Environment

```bash
python -m venv .venv
```

Activate it:

**Linux/macOS**

```bash
source .venv/bin/activate
```

**Windows**

```bash
.venv\Scripts\activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure the API Key

The project does **not** store API keys inside the notebook.

Set the required API credential as an environment variable before running the LLM section.

Example:

```bash
export GROQ_API_KEY="your_api_key"
```

Never commit the actual API key to GitHub.

### 5. Run the Notebook

Open:

```text
Njoroge_Dorcas_RAG_Assignment.ipynb
```

Then run the notebook from top to bottom.

The notebook contains the complete implementation from document collection and preprocessing through retrieval, generation, interactive questioning, and evaluation.

---

## Reproducibility

The project is designed so that the vector database can be persisted and loaded from disk.

After the initial embedding process, subsequent runs can reload the existing ChromaDB collection rather than embedding every document again.

The notebook also contains dataset statistics, processing logs, retrieval tests, and evaluation outputs to make the pipeline easier to inspect and reproduce.

---

## Limitations

This system has several limitations:

1. Its answers are limited by the documents included in the dataset.
2. Retrieval quality depends on the quality and completeness of the collected documents.
3. Semantic similarity does not guarantee that a retrieved passage is legally sufficient to answer a question.
4. PDF extraction can introduce formatting or text-order errors.
5. Chunking can separate legally related information across different chunks.
6. The LLM can still produce incorrect interpretations even when relevant context is retrieved.
7. The system should not be treated as a legal advice tool.
8. Legal information should always be verified against the original authoritative source.

---

## Academic Integrity

This project was developed as an individual RAG implementation for the AkiraChix DAS assignment.

The implementation, dataset selection, preprocessing decisions, chunking strategy, prompt design, evaluation questions, and analysis are specific to this project.

External libraries and frameworks are used where appropriate, but their purpose and role in the RAG pipeline are explained in the notebook.

---

## Author

**Dorcas Wanjiru**


**Project:** RAG Pipeline: Cybercrime Act Prosecutions in Kenya

---

## License

This repository contains an educational implementation and does not claim ownership of the underlying legal documents.

The licensing and terms applicable to individual source documents are documented in the notebook's collection log.
