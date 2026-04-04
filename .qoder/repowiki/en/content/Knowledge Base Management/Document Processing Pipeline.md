# Document Processing Pipeline

<cite>
**Referenced Files in This Document**
- [factory.py](file://backend/package/yuxi/plugins/parser/factory.py)
- [base.py](file://backend/package/yuxi/plugins/parser/base.py)
- [rapid_ocr.py](file://backend/package/yuxi/plugins/parser/rapid_ocr.py)
- [mineru.py](file://backend/package/yuxi/plugins/parser/mineru.py)
- [mineru_official.py](file://backend/package/yuxi/plugins/parser/mineru_official.py)
- [pp_structure_v3.py](file://backend/package/yuxi/plugins/parser/pp_structure_v3.py)
- [deepseek_ocr.py](file://backend/package/yuxi/plugins/parser/deepseek_ocr.py)
- [unified.py](file://backend/package/yuxi/plugins/parser/unified.py)
- [zip_utils.py](file://backend/package/yuxi/plugins/parser/zip_utils.py)
- [dispatcher.py](file://backend/package/yuxi/knowledge/chunking/ragflow_like/dispatcher.py)
- [presets.py](file://backend/package/yuxi/knowledge/chunking/ragflow_like/presets.py)
- [nlp.py](file://backend/package/yuxi/knowledge/chunking/ragflow_like/nlp.py)
- [general.py](file://backend/package/yuxi/knowledge/chunking/ragflow_like/parsers/general.py)
- [book.py](file://backend/package/yuxi/knowledge/chunking/ragflow_like/parsers/book.py)
- [qa.py](file://backend/package/yuxi/knowledge/chunking/ragflow_like/parsers/qa.py)
- [upload.py](file://backend/package/yuxi/knowledge/graphs/adapters/upload.py)
- [upload_graph_service.py](file://backend/package/yuxi/knowledge/graphs/upload_graph_service.py)
- [kb_utils.py](file://backend/package/yuxi/knowledge/utils/kb_utils.py)
- [url_fetcher.py](file://backend/package/yuxi/knowledge/utils/url_fetcher.py)
- [url_validator.py](file://backend/package/yuxi/knowledge/utils/url_validator.py)
- [filesystem_service.py](file://backend/package/yuxi/services/filesystem_service.py)
- [viewer_filesystem_service.py](file://backend/package/yuxi/services/viewer_filesystem_service.py)
- [knowledge_fs_service.py](file://backend/package/yuxi/services/knowledge_fs_service.py)
- [logger.py](file://backend/package/yuxi/utils/logging_config.py)
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
This document explains the end-to-end document processing pipeline used to transform uploaded documents into vector-ready chunks. It covers:
- Multi-format ingestion and OCR support for PDF, Word, Markdown, HTML, and images
- A factory-driven plugin architecture for pluggable document processors
- Chunking strategies: sentence/paragraph-based and semantic-aware hierarchical merging
- Text preprocessing, NLP utilities, and content validation
- Practical configuration examples, performance tuning, memory management, batching, and error handling

## Project Structure
The pipeline spans two primary areas:
- Plugins: OCR/document parsing plugins and a factory to manage them
- Knowledge chunking: preset-driven chunking engine with parsers and NLP utilities

```mermaid
graph TB
subgraph "Plugins: Parser Factory"
F["DocumentProcessorFactory<br/>factory.py"]
B["BaseDocumentProcessor<br/>base.py"]
R["RapidOCRParser<br/>rapid_ocr.py"]
M["MinerUParser<br/>mineru.py"]
MO["MinerUOfficialParser<br/>mineru_official.py"]
P["PPStructureV3Parser<br/>pp_structure_v3.py"]
D["DeepSeekOCRParser<br/>deepseek_ocr.py"]
U["Unified OCR Facade<br/>unified.py"]
Z["Zip Utilities<br/>zip_utils.py"]
end
subgraph "Knowledge: Chunking Engine"
DI["Dispatcher<br/>dispatcher.py"]
PR["Presets & Defaults<br/>presets.py"]
NL["NLP Utilities<br/>nlp.py"]
PG["General Parser<br/>general.py"]
PB["Book Parser<br/>book.py"]
PQ["QA Parser<br/>qa.py"]
end
subgraph "Graph Adapters"
GA["Upload Adapter<br/>upload.py"]
GS["Upload Graph Service<br/>upload_graph_service.py"]
end
subgraph "Utilities"
KB["KB Utils<br/>kb_utils.py"]
UF["URL Fetcher<br/>url_fetcher.py"]
UV["URL Validator<br/>url_validator.py"]
end
F --> B
F --> R
F --> M
F --> MO
F --> P
F --> D
F --> U
F --> Z
DI --> PR
DI --> PG
DI --> PB
DI --> PQ
PG --> NL
PB --> NL
PQ --> NL
DI --> NL
GA --> GS
GS --> DI
GS --> KB
GS --> UF
GS --> UV
```

**Diagram sources**
- [factory.py:18-148](file://backend/package/yuxi/plugins/parser/factory.py#L18-L148)
- [base.py:44-100](file://backend/package/yuxi/plugins/parser/base.py#L44-L100)
- [dispatcher.py:1-65](file://backend/package/yuxi/knowledge/chunking/ragflow_like/dispatcher.py#L1-L65)
- [presets.py:1-239](file://backend/package/yuxi/knowledge/chunking/ragflow_like/presets.py#L1-L239)
- [nlp.py:1-535](file://backend/package/yuxi/knowledge/chunking/ragflow_like/nlp.py#L1-L535)
- [general.py:1-47](file://backend/package/yuxi/knowledge/chunking/ragflow_like/parsers/general.py#L1-L47)
- [book.py:1-62](file://backend/package/yuxi/knowledge/chunking/ragflow_like/parsers/book.py#L1-L62)
- [qa.py:1-261](file://backend/package/yuxi/knowledge/chunking/ragflow_like/parsers/qa.py#L1-L261)
- [upload.py:1-200](file://backend/package/yuxi/knowledge/graphs/adapters/upload.py#L1-L200)
- [upload_graph_service.py:1-200](file://backend/package/yuxi/knowledge/graphs/upload_graph_service.py#L1-L200)
- [kb_utils.py:1-200](file://backend/package/yuxi/knowledge/utils/kb_utils.py#L1-L200)
- [url_fetcher.py:1-200](file://backend/package/yuxi/knowledge/utils/url_fetcher.py#L1-L200)
- [url_validator.py:1-200](file://backend/package/yuxi/knowledge/utils/url_validator.py#L1-L200)

**Section sources**
- [factory.py:18-148](file://backend/package/yuxi/plugins/parser/factory.py#L18-L148)
- [dispatcher.py:1-65](file://backend/package/yuxi/knowledge/chunking/ragflow_like/dispatcher.py#L1-L65)

## Core Components
- Plugin Factory: Creates and caches processor instances, exposes health checks, and provides convenience methods for processing files.
- Base Processor Interface: Defines the contract for all OCR/document parsers.
- Chunking Dispatcher: Routes markdown content to appropriate parsers based on preset IDs and builds chunk records.
- Preset Engine: Normalizes chunking presets, merges defaults and overrides, and resolves effective parameters.
- NLP Utilities: Token counting, heading detection, bullet categorization, hierarchical merging, and QA pair extraction.
- Parsers: General, Book, and QA-specific chunkers that leverage NLP utilities.

**Section sources**
- [factory.py:18-148](file://backend/package/yuxi/plugins/parser/factory.py#L18-L148)
- [base.py:44-100](file://backend/package/yuxi/plugins/parser/base.py#L44-L100)
- [dispatcher.py:32-65](file://backend/package/yuxi/knowledge/chunking/ragflow_like/dispatcher.py#L32-L65)
- [presets.py:79-214](file://backend/package/yuxi/knowledge/chunking/ragflow_like/presets.py#L79-L214)
- [nlp.py:49-535](file://backend/package/yuxi/knowledge/chunking/ragflow_like/nlp.py#L49-L535)
- [general.py:33-47](file://backend/package/yuxi/knowledge/chunking/ragflow_like/parsers/general.py#L33-L47)
- [book.py:26-62](file://backend/package/yuxi/knowledge/chunking/ragflow_like/parsers/book.py#L26-L62)
- [qa.py:213-261](file://backend/package/yuxi/knowledge/chunking/ragflow_like/parsers/qa.py#L213-L261)

## Architecture Overview
The pipeline transforms raw files into structured, chunked text suitable for embedding and retrieval.

```mermaid
sequenceDiagram
participant Client as "Client"
participant Adapter as "Upload Adapter<br/>upload.py"
participant Service as "Upload Graph Service<br/>upload_graph_service.py"
participant Factory as "DocumentProcessorFactory<br/>factory.py"
participant Proc as "Document Processor<br/>base.py impl"
participant Dispatcher as "Chunking Dispatcher<br/>dispatcher.py"
participant Presets as "Preset Resolver<br/>presets.py"
participant Parser as "Parser (General/Book/QA)<br/>general.py/book.py/qa.py"
participant NLP as "NLP Utilities<br/>nlp.py"
Client->>Adapter : Upload file
Adapter->>Service : Submit upload job
Service->>Factory : get_processor(type, kwargs)
Factory-->>Service : BaseDocumentProcessor instance
Service->>Proc : process_file(path, params)
Proc-->>Service : Extracted text (Markdown)
Service->>Dispatcher : chunk_markdown(md, file_id, filename, params)
Dispatcher->>Presets : resolve_chunk_processing_params(...)
Presets-->>Dispatcher : preset_id + parser_config
Dispatcher->>Parser : chunk_markdown(...)
Parser->>NLP : hierarchical_merge / naive_merge / QA helpers
NLP-->>Parser : Chunks
Parser-->>Dispatcher : List of chunks
Dispatcher-->>Service : List of chunk records
Service-->>Client : Chunk metadata for embedding
```

**Diagram sources**
- [upload.py:1-200](file://backend/package/yuxi/knowledge/graphs/adapters/upload.py#L1-L200)
- [upload_graph_service.py:1-200](file://backend/package/yuxi/knowledge/graphs/upload_graph_service.py#L1-L200)
- [factory.py:46-94](file://backend/package/yuxi/plugins/parser/factory.py#L46-L94)
- [dispatcher.py:49-65](file://backend/package/yuxi/knowledge/chunking/ragflow_like/dispatcher.py#L49-L65)
- [presets.py:161-213](file://backend/package/yuxi/knowledge/chunking/ragflow_like/presets.py#L161-L213)
- [general.py:33-47](file://backend/package/yuxi/knowledge/chunking/ragflow_like/parsers/general.py#L33-L47)
- [book.py:26-62](file://backend/package/yuxi/knowledge/chunking/ragflow_like/parsers/book.py#L26-L62)
- [qa.py:213-261](file://backend/package/yuxi/knowledge/chunking/ragflow_like/parsers/qa.py#L213-L261)
- [nlp.py:411-482](file://backend/package/yuxi/knowledge/chunking/ragflow_like/nlp.py#L411-L482)

## Detailed Component Analysis

### Plugin Factory and Base Interface
- Factory manages processor lifecycle via a cache keyed by type and parameters. It supports health checks and async health polling.
- Base interface defines the contract for OCR/document parsers, including health checks and supported file extensions.

```mermaid
classDiagram
class BaseDocumentProcessor {
+process_file(file_path, params) str
+check_health() dict
+get_service_name() str
+supports_file_type(ext) bool
+get_supported_extensions() str[]
}
class DocumentProcessorFactory {
+get_processor(processor_type, **kwargs) BaseDocumentProcessor
+process_file(processor_type, file_path, params) str
+check_health(processor_type) dict
+check_all_health() dict
+check_all_health_async() dict
+get_available_processors() str[]
+clear_cache() void
}
class RapidOCRParser
class MinerUParser
class MinerUOfficialParser
class PPStructureV3Parser
class DeepSeekOCRParser
DocumentProcessorFactory --> BaseDocumentProcessor : "creates"
BaseDocumentProcessor <|-- RapidOCRParser
BaseDocumentProcessor <|-- MinerUParser
BaseDocumentProcessor <|-- MinerUOfficialParser
BaseDocumentProcessor <|-- PPStructureV3Parser
BaseDocumentProcessor <|-- DeepSeekOCRParser
```

**Diagram sources**
- [base.py:44-100](file://backend/package/yuxi/plugins/parser/base.py#L44-L100)
- [factory.py:18-148](file://backend/package/yuxi/plugins/parser/factory.py#L18-L148)
- [rapid_ocr.py:1-200](file://backend/package/yuxi/plugins/parser/rapid_ocr.py#L1-L200)
- [mineru.py:1-200](file://backend/package/yuxi/plugins/parser/mineru.py#L1-L200)
- [mineru_official.py:1-200](file://backend/package/yuxi/plugins/parser/mineru_official.py#L1-L200)
- [pp_structure_v3.py:1-200](file://backend/package/yuxi/plugins/parser/pp_structure_v3.py#L1-L200)
- [deepseek_ocr.py:1-200](file://backend/package/yuxi/plugins/parser/deepseek_ocr.py#L1-L200)

**Section sources**
- [factory.py:18-148](file://backend/package/yuxi/plugins/parser/factory.py#L18-L148)
- [base.py:44-100](file://backend/package/yuxi/plugins/parser/base.py#L44-L100)

### Multi-format Support and OCR Capabilities
- Supported formats are determined by each processor’s extension list. The factory exposes a unified interface to select and reuse processors.
- OCR backends include local and cloud APIs:
  - Local OCR: RapidOCR
  - HTTP API: MinerU
  - Official cloud API: MinerU Official
  - PP-Structure-V3 layout understanding
  - DeepSeek-OCR via external API
- A unified facade consolidates OCR behavior for downstream chunking.

Practical notes:
- Choose processor based on accuracy vs latency trade-offs.
- Use cache clearing when changing OCR configurations.

**Section sources**
- [factory.py:22-28](file://backend/package/yuxi/plugins/parser/factory.py#L22-L28)
- [factory.py:139-148](file://backend/package/yuxi/plugins/parser/factory.py#L139-L148)
- [unified.py:1-200](file://backend/package/yuxi/plugins/parser/unified.py#L1-L200)

### Chunking Strategies and Presets
- Preset IDs: general, qa, book, laws.
- Defaults include token budgets, delimiters, overlapped percentages, and optional graph features.
- Parameter resolution merges knowledge base defaults, per-file overrides, and request-time parameters.

```mermaid
flowchart TD
Start(["Resolve Chunk Params"]) --> Normalize["Normalize preset id"]
Normalize --> Defaults["Load preset defaults"]
Defaults --> MergeKB["Merge KB parser config"]
MergeKB --> MergeFile["Merge file parser config"]
MergeFile --> MergeReq["Merge request parser config"]
MergeReq --> Legacy["Convert legacy params (chunk_size/overlap/delimiter)"]
Legacy --> Snapshot["Build snapshot and backward-compatible fields"]
Snapshot --> End(["Resolved params"])
```

**Diagram sources**
- [presets.py:161-213](file://backend/package/yuxi/knowledge/chunking/ragflow_like/presets.py#L161-L213)

**Section sources**
- [presets.py:8-66](file://backend/package/yuxi/knowledge/chunking/ragflow_like/presets.py#L8-L66)
- [presets.py:161-213](file://backend/package/yuxi/knowledge/chunking/ragflow_like/presets.py#L161-L213)

### Sentence-based and Paragraph-based Chunking
- General parser splits on custom delimiters or falls back to newline-based sections, then merges tokens within budget and optional overlap.
- Uses token counting and optional custom delimiter extraction.

```mermaid
flowchart TD
S(["Start"]) --> Split["Split markdown into sections"]
Split --> HasCustom{"Custom delimiter?"}
HasCustom --> |Yes| UseCustom["Split by custom delimiters"]
HasCustom --> |No| UseNewline["Split by newlines"]
UseCustom --> Merge["naive_merge with token budget and overlap"]
UseNewline --> Merge
Merge --> Out(["Chunks"])
```

**Diagram sources**
- [general.py:12-47](file://backend/package/yuxi/knowledge/chunking/ragflow_like/parsers/general.py#L12-L47)
- [nlp.py:411-482](file://backend/package/yuxi/knowledge/chunking/ragflow_like/nlp.py#L411-L482)

**Section sources**
- [general.py:33-47](file://backend/package/yuxi/knowledge/chunking/ragflow_like/parsers/general.py#L33-L47)
- [nlp.py:411-482](file://backend/package/yuxi/knowledge/chunking/ragflow_like/nlp.py#L411-L482)

### Semantic Chunking (Hierarchical Merging)
- Detects heading-like bullet patterns and languages, removes table of contents, normalizes colon-as-title cases, and performs hierarchical merging by inferred depth.
- Falls back to naive merge if no strong hierarchy detected.

```mermaid
flowchart TD
BS(["Begin Book Chunking"]) --> Sections["Iterate lines to sections"]
Sections --> RemoveTOC["Remove contents table"]
RemoveTOC --> ColonTitles["Treat colon lines as titles"]
ColonTitles --> Category["Classify bullet categories"]
Category --> HasHierarchy{"Bullet category found?"}
HasHierarchy --> |Yes| Hier["hierarchical_merge(depth=5)"]
HasHierarchy --> |No| Fallback["naive_merge fallback"]
Hier --> Done(["Chunks"])
Fallback --> Done
```

**Diagram sources**
- [book.py:26-62](file://backend/package/yuxi/knowledge/chunking/ragflow_like/parsers/book.py#L26-L62)
- [nlp.py:187-401](file://backend/package/yuxi/knowledge/chunking/ragflow_like/nlp.py#L187-L401)

**Section sources**
- [book.py:26-62](file://backend/package/yuxi/knowledge/chunking/ragflow_like/parsers/book.py#L26-L62)
- [nlp.py:187-401](file://backend/package/yuxi/knowledge/chunking/ragflow_like/nlp.py#L187-L401)

### QA-focused Chunking
- Extracts question-answer pairs from CSV/XLSX/Markdown/TXT/DOCX by:
  - Parsing markdown tables
  - Detecting heading-based Q&A
  - Prefix-based extraction (Q/A markers)
  - Delimiter-based splitting
  - Final deduplication and fallback pairing
- Outputs tab-separated QA chunks.

```mermaid
flowchart TD
QS(["Start QA Chunking"]) --> DetectSuffix["Detect file suffix"]
DetectSuffix --> Parse["Parse content by type"]
Parse --> Pairs["Assemble (Q,A) pairs"]
Pairs --> Dedupe["Deduplicate pairs"]
Dedupe --> Fallback{"Pairs found?"}
Fallback --> |No| PairFallback["Pair lines 2-by-2"]
Fallback --> |Yes| Build["Build tab-separated chunks"]
PairFallback --> Build
Build --> QOut(["QA Chunks"])
```

**Diagram sources**
- [qa.py:213-261](file://backend/package/yuxi/knowledge/chunking/ragflow_like/parsers/qa.py#L213-L261)

**Section sources**
- [qa.py:213-261](file://backend/package/yuxi/knowledge/chunking/ragflow_like/parsers/qa.py#L213-L261)

### Text Preprocessing and Validation
- Token estimation avoids heavy dependencies by splitting on words/CJK and digits.
- Heading heuristics and bullet classification improve semantic grouping.
- Content table removal and colon-as-title normalization reduce noise.
- QA-specific validations ensure non-empty, non-duplicated pairs.

**Section sources**
- [nlp.py:49-116](file://backend/package/yuxi/knowledge/chunking/ragflow_like/nlp.py#L49-L116)
- [nlp.py:187-244](file://backend/package/yuxi/knowledge/chunking/ragflow_like/nlp.py#L187-L244)
- [qa.py:195-211](file://backend/package/yuxi/knowledge/chunking/ragflow_like/parsers/qa.py#L195-L211)

### Practical Configuration Examples
- Select preset:
  - Use general for most documents; book for long-form texts; qa for FAQ-like content; laws for regulatory content.
- Configure chunk size and overlap:
  - Set token budget and overlap percentage; the resolver converts legacy fields and computes derived values.
- Delimiter customization:
  - Provide custom delimiters for general chunking; QA parsers auto-detect separators or rely on markdown headings.
- Enable/disable auxiliary features:
  - GraphRAG and RAPTOR toggles are preset-scoped; adjust via parser config merges.

**Section sources**
- [presets.py:161-213](file://backend/package/yuxi/knowledge/chunking/ragflow_like/presets.py#L161-L213)
- [general.py:33-47](file://backend/package/yuxi/knowledge/chunking/ragflow_like/parsers/general.py#L33-L47)
- [book.py:26-62](file://backend/package/yuxi/knowledge/chunking/ragflow_like/parsers/book.py#L26-L62)
- [qa.py:213-261](file://backend/package/yuxi/knowledge/chunking/ragflow_like/parsers/qa.py#L213-L261)

## Dependency Analysis
- Factory depends on Base interface and concrete processors; it caches instances to avoid repeated initialization.
- Dispatcher depends on presets and parsers; parsers depend on NLP utilities.
- Graph adapters orchestrate upload and chunking; utilities support URL fetching/validation and KB operations.

```mermaid
graph LR
Factory["factory.py"] --> Base["base.py"]
Factory --> Rapid["rapid_ocr.py"]
Factory --> Mineru["mineru.py"]
Factory --> MineruOff["mineru_official.py"]
Factory --> PP["pp_structure_v3.py"]
Factory --> Deep["deepseek_ocr.py"]
Factory --> Unified["unified.py"]
Dispatcher["dispatcher.py"] --> Presets["presets.py"]
Dispatcher --> General["general.py"]
Dispatcher --> Book["book.py"]
Dispatcher --> QA["qa.py"]
General --> NLP["nlp.py"]
Book --> NLP
QA --> NLP
Adapter["upload.py"] --> Service["upload_graph_service.py"]
Service --> Dispatcher
Service --> KB["kb_utils.py"]
Service --> URLF["url_fetcher.py"]
Service --> URLV["url_validator.py"]
```

**Diagram sources**
- [factory.py:18-148](file://backend/package/yuxi/plugins/parser/factory.py#L18-L148)
- [dispatcher.py:1-65](file://backend/package/yuxi/knowledge/chunking/ragflow_like/dispatcher.py#L1-L65)
- [presets.py:1-239](file://backend/package/yuxi/knowledge/chunking/ragflow_like/presets.py#L1-L239)
- [nlp.py:1-535](file://backend/package/yuxi/knowledge/chunking/ragflow_like/nlp.py#L1-L535)
- [upload.py:1-200](file://backend/package/yuxi/knowledge/graphs/adapters/upload.py#L1-L200)
- [upload_graph_service.py:1-200](file://backend/package/yuxi/knowledge/graphs/upload_graph_service.py#L1-L200)

**Section sources**
- [factory.py:18-148](file://backend/package/yuxi/plugins/parser/factory.py#L18-L148)
- [dispatcher.py:1-65](file://backend/package/yuxi/knowledge/chunking/ragflow_like/dispatcher.py#L1-L65)

## Performance Considerations
- Processor caching: Reuse cached instances to avoid repeated initialization overhead.
- Async health checks: Poll multiple OCR backends concurrently to detect availability quickly.
- Token budget tuning: Adjust chunk_token_num and overlapped_percent to balance recall and embedding costs.
- Memory management:
  - Prefer streaming or chunked processing for large files.
  - Avoid retaining intermediate markdown buffers longer than necessary.
- Batching:
  - Group uploads by similar presets to maximize cache hits.
  - Batch health checks and OCR requests where supported by the underlying services.
- Logging: Use structured logs to track OCR latency, chunk sizes, and failures.

[No sources needed since this section provides general guidance]

## Troubleshooting Guide
- Processor selection errors:
  - Ensure the requested processor type is registered in the factory mapping.
- Health check failures:
  - Inspect returned status and details; the factory wraps exceptions into health dictionaries.
- OCR failures:
  - Verify credentials and endpoints for cloud OCR services; retry with local OCR as fallback.
- Chunking anomalies:
  - Adjust delimiter and token budget; for books, ensure sufficient content after TOC removal.
  - For QA, confirm presence of Q/A markers or delimiter consistency.
- Logging:
  - Use the configured logger to capture processor messages and exceptions.

**Section sources**
- [factory.py:65-66](file://backend/package/yuxi/plugins/parser/factory.py#L65-L66)
- [factory.py:107-115](file://backend/package/yuxi/plugins/parser/factory.py#L107-L115)
- [base.py:11-42](file://backend/package/yuxi/plugins/parser/base.py#L11-L42)
- [logger.py:1-200](file://backend/package/yuxi/utils/logging_config.py#L1-L200)

## Conclusion
The pipeline integrates a flexible plugin architecture for OCR/document parsing with a robust, preset-driven chunking engine. By leveraging sentence/paragraph-based and semantic-aware hierarchical merging, it prepares diverse document types for embedding and retrieval. Proper configuration of chunking parameters, careful selection of OCR backends, and attention to memory and batching yield reliable, scalable processing.