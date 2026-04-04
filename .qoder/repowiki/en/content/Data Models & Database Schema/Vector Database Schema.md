# Vector Database Schema

<cite>
**Referenced Files in This Document**
- [milvus.py](file://backend/package/yuxi/knowledge/implementations/milvus.py)
- [lightrag.py](file://backend/package/yuxi/knowledge/implementations/lightrag.py)
- [base.py](file://backend/package/yuxi/knowledge/base.py)
- [embed.py](file://backend/package/yuxi/models/embed.py)
- [dispatcher.py](file://backend/package/yuxi/knowledge/chunking/ragflow_like/dispatcher.py)
- [presets.py](file://backend/package/yuxi/knowledge/chunking/ragflow_like/presets.py)
- [kb_utils.py](file://backend/package/yuxi/knowledge/utils/kb_utils.py)
- [factory.py](file://backend/package/yuxi/knowledge/factory.py)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [Project Structure](#project-structure)
3. [Core Components](#core-components)
4. [Architecture Overview](#architecture-overview)
5. [Detailed Component Analysis](#detailed-component-analysis)
6. [Dependency Analysis](#dependency-analysis)
7. [Performance Considerations](#performance-considerations)
8. [Troubleshooting Guide](#troubleshooting-guide)
9. [Conclusion](#conclusion)

## Introduction
This document describes the vector database schema used in the Yuxi platform’s knowledge management system. It covers embedding storage models, vector indexing strategies, and similarity search implementations. It explains the schema design for storing document embeddings, metadata, and chunk information, and details the integration with Milvus and LightRAG vector databases, including collection schemas, index types, and query optimization techniques. It also documents embedding dimension requirements, similarity metrics, and search parameters, and provides examples of vector insertion, querying, and retrieval operations. Finally, it addresses performance optimization strategies, batch processing capabilities, and memory management considerations for large-scale vector operations.

## Project Structure
The vector database functionality is implemented as pluggable knowledge base backends:
- Milvus-backed vector storage for pure vector search
- LightRAG-backed hybrid storage integrating vector and graph components

```mermaid
graph TB
subgraph "Knowledge Management"
KBFactory["KnowledgeBaseFactory"]
KBBase["KnowledgeBase (abstract)"]
MilvusKB["MilvusKB"]
LightRagKB["LightRagKB"]
end
subgraph "Embeddings"
EmbedModel["BaseEmbeddingModel<br/>OllamaEmbedding<br/>OtherEmbedding"]
EmbedSelect["select_embedding_model()"]
end
subgraph "Chunking"
Presets["Chunking Presets & Defaults"]
Dispatcher["Markdown Chunk Dispatcher"]
end
subgraph "Vector Databases"
Milvus["Milvus Collection"]
LightRAG["LightRAG (Milvus + Neo4j)"]
end
KBFactory --> MilvusKB
KBFactory --> LightRagKB
KBBase --> MilvusKB
KBBase --> LightRagKB
MilvusKB --> EmbedSelect
LightRagKB --> EmbedSelect
EmbedSelect --> EmbedModel
MilvusKB --> Milvus
LightRagKB --> LightRAG
Dispatcher --> Presets
```

**Diagram sources**
- [factory.py:32-64](file://backend/package/yuxi/knowledge/factory.py#L32-L64)
- [base.py:46-190](file://backend/package/yuxi/knowledge/base.py#L46-L190)
- [milvus.py:24-164](file://backend/package/yuxi/knowledge/implementations/milvus.py#L24-L164)
- [lightrag.py:23-174](file://backend/package/yuxi/knowledge/implementations/lightrag.py#L23-L174)
- [embed.py:13-296](file://backend/package/yuxi/models/embed.py#L13-L296)
- [dispatcher.py:49-65](file://backend/package/yuxi/knowledge/chunking/ragflow_like/dispatcher.py#L49-L65)
- [presets.py:161-213](file://backend/package/yuxi/knowledge/chunking/ragflow_like/presets.py#L161-L213)

**Section sources**
- [factory.py:5-108](file://backend/package/yuxi/knowledge/factory.py#L5-L108)
- [base.py:46-190](file://backend/package/yuxi/knowledge/base.py#L46-L190)

## Core Components
- KnowledgeBase abstraction defines the unified interface for indexing, querying, and managing knowledge base operations.
- MilvusKB implements a pure vector search backend using Milvus collections with IVF_FLAT index and COSINE similarity.
- LightRagKB integrates LightRAG with Milvus for vector storage and Neo4j for graph storage, enabling hybrid retrieval.
- Embedding models encapsulate batch encoding and async embedding generation for vectorization.
- Chunking pipeline produces document chunks with deterministic IDs and metadata for vectorization.

**Section sources**
- [base.py:46-190](file://backend/package/yuxi/knowledge/base.py#L46-L190)
- [milvus.py:24-164](file://backend/package/yuxi/knowledge/implementations/milvus.py#L24-L164)
- [lightrag.py:23-174](file://backend/package/yuxi/knowledge/implementations/lightrag.py#L23-L174)
- [embed.py:13-296](file://backend/package/yuxi/models/embed.py#L13-L296)
- [dispatcher.py:9-29](file://backend/package/yuxi/knowledge/chunking/ragflow_like/dispatcher.py#L9-L29)

## Architecture Overview
The system orchestrates ingestion and retrieval across two backends:

```mermaid
sequenceDiagram
participant Client as "Client"
participant KB as "KnowledgeBase"
participant Milvus as "MilvusKB"
participant LR as "LightRagKB"
participant Embed as "Embedding Model"
participant DB as "Vector DB"
Client->>KB : index_file(db_id, file_id)
KB->>Milvus : _get_milvus_collection(db_id)
Milvus->>Embed : abatch_encode(texts)
Embed-->>Milvus : embeddings
Milvus->>DB : insert(entities)
DB-->>Milvus : ack
Milvus-->>KB : status=INDEXED
Client->>KB : aquery(query_text, db_id, kwargs)
alt Milvus mode
KB->>Milvus : search(query_embedding, filters)
Milvus->>DB : search(limit=recall_top_k)
DB-->>Milvus : hits
Milvus-->>KB : ranked chunks
else LightRAG mode
KB->>LR : aquery(query_text, params)
LR->>DB : hybrid retrieval (vector + graph)
DB-->>LR : results
LR-->>KB : structured response
end
KB-->>Client : retrieved chunks
```

**Diagram sources**
- [milvus.py:227-355](file://backend/package/yuxi/knowledge/implementations/milvus.py#L227-L355)
- [milvus.py:471-676](file://backend/package/yuxi/knowledge/implementations/milvus.py#L471-L676)
- [lightrag.py:305-421](file://backend/package/yuxi/knowledge/implementations/lightrag.py#L305-L421)
- [lightrag.py:526-600](file://backend/package/yuxi/knowledge/implementations/lightrag.py#L526-L600)

## Detailed Component Analysis

### Milvus Vector Storage Schema
MilvusKB creates a collection per database with a fixed schema optimized for fast similarity search:

- Fields
  - id: primary key (VARCHAR, max_length 100)
  - content: chunk text (VARCHAR, max_length 65535)
  - source: filename or identifier (VARCHAR, max_length 500)
  - chunk_id: unique chunk identifier (VARCHAR, max_length 100)
  - file_id: document identifier (VARCHAR, max_length 100)
  - chunk_index: order within the document (INT64)
  - embedding: FLOAT_VECTOR with dimension from embedding model

- Index
  - Index type: IVF_FLAT
  - Metric type: COSINE
  - Parameters: nlist (proportional to dataset size)

- Insertion
  - Chunks are generated via the chunking pipeline and inserted in bulk using Collection.insert.
  - Re-indexing deletes existing file chunks first, then inserts new vectors.

- Querying
  - Vector search with configurable top-k and similarity threshold.
  - Optional keyword search and hybrid modes.
  - Optional re-ranking with a reranker model.

```mermaid
classDiagram
class MilvusKB {
+string kb_type
+dict collections
+float chunk_size
+float chunk_overlap
+index_file(db_id, file_id, operator_id) dict
+update_content(db_id, file_ids, params) list[dict]
+aquery(query_text, db_id, agent_call, kwargs) list[dict]
+delete_file_chunks_only(db_id, file_id) void
+delete_file(db_id, file_id) void
+get_file_content(db_id, file_id) dict
}
class Collection {
+create_index(field, index_params) void
+insert(entities) void
+search(data, anns_field, param, limit, expr, output_fields) list
+query(expr, output_fields, limit) list
+delete(expr) void
+load() void
}
class EmbeddingModel {
+abatch_encode(messages, batch_size) list[list[float]]
+batch_encode(messages, batch_size) list[list[float]]
}
MilvusKB --> Collection : "manages"
MilvusKB --> EmbeddingModel : "uses"
```

**Diagram sources**
- [milvus.py:24-164](file://backend/package/yuxi/knowledge/implementations/milvus.py#L24-L164)
- [milvus.py:227-355](file://backend/package/yuxi/knowledge/implementations/milvus.py#L227-L355)
- [milvus.py:471-676](file://backend/package/yuxi/knowledge/implementations/milvus.py#L471-L676)
- [embed.py:13-296](file://backend/package/yuxi/models/embed.py#L13-L296)

**Section sources**
- [milvus.py:135-164](file://backend/package/yuxi/knowledge/implementations/milvus.py#L135-L164)
- [milvus.py:227-355](file://backend/package/yuxi/knowledge/implementations/milvus.py#L227-L355)
- [milvus.py:471-676](file://backend/package/yuxi/knowledge/implementations/milvus.py#L471-L676)

### LightRAG Hybrid Storage Schema
LightRagKB integrates vector and graph storage:
- Vector storage: MilvusVectorDBStorage (via LightRAG)
- Graph storage: Neo4JStorage (entities and relationships)
- KV storage: JsonKVStorage (text chunks and metadata)
- Document status storage: JsonDocStatusStorage

Key behaviors:
- Creates three Milvus collections per workspace: {db_id}_chunks, {db_id}_entities, {db_id}_relationships
- Supports retrieval modes: local, global, hybrid, naive, mix
- Exposes retrieval scopes: chunks, graph, all
- Ensures document processing status before retrieval

```mermaid
sequenceDiagram
participant Client as "Client"
participant LR as "LightRagKB"
participant RAG as "LightRAG"
participant VDB as "MilvusVectorDBStorage"
participant GDB as "Neo4JStorage"
Client->>LR : index_file(db_id, file_id)
LR->>RAG : ainsert(input, ids, file_paths, ...)
RAG->>VDB : persist chunks
RAG->>GDB : extract entities & relations
RAG-->>LR : processed status
LR-->>Client : status=INDEXED
Client->>LR : aquery(query_text, params)
LR->>RAG : aquery_data(query_text, QueryParam)
RAG->>VDB : vector search
RAG->>GDB : graph traversal
RAG-->>LR : mixed results
LR-->>Client : chunks/entities/relationships
```

**Diagram sources**
- [lightrag.py:136-174](file://backend/package/yuxi/knowledge/implementations/lightrag.py#L136-L174)
- [lightrag.py:305-421](file://backend/package/yuxi/knowledge/implementations/lightrag.py#L305-L421)
- [lightrag.py:526-600](file://backend/package/yuxi/knowledge/implementations/lightrag.py#L526-L600)

**Section sources**
- [lightrag.py:23-174](file://backend/package/yuxi/knowledge/implementations/lightrag.py#L23-L174)
- [lightrag.py:305-421](file://backend/package/yuxi/knowledge/implementations/lightrag.py#L305-L421)
- [lightrag.py:526-600](file://backend/package/yuxi/knowledge/implementations/lightrag.py#L526-L600)

### Embedding Models and Batch Processing
- BaseEmbeddingModel defines sync/async encoding and batch encoding with progress tracking.
- OllamaEmbedding and OtherEmbedding implement provider-specific embedding generation.
- select_embedding_model resolves provider and constructs the appropriate embedding model.
- Batch sizes are configurable and used during vectorization.

```mermaid
classDiagram
class BaseEmbeddingModel {
+int dimension
+int batch_size
+encode(message) list[list[float]]
+aencode(message) list[list[float]]
+batch_encode(messages, batch_size) list[list[float]]
+abatch_encode(messages, batch_size) list[list[float]]
+test_connection() (bool, str)
}
class OllamaEmbedding {
+encode(message) list[list[float]]
+aencode(message) list[list[float]]
}
class OtherEmbedding {
+encode(message) list[list[float]]
+aencode(message) list[list[float]]
}
BaseEmbeddingModel <|-- OllamaEmbedding
BaseEmbeddingModel <|-- OtherEmbedding
```

**Diagram sources**
- [embed.py:13-296](file://backend/package/yuxi/models/embed.py#L13-L296)

**Section sources**
- [embed.py:13-296](file://backend/package/yuxi/models/embed.py#L13-L296)

### Chunking Pipeline and Metadata
- Chunking presets define general, QA, book, and laws strategies with defaults and overrides.
- The dispatcher selects a parser based on preset ID and builds chunk records with deterministic IDs and metadata.
- Processing parameters are resolved from KB defaults, file params, and request params, with legacy compatibility.

```mermaid
flowchart TD
Start(["Start chunking"]) --> Resolve["Resolve preset and parser config"]
Resolve --> Dispatch["Dispatch to parser by preset"]
Dispatch --> Parse["Parse markdown content"]
Parse --> Merge["Merge sections and apply token limits"]
Merge --> Build["Build chunk records with IDs and metadata"]
Build --> End(["Return chunks"])
```

**Diagram sources**
- [presets.py:161-213](file://backend/package/yuxi/knowledge/chunking/ragflow_like/presets.py#L161-L213)
- [dispatcher.py:32-57](file://backend/package/yuxi/knowledge/chunking/ragflow_like/dispatcher.py#L32-L57)

**Section sources**
- [presets.py:161-213](file://backend/package/yuxi/knowledge/chunking/ragflow_like/presets.py#L161-L213)
- [dispatcher.py:9-29](file://backend/package/yuxi/knowledge/chunking/ragflow_like/dispatcher.py#L9-L29)

### Relationship Between Documents and Embeddings
- Each file is parsed to markdown and split into chunks.
- Each chunk receives a unique chunk_id and file_id, preserving ordering via chunk_index.
- Embeddings are computed for chunk contents and stored alongside metadata in the vector database.
- Retrieval returns chunk content, metadata (source, chunk_id, file_id, chunk_index), and similarity scores.

**Section sources**
- [dispatcher.py:9-29](file://backend/package/yuxi/knowledge/chunking/ragflow_like/dispatcher.py#L9-L29)
- [milvus.py:316-324](file://backend/package/yuxi/knowledge/implementations/milvus.py#L316-L324)
- [milvus.py:534-545](file://backend/package/yuxi/knowledge/implementations/milvus.py#L534-L545)

## Dependency Analysis
- MilvusKB depends on:
  - pymilvus for collection creation, indexing, search, and query
  - Embedding models for vectorization
  - Chunking pipeline for content preparation
- LightRagKB depends on:
  - LightRAG for ingestion and retrieval
  - MilvusVectorDBStorage for vector storage
  - Neo4j driver for graph storage
  - Embedding models for vectorization
- Both backends depend on the KnowledgeBase abstraction for lifecycle and metadata management.

```mermaid
graph TB
KB["KnowledgeBase (abstract)"]
MKB["MilvusKB"]
LKB["LightRagKB"]
PM["Embedding Models"]
CH["Chunking Pipeline"]
MV["Milvus (pymilvus)"]
LR["LightRAG"]
NG["Neo4j"]
KB --> MKB
KB --> LKB
MKB --> PM
MKB --> CH
MKB --> MV
LKB --> PM
LKB --> LR
LR --> MV
LR --> NG
```

**Diagram sources**
- [base.py:46-190](file://backend/package/yuxi/knowledge/base.py#L46-L190)
- [milvus.py:9-19](file://backend/package/yuxi/knowledge/implementations/milvus.py#L9-L19)
- [lightrag.py:6-11](file://backend/package/yuxi/knowledge/implementations/lightrag.py#L6-L11)

**Section sources**
- [base.py:46-190](file://backend/package/yuxi/knowledge/base.py#L46-L190)
- [milvus.py:9-19](file://backend/package/yuxi/knowledge/implementations/milvus.py#L9-L19)
- [lightrag.py:6-11](file://backend/package/yuxi/knowledge/implementations/lightrag.py#L6-L11)

## Performance Considerations
- Index selection
  - IVF_FLAT with COSINE metric is used for Milvus. Parameter nlist controls inverted list count; larger datasets typically require higher nlist for better recall.
- Search parameters
  - recall_top_k determines initial vector search results; final_top_k trims to desired output size.
  - similarity_threshold filters low-similarity matches.
  - nprobe controls the number of inverted lists to probe during search; higher increases accuracy but latency.
- Batch processing
  - Embedding batch sizes are configurable; batching reduces overhead and improves throughput.
  - Async batch encoding is supported to overlap network I/O with computation.
- Memory management
  - Collections are loaded into memory after creation; ensure adequate memory allocation for large collections.
  - Limit returned fields to reduce bandwidth and memory footprint.
- Re-indexing
  - Existing chunks are deleted before re-inserting to avoid duplication; this minimizes stale data but requires extra write operations.

[No sources needed since this section provides general guidance]

## Troubleshooting Guide
Common issues and resolutions:
- Connection failures to Milvus
  - Verify URI and token configuration; ensure database exists and is selected.
- Index mismatch
  - If embedding model name changes, the collection description is checked and recreated automatically.
- Query errors
  - Validate metric type and index parameters; ensure embeddings are computed with matching dimensionality.
- Reranking failures
  - If reranker fails, results fall back to vector scores; confirm reranker model availability and configuration.
- File deletion anomalies
  - delete_file_chunks_only removes only vector data while preserving metadata; delete_file removes both vector data and metadata entries.

**Section sources**
- [milvus.py:70-89](file://backend/package/yuxi/knowledge/implementations/milvus.py#L70-L89)
- [milvus.py:104-133](file://backend/package/yuxi/knowledge/implementations/milvus.py#L104-L133)
- [milvus.py:677-701](file://backend/package/yuxi/knowledge/implementations/milvus.py#L677-L701)
- [milvus.py:703-716](file://backend/package/yuxi/knowledge/implementations/milvus.py#L703-L716)

## Conclusion
The Yuxi platform implements a robust vector database schema supporting both pure vector search (Milvus) and hybrid retrieval (LightRAG). The schema emphasizes deterministic chunking, flexible embedding models, and configurable indexing and search parameters. By leveraging batch processing, async encoding, and careful index tuning, the system scales to large document sets while maintaining responsive query performance. The modular design allows seamless integration with external vector and graph databases, enabling future enhancements and migrations.