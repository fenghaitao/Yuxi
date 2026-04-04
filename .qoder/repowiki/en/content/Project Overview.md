# Project Overview

<cite>
**Referenced Files in This Document**
- [README.md](file://README.md)
- [project-overview.md](file://docs/intro/project-overview.md)
- [main.js](file://web/src/main.js)
- [App.vue](file://web/src/App.vue)
- [main.py](file://backend/server/main.py)
- [routers/__init__.py](file://backend/server/routers/__init__.py)
- [base.py](file://backend/package/yuxi/agents/base.py)
- [graph.py](file://backend/package/yuxi/agents/buildin/deep_agent/graph.py)
- [chat.py](file://backend/package/yuxi/models/chat.py)
- [presets.py](file://backend/package/yuxi/knowledge/chunking/ragflow_like/presets.py)
- [lightrag.py](file://backend/package/yuxi/knowledge/graphs/adapters/lightrag.py)
- [docker-compose.yml](file://docker-compose.yml)
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
Yuxi (语析) is a business-oriented AI agent development platform that combines Retrieval-Augmented Generation (RAG) with knowledge graph construction. It enables organizations to build production-grade AI applications that understand and reason over internal knowledge. The platform integrates LangGraph-based agents, a Vue.js frontend, and a FastAPI backend, and it supports enterprise-grade capabilities such as knowledge base ingestion, graph construction, sandboxed execution, and containerized deployment.

Key value propositions:
- Intelligent agent development powered by LangGraph v1 with subagents, skills, MCPs, tools, and middleware.
- Knowledge base management with multi-format document ingestion, chunking presets, embedding/rerank configuration, and evaluation.
- Knowledge graph construction and reasoning via LightRAG and Neo4j, integrated into agent workflows.
- Production-ready deployment with Docker Compose, hot-reload development, dark mode, and API key authentication.

Practical outcomes users can achieve:
- Build RAG-powered agents that answer complex questions using structured knowledge.
- Construct and query knowledge graphs from unstructured documents for deeper reasoning.
- Orchestrate multi-step tasks with subagents and tools, including file system operations and web search.
- Manage permissions, skills, tools, and subagents through a centralized admin interface.

**Section sources**
- [README.md:22-36](file://README.md#L22-L36)
- [project-overview.md:3-81](file://docs/intro/project-overview.md#L3-L81)

## Project Structure
The repository is organized into three major layers:
- Frontend (Vue.js): Single-page application with routing, stores, and UI components.
- Backend (FastAPI): REST API with modular routers, middleware, and domain services.
- Platform services: Docker Compose-managed infrastructure including databases, graph engine, vector store, object storage, and OCR/document parsing services.

```mermaid
graph TB
subgraph "Frontend (Vue.js)"
WEB_MAIN["web/src/main.js"]
APP["web/src/App.vue"]
end
subgraph "Backend (FastAPI)"
API_MAIN["backend/server/main.py"]
ROUTERS["backend/server/routers/__init__.py"]
end
subgraph "Platform Services (Docker)"
DC["docker-compose.yml"]
end
WEB_MAIN --> API_MAIN
APP --> WEB_MAIN
API_MAIN --> ROUTERS
ROUTERS --> DC
```

**Diagram sources**
- [main.js:1-26](file://web/src/main.js#L1-L26)
- [App.vue:1-23](file://web/src/App.vue#L1-L23)
- [main.py:139-150](file://backend/server/main.py#L139-L150)
- [routers/__init__.py:20-49](file://backend/server/routers/__init__.py#L20-L49)
- [docker-compose.yml:37-84](file://docker-compose.yml#L37-L84)

**Section sources**
- [docker-compose.yml:37-84](file://docker-compose.yml#L37-L84)
- [main.js:1-26](file://web/src/main.js#L1-L26)
- [main.py:139-150](file://backend/server/main.py#L139-L150)

## Core Components
- LangGraph-based agents: Base agent abstraction and built-in DeepAgent graph with middleware pipeline for tools, skills, subagents, and knowledge base integration.
- Knowledge base and chunking: Preset-driven chunking strategies (general, QA, book, laws) with defaults and legacy parameter compatibility.
- Knowledge graph adapter: Neo4j-backed adapter for LightRAG, exposing standardized node/edge queries and statistics.
- Chat model abstraction: Unified OpenAI-compatible client with retry/backoff and model selection logic.
- Frontend: Vue 3 app bootstrapped with Pinia, Ant Design Vue, and router; mounts and initializes stores on startup.
- Backend: FastAPI app with CORS, access logging, rate-limiting, and modular routers; includes system, auth, chat, dashboard, tasks, tools, skills, subagents, filesystem, knowledge, evaluation, and graph routes.

**Section sources**
- [base.py:17-263](file://backend/package/yuxi/agents/base.py#L17-L263)
- [graph.py:27-124](file://backend/package/yuxi/agents/buildin/deep_agent/graph.py#L27-L124)
- [presets.py:1-239](file://backend/package/yuxi/knowledge/chunking/ragflow_like/presets.py#L1-L239)
- [lightrag.py:8-329](file://backend/package/yuxi/knowledge/graphs/adapters/lightrag.py#L8-L329)
- [chat.py:26-203](file://backend/package/yuxi/models/chat.py#L26-L203)
- [main.js:1-26](file://web/src/main.js#L1-L26)
- [main.py:40-137](file://backend/server/main.py#L40-L137)
- [routers/__init__.py:20-49](file://backend/server/routers/__init__.py#L20-L49)

## Architecture Overview
The platform’s runtime architecture connects the frontend UI to the backend API, which orchestrates agent execution, knowledge base operations, and graph queries. Platform services provide persistent storage, vector search, graph traversal, and document parsing.

```mermaid
graph TB
FE["Vue.js Frontend<br/>web/src/main.js, App.vue"]
API["FastAPI Backend<br/>backend/server/main.py"]
ROUTES["Routers<br/>server/routers/__init__.py"]
AGENTS["LangGraph Agents<br/>agents/base.py, agents/buildin/deep_agent/graph.py"]
KB["Knowledge Base<br/>chunking presets"]
GRAPH["Knowledge Graph Adapter<br/>graphs/adapters/lightrag.py"]
CHAT["Chat Model Abstraction<br/>models/chat.py"]
DB["PostgreSQL"]
VECTOR["Milvus"]
GRAPHDB["Neo4j"]
STORE["MinIO"]
OCR["MinerU / PaddleX / RapidOCR"]
FE --> API
API --> ROUTES
ROUTES --> AGENTS
ROUTES --> KB
ROUTES --> GRAPH
ROUTES --> CHAT
AGENTS --> DB
KB --> VECTOR
GRAPH --> GRAPHDB
KB --> STORE
KB --> OCR
```

**Diagram sources**
- [main.js:1-26](file://web/src/main.js#L1-L26)
- [App.vue:1-23](file://web/src/App.vue#L1-L23)
- [main.py:40-137](file://backend/server/main.py#L40-L137)
- [routers/__init__.py:20-49](file://backend/server/routers/__init__.py#L20-L49)
- [base.py:17-263](file://backend/package/yuxi/agents/base.py#L17-L263)
- [graph.py:27-124](file://backend/package/yuxi/agents/buildin/deep_agent/graph.py#L27-L124)
- [presets.py:1-239](file://backend/package/yuxi/knowledge/chunking/ragflow_like/presets.py#L1-L239)
- [lightrag.py:8-329](file://backend/package/yuxi/knowledge/graphs/adapters/lightrag.py#L8-L329)
- [chat.py:26-203](file://backend/package/yuxi/models/chat.py#L26-L203)

## Detailed Component Analysis

### Agent Runtime and Middleware Pipeline
The agent runtime compiles a LangGraph workflow with a checkpointer for state persistence and streams events/messages. The DeepAgent demonstrates a middleware pipeline integrating filesystem access, skills, knowledge base tools, subagents, and tool-call limits.

```mermaid
sequenceDiagram
participant UI as "Frontend"
participant API as "FastAPI"
participant Agent as "DeepAgent"
participant MW as "Middleware Chain"
participant KB as "Knowledge Base"
participant FS as "Filesystem"
participant Graph as "Graph Adapter"
UI->>API : "Invoke agent with messages"
API->>Agent : "Compile/get_graph(context)"
Agent->>MW : "Apply middleware pipeline"
MW->>FS : "Access sandboxed filesystem"
MW->>KB : "Query knowledge chunks"
MW->>Graph : "Query entities/relations"
MW-->>Agent : "Tool outputs and state updates"
Agent-->>API : "Streamed messages/events"
API-->>UI : "Real-time response"
```

**Diagram sources**
- [graph.py:52-124](file://backend/package/yuxi/agents/buildin/deep_agent/graph.py#L52-L124)
- [base.py:64-150](file://backend/package/yuxi/agents/base.py#L64-L150)

**Section sources**
- [graph.py:27-124](file://backend/package/yuxi/agents/buildin/deep_agent/graph.py#L27-L124)
- [base.py:17-263](file://backend/package/yuxi/agents/base.py#L17-L263)

### Knowledge Base Chunking and Presets
The chunking subsystem normalizes preset IDs, merges defaults and overrides, and resolves effective configuration snapshots. It supports legacy parameters and exposes preset options for UI consumption.

```mermaid
flowchart TD
Start(["Resolve Chunk Params"]) --> Normalize["Normalize preset ID"]
Normalize --> Defaults["Load preset defaults"]
Defaults --> MergeKB["Merge KB-level overrides"]
MergeKB --> MergeFile["Merge file-level overrides"]
MergeFile --> MergeReq["Merge request-level overrides"]
MergeReq --> Legacy["Convert legacy params"]
Legacy --> Snapshot["Build processing snapshot"]
Snapshot --> Options["Expose preset options"]
Options --> End(["Return config"])
```

**Diagram sources**
- [presets.py:79-213](file://backend/package/yuxi/knowledge/chunking/ragflow_like/presets.py#L79-L213)

**Section sources**
- [presets.py:1-239](file://backend/package/yuxi/knowledge/chunking/ragflow_like/presets.py#L1-L239)

### Knowledge Graph Adapter (LightRAG)
The LightRAG adapter wraps a Neo4j-based graph, normalizing nodes and edges, building Cypher queries, and returning standardized graph structures. It supports keyword search, label discovery, and statistics.

```mermaid
classDiagram
class LightRAGGraphAdapter {
+config dict
+kb_id str
+query_nodes(keyword, **kwargs) dict
+get_labels() list
+get_stats(**kwargs) dict
+normalize_node(raw_node) dict
+normalize_edge(raw_edge) dict
}
class BaseNeo4jAdapter {
+driver
+session()
}
LightRAGGraphAdapter --> BaseNeo4jAdapter : "uses"
```

**Diagram sources**
- [lightrag.py:8-329](file://backend/package/yuxi/knowledge/graphs/adapters/lightrag.py#L8-L329)

**Section sources**
- [lightrag.py:8-329](file://backend/package/yuxi/knowledge/graphs/adapters/lightrag.py#L8-L329)

### Chat Model Abstraction
The chat model abstraction encapsulates provider selection, model resolution, and streaming responses with exponential backoff and retry logic.

```mermaid
classDiagram
class OpenAIBase {
+api_key str
+base_url str
+model_name str
+call(message, stream) response
+get_models() list
}
class OpenModel {
+__init__(model_name)
}
OpenModel <|-- OpenAIBase
```

**Diagram sources**
- [chat.py:26-203](file://backend/package/yuxi/models/chat.py#L26-L203)

**Section sources**
- [chat.py:26-203](file://backend/package/yuxi/models/chat.py#L26-L203)

### Frontend Bootstrapping and Routing
The Vue application initializes Pinia, registers Ant Design Vue, and mounts the root component. It loads pre-configured branding and info, and routes users to the appropriate views.

```mermaid
sequenceDiagram
participant Main as "main.js"
participant App as "App.vue"
participant Router as "Router"
participant Stores as "Pinia Stores"
Main->>Stores : "createPinia() and plugin"
Main->>App : "createApp(App)"
App->>Stores : "loadInfoConfig()"
App->>Router : "router-view"
```

**Diagram sources**
- [main.js:1-26](file://web/src/main.js#L1-L26)
- [App.vue:1-23](file://web/src/App.vue#L1-L23)

**Section sources**
- [main.js:1-26](file://web/src/main.js#L1-L26)
- [App.vue:1-23](file://web/src/App.vue#L1-L23)

### Backend API and Routers
The FastAPI app sets up CORS, access logging, rate limiting, and authentication middleware. It includes modular routers for system, auth, chat, dashboard, tasks, tools, skills, subagents, filesystem, knowledge, evaluation, and graph.

```mermaid
graph LR
A["FastAPI app"] --> B["CORS"]
A --> C["AccessLogMiddleware"]
A --> D["LoginRateLimitMiddleware"]
A --> E["AuthMiddleware"]
A --> F["include_router(router, prefix='/api')"]
F --> G["system, auth, chat"]
F --> H["dashboard, department, tasks"]
F --> I["mcp, skills, subagents, tools, apikey"]
F --> J["filesystem"]
F --> K["knowledge, evaluation, mindmap (non-LITE)"]
F --> L["graph (non-LITE)"]
```

**Diagram sources**
- [main.py:40-137](file://backend/server/main.py#L40-L137)
- [routers/__init__.py:20-49](file://backend/server/routers/__init__.py#L20-L49)

**Section sources**
- [main.py:40-137](file://backend/server/main.py#L40-L137)
- [routers/__init__.py:20-49](file://backend/server/routers/__init__.py#L20-L49)

## Dependency Analysis
- Frontend depends on Vue 3 ecosystem (Pinia, Ant Design Vue) and router; it communicates with the backend API.
- Backend depends on LangGraph for agent orchestration, PostgreSQL for persistence, Milvus for vector search, Neo4j for graph storage, MinIO for object storage, and OCR/document parsing services.
- Docker Compose defines service dependencies and environment variables for seamless orchestration.

```mermaid
graph TB
WEB["web/src/main.js"] --> API["backend/server/main.py"]
API --> AGENTS["agents/base.py, agents/buildin/deep_agent/graph.py"]
API --> MODELS["models/chat.py"]
API --> ROUTES["server/routers/__init__.py"]
ROUTES --> KB["knowledge/chunking/ragflow_like/presets.py"]
ROUTES --> GRAPH["knowledge/graphs/adapters/lightrag.py"]
API --> DB["PostgreSQL"]
KB --> VECTOR["Milvus"]
GRAPH --> GRAPHDB["Neo4j"]
KB --> STORE["MinIO"]
KB --> OCR["MinerU/PaddleX/RapidOCR"]
```

**Diagram sources**
- [main.js:1-26](file://web/src/main.js#L1-L26)
- [main.py:40-137](file://backend/server/main.py#L40-L137)
- [base.py:17-263](file://backend/package/yuxi/agents/base.py#L17-L263)
- [graph.py:27-124](file://backend/package/yuxi/agents/buildin/deep_agent/graph.py#L27-L124)
- [chat.py:26-203](file://backend/package/yuxi/models/chat.py#L26-L203)
- [routers/__init__.py:20-49](file://backend/server/routers/__init__.py#L20-L49)
- [presets.py:1-239](file://backend/package/yuxi/knowledge/chunking/ragflow_like/presets.py#L1-L239)
- [lightrag.py:8-329](file://backend/package/yuxi/knowledge/graphs/adapters/lightrag.py#L8-L329)

**Section sources**
- [docker-compose.yml:37-436](file://docker-compose.yml#L37-L436)

## Performance Considerations
- Streaming responses: Use agent streaming APIs to deliver incremental updates and reduce perceived latency.
- Chunking presets: Choose appropriate chunk sizes and overlap to balance retrieval accuracy and latency.
- Middleware limits: Enforce tool-call limits and summary offloading to prevent long-running runs and excessive context growth.
- Vector and graph indexing: Tune embedding dimensions, similarity thresholds, and graph traversal depth for query performance.
- Container resource allocation: Configure GPU/CPU/memory for OCR and embedding services to avoid bottlenecks.

[No sources needed since this section provides general guidance]

## Troubleshooting Guide
Common issues and remedies:
- Backend health checks failing: Verify service readiness and network connectivity among api, postgres, redis, minio, and sandbox-provisioner.
- Authentication and rate limiting: Review login attempts and ensure proper credentials; adjust rate-limit windows if needed.
- Agent history and checkpoints: Confirm checkpointer configuration (SQLite/PostgreSQL) and availability of persisted state.
- Knowledge base ingestion: Validate chunking presets and ensure OCR/document parsing services are healthy.

**Section sources**
- [docker-compose.yml:69-84](file://docker-compose.yml#L69-L84)
- [main.py:63-96](file://backend/server/main.py#L63-L96)
- [base.py:199-254](file://backend/package/yuxi/agents/base.py#L199-L254)

## Conclusion
Yuxi provides a cohesive, production-ready framework for building intelligent agents that combine RAG and knowledge graphs. Its modular architecture, robust middleware pipeline, and containerized deployment simplify real-world adoption. Teams can rapidly prototype and operate scalable AI applications tailored to business needs.

[No sources needed since this section summarizes without analyzing specific files]

## Appendices
- Quick start: Clone the repository, initialize environment, and bring up services with Docker Compose. Access the frontend at the configured port and log in to explore agents, knowledge bases, and graphs.
- Technology stack highlights: Vue 3 + Pinia + Ant Design Vue for frontend; FastAPI + Uvicorn for backend; LangGraph v1 for agents; LightRAG + Neo4j for graphs; Milvus for vectors; PostgreSQL, Redis, MinIO for persistence and storage; Docker Compose for orchestration.

[No sources needed since this section provides general guidance]