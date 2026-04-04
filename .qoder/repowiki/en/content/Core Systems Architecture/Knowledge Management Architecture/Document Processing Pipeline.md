# Document Processing Pipeline

<cite>
**Referenced Files in This Document**
- [unified.py](file://backend/package/yuxi/plugins/parser/unified.py)
- [factory.py](file://backend/package/yuxi/plugins/parser/factory.py)
- [base.py](file://backend/package/yuxi/plugins/parser/base.py)
- [__init__.py](file://backend/package/yuxi/plugins/parser/__init__.py)
- [rapid_ocr.py](file://backend/package/yuxi/plugins/parser/rapid_ocr.py)
- [mineru.py](file://backend/package/yuxi/plugins/parser/mineru.py)
- [mineru_official.py](file://backend/package/yuxi/plugins/parser/mineru_official.py)
- [pp_structure_v3.py](file://backend/package/yuxi/plugins/parser/pp_structure_v3.py)
- [deepseek_ocr.py](file://backend/package/yuxi/plugins/parser/deepseek_ocr.py)
- [zip_utils.py](file://backend/package/yuxi/plugins/parser/zip_utils.py)
- [url_fetcher.py](file://backend/package/yuxi/knowledge/utils/url_fetcher.py)
- [url_validator.py](file://backend/package/yuxi/knowledge/utils/url_validator.py)
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
10. [Appendices](#appendices)

## Introduction
This document describes the end-to-end document processing pipeline that ingests files and URLs, parses them into structured content, and converts them into Markdown. It covers:
- The ingestion workflow from upload through parsing to Markdown conversion
- The unified parser architecture supporting multiple formats (PDF, Word, Markdown, images, spreadsheets, presentations, HTML, CSV, JSON, ZIP)
- The factory pattern and plugin system enabling extensible document processing
- URL fetching and validation mechanisms for web-based content
- Error handling, content sanitization, and metadata extraction
- Examples of supported formats, custom parser development, and performance optimization techniques for large documents

## Project Structure
The document processing pipeline is primarily implemented under the parser plugin subsystem and integrated with utilities for ZIP handling, MinIO storage, and URL fetching/validation.

```mermaid
graph TB
subgraph "Parser Plugins"
U["unified.py"]
F["factory.py"]
B["base.py"]
R["rapid_ocr.py"]
M["mineru.py"]
MO["mineru_official.py"]
P["pp_structure_v3.py"]
D["deepseek_ocr.py"]
Z["zip_utils.py"]
end
subgraph "Utilities"
UF["url_fetcher.py"]
UV["url_validator.py"]
end
U --> F
U --> Z
U --> R
U --> M
U --> MO
U --> P
U --> D
UF --> UV
```

**Diagram sources**
- [unified.py:1-445](file://backend/package/yuxi/plugins/parser/unified.py#L1-L445)
- [factory.py:1-148](file://backend/package/yuxi/plugins/parser/factory.py#L1-L148)
- [base.py:1-100](file://backend/package/yuxi/plugins/parser/base.py#L1-L100)
- [rapid_ocr.py:1-254](file://backend/package/yuxi/plugins/parser/rapid_ocr.py#L1-L254)
- [mineru.py:1-240](file://backend/package/yuxi/plugins/parser/mineru.py#L1-L240)
- [mineru_official.py:1-366](file://backend/package/yuxi/plugins/parser/mineru_official.py#L1-L366)
- [pp_structure_v3.py:1-275](file://backend/package/yuxi/plugins/parser/pp_structure_v3.py#L1-L275)
- [deepseek_ocr.py:1-182](file://backend/package/yuxi/plugins/parser/deepseek_ocr.py#L1-L182)
- [zip_utils.py:1-211](file://backend/package/yuxi/plugins/parser/zip_utils.py#L1-L211)
- [url_fetcher.py:1-143](file://backend/package/yuxi/knowledge/utils/url_fetcher.py#L1-L143)
- [url_validator.py:1-88](file://backend/package/yuxi/knowledge/utils/url_validator.py#L1-L88)

**Section sources**
- [unified.py:1-445](file://backend/package/yuxi/plugins/parser/unified.py#L1-L445)
- [factory.py:1-148](file://backend/package/yuxi/plugins/parser/factory.py#L1-L148)
- [base.py:1-100](file://backend/package/yuxi/plugins/parser/base.py#L1-L100)
- [zip_utils.py:1-211](file://backend/package/yuxi/plugins/parser/zip_utils.py#L1-L211)
- [url_fetcher.py:1-143](file://backend/package/yuxi/knowledge/utils/url_fetcher.py#L1-L143)
- [url_validator.py:1-88](file://backend/package/yuxi/knowledge/utils/url_validator.py#L1-L88)

## Core Components
- Unified parser facade: Orchestrates format-specific parsing and Markdown conversion, handles MinIO downloads, ZIP extraction, and image uploads.
- Factory and base: Defines the processor interface and a factory that instantiates OCR/document processors with caching and health checks.
- Plugin processors: Implement OCR and document understanding for PDFs, images, and office formats via local models or external APIs.
- ZIP utilities: Extract Markdown and images from ZIP archives returned by external services, upload images to MinIO, and normalize links.
- URL utilities: Securely fetch and validate web content with size limits, content-type restrictions, and private IP blocking.

**Section sources**
- [unified.py:417-445](file://backend/package/yuxi/plugins/parser/unified.py#L417-L445)
- [factory.py:18-148](file://backend/package/yuxi/plugins/parser/factory.py#L18-L148)
- [base.py:44-100](file://backend/package/yuxi/plugins/parser/base.py#L44-L100)
- [zip_utils.py:21-112](file://backend/package/yuxi/plugins/parser/zip_utils.py#L21-L112)
- [url_fetcher.py:37-143](file://backend/package/yuxi/knowledge/utils/url_fetcher.py#L37-L143)

## Architecture Overview
The pipeline follows a layered design:
- Entry point: A synchronous or asynchronous parser facade that routes to format-specific handlers.
- Format dispatch: Based on file extension, the unified parser selects the appropriate handler (PDF, DOCX/PPTX/XLSX via Docling, images via OCR, HTML/CSV/JSON/ZIP via dedicated logic).
- OCR and external services: For PDFs/images, the factory selects an OCR processor (RapidOCR, MinerU, PP-Structure-V3, DeepSeek-OCR) depending on configuration.
- Storage and artifacts: Images extracted during Docling or OCR flows are uploaded to MinIO; ZIP archives are processed to extract Markdown and images.
- Output: The final Markdown string and optional artifacts are returned.

```mermaid
sequenceDiagram
participant Client as "Caller"
participant Parser as "Parser.aparse()"
participant Unified as "parse_source_to_markdown()"
participant Handler as "_process_file_to_markdown_core()"
participant Factory as "DocumentProcessorFactory"
participant Proc as "BaseDocumentProcessor"
Client->>Parser : aparse(source, params)
Parser->>Unified : parse_source_to_markdown(source, params)
Unified->>Handler : _process_file_to_markdown_core(source, params)
alt PDF/image with OCR
Handler->>Factory : get_processor(type)
Factory-->>Handler : processor instance
Handler->>Proc : process_file(path, params)
Proc-->>Handler : text/markdown
else Office/HTML/CSV/JSON/ZIP
Handler-->>Unified : text/markdown
end
Unified-->>Parser : MarkdownParseResult
Parser-->>Client : markdown string
```

**Diagram sources**
- [unified.py:417-445](file://backend/package/yuxi/plugins/parser/unified.py#L417-L445)
- [unified.py:270-414](file://backend/package/yuxi/plugins/parser/unified.py#L270-L414)
- [factory.py:46-94](file://backend/package/yuxi/plugins/parser/factory.py#L46-L94)
- [base.py:44-83](file://backend/package/yuxi/plugins/parser/base.py#L44-L83)

## Detailed Component Analysis

### Unified Parser Facade and Dispatcher
Responsibilities:
- Determine file type and route to the correct handler
- Handle MinIO URLs by downloading to a temporary file
- Convert various formats to Markdown
- Manage artifacts (e.g., ZIP metadata, image info)
- Provide synchronous and asynchronous entry points

Key behaviors:
- Supported extensions include text, Markdown, Word, HTML, Excel, PowerPoint, PDF, images, CSV, JSON, and ZIP.
- PDF handling supports disabling OCR or delegating to OCR processors via the factory.
- Image handling requires OCR; otherwise raises a validation error.
- Office formats (DOCX/PPTX/XLSX) are converted using Docling with a fallback to python-docx for DOC.
- ZIP archives are extracted asynchronously; images are uploaded to MinIO and Markdown links are normalized.

```mermaid
flowchart TD
Start(["Entry: parse_source_to_markdown"]) --> CheckMinio["Is MinIO URL?"]
CheckMinio --> |Yes| Download["Download to temp file"]
CheckMinio --> |No| UsePath["Use provided path"]
Download --> SetPath["Set actual_file_path"]
UsePath --> Ext["Detect extension"]
SetPath --> Ext
Ext --> PDF{"pdf?"}
PDF --> |Yes| OCRParam["Read enable_ocr param"]
OCRParam --> OCRChoice{"OCR enabled?"}
OCRChoice --> |No| PyPDF["PyPDFLoader"]
OCRChoice --> |Yes| Factory["Factory.process_file()"]
PDF --> |No| Office{"docx/pptx/xlsx?"}
Office --> |Yes| Docling["Docling convert"]
Office --> |No| Other{"txt/md/doc/html/csv/json/jpg/png/..."}
Other --> |txt/md| ReadText["Read UTF-8"]
Other --> |doc| Unstructured["Unstructured loader"]
Other --> |html| MD["markdownify"]
Other --> |csv| ToMD["DataFrame.to_markdown()"]
Other --> |json| AsCode["JSON fenced code"]
Other --> |jpg/png/...| ImgOCR["Factory.process_file()"]
Other --> |zip| Zip["process_zip_file()"]
Docling --> Merge["Assemble Markdown"]
PyPDF --> Merge
ReadText --> Merge
Unstructured --> Merge
MD --> Merge
ToMD --> Merge
AsCode --> Merge
ImgOCR --> Merge
Zip --> Merge
Merge --> Artifacts["Attach artifacts if any"]
Artifacts --> Return(["Return MarkdownParseResult"])
```

**Diagram sources**
- [unified.py:417-445](file://backend/package/yuxi/plugins/parser/unified.py#L417-L445)
- [unified.py:270-414](file://backend/package/yuxi/plugins/parser/unified.py#L270-L414)
- [zip_utils.py:21-76](file://backend/package/yuxi/plugins/parser/zip_utils.py#L21-L76)

**Section sources**
- [unified.py:25-44](file://backend/package/yuxi/plugins/parser/unified.py#L25-L44)
- [unified.py:417-445](file://backend/package/yuxi/plugins/parser/unified.py#L417-L445)
- [unified.py:270-414](file://backend/package/yuxi/plugins/parser/unified.py#L270-L414)

### Factory Pattern and Plugin System
Responsibilities:
- Provide a single interface to instantiate OCR/document processors
- Cache instances to avoid repeated initialization
- Expose health checks for all processors
- Offer convenience methods for processing files and checking health

Implementation highlights:
- Processor types mapped to module and class names
- Cache keyed by processor type plus parameter representation
- Health checks return structured status with details
- Async health checks leverage threads to keep the event loop responsive

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
BaseDocumentProcessor <|-- RapidOCRParser
BaseDocumentProcessor <|-- MinerUParser
BaseDocumentProcessor <|-- MinerUOfficialParser
BaseDocumentProcessor <|-- PPStructureV3Parser
BaseDocumentProcessor <|-- DeepSeekOCRParser
DocumentProcessorFactory --> BaseDocumentProcessor : "creates/instantiates"
```

**Diagram sources**
- [base.py:44-100](file://backend/package/yuxi/plugins/parser/base.py#L44-L100)
- [factory.py:18-148](file://backend/package/yuxi/plugins/parser/factory.py#L18-L148)
- [rapid_ocr.py:21-254](file://backend/package/yuxi/plugins/parser/rapid_ocr.py#L21-L254)
- [mineru.py:19-240](file://backend/package/yuxi/plugins/parser/mineru.py#L19-L240)
- [mineru_official.py:21-366](file://backend/package/yuxi/plugins/parser/mineru_official.py#L21-L366)
- [pp_structure_v3.py:19-275](file://backend/package/yuxi/plugins/parser/pp_structure_v3.py#L19-L275)
- [deepseek_ocr.py:21-182](file://backend/package/yuxi/plugins/parser/deepseek_ocr.py#L21-L182)

**Section sources**
- [factory.py:18-148](file://backend/package/yuxi/plugins/parser/factory.py#L18-L148)
- [base.py:44-100](file://backend/package/yuxi/plugins/parser/base.py#L44-L100)

### OCR and Document Understanding Plugins
- RapidOCRParser: Local ONNXRuntime-based OCR for PDFs and images; streams PDF pages to minimize memory usage; supports configurable detection threshold.
- MinerUParser: HTTP API-based document understanding with ZIP-returning responses; supports multiple backends and languages; returns Markdown via ZIP processing.
- MinerUOfficialParser: Cloud API with batch upload flow; polls for results; extracts Markdown from ZIP or falls back to first available text.
- PPStructureV3Parser: Layout parsing API for PDFs and images; returns per-page Markdown; aggregates statistics.
- DeepSeekOCRParser: Uses SiliconFlow API with DeepSeek-OCR model; converts PDF pages to images and sends to API; cleans special tags from output.

```mermaid
sequenceDiagram
participant Caller as "Caller"
participant Factory as "DocumentProcessorFactory"
participant Proc as "Selected OCR Parser"
participant API as "External Service"
Caller->>Factory : get_processor(type, **kwargs)
Factory-->>Caller : processor instance (cached)
Caller->>Proc : process_file(path, params)
alt Local model (RapidOCR)
Proc-->>Caller : text
else HTTP API (MinerU/PP-Structure-V3/DeepSeek)
Proc->>API : request (file or URL/Base64)
API-->>Proc : response (ZIP or JSON)
Proc-->>Caller : text/markdown
end
```

**Diagram sources**
- [factory.py:46-94](file://backend/package/yuxi/plugins/parser/factory.py#L46-L94)
- [rapid_ocr.py:178-253](file://backend/package/yuxi/plugins/parser/rapid_ocr.py#L178-L253)
- [mineru.py:93-239](file://backend/package/yuxi/plugins/parser/mineru.py#L93-L239)
- [mineru_official.py:96-198](file://backend/package/yuxi/plugins/parser/mineru_official.py#L96-L198)
- [pp_structure_v3.py:196-274](file://backend/package/yuxi/plugins/parser/pp_structure_v3.py#L196-L274)
- [deepseek_ocr.py:80-114](file://backend/package/yuxi/plugins/parser/deepseek_ocr.py#L80-L114)

**Section sources**
- [rapid_ocr.py:21-254](file://backend/package/yuxi/plugins/parser/rapid_ocr.py#L21-L254)
- [mineru.py:19-240](file://backend/package/yuxi/plugins/parser/mineru.py#L19-L240)
- [mineru_official.py:21-366](file://backend/package/yuxi/plugins/parser/mineru_official.py#L21-L366)
- [pp_structure_v3.py:19-275](file://backend/package/yuxi/plugins/parser/pp_structure_v3.py#L19-L275)
- [deepseek_ocr.py:21-182](file://backend/package/yuxi/plugins/parser/deepseek_ocr.py#L21-L182)

### ZIP Handling and Image Upload
- Extract Markdown from ZIP archives; locate images directory; upload images to MinIO; replace image links in Markdown with MinIO URLs.
- Compute content hash for deduplication or change detection.
- Provide both async and sync entry points for ZIP processing.

```mermaid
flowchart TD
ZStart["process_zip_file(zip_path)"] --> Validate["Validate entries (no .., no leading slash)"]
Validate --> FindMD["Find .md file (prefer full.md)"]
FindMD --> ReadMD["Read Markdown content"]
ReadMD --> FindImgDir["Locate images directory"]
FindImgDir --> |Found| Upload["Upload images to MinIO"]
FindImgDir --> |Not Found| Hash["Compute content hash"]
Upload --> Replace["Replace image links"]
Replace --> Hash
Hash --> ZReturn["Return {markdown, hash, images_info}"]
```

**Diagram sources**
- [zip_utils.py:21-76](file://backend/package/yuxi/plugins/parser/zip_utils.py#L21-L76)
- [zip_utils.py:132-179](file://backend/package/yuxi/plugins/parser/zip_utils.py#L132-L179)
- [zip_utils.py:182-211](file://backend/package/yuxi/plugins/parser/zip_utils.py#L182-L211)

**Section sources**
- [zip_utils.py:21-112](file://backend/package/yuxi/plugins/parser/zip_utils.py#L21-L112)

### URL Fetching and Validation
- URL parsing is controlled by a whitelist environment variable; domains can use wildcards.
- Fetching enforces size limits, allowed content types, and blocks private IPs.
- Redirects are followed with validation at each hop.

```mermaid
flowchart TD
UStart["fetch_url_content(url)"] --> Enabled{"URL parsing enabled?"}
Enabled --> |No| RaiseDisabled["Raise error: disabled"]
Enabled --> |Yes| Validate["validate_url(url)"]
Validate --> Valid{"Valid?"}
Valid --> |No| RaiseInvalid["Raise error: invalid"]
Valid --> Private["Resolve hostname and check private IP"]
Private --> IsPrivate{"Private IP?"}
IsPrivate --> |Yes| RaisePrivate["Raise error: private IP"]
IsPrivate --> |No| Request["HTTP GET with headers"]
Request --> Redirect{"3xx?"}
Redirect --> |Yes| NextURL["Follow redirect with validation"]
Redirect --> |No| CheckCT["Check Content-Type"]
CheckCT --> Allowed{"Allowed?"}
Allowed --> |No| RaiseCT["Raise error: unsupported Content-Type"]
Allowed --> Size["Stream and enforce size limit"]
Size --> UReturn["Return (content, final_url)"]
```

**Diagram sources**
- [url_fetcher.py:37-143](file://backend/package/yuxi/knowledge/utils/url_fetcher.py#L37-L143)
- [url_validator.py:19-71](file://backend/package/yuxi/knowledge/utils/url_validator.py#L19-L71)

**Section sources**
- [url_fetcher.py:1-143](file://backend/package/yuxi/knowledge/utils/url_fetcher.py#L1-L143)
- [url_validator.py:1-88](file://backend/package/yuxi/knowledge/utils/url_validator.py#L1-L88)

## Dependency Analysis
- Unified parser depends on:
  - Factory for processor instantiation
  - ZIP utilities for archive handling
  - MinIO client for image storage
  - LangChain loaders for PDF/DOC and Docling for office formats
- OCR plugins depend on:
  - External services or local models
  - Requests for HTTP communication
  - PyMuPDF/Pillow for PDF/image processing
- URL utilities depend on:
  - httpx for async fetching
  - validators from environment variables

```mermaid
graph LR
Unified["unified.py"] --> Factory["factory.py"]
Unified --> ZIP["zip_utils.py"]
Unified --> MinIO["storage.minio (via utils)"]
Unified --> LangChain["langchain_community loaders"]
Unified --> Docling["docling"]
Rapid["rapid_ocr.py"] --> Requests["requests"]
Rapid --> FitZ["fitz (PyMuPDF)"]
Rapid --> PIL["PIL"]
MinerU["mineru.py"] --> Requests
MinerU --> ZIP
MinerUOff["mineru_official.py"] --> Requests
MinerUOff --> ZIP
PP["pp_structure_v3.py"] --> Requests
Deep["deepseek_ocr.py"] --> Requests
Deep --> FitZ
URLF["url_fetcher.py"] --> URLV["url_validator.py"]
```

**Diagram sources**
- [unified.py:1-445](file://backend/package/yuxi/plugins/parser/unified.py#L1-L445)
- [factory.py:1-148](file://backend/package/yuxi/plugins/parser/factory.py#L1-L148)
- [zip_utils.py:1-211](file://backend/package/yuxi/plugins/parser/zip_utils.py#L1-L211)
- [rapid_ocr.py:1-254](file://backend/package/yuxi/plugins/parser/rapid_ocr.py#L1-L254)
- [mineru.py:1-240](file://backend/package/yuxi/plugins/parser/mineru.py#L1-L240)
- [mineru_official.py:1-366](file://backend/package/yuxi/plugins/parser/mineru_official.py#L1-L366)
- [pp_structure_v3.py:1-275](file://backend/package/yuxi/plugins/parser/pp_structure_v3.py#L1-L275)
- [deepseek_ocr.py:1-182](file://backend/package/yuxi/plugins/parser/deepseek_ocr.py#L1-L182)
- [url_fetcher.py:1-143](file://backend/package/yuxi/knowledge/utils/url_fetcher.py#L1-L143)
- [url_validator.py:1-88](file://backend/package/yuxi/knowledge/utils/url_validator.py#L1-L88)

**Section sources**
- [unified.py:1-445](file://backend/package/yuxi/plugins/parser/unified.py#L1-L445)
- [factory.py:1-148](file://backend/package/yuxi/plugins/parser/factory.py#L1-L148)
- [zip_utils.py:1-211](file://backend/package/yuxi/plugins/parser/zip_utils.py#L1-L211)
- [url_fetcher.py:1-143](file://backend/package/yuxi/knowledge/utils/url_fetcher.py#L1-L143)
- [url_validator.py:1-88](file://backend/package/yuxi/knowledge/utils/url_validator.py#L1-L88)

## Performance Considerations
- Streaming and chunking:
  - PDF OCR streams pages via PyMuPDF to avoid loading entire PDF into memory.
  - ZIP processing reads files lazily and uploads images concurrently.
- Caching:
  - Factory caches processor instances keyed by type and parameters to reduce initialization overhead.
- Async I/O:
  - MinIO uploads and ZIP processing are async-aware; URL fetching uses streaming.
- Image optimization:
  - Images are uploaded with timestamps and normalized prefixes; Markdown image links are replaced after upload.
- Health checks:
  - Factory exposes health checks for all processors; async variant uses threads to keep the event loop responsive.

[No sources needed since this section provides general guidance]

## Troubleshooting Guide
Common issues and resolutions:
- Unsupported file type:
  - Ensure the extension is in the supported list; otherwise, the unified parser raises an error.
- OCR failures:
  - RapidOCR: Verify model paths and thresholds; check health status.
  - MinerU/PP-Structure-V3/DeepSeek: Confirm service availability, credentials, and timeouts; review logs for HTTP errors.
- ZIP extraction errors:
  - Ensure ZIP contains a .md file; verify images directory exists and is accessible; check for unsafe paths.
- URL parsing disabled:
  - Configure the whitelist environment variable; validate domain patterns and wildcard usage.
- Private IP or redirect issues:
  - Private IP access is blocked; redirects are validated at each hop; adjust whitelist or disable parsing.

**Section sources**
- [unified.py:394-404](file://backend/package/yuxi/plugins/parser/unified.py#L394-L404)
- [rapid_ocr.py:44-91](file://backend/package/yuxi/plugins/parser/rapid_ocr.py#L44-L91)
- [mineru.py:33-91](file://backend/package/yuxi/plugins/parser/mineru.py#L33-L91)
- [mineru_official.py:42-94](file://backend/package/yuxi/plugins/parser/mineru_official.py#L42-L94)
- [pp_structure_v3.py:159-194](file://backend/package/yuxi/plugins/parser/pp_structure_v3.py#L159-L194)
- [deepseek_ocr.py:56-78](file://backend/package/yuxi/plugins/parser/deepseek_ocr.py#L56-L78)
- [zip_utils.py:41-50](file://backend/package/yuxi/plugins/parser/zip_utils.py#L41-L50)
- [url_validator.py:74-87](file://backend/package/yuxi/knowledge/utils/url_validator.py#L74-L87)
- [url_fetcher.py:61-62](file://backend/package/yuxi/knowledge/utils/url_fetcher.py#L61-L62)

## Conclusion
The document processing pipeline integrates a unified parser facade with a flexible factory and plugin system to support diverse file formats and OCR/document understanding services. It emphasizes robustness through health checks, secure URL handling, artifact extraction, and efficient processing strategies for large documents.

[No sources needed since this section summarizes without analyzing specific files]

## Appendices

### Supported File Formats
- Text and markup: txt, md
- Office: docx, pptx, xlsx, doc
- PDF: pdf
- Images: jpg, jpeg, png, bmp, tiff, tif
- Web: html, htm
- Data: csv, json
- Archives: zip

**Section sources**
- [unified.py:25-44](file://backend/package/yuxi/plugins/parser/unified.py#L25-L44)

### Custom Parser Development
Steps to add a new OCR/document parser:
1. Define a class inheriting from the base processor interface and implement required methods.
2. Register the processor type in the factory mapping with module and class names.
3. Optionally expose health checks and parameter handling.
4. Integrate into the unified parser if needed for automatic routing.

References:
- Base interface and exceptions: [base.py:44-100](file://backend/package/yuxi/plugins/parser/base.py#L44-L100)
- Factory registration and caching: [factory.py:18-76](file://backend/package/yuxi/plugins/parser/factory.py#L18-L76)
- Example implementations: [rapid_ocr.py:21-254](file://backend/package/yuxi/plugins/parser/rapid_ocr.py#L21-L254), [pp_structure_v3.py:19-275](file://backend/package/yuxi/plugins/parser/pp_structure_v3.py#L19-L275)

**Section sources**
- [base.py:44-100](file://backend/package/yuxi/plugins/parser/base.py#L44-L100)
- [factory.py:18-76](file://backend/package/yuxi/plugins/parser/factory.py#L18-L76)
- [rapid_ocr.py:21-254](file://backend/package/yuxi/plugins/parser/rapid_ocr.py#L21-L254)
- [pp_structure_v3.py:19-275](file://backend/package/yuxi/plugins/parser/pp_structure_v3.py#L19-L275)