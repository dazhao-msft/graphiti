# Graphiti Core Architecture Analysis

## Overview

Graphiti Core is a Python framework for building temporally-aware knowledge graphs designed for AI agents. The architecture follows a modular design with clear separation of concerns across different layers and responsibilities.

## Core Architecture Components

### 1. **Main Entry Point (`graphiti.py`)**
- **Graphiti Class**: Central orchestrator that manages all framework functionality
- Provides unified interface for episode processing, search, and graph operations
- Supports both single episode processing and bulk operations
- Integrates all subsystems through the `GraphitiClients` container

### 2. **Graph Data Model**

#### **Nodes (`nodes.py`)**
- **EpisodicNode**: Represents time-bound episodes/events with source content
- **EntityNode**: Core entities with embeddings, summaries, and custom attributes
- **CommunityNode**: Clusters of related entities for hierarchical organization
- All nodes share common base class with UUID, timestamps, and group partitioning

#### **Edges (`edges.py`)**
- **EpisodicEdge**: Links episodes to mentioned entities (MENTIONS relationship)
- **EntityEdge**: Semantic relationships between entities (RELATES_TO)
- **CommunityEdge**: Membership links between communities and entities
- Supports temporal validity periods and episode provenance tracking

### 3. **Storage Layer (`driver/`)**

#### **Abstract Driver Interface**
- `GraphDriver`: Database-agnostic interface for graph operations
- `GraphDriverSession`: Transaction management abstraction

#### **Concrete Implementations**
- **Neo4jDriver**: Production-ready implementation using Neo4j Python driver
- **FalkorDriver**: Alternative backend using FalkorDB with Redis-like syntax
- Both support multi-tenancy through database/graph selection

### 4. **AI Integration Layer**

#### **LLM Clients (`llm_client/`)**
- Abstract `LLMClient` interface with retry logic and caching
- Multiple provider support: OpenAI, Anthropic, Gemini, Groq, Azure
- Structured output support with Pydantic models
- Input sanitization and multilingual support

#### **Embedding Services (`embedder/`)**
- `EmbedderClient` abstraction for vector embeddings
- Provider implementations: OpenAI, Voyage, Gemini, Azure
- Batch processing capabilities for efficiency

#### **Cross-Encoder Reranking (`cross_encoder/`)**
- `CrossEncoderClient` for passage ranking and relevance scoring
- Used in hybrid search for final result reranking

### 5. **Search System (`search/`)**

#### **Hybrid Search Architecture**
- **Multiple search methods**: Vector similarity, BM25 full-text, graph traversal (BFS)
- **Multi-layer reranking**: RRF, MMR, cross-encoder, node distance, episode mentions
- **Configurable search strategies** through `SearchConfig` recipes
- **Search across all graph elements**: nodes, edges, episodes, communities

#### **Search Components**
- `search.py`: Main search orchestration
- `search_config.py`: Configuration schemas and result types
- `search_utils.py`: Core search algorithms and utilities

### 6. **Processing Pipeline**

#### **Episode Processing Workflow**
1. **Entity Extraction**: LLM-based entity identification from episode content
2. **Edge Extraction**: Relationship discovery between entities
3. **Deduplication**: Semantic and syntactic duplicate detection
4. **Graph Integration**: Node/edge resolution against existing graph
5. **Embedding Generation**: Vector representations for search
6. **Community Updates**: Optional hierarchical clustering updates

#### **Bulk Processing (`utils/bulk_utils.py`)**
- Optimized batch processing for multiple episodes
- In-memory deduplication using Union-Find algorithm
- Parallel LLM calls with semaphore-based concurrency control

### 7. **Maintenance Operations (`utils/maintenance/`)**

#### **Core Operations**
- **Node Operations**: Entity extraction, attribute hydration, resolution
- **Edge Operations**: Relationship extraction, validation, invalidation
- **Community Operations**: Hierarchical clustering and summarization
- **Temporal Operations**: Time-based graph maintenance
- **Graph Data Operations**: Schema management, indexing, constraints

### 8. **Supporting Infrastructure**

#### **Utilities**
- **DateTime utilities**: UTC normalization and timezone handling  
- **Telemetry**: Usage tracking and performance monitoring
- **Error handling**: Custom exception hierarchy
- **Helpers**: Concurrency management, normalization, validation

#### **Configuration Management**
- Environment variable support for API keys and settings
- Provider-specific configuration classes
- Flexible client injection for testing and customization

## Architectural Patterns

### **1. Dependency Injection**
The `GraphitiClients` container manages all external dependencies, enabling:
- Easy testing with mock implementations
- Runtime provider switching
- Configuration centralization

### **2. Strategy Pattern**
Search system uses configurable strategies:
- Multiple search methods can be combined
- Pluggable reranking algorithms
- Recipe-based configuration presets

### **3. Template Method**
Abstract base classes define common workflows:
- Driver interface standardizes database operations
- Client abstractions enable provider switching
- Consistent error handling and retry logic

### **4. Event Sourcing Characteristics**
- Episodes represent immutable events
- Temporal validity tracking on edges
- Provenance tracking through episode references

## Key Design Decisions

### **Temporal Awareness**
- Bi-temporal model with event time vs. validity time
- Edge invalidation for knowledge updates
- Episode-based provenance tracking

### **Hybrid Search**
- Multiple search modalities for comprehensive retrieval
- Configurable fusion strategies
- Cross-encoder reranking for quality

### **Scalability Considerations**
- Asynchronous operations throughout
- Bulk processing optimizations
- Database-agnostic design for deployment flexibility

### **AI-First Design**
- LLM integration for entity/relationship extraction
- Embedding-based semantic search
- Structured output validation

## File Structure Analysis

```
graphiti_core/
├── __init__.py
├── graphiti.py                 # Main entry point and orchestrator
├── nodes.py                    # Graph node definitions
├── edges.py                    # Graph edge definitions
├── graphiti_types.py          # Core type definitions
├── errors.py                   # Custom exception hierarchy
├── helpers.py                  # Utility functions
├── graph_queries.py           # Graph query templates
├── py.typed                    # Type checking marker
│
├── driver/                     # Database abstraction layer
│   ├── driver.py              # Abstract driver interface
│   ├── neo4j_driver.py        # Neo4j implementation
│   └── falkordb_driver.py     # FalkorDB implementation
│
├── llm_client/                 # LLM integration
│   ├── client.py              # Abstract LLM client
│   ├── config.py              # LLM configuration
│   ├── errors.py              # LLM-specific errors
│   ├── utils.py               # LLM utilities
│   ├── openai_client.py       # OpenAI implementation
│   ├── anthropic_client.py    # Anthropic implementation
│   ├── gemini_client.py       # Gemini implementation
│   ├── groq_client.py         # Groq implementation
│   ├── azure_openai_client.py # Azure OpenAI implementation
│   ├── openai_base_client.py  # OpenAI base class
│   └── openai_generic_client.py # Generic OpenAI client
│
├── embedder/                   # Embedding services
│   ├── client.py              # Abstract embedder client
│   ├── openai.py              # OpenAI embeddings
│   ├── azure_openai.py        # Azure OpenAI embeddings
│   ├── gemini.py              # Gemini embeddings
│   └── voyage.py              # Voyage embeddings
│
├── cross_encoder/              # Reranking services
│   ├── client.py              # Abstract cross-encoder client
│   ├── openai_reranker_client.py # OpenAI reranker
│   ├── gemini_reranker_client.py # Gemini reranker
│   └── bge_reranker_client.py # BGE reranker
│
├── search/                     # Search system
│   ├── search.py              # Main search orchestration
│   ├── search_config.py       # Search configuration schemas
│   ├── search_config_recipes.py # Predefined search strategies
│   ├── search_filters.py      # Search filtering logic
│   ├── search_helpers.py      # Search helper functions
│   └── search_utils.py        # Core search algorithms
│
├── prompts/                    # LLM prompt templates
│   ├── lib.py                 # Prompt library
│   ├── models.py              # Prompt data models
│   ├── prompt_helpers.py      # Prompt utilities
│   ├── extract_nodes.py       # Node extraction prompts
│   ├── extract_edges.py       # Edge extraction prompts
│   ├── extract_edge_dates.py  # Date extraction prompts
│   ├── dedupe_nodes.py        # Node deduplication prompts
│   ├── dedupe_edges.py        # Edge deduplication prompts
│   ├── invalidate_edges.py    # Edge invalidation prompts
│   ├── summarize_nodes.py     # Node summarization prompts
│   └── eval.py                # Evaluation prompts
│
├── models/                     # Database query models
│   ├── nodes/
│   │   ├── node_db_queries.py # Node database queries
│   └── edges/
│       └── edge_db_queries.py # Edge database queries
│
├── utils/                      # Utility modules
│   ├── bulk_utils.py          # Bulk processing utilities
│   ├── datetime_utils.py      # DateTime handling
│   ├── maintenance/           # Graph maintenance operations
│   │   ├── community_operations.py # Community clustering
│   │   ├── edge_operations.py      # Edge processing
│   │   ├── node_operations.py      # Node processing
│   │   ├── temporal_operations.py  # Time-based operations
│   │   ├── graph_data_operations.py # Schema operations
│   │   └── utils.py               # Maintenance utilities
│   └── ontology_utils/
│       └── entity_types_utils.py  # Entity type validation
│
└── telemetry/                  # Usage tracking
    └── telemetry.py           # Telemetry implementation
```

## Conclusion

The architecture balances flexibility with performance, providing a robust foundation for temporal knowledge graph applications while maintaining extensibility for future enhancements. The modular design enables easy testing, provider switching, and customization while the AI-first approach leverages modern LLM capabilities for intelligent graph construction and retrieval.