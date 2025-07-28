---
title: "1-4 Memory & Basic Retrieval-Augmented Generation (RAG)"
---

# 🧠 Lesson 1-4: Memory & Basic RAG

!!! note "Learning Objectives"
    By the end of this lesson, you will be able to:

    - Distinguish between short-term and long-term agent memory
    - Explain how embeddings numerically represent text and capture semantic similarity
    - Configure and use a vector store (FAISS or Chroma) for embedding storage and retrieval
    - Describe the core workflow of Retrieval-Augmented Generation (RAG)
    - Build a simple document-question-answering agent grounded by RAG

---

## 💾 1. Memory in AI Agents

### 1.1 Short-Term Memory

!!! info "Purpose"
    Keep context and intermediate results within one session.

!!! example "Implementation"
    ```python
    short_term_memory = []
    short_term_memory.append({
        "step": "fetch_pricing_docs",
        "output": raw_data
    })
    ```

!!! tip "Use Cases"
    Maintain conversation turns, avoid repeat tool calls.

### 1.2 Long-Term Memory

!!! info "Purpose"
    Persist knowledge across sessions—documents, past interactions.

!!! info "Implementation"
    Vector database storing embeddings with metadata.

!!! example "Schema Example"
    | id          | embedding            | source         | chunk_text                 |
    |-------------|----------------------|----------------|----------------------------|
    | doc1_0      | [0.12, −0.03, …]     | policy.pdf     | "Employees may request…"   |

---

## 🔢 2. Numeric Representations of Text (Embeddings)

### 2.1 What Are Embeddings?

!!! info "Definition"
    Vectors produced by an embedding model (OpenAI, Cohere, Hugging Face) that map text to numeric space.

!!! tip "Key Feature"
    Capture semantic similarity: similar meanings yield nearby vectors.

### 2.2 Embedding Providers Comparison

!!! info "Provider Comparison"
    | Provider         | Model Variant       | Dimensionality | Speed         | Cost       |
    |------------------|---------------------|----------------|---------------|------------|
    | OpenAI           | text-embedding-3-small | 1,536          | Moderate      | Moderate   |
    | Cohere           | embed-multilingual  | 1,024          | Fast          | Low-Medium |
    | Hugging Face     | sentence-transformers | 768            | Variable      | Free/Custom|

### 2.3 Generating Embeddings

!!! example "Embedding Generation"
    ```python
    from langchain.embeddings import OpenAIEmbeddings, CohereEmbeddings
    embedder = OpenAIEmbeddings()       # or CohereEmbeddings(), HuggingFaceEmbeddings()
    vector = embedder.embed_query("What is our leave policy?")
    ```

---

## 🗄️ 3. Vector Stores: FAISS vs. Chroma

### 3.1 FAISS (Facebook AI Similarity Search)

!!! info "FAISS Characteristics"
    - **Type:** In-memory, fast prototype
    - **Use:** `FAISS.from_documents(texts, embedder)`

### 3.2 Chroma

!!! info "Chroma Characteristics"
    - **Type:** Persistent on-disk, metadata support

!!! example "Chroma Usage"
    ```python
    from langchain.vectorstores import Chroma
    store = Chroma.from_documents(
        texts, embedder,
        persist_directory="chroma_store",
        collection_name="company_docs"
    )
    store.persist()
    ```

!!! tip "Configuration Tips"
    - Choose similarity metric (cosine vs. Euclidean)
    - Select `k` (top-k results) based on chunk granularity

---

## 🔍 4. Retrieval-Augmented Generation (RAG) Workflow

!!! example "RAG Steps"
    1. **Embed Query**
       ```python
       query_vector = embedder.embed_query(user_question)
       ```
    2. **Retrieve Documents**
       ```python
       docs = vector_store.similarity_search_by_vector(
           query_vector, k=3
       )
       ```
    3. **Compose Prompt**
       ```python
       Context:
       {doc1}
       {doc2}
       {doc3}

       Question: {user_question}
       Answer:
       ```
    4. **LLM Completion**
       ```python
       from langchain.llms import OpenAI
       llm = OpenAI(model_name="gpt-4o")
       response = llm(context_and_question)
       ```

!!! tip "RAG Benefits"
    Grounded answers, reduced hallucinations, up-to-date knowledge without retraining.

---

## 🏗️ 5. Building a Basic RAG Agent

### 5.1 Ingest & Split Documents

!!! example "Document Processing"
    ```python
    from langchain.document_loaders import TextLoader
    from langchain.text_splitter import RecursiveCharacterTextSplitter

    loader = TextLoader("company_policy.pdf")
    docs = loader.load()
    splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
    texts = splitter.split_documents(docs)
    ```

### 5.2 Embed & Index

!!! example "Vector Store Setup"
    ```python
    from langchain.embeddings import OpenAIEmbeddings
    from langchain.vectorstores import FAISS

    embeddings = OpenAIEmbeddings()
    vector_store = FAISS.from_documents(texts, embeddings)
    ```

### 5.3 Construct RetrievalQA Chain

!!! example "QA Chain Implementation"
    ```python
    from langchain.chains import RetrievalQA
    from langchain.llms import OpenAI

    qa_chain = RetrievalQA.from_chain_type(
        llm=OpenAI(model_name="gpt-4o"),
        chain_type="stuff",
        retriever=vector_store.as_retriever(search_kwargs={"k": 3})
    )
    answer = qa_chain.run("What is our leave-of-absence policy?")
    print(answer)
    ```

---

## 💻 6. Mini-Project: Document Q&A Agent

!!! success "Document Q&A Challenge"
    **Build a Document-QA agent:**

    1. Ingest and chunk a PDF or text file.
    2. Embed and index with FAISS or Chroma.
    3. Instantiate `RetrievalQA` with `k=3`.
    4. Ask three distinct questions; evaluate answer relevance and accuracy.
    5. Adjust `k`, chunk size, or prompt formatting to improve grounding.

---

## ❓ 7. Self-Check Questions

!!! question "Knowledge Check"
    1. How do FAISS and Chroma differ in persistence and metadata support?
    2. Why does chunk overlap matter when splitting documents?
    3. What happens if you set `k` too low or too high in similarity search?
    4. Explain how embeddings capture semantic similarity across different providers.

---

## 🧭 Navigation

!!! success "Next Up"
    **[Lesson 1-5: Model APIs →](lesson-5.md)**

    Lesson 1-5 will evaluate **Model APIs**—weighing GPT-4o, Claude 4, and open-source models on cost, latency, and capability to choose the right "brain" for your agents.  