# Agent Framework Overview

<cite>
**Referenced Files in This Document**
- [base.py](file://backend/package/yuxi/agents/base.py)
- [context.py](file://backend/package/yuxi/agents/context.py)
- [state.py](file://backend/package/yuxi/agents/state.py)
- [models.py](file://backend/package/yuxi/agents/models.py)
- [__init__.py](file://backend/package/yuxi/agents/__init__.py)
- [graph.py](file://backend/package/yuxi/agents/buildin/deep_agent/graph.py)
- [__init__.py](file://backend/package/yuxi/agents/buildin/deep_agent/__init__.py)
- [skills_middleware.py](file://backend/package/yuxi/agents/middlewares/skills_middleware.py)
- [knowledge_base_middleware.py](file://backend/package/yuxi/agents/middlewares/knowledge_base_middleware.py)
- [context_middlewares.py](file://backend/package/yuxi/agents/middlewares/context_middlewares.py)
- [composite.py](file://backend/package/yuxi/agents/backends/composite.py)
- [registry.py](file://backend/package/yuxi/agents/toolkits/registry.py)
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
This document explains the agent framework’s foundational architecture and operational patterns. It focuses on the BaseAgent class as the core abstraction for all agent implementations, the agent lifecycle from initialization through execution, context and state management, metadata and capability declarations, and how agents integrate with middlewares, backends, and toolkits. Practical usage patterns and registration mechanisms are included to help developers implement and deploy new agents effectively.

## Project Structure
The agent framework is organized around a small set of core abstractions and a rich ecosystem of middlewares, backends, and toolkits. The most important modules are:
- BaseAgent: The abstract base class that defines the agent interface and lifecycle.
- BaseContext: The typed configuration schema passed into agent graphs at runtime.
- BaseState: The shared state schema used across agents.
- Middlewares: Extensible components that augment agent behavior (skills, knowledge base, runtime configuration, summarization, etc.).
- Backends: Pluggable storage and routing backends for file systems, knowledge bases, and sandbox environments.
- Toolkits: Registry and decorators for tools, with automatic discovery and metadata collection.
- Models: Unified model loading utilities for various providers.

```mermaid
graph TB
subgraph "Core Abstractions"
BA["BaseAgent<br/>base.py"]
BCtx["BaseContext<br/>context.py"]
BState["BaseState<br/>state.py"]
LCM["load_chat_model<br/>models.py"]
end
subgraph "Built-in Agent"
DA["DeepAgent<br/>deep_agent/graph.py"]
end
subgraph "Middlewares"
MW1["SkillsMiddleware<br/>skills_middleware.py"]
MW2["KnowledgeBaseMiddleware<br/>knowledge_base_middleware.py"]
MW3["Context Middlewares<br/>context_middlewares.py"]
end
subgraph "Backends"
CB["CustomCompositeBackend<br/>backends/composite.py"]
end
subgraph "Toolkits"
REG["Tool Registry<br/>toolkits/registry.py"]
end
BA --> DA
DA --> BCtx
DA --> BState
DA --> LCM
DA --> MW1
DA --> MW2
DA --> MW3
MW3 --> LCM
MW1 --> REG
MW2 --> REG
DA --> CB
```

**Diagram sources**
- [base.py:17-263](file://backend/package/yuxi/agents/base.py#L17-L263)
- [context.py:11-191](file://backend/package/yuxi/agents/context.py#L11-L191)
- [state.py:19-31](file://backend/package/yuxi/agents/state.py#L19-L31)
- [models.py:12-58](file://backend/package/yuxi/agents/models.py#L12-L58)
- [graph.py:27-124](file://backend/package/yuxi/agents/buildin/deep_agent/graph.py#L27-L124)
- [skills_middleware.py:145-477](file://backend/package/yuxi/agents/middlewares/skills_middleware.py#L145-L477)
- [knowledge_base_middleware.py:12-33](file://backend/package/yuxi/agents/middlewares/knowledge_base_middleware.py#L12-L33)
- [context_middlewares.py:11-26](file://backend/package/yuxi/agents/middlewares/context_middlewares.py#L11-L26)
- [composite.py:18-134](file://backend/package/yuxi/agents/backends/composite.py#L18-L134)
- [registry.py:38-97](file://backend/package/yuxi/agents/toolkits/registry.py#L38-L97)

**Section sources**
- [__init__.py:1-29](file://backend/package/yuxi/agents/__init__.py#L1-L29)

## Core Components
- BaseAgent: Defines the agent contract, lifecycle hooks, streaming and invocation APIs, and persistence via checkpointer backends (SQLite, Postgres, or memory).
- BaseContext: A typed dataclass that carries runtime configuration such as thread identifiers, user identifiers, system prompts, model selection, tools, knowledge bases, MCP servers, skills, subagents, and summarization thresholds.
- BaseState: A shared state schema extending AgentState with artifact tracking.
- Model loader: A unified function to initialize chat models from provider-specific configurations.

Key responsibilities:
- BaseAgent orchestrates graph compilation, streaming, invocation, history retrieval, and persistence.
- BaseContext provides a schema-driven configuration surface with metadata for UI rendering and validation.
- BaseState standardizes stateful artifacts across agents.
- Models loader encapsulates provider-specific initialization and environment variable handling.

**Section sources**
- [base.py:17-263](file://backend/package/yuxi/agents/base.py#L17-L263)
- [context.py:11-191](file://backend/package/yuxi/agents/context.py#L11-L191)
- [state.py:19-31](file://backend/package/yuxi/agents/state.py#L19-L31)
- [models.py:12-58](file://backend/package/yuxi/agents/models.py#L12-L58)

## Architecture Overview
The agent framework is built on LangGraph’s CompiledStateGraph. Agents declare capabilities and metadata, configure runtime context, and compose middleware stacks to extend behavior. The graph is compiled with a checkpointer to support state persistence and history retrieval. Backends provide secure, routed access to file systems, knowledge bases, and sandbox environments. Toolkits register tools with metadata for discoverability and UI presentation.

```mermaid
sequenceDiagram
participant Client as "Caller"
participant Agent as "BaseAgent<br/>base.py"
participant Graph as "CompiledStateGraph"
participant Ctx as "BaseContext<br/>context.py"
participant MW as "Middlewares<br/>skills_middleware.py"
participant BK as "Backends<br/>backends/composite.py"
Client->>Agent : "invoke_messages(messages, input_context)"
Agent->>Ctx : "construct/update_from_dict"
Agent->>Agent : "get_graph(context)"
Agent->>Graph : "compile with checkpointer"
Agent->>MW : "apply middleware stack"
MW->>BK : "resolve backends (sandbox, skills, kbs)"
Agent->>Graph : "ainvoke({messages}, context, config)"
Graph-->>Agent : "final state/messages"
Agent-->>Client : "result"
```

**Diagram sources**
- [base.py:125-150](file://backend/package/yuxi/agents/base.py#L125-L150)
- [context.py:186-191](file://backend/package/yuxi/agents/context.py#L186-L191)
- [skills_middleware.py:145-272](file://backend/package/yuxi/agents/middlewares/skills_middleware.py#L145-L272)
- [composite.py:122-134](file://backend/package/yuxi/agents/backends/composite.py#L122-L134)

## Detailed Component Analysis

### BaseAgent: Lifecycle, Streaming, Invocation, Persistence
BaseAgent centralizes agent lifecycle and execution:
- Initialization: Creates a working directory under a per-agent module path and prepares persistence resources.
- Metadata exposure: Provides agent metadata and configurable items derived from the context schema.
- Execution: Supports streaming values, streaming messages, streaming with state, and single-shot invocation.
- Persistence: Selects a checkpointer backend (Postgres, SQLite, or memory) and exposes history retrieval.
- Graph lifecycle: Enforces that subclasses compile graphs with a checkpointer to enable state restoration.

```mermaid
classDiagram
class BaseAgent {
+name : string
+description : string
+capabilities : string[]
+context_schema
+get_info(include_configurable_items)
+get_config()
+stream_values(messages, input_context, **kwargs)
+stream_messages(messages, input_context, **kwargs)
+stream_messages_with_state(messages, input_context, **kwargs)
+invoke_messages(messages, input_context, **kwargs)
+get_history(user_id, thread_id)
+reload_graph()
+get_graph(**kwargs)
+_get_checkpointer()
+get_async_conn()
+get_aio_memory()
+load_metadata()
}
```

**Diagram sources**
- [base.py:17-263](file://backend/package/yuxi/agents/base.py#L17-L263)

**Section sources**
- [base.py:27-60](file://backend/package/yuxi/agents/base.py#L27-L60)
- [base.py:64-150](file://backend/package/yuxi/agents/base.py#L64-L150)
- [base.py:158-184](file://backend/package/yuxi/agents/base.py#L158-L184)
- [base.py:199-254](file://backend/package/yuxi/agents/base.py#L199-L254)
- [base.py:256-263](file://backend/package/yuxi/agents/base.py#L256-L263)

### BaseContext: Typed Runtime Configuration
BaseContext defines a strongly-typed configuration surface:
- Identity fields: thread_id and user_id for conversation threading and user scoping.
- Prompting and model: system_prompt and model selection with provider-specific metadata.
- Tooling: tools list and dynamic tool injection via middlewares.
- Knowledge and MCP: lists of knowledge bases and MCP servers with dynamic resolution.
- Skills and subagents: lists of skills and subagents with defaults and UI metadata.
- Summarization threshold: controls when context offloading is triggered.

It also provides:
- get_configurable_items(): introspects fields and extracts UI-friendly metadata, including type inference and template hints.
- update_from_dict(): merges runtime configuration into the context.

```mermaid
classDiagram
class BaseContext {
+thread_id : string
+user_id : string
+system_prompt : string
+model : string
+tools : string[]
+knowledges : string[]
+mcps : string[]
+skills : string[]
+subagents_model : string
+subagents : string[]
+summary_threshold : int
+update(data)
+update_from_dict(data)
+get_configurable_items()
}
```

**Diagram sources**
- [context.py:11-191](file://backend/package/yuxi/agents/context.py#L11-L191)

**Section sources**
- [context.py:27-116](file://backend/package/yuxi/agents/context.py#L27-L116)
- [context.py:118-149](file://backend/package/yuxi/agents/context.py#L118-L149)
- [context.py:186-191](file://backend/package/yuxi/agents/context.py#L186-L191)

### BaseState and Artifacts
BaseState extends AgentState with an artifacts field annotated with a merge function to preserve ordering and deduplicate file paths. AgentStatePayload describes serialized state consumable by the frontend.

```mermaid
classDiagram
class BaseState {
+artifacts : string[]
}
class AgentStatePayload {
+todos : list
+files : dict
+artifacts : string[]
}
BaseState <|-- AgentStatePayload
```

**Diagram sources**
- [state.py:19-31](file://backend/package/yuxi/agents/state.py#L19-L31)

**Section sources**
- [state.py:10-17](file://backend/package/yuxi/agents/state.py#L10-L17)
- [state.py:25-31](file://backend/package/yuxi/agents/state.py#L25-L31)

### Model Loader: Provider-Agnostic Chat Model Initialization
The model loader resolves provider-specific initialization, environment variables, and base URLs. It supports official providers and others via LangChain integrations.

```mermaid
flowchart TD
Start(["load_chat_model(fully_specified_name)"]) --> Parse["Parse provider/model"]
Parse --> Lookup["Lookup provider config"]
Lookup --> Env["Resolve API key and base URL"]
Env --> Build{"Provider type?"}
Build --> |Official| InitOfficial["init_chat_model(spec)"]
Build --> |DashScope| InitDash["ChatDeepSeek(...)"]
Build --> |Others| InitOpenAI["ChatOpenAI(...)"]
InitOfficial --> End(["Return BaseChatModel"])
InitDash --> End
InitOpenAI --> End
```

**Diagram sources**
- [models.py:12-58](file://backend/package/yuxi/agents/models.py#L12-L58)

**Section sources**
- [models.py:12-58](file://backend/package/yuxi/agents/models.py#L12-L58)

### Built-in Agent Example: DeepAgent
DeepAgent demonstrates a real-world agent implementation:
- Declares name, description, capabilities, and metadata.
- Builds a system prompt by combining a deep prompt with the runtime context’s system prompt.
- Loads models for the main agent and subagents.
- Composes a middleware pipeline including filesystem, runtime configuration, skills, knowledge base, subagents, and summarization.
- Compiles the graph with a checkpointer and returns it.

```mermaid
sequenceDiagram
participant Dev as "Developer"
participant DA as "DeepAgent<br/>graph.py"
participant Ctx as "BaseContext"
participant LCM as "load_chat_model"
participant MW as "Middleware Stack"
participant CG as "Compiled Graph"
Dev->>DA : "new DeepAgent()"
Dev->>DA : "get_graph(context)"
DA->>Ctx : "context or context_schema()"
DA->>LCM : "load main model"
DA->>LCM : "load subagents model"
DA->>MW : "compose middleware"
MW-->>DA : "ready middleware"
DA->>CG : "compile with checkpointer"
CG-->>Dev : "graph"
```

**Diagram sources**
- [graph.py:27-124](file://backend/package/yuxi/agents/buildin/deep_agent/graph.py#L27-L124)

**Section sources**
- [graph.py:27-124](file://backend/package/yuxi/agents/buildin/deep_agent/graph.py#L27-L124)
- [__init__.py:14-23](file://backend/package/yuxi/agents/buildin/deep_agent/__init__.py#L14-L23)

### Middlewares: Capabilities and Behavior Extension
- SkillsMiddleware: Injects skills prompts, expands dependencies, dynamically activates skills, and loads tools and MCP servers accordingly.
- KnowledgeBaseMiddleware: Provides common knowledge base tools and resolves visible knowledge bases for the runtime context.
- Context Middlewares: Dynamically adjust system prompts and model selection based on runtime context.

```mermaid
classDiagram
class SkillsMiddleware {
+state_schema
+abefore_agent(state, runtime)
+awrap_model_call(request, handler)
+awrap_tool_call(request, handler)
}
class KnowledgeBaseMiddleware {
+tools
+awrap_model_call(request, handler)
}
class ContextAwarePrompt {
+context_aware_prompt(request)
}
class ContextBasedModel {
+context_based_model(request, handler)
}
```

**Diagram sources**
- [skills_middleware.py:145-477](file://backend/package/yuxi/agents/middlewares/skills_middleware.py#L145-L477)
- [knowledge_base_middleware.py:12-33](file://backend/package/yuxi/agents/middlewares/knowledge_base_middleware.py#L12-L33)
- [context_middlewares.py:11-26](file://backend/package/yuxi/agents/middlewares/context_middlewares.py#L11-L26)

**Section sources**
- [skills_middleware.py:145-272](file://backend/package/yuxi/agents/middlewares/skills_middleware.py#L145-L272)
- [knowledge_base_middleware.py:12-33](file://backend/package/yuxi/agents/middlewares/knowledge_base_middleware.py#L12-L33)
- [context_middlewares.py:11-26](file://backend/package/yuxi/agents/middlewares/context_middlewares.py#L11-L26)

### Backends: Composite Routing and Access Control
The composite backend routes requests across multiple backends:
- Default: Sandbox backend scoped to thread and user with visible skills.
- Routes: Read-only access to selected skills and knowledge bases resolved from runtime context.

```mermaid
flowchart TD
RT["Runtime Context"] --> VS["Visible Skills"]
RT --> VT["Visible Knowledge Bases"]
RT --> TID["Thread ID"]
RT --> UID["User ID"]
VS --> CB["CustomCompositeBackend"]
VT --> CB
TID --> CB
UID --> CB
CB --> DEF["Default: SandboxBackend(thread_id, user_id, visible_skills)"]
CB --> SK["Route '/skills/': SelectedSkillsReadonlyBackend(selected_slugs)"]
CB --> KB["Route '/home/gem/kbs/': KnowledgeBaseReadonlyBackend(visible_kbs)"]
```

**Diagram sources**
- [composite.py:18-134](file://backend/package/yuxi/agents/backends/composite.py#L18-L134)

**Section sources**
- [composite.py:122-134](file://backend/package/yuxi/agents/backends/composite.py#L122-L134)

### Toolkits: Registration and Metadata
The toolkit registry enables:
- Defining tools with extra metadata (category, tags, display name, icon, configuration guide).
- Automatic collection of tool instances for dynamic activation.
- Discovery and UI presentation of tools.

```mermaid
classDiagram
class ToolExtraMetadata {
+category : string
+tags : string[]
+display_name : string
+icon : string
+config_guide : string
}
class Registry {
+get_extra_metadata(tool_name)
+get_all_extra_metadata()
+get_all_tool_instances()
+tool(...)
}
Registry --> ToolExtraMetadata : "stores"
```

**Diagram sources**
- [registry.py:5-97](file://backend/package/yuxi/agents/toolkits/registry.py#L5-L97)

**Section sources**
- [registry.py:38-97](file://backend/package/yuxi/agents/toolkits/registry.py#L38-L97)

## Dependency Analysis
The framework exhibits layered cohesion:
- BaseAgent depends on BaseContext, BaseState, and model loading utilities.
- Built-in agents depend on middlewares and backends to assemble capabilities.
- Middlewares depend on toolkits and services for dynamic tool activation and MCP integration.
- Backends depend on runtime context to resolve visibility and routing.

```mermaid
graph LR
BA["BaseAgent"] --> BCtx["BaseContext"]
BA --> BState["BaseState"]
BA --> LCM["load_chat_model"]
DA["DeepAgent"] --> BA
DA --> MW1["SkillsMiddleware"]
DA --> MW2["KnowledgeBaseMiddleware"]
DA --> MW3["Context Middlewares"]
MW1 --> REG["Tool Registry"]
MW2 --> REG
DA --> CB["CompositeBackend"]
```

**Diagram sources**
- [base.py:17-263](file://backend/package/yuxi/agents/base.py#L17-L263)
- [graph.py:27-124](file://backend/package/yuxi/agents/buildin/deep_agent/graph.py#L27-L124)
- [skills_middleware.py:145-477](file://backend/package/yuxi/agents/middlewares/skills_middleware.py#L145-L477)
- [knowledge_base_middleware.py:12-33](file://backend/package/yuxi/agents/middlewares/knowledge_base_middleware.py#L12-L33)
- [context_middlewares.py:11-26](file://backend/package/yuxi/agents/middlewares/context_middlewares.py#L11-L26)
- [composite.py:122-134](file://backend/package/yuxi/agents/backends/composite.py#L122-L134)
- [registry.py:38-97](file://backend/package/yuxi/agents/toolkits/registry.py#L38-L97)

**Section sources**
- [__init__.py:1-29](file://backend/package/yuxi/agents/__init__.py#L1-L29)

## Performance Considerations
- Streaming vs. single-shot: Prefer streaming APIs for long-running tasks to reduce latency and improve responsiveness.
- Checkpointer selection: Use Postgres-backed checkpointer in production for reliability; fall back to SQLite or memory when unavailable.
- Middleware overhead: Skills and knowledge base middlewares add dynamic resolution; tune thresholds and limits to avoid excessive tool activation.
- Summarization offload: Configure summary thresholds to keep context windows manageable for long conversations.
- Backend routing: Composite backend routing is efficient but ensure thread and user scoping avoids unnecessary cross-backend scans.

## Troubleshooting Guide
Common issues and resolutions:
- Missing or invalid checkpointer: Ensure get_graph compiles with a checkpointer; BaseAgent falls back to memory if needed.
- History retrieval errors: Verify thread_id and user_id are present in runtime configuration; handle empty histories gracefully.
- Skills not activating: Confirm skills are configured and visible; inspect middleware logs for dependency resolution and dynamic activation.
- MCP tool availability: Ensure MCP servers are reachable and enabled; verify tool loading and merging logic.
- Model provider errors: Validate provider configuration and environment variables; confirm base URLs and API keys.

**Section sources**
- [base.py:158-184](file://backend/package/yuxi/agents/base.py#L158-L184)
- [skills_middleware.py:216-272](file://backend/package/yuxi/agents/middlewares/skills_middleware.py#L216-L272)
- [knowledge_base_middleware.py:28-33](file://backend/package/yuxi/agents/middlewares/knowledge_base_middleware.py#L28-L33)
- [models.py:12-58](file://backend/package/yuxi/agents/models.py#L12-L58)

## Conclusion
The agent framework centers on BaseAgent as a robust, extensible foundation. Through typed contexts, standardized state, provider-agnostic model loading, and a modular middleware/backends/toolkit ecosystem, it enables consistent agent behavior, powerful capabilities, and seamless integration. Developers can implement new agents by extending BaseAgent, composing middlewares, and leveraging backends and toolkits tailored to their use cases.

## Appendices

### Agent Lifecycle: From Initialization to Execution
- Initialization: BaseAgent sets up working directories and persistence resources.
- Configuration: BaseContext is constructed and updated from input; configurable items are exposed for UI.
- Graph compilation: get_graph builds the middleware stack, loads models, and compiles the graph with a checkpointer.
- Execution: Invoke or stream messages; BaseAgent manages runtime configuration, callbacks, and metadata.
- Persistence: Retrieve history using thread_id and user_id; manage checkpointer backends.

```mermaid
flowchart TD
A["Initialize BaseAgent"] --> B["Build BaseContext"]
B --> C["Compile Graph with Checkpointer"]
C --> D["Apply Middlewares"]
D --> E["Invoke or Stream Messages"]
E --> F["Persist State and Retrieve History"]
```

**Diagram sources**
- [base.py:27-60](file://backend/package/yuxi/agents/base.py#L27-L60)
- [base.py:64-150](file://backend/package/yuxi/agents/base.py#L64-L150)
- [base.py:158-184](file://backend/package/yuxi/agents/base.py#L158-L184)

### Practical Usage Patterns
- Instantiate and configure:
  - Construct a BaseContext with desired fields and update from a dictionary.
  - Call get_config to obtain a typed configuration instance.
- Execute:
  - Use stream_messages or invoke_messages with optional callbacks, metadata, and tags.
- Expose metadata and capabilities:
  - Call get_info to retrieve agent metadata, capabilities, and configurable items.
- Manage state and history:
  - Use get_history with thread_id and user_id to retrieve persisted messages.

**Section sources**
- [base.py:44-59](file://backend/package/yuxi/agents/base.py#L44-L59)
- [base.py:64-150](file://backend/package/yuxi/agents/base.py#L64-L150)
- [base.py:158-184](file://backend/package/yuxi/agents/base.py#L158-L184)
- [context.py:186-191](file://backend/package/yuxi/agents/context.py#L186-L191)

### Agent Registration and Integration
- Built-in agents are surfaced via their package’s __init__ exports.
- Toolkits auto-register tools via decorators; middleware consumes registered tools and MCP servers.
- Backends are resolved per-runtime to enforce visibility and access control.

**Section sources**
- [__init__.py:14-23](file://backend/package/yuxi/agents/buildin/deep_agent/__init__.py#L14-L23)
- [registry.py:38-97](file://backend/package/yuxi/agents/toolkits/registry.py#L38-L97)
- [composite.py:122-134](file://backend/package/yuxi/agents/backends/composite.py#L122-L134)