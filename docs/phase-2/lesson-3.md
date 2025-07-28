---
title: "2-3: Hybrid RAG & Context Management"
---

# 🔍 Lesson 2-3: Hybrid RAG & Context Management

!!! note "Learning Objectives"
    By the end this lesson, you'll be able to:

    - Explain the differences between vector, graph, and hybrid RAG approaches and when to use each
    - Design knowledge graphs and integrate them with vector stores for enhanced retrieval precision
    - Implement long-context window strategies for handling documents beyond traditional token limits
    - Optimize chunking strategies and context management for different document types and query patterns
    - Build hybrid retrieval systems that combine multiple retrieval methods for improved accuracy

---

## 🔄 1. RAG Approaches Comparison

### 1.1 Vector RAG

Vector RAG ("Vector Retrieval-Augmented Generation") uses dense embeddings to perform semantic similarity search over a document.

!!! info "Vector RAG Mechanism"
    1. Encode query and document chunks into high-dimensional vectors via an embedding model (e.g., OpenAI Embeddings, Cohere, Sentence Transformers).
    2. Index document vectors in a vector store (FAISS, Chroma, Pinecone).
    3. At query time, retrieve the top-k nearest neighbors by cosine (or Euclidean) distance.
    4. Supply retrieved text as context to an LLM for generation.

!!! tip "Vector RAG Strengths"
    - Grounded, up-to-date retrieval
    - Robust to lexical variation
    - Fast approximate nearest-neighbor lookup

!!! warning "Vector RAG Limitations"
    - May retrieve semantically related but contextually irrelevant passages ("false positives")
    - Performance depends on embedding quality and vector index configuration

### 1.2 Graph RAG

Graph RAG ("Graph-based Retrieval-Augmented Generation") leverages an explicit knowledge graph to perform relationship-based retrieval.

!!! info "Graph RAG Mechanism"
    1. Extract entities and relations from documents to construct a graph (nodes = entities, edges = relations, with metadata).
    2. Store the graph in a graph database (Neo4j, TigerGraph) or in-memory structure (NetworkX).
    3. At query time, translate the user question into a graph query (Cypher or custom traversal) to fetch subgraphs or paths that directly connect relevant entities.
    4. Convert the retrieved subgraph (node/edge text) into a prompt context for the LLM.

!!! tip "Graph RAG Strengths"
    - Precise retrieval of relational knowledge and multi-hop connections
    - Provenance and explainability via explicit graph paths

!!! warning "Graph RAG Limitations"
    - Graph construction requires reliable entity and relation extraction
    - Scalability challenges for very large or densely connected graphs

### 1.3 Hybrid RAG

Hybrid RAG combines vector and graph retrieval to leverage the strengths of both approaches.

!!! info "Hybrid RAG Mechanism"
    1. Perform a vector search to retrieve semantically related document chunks.
    2. Perform a graph traversal to retrieve relationship-centric passages or multi-hop paths.
    3. Fuse and rank results from both methods (e.g., weighted scoring or re-ranking).
    4. Present the combined context to the LLM for grounded generation.

!!! tip "Hybrid RAG Strengths"
    - Higher precision and recall than either approach alone
    - Balances semantic breadth (vector) with relational depth (graph)

!!! warning "Hybrid RAG Limitations"
    - Increased system complexity: maintenance of both vector store and graph database
    - Higher computational cost due to dual retrieval pipelines

### Performance Trade-offs

!!! info "RAG Approach Comparison"
    | Criterion                | Vector RAG                 | Graph RAG                      | Hybrid RAG                                                   |
    |--------------------------|----------------------------|--------------------------------|--------------------------------------------------------------|
    | Precision                | Moderate–High              | High (for relational queries)  | Very High (combines strengths)                               |
    | Recall                   | High                       | Moderate (limited by graph scope) | Very High (captures semantic and relational context)         |
    | Computational Cost       | Low–Moderate (ANN search)  | Moderate–High (graph traversal)| High (two retrieval pipelines + fusion logic)               |
    | Implementation Complexity| Low (vector store setup)    | Moderate–High (graph ETL & schema) | Very High (both vector and graph components)                 |
    | Explainability           | Low–Moderate (document snippets) | High (explicit graph paths)  | High (graph provenance + vector similarity scores)           |

!!! tip "Selection Guide"
    By understanding these distinctions, you can choose or design the RAG approach that best fits your domain's needs for precision, explainability, and scalability.

---

## 🕸️ 2. Knowledge Graph Construction for RAG

!!! note "Knowledge Graph Learning Objectives"
    By the end of this section, you will be able to:

    - Transform unstructured text into a knowledge graph via entity and relation extraction
    - Design a graph schema with appropriate node and edge types to represent domain knowledge
    - Ingest, store, and query a knowledge graph using popular graph databases or in-memory frameworks
    - Generate graph embeddings to integrate with vector-based retrieval for hybrid RAG

### 2.1 From Unstructured to Structured Knowledge

!!! example "Entity Extraction Process"
    1. **Entity Extraction**
       - Identify key concepts (entities) in text: people, organizations, products, dates, metrics.
       - **Techniques:**
         - Rule-based: regular expressions, gazetteers for known terms.
         - Statistical/NLP: Named-Entity Recognition (NER) models (spaCy, Hugging Face Transformers).
       - **Example:** From "Acme Corp raised $50 M in Series B on June 1, 2024," extract
         - Entity types: Organization("Acme Corp"), Money("$50 M"), Event("Series B"), Date("June 1, 2024").

    2. **Relation Extraction**
       - Determine relationships between entities: "Acme Corp ➔ raised ➔ $50 M" and "Series B ➔ occurred on ➔ June 1, 2024."
       - **Techniques:**
         - Pattern-based: dependency-parse patterns (e.g., " raised ").
         - Supervised models: relation-classification using labeled corpora (Stanford OpenIE, spaCy's Matcher).
       - **Output:** Triples of the form `(subject, predicate, object)`.

    3. **Graph Schema Design**
       - **Nodes:** Represent entities; include properties/metadata (e.g., node type, source document, confidence score).
       - **Edges:** Represent relations; include edge type and any attributes.
       - **Metadata:** Track provenance (document ID, paragraph index), extraction timestamp, and model version.

!!! example "Graph Schema Example"
    ```python
    # Node structure
    Node: 
      id: unique_identifier
      label: EntityType (e.g., "Organization")
      properties:
        name: string
        source: document_id
        confidence: float

    # Edge structure
    Edge:
      source: node_id
      target: node_id
      label: RelationType (e.g., "raised")
      properties:
        sentence: original_text
        confidence: float
    ```

---

## 🗄️ 3. Graph Database Integration

### 3.1 Neo4j Example (Cypher)

!!! example "Neo4j Setup and Ingestion"
    ```python
    from neo4j import GraphDatabase

    driver = GraphDatabase.driver("bolt://localhost:7687", auth=("neo4j","password"))

    def ingest_triplets(tx, triplets):
        for subj, rel, obj, props in triplets:
            tx.run(
              """
              MERGE (a:Entity {name:$subj, type:$subj_type})
              MERGE (b:Entity {name:$obj, type:$obj_type})
              MERGE (a)-[r:RELATION {type:$rel}]->(b)
              SET r += $props
              """,
              subj=subj, subj_type=props["subj_type"],
              obj=obj, obj_type=props["obj_type"],
              rel=rel, props=props
            )

    with driver.session() as session:
        session.write_transaction(ingest_triplets, triplets_list)
    ```

!!! example "Neo4j Querying"
    ```cypher
    MATCH (a:Entity {name:"Acme Corp"})-[r:RELATION]->(b)
    RETURN b.name AS object, r.type AS relation
    ```

### 3.2 NetworkX In-Memory Graph

!!! example "NetworkX Implementation"
    ```python
    import networkx as nx

    G = nx.DiGraph()

    # Add nodes and edges
    for subj, rel, obj, props in triplets_list:
        G.add_node(subj, type=props["subj_type"], source=props["source"])
        G.add_node(obj, type=props["obj_type"], source=props["source"])
        G.add_edge(subj, obj, type=rel, **props)

    # Query: neighbors
    successors = list(G.successors("Acme Corp"))
    ```

---

## 🧠 4. Graph Embedding Generation

!!! info "Graph Embedding Purpose"
    Convert graph structure into dense vectors capturing relational patterns, enabling hybrid semantic-graph retrieval.

!!! example "Node2Vec Implementation"
    ```python
    from node2vec import Node2Vec

    # Initialize Node2Vec on NetworkX graph
    node2vec = Node2Vec(G, dimensions=128, walk_length=30, num_walks=200, workers=4)
    model = node2vec.fit(window=10, min_count=1)

    # Get embedding for a node
    acme_vec = model.wv["Acme Corp"]
    ```

!!! tip "Integration Strategy"
    - Index node embeddings in FAISS/Chroma alongside document chunk embeddings.
    - At query time, perform a vector search over both document and graph embedding spaces; fuse results.

---

## 📏 5. Long-Context Window Strategies

### 5.1 Hierarchical Summarization

!!! info "Hierarchical Approach"
    - Summarize long documents at multiple levels (paragraph → section → document)
    - Store summaries in vector store for retrieval
    - Use summaries as context for detailed sections

### 5.2 Selective Context Extraction

!!! example "Context Selection Strategy"
    ```python
    def selective_context(query, document_chunks, max_tokens=8000):
        # 1. Retrieve relevant chunks
        relevant_chunks = vector_search(query, document_chunks)
        
        # 2. Rank by relevance score
        ranked_chunks = rank_by_relevance(query, relevant_chunks)
        
        # 3. Select chunks up to token limit
        selected_context = []
        token_count = 0
        
        for chunk in ranked_chunks:
            if token_count + chunk.token_count <= max_tokens:
                selected_context.append(chunk)
                token_count += chunk.token_count
            else:
                break
        
        return selected_context
    ```

---

## 💻 6. Mini-Project: Hybrid RAG System

!!! success "Hybrid RAG Challenge"
    **Build a hybrid RAG system that combines:**

    1. **Vector Search:** Use FAISS for semantic similarity
    2. **Graph Retrieval:** Use NetworkX for relationship-based search
    3. **Fusion Logic:** Combine results with weighted scoring
    4. **Evaluation:** Compare hybrid vs. single-method performance

!!! example "Fusion Implementation"
    ```python
    def hybrid_retrieval(query, vector_results, graph_results, alpha=0.7):
        # Weighted fusion of results
        fused_scores = {}
        
        for doc_id, score in vector_results:
            fused_scores[doc_id] = alpha * score
        
        for doc_id, score in graph_results:
            if doc_id in fused_scores:
                fused_scores[doc_id] += (1 - alpha) * score
            else:
                fused_scores[doc_id] = (1 - alpha) * score
        
        # Return top results
        return sorted(fused_scores.items(), key=lambda x: x[1], reverse=True)
    ```

---

## ❓ 7. Self-Check Questions

!!! question "Knowledge Check"
    1. When would you choose Graph RAG over Vector RAG?
    2. How do you handle documents that exceed token limits in RAG systems?
    3. What are the trade-offs between Neo4j and NetworkX for graph storage?
    4. How would you implement result fusion in a hybrid RAG system?

---

## 🧭 Navigation

!!! success "Next Up"
    **[Lesson 2-4: Evaluation & Tracing →](lesson-4.md)**

    Learn about evaluation metrics and tracing for agent systems.

---

## 🧭 Navigation

!!! success "Phase 2 Complete!"
    **[Phase 3: Multi-Agent Orchestration →](../phase-3/)**

    Phase 3 explores **Multi-Agent Orchestration**—coordinating multiple agents, managing conversations, and building complex multi-agent systems.