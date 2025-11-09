# Implementation Plan: Ethnobotany Vectorized Database

**Created**: 2025-11-01
**Updated**: 2025-11-09 (Added docling-rag-agent reference architecture)
**Status**: Ready for Implementation
**Reference Architecture**: [docling-rag-agent](https://github.com/coleam00/ottomator-agents/tree/main/docling-rag-agent)

**Documentation**:
  - [Feature Specification](./spec.md)
  - [Updated Specification](./specs/001-docling-rag-update/spec.md) - docling-rag-agent patterns integrated
  - [Data Model](./data-model.md)
  - [API Specification](./api-specification.md)
  - Research: [Embeddings](./research/research-embedding-models.md) • [Databases](./research/research-databases.md) • [LLM APIs](./research/research-llm-apis.md)

---

## Technical Context

### Problem Statement

Ethnobotany researchers need a semantic search engine over scientific literature to discover relationships between traditional plant use, regional communities, and research findings that keyword search cannot surface.

### Solution Architecture

**Based on [docling-rag-agent](https://github.com/coleam00/ottomator-agents/tree/main/docling-rag-agent)** - A containerized RAG application that:
- Ingests documents in multiple formats (PDF, DOCX, PPTX, XLSX, HTML, MD, TXT) via Docling
- Stores metadata and vector embeddings in PostgreSQL + pgvector
- Provides chat interface via PydanticAI agent framework with tool-calling
- Supports multi-provider LLMs (OpenAI, Claude, Gemini) with user-provided API keys
- Implements streaming responses and async connection pooling

**Key Design Principles**:
1. **Reference Implementation First**: Follow docling-rag-agent patterns before customizing
2. **No Full-Text Retention**: Metadata + embeddings + source links only
3. **CARE Principles**: Add ethnobotany-specific governance on top of base RAG

---

## Architecture Overview

### System Components

```
┌─────────────────────────────────────────────────────────────────┐
│                     Web Frontend (React/Vue)                     │
│         - Article upload/URL submission interface                │
│         - Chat-based semantic search                             │
│         - Recommendation explorer                                │
│         - Trend analysis dashboard                               │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                    REST/WebSocket
                           │
┌──────────────────────────▼──────────────────────────────────────┐
│                  Backend API (FastAPI/Python)                    │
│  ┌────────────────┬──────────────┬──────────────────────────┐   │
│  │ Article Ingest │ Search Service│ Recommendation Engine  │   │
│  │ - PDF Parser   │ - Semantic    │ - Similarity Analysis  │   │
│  │ - URL Fetcher  │ - Chat LLM    │ - Cross-domain Links   │   │
│  │ - Metadata Ext │  Interface    │                        │   │
│  └────────┬───────┴──────┬────────┴─────────────┬──────────┘   │
│           │              │                      │               │
├───────────┼──────────────┼──────────────────────┼───────────────┤
│    Embedding Service (Python + SPECTER2)       │               │
│    - Batch embedding generation                │               │
│    - Async processing                          │               │
└────────────┬──────────────────────────────────┬─────────────────┘
             │                                  │
   ┌─────────▼────────────┐        ┌─────────────▼────────┐
   │  Vector Database     │        │  SQL Database        │
   │  (PostgreSQL pgvector)       │  (PostgreSQL)        │
   │  - 768-dim embeddings        │  - Article metadata  │
   │  - Similarity search         │  - User preferences  │
   │  - Payload filtering         │  - Search history    │
   └────────────────────┘        └─────────────────────┘
```

### Technology Stack

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| **Frontend** | React 18 + TypeScript | Modern, type-safe, large ecosystem |
| **Backend** | FastAPI (Python 3.11+) | Async-first, excellent for concurrent tasks |
| **Embeddings** | SPECTER2 (Hugging Face) | Scientific-domain optimized, task-adaptive |
| **Vector DB** | PostgreSQL pgvector (MVP) + Qdrant (Prod) | Open-source, cost-effective, reliable |
| **SQL DB** | PostgreSQL 15+ | ACID transactions, full-text search, pgvector |
| **LLM Integration** | LiteLLM + Claude/Gemini/ChatGPT APIs | Multi-provider, unified interface |
| **Article Fetching** | httpx, crossref-commons, arxiv | Async HTTP, native metadata APIs |
| **PDF Processing** | PyPDF2, pdfplumber | Text extraction, metadata parsing |
| **Container** | Docker + Docker Compose | Reproducible deployment, local development |
| **Queue** | Celery + Redis | Async task processing (optional for MVP) |

---

## Data Model & Persistence

### PostgreSQL Schema (Core)

```sql
-- Articles table (core metadata)
CREATE TABLE articles (
  id BIGSERIAL PRIMARY KEY,
  doi VARCHAR(255) UNIQUE,
  title TEXT NOT NULL,
  authors TEXT[] NOT NULL,  -- Array of author names
  year INTEGER,
  publication_venue TEXT,
  abstract TEXT,
  keywords TEXT[],
  source_url TEXT,          -- URL to original article (journal/DOI/arXiv)
  source_type VARCHAR(50),  -- 'doi', 'journal_url', 'arxiv', 'researchgate'
  language VARCHAR(10) DEFAULT 'en',  -- 'en' or 'pt'
  ingestion_method VARCHAR(20),       -- 'pdf_upload' or 'url_fetch'
  metadata_confidence NUMERIC(3,2),   -- 0.00-1.00 extraction quality
  embedding_status VARCHAR(20) DEFAULT 'pending',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),

  -- Full-text search
  search_vector tsvector GENERATED ALWAYS AS (
    to_tsvector('english', COALESCE(title, '') || ' ' ||
               COALESCE(abstract, ''))
  ) STORED
);

CREATE INDEX idx_articles_doi ON articles(doi);
CREATE INDEX idx_articles_year ON articles(year);
CREATE INDEX idx_articles_search ON articles USING GIN(search_vector);

-- Embeddings table
CREATE TABLE embeddings (
  id BIGSERIAL PRIMARY KEY,
  article_id BIGINT UNIQUE REFERENCES articles(id) ON DELETE CASCADE,
  embedding vector(768),  -- SPECTER2 output dimension
  model_version VARCHAR(50),
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_embeddings_similarity ON embeddings USING HNSW (embedding vector_cosine_ops);

-- CARE Principles: Community metadata
CREATE TABLE communities (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL UNIQUE,
  region VARCHAR(255),      -- e.g., "Amazonia", "Cerrado", "Caatinga"
  country VARCHAR(100),
  research_domain VARCHAR(255),  -- e.g., "ethnobotany", "ethnopharmacology"
  approval_status VARCHAR(20),   -- 'pending', 'approved', 'declined'
  notes TEXT
);

CREATE TABLE article_communities (
  id BIGSERIAL PRIMARY KEY,
  article_id BIGINT REFERENCES articles(id) ON DELETE CASCADE,
  community_id INTEGER REFERENCES communities(id),
  knowledge_origin TEXT,  -- Description of traditional knowledge
  validation_status VARCHAR(20),  -- 'pending', 'approved', 'disputed'
  validated_by VARCHAR(255),  -- Community representative name
  validated_at TIMESTAMP,
  UNIQUE(article_id, community_id)
);

-- User interactions
CREATE TABLE search_queries (
  id BIGSERIAL PRIMARY KEY,
  user_id VARCHAR(255),
  query_text TEXT NOT NULL,
  query_embedding vector(768),  -- For analytics
  results_count INTEGER,
  selected_articles INTEGER[],  -- Which articles user clicked
  llm_provider VARCHAR(50),
  created_at TIMESTAMP DEFAULT NOW()
);

-- User preferences & LLM config
CREATE TABLE user_settings (
  user_id VARCHAR(255) PRIMARY KEY,
  preferred_llm VARCHAR(50) DEFAULT 'claude',  -- 'claude', 'gemini', 'chatgpt'
  api_keys_encrypted JSONB,  -- Encrypted: {claude: enc_key, gemini: enc_key, ...}
  language_preference VARCHAR(10) DEFAULT 'pt',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Annotations & community feedback
CREATE TABLE annotations (
  id BIGSERIAL PRIMARY KEY,
  article_id BIGINT REFERENCES articles(id) ON DELETE CASCADE,
  user_id VARCHAR(255),
  annotation_type VARCHAR(50),  -- 'correction', 'clarification', 'dispute', 'confirmation'
  text TEXT NOT NULL,
  evidence_url TEXT,  -- Link supporting annotation
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## Implementation Phases

### Phase 1: Foundation (Weeks 1-3)
**Deliverables**: Working MVP with PDF upload and basic semantic search

#### 1.1 Infrastructure Setup
- [ ] Docker environment (PostgreSQL + pgvector, FastAPI, Redis)
- [ ] Project structure (backend/frontend folder layout)
- [ ] Environment configuration (.env, secrets management)
- [ ] CI/CD pipeline skeleton (GitHub Actions)

#### 1.2 Core Backend Services
- [ ] FastAPI application with basic endpoints
- [ ] PostgreSQL connection and schema initialization
- [ ] SPECTER2 model loading and inference service
- [ ] pgvector index configuration

#### 1.3 Article Ingestion (PDF Upload)
- [ ] PDF upload endpoint with file validation
- [ ] PDF text extraction service (pdfplumber)
- [ ] Metadata extraction from PDF (title, authors, year via heuristics)
- [ ] Async metadata enrichment from CrossRef API (when DOI found)
- [ ] Duplicate detection (by DOI or title similarity)

#### 1.4 Embedding Generation
- [ ] Batch embedding service using SPECTER2
- [ ] Storage in pgvector
- [ ] Async processing queue (Celery optional for MVP)

#### 1.5 Frontend Basics
- [ ] Upload form component
- [ ] Article listing/search results page
- [ ] Basic styling and layout

**Success Metrics**:
- Upload 10 sample PDFs successfully
- Metadata extraction works for 95%+ of uploads
- Embeddings generated and stored
- Basic search UI functional

---

### Phase 2: Search & Chat Interface (Weeks 4-5)
**Deliverables**: Semantic search + LLM-powered chat

#### 2.1 Semantic Search Service
- [ ] Vector similarity search implementation
- [ ] Ranking and filtering logic
- [ ] Full-text search hybrid (SQL + vector)
- [ ] Search result formatting with context

#### 2.2 Chat Interface & LLM Integration
- [ ] WebSocket connection for real-time chat
- [ ] Prompt engineering for article synthesis
- [ ] LiteLLM wrapper for Claude/Gemini/ChatGPT
- [ ] Secure API key storage (encrypted)
- [ ] Response streaming to frontend

#### 2.3 URL-Based Submission
- [ ] URL submission endpoint
- [ ] Article metadata fetching from CrossRef/arXiv/PubMed APIs
- [ ] Fallback web scraping for unsupported sources
- [ ] Manual metadata entry as fallback

#### 2.4 Chat Frontend
- [ ] Message display with streaming
- [ ] Chat history persistence
- [ ] Article reference links
- [ ] Follow-up suggestion UI

**Success Metrics**:
- Semantic search returns relevant results (manual validation)
- Chat responses synthesize article findings correctly
- URL submission works for 90%+ of common sources
- Sub-2 second response times for typical queries

---

### Phase 3: Recommendations & Analytics (Weeks 6-7)
**Deliverables**: Article recommendations + trend analysis dashboard

#### 3.1 Recommendation Engine
- [ ] Co-citation analysis
- [ ] Semantic similarity clustering
- [ ] Regional cross-reference detection
- [ ] Related article ranking

#### 3.2 Trend Analysis Service
- [ ] Plant frequency aggregation
- [ ] Regional research patterns
- [ ] Gap identification (plants, regions, properties)
- [ ] Temporal trends (publication year analysis)

#### 3.3 Analytics Dashboard
- [ ] Plant study frequency visualization
- [ ] Regional distribution maps
- [ ] Research gap heatmaps
- [ ] Trend time-series

**Success Metrics**:
- Recommendations identify 10+ related articles per article
- 70%+ user satisfaction on recommendation relevance
- Gap analysis identifies known underresearched areas

---

### Phase 4: CARE Principles & Monitoring (Weeks 8-9)
**Deliverables**: Community governance + publication monitoring

#### 4.1 CARE Implementation
- [ ] Community registration system
- [ ] Knowledge attribution framework
- [ ] Community approval workflows
- [ ] Audit logging

#### 4.2 Publication Monitoring
- [ ] Scheduled journal API polling (CrossRef, arXiv)
- [ ] Ethnobotany query automation
- [ ] Automatic ingestion pipeline
- [ ] Duplicate detection across sources

#### 4.3 Governance UI
- [ ] Community management dashboard
- [ ] Knowledge attribution display
- [ ] Approval status indicators
- [ ] Usage analytics for communities

**Success Metrics**:
- 100% of articles tagged with community origin
- New publications discovered within 7 days
- Automated ingestion succeeds for 90%+ of discoveries

---

### Phase 5: Polish & Deployment (Weeks 10-11)
**Deliverables**: Production-ready system

#### 5.1 Quality Assurance
- [ ] End-to-end integration tests
- [ ] Load testing (50+ concurrent users)
- [ ] Security audit (API keys, authentication)
- [ ] Performance optimization

#### 5.2 Documentation
- [ ] API documentation (OpenAPI/Swagger)
- [ ] User guides (Portuguese + English)
- [ ] Deployment guide
- [ ] Contributing guidelines

#### 5.3 Deployment
- [ ] Production Docker Compose setup
- [ ] Database backup strategy
- [ ] Monitoring and logging (Prometheus, ELK)
- [ ] Disaster recovery procedures

**Success Metrics**:
- 99.5% uptime target
- All endpoints documented
- Deployable in under 30 minutes

---

## Technology Decisions & Justification

### Why SPECTER2 for Embeddings?

**Decision**: Use SPECTER2 model from Allenai for all article embeddings

**Rationale**:
- Trained on 6M scientific paper triplets across 23 fields (includes botany, medicine, biology)
- Task-adaptive: generates different embeddings for different tasks
- Citation network aware: papers close in citation graph are close in embedding space
- Free and open-source (Apache 2.0)
- Supports future fine-tuning on ethnobotany-specific data

**Alternatives Considered**:
- SciBERT: Simpler but less sophisticated for document-level similarity
- Sentence-Transformers: General purpose, requires fine-tuning for scientific content
- OpenAI Embeddings: Excellent quality but proprietary and recurring API costs ($0.02/1M tokens)

**Recommendation**: Use SPECTER2 for MVP. Switch to fine-tuned models or OpenAI only if user feedback indicates insufficient quality.

---

### Why PostgreSQL + pgvector (MVP) + Qdrant (Production)?

**Decision**:
- **MVP**: Single PostgreSQL database with pgvector extension
- **Production**: PostgreSQL for metadata + Qdrant for dedicated vector search

**Rationale for MVP**:
- Single database reduces operational complexity
- pgvector provides native vector support in familiar SQL
- Sufficient for 50k articles (50-100ms query latency acceptable)
- Cost-effective ($5-30/month for self-hosted or managed)
- Hybrid search in single query (SQL filter + vector similarity)

**Rationale for Production**:
- Qdrant provides sub-20ms query latency
- Scales to billions of vectors if needed
- Separates concerns: PostgreSQL (metadata+transactions) vs Qdrant (vectors)
- Both open-source, zero vendor lock-in
- Cost-effective ($9-15/month Qdrant Cloud + PostgreSQL)

**Alternatives Considered**:
- Milvus: Best performance but requires Kubernetes expertise (overkill for 50k articles)
- Weaviate: Feature-rich but higher operational complexity
- Pinecone: Fully managed but expensive ($50-100+/month) and vendor lock-in

---

### Why LiteLLM for Multi-LLM Support?

**Decision**: Use LiteLLM library to abstract LLM provider differences

**Rationale**:
- Single interface for Claude, Gemini, ChatGPT
- Easy provider switching (configuration change only)
- Built-in error handling and retries
- Cost tracking per provider
- Community-maintained, well-tested

**Implementation Pattern**:
```python
import litellm

response = litellm.completion(
    model=user_settings.preferred_llm,  # "claude-3-5-sonnet" or "gemini-2.5-flash"
    messages=messages,
    temperature=0.7,
    stream=True
)
```

**User Cost Model**: Each user provides their own API keys (encrypted storage)
- Claude: $1-3/month for typical usage
- Gemini: $0.10-0.50/month (cheapest)
- ChatGPT: $3-8/month

**Zero platform API costs** with this model.

---

### Why URL-Based Submission?

**Decision**: Support both PDF upload and article URL submission

**Rationale**:
- PDF upload: For offline articles or when users have local copies
- URL submission: Persistent access to original sources, no storage burden
- Metadata APIs (CrossRef, arXiv): Automated extraction, high success rate
- No full-text retention: Respects publisher rights, minimizes storage

**Implementation**:
```
URL Input (DOI/journal link/arXiv)
  → Fetch metadata via API
  → If API fails → Attempt web scraping
  → If both fail → Allow manual entry
  → Extract abstract for embedding
  → Link to original source
```

**Success Target**: 90%+ of URL submissions processed automatically without manual intervention

---

## Development Workflow

### Git & Branching (Main-Only)
- All work commits to **main** branch
- No feature branches (per user requirement)
- Atomic, descriptive commits with full context

### Code Organization

```
project/
├── backend/
│   ├── app/
│   │   ├── main.py              # FastAPI app entry
│   │   ├── models.py            # Pydantic models
│   │   ├── database.py          # SQLAlchemy setup
│   │   ├── api/
│   │   │   ├── articles.py      # Article ingestion endpoints
│   │   │   ├── search.py        # Search endpoints
│   │   │   ├── recommendations.py
│   │   │   ├── trends.py
│   │   │   └── chat.py          # WebSocket chat
│   │   ├── services/
│   │   │   ├── embedding.py     # SPECTER2 service
│   │   │   ├── pdf_extraction.py
│   │   │   ├── url_fetcher.py   # Article URL fetching
│   │   │   ├── metadata.py      # Extraction & enrichment
│   │   │   ├── search_service.py
│   │   │   └── llm_service.py   # LLM integration
│   │   └── utils/
│   │       ├── crypto.py        # API key encryption
│   │       ├── validators.py
│   │       └── logging.py
│   ├── requirements.txt
│   ├── Dockerfile
│   └── .env.example
│
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── Upload.tsx
│   │   │   ├── SearchChat.tsx
│   │   │   ├── Recommendations.tsx
│   │   │   └── TrendsDashboard.tsx
│   │   ├── services/
│   │   │   ├── api.ts
│   │   │   └── websocket.ts
│   │   └── pages/
│   ├── package.json
│   └── Dockerfile
│
├── docker-compose.yml
├── postgres-init.sql
└── README.md
```

### Testing Strategy

**Unit Tests**:
- Metadata extraction (PDF, URL)
- Embedding service
- Search ranking
- LLM response formatting

**Integration Tests**:
- End-to-end article upload → search
- URL ingestion → embedding
- Chat context handling
- API key encryption/decryption

**Load Tests**:
- 50 concurrent users
- 100 simultaneous searches
- Batch embedding performance

---

## Deployment & Operations

### Docker Compose (Development)

```yaml
version: '3.8'
services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: ethnobotany
      POSTGRES_PASSWORD: dev_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./postgres-init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  backend:
    build: ./backend
    environment:
      DATABASE_URL: postgresql://postgres:dev_password@postgres:5432/ethnobotany
      REDIS_URL: redis://redis:6379
    ports:
      - "8000:8000"
    depends_on:
      - postgres
      - redis

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - backend
```

### Production Deployment

**Recommended Setup**:
- PostgreSQL on managed service (AWS RDS, DigitalOcean, etc.)
- Qdrant Cloud for vector search ($9-15/month)
- Backend on container orchestration (Docker Swarm, Kubernetes minimal)
- Frontend on static hosting (Vercel, Netlify, S3 + CloudFront)
- CloudFlare for CDN and DDoS protection

**Cost Estimate**: $40-80/month for production

---

## Risk Mitigation

| Risk | Impact | Mitigation |
|------|--------|-----------|
| PDF extraction failure on some papers | Blocks article indexing | Implement OCR fallback, allow manual entry |
| URL API rate limits (CrossRef, arXiv) | Slow ingestion | Implement caching, backoff strategy |
| Embedding model quality issues | Poor search results | Allow fine-tuning, fallback to keyword search |
| LLM API outages | Chat unavailable | Fallback to previous responses, offline mode |
| Community approval bottleneck | Slow CARE compliance | Automate binary approvals, escalate complex cases |

---

## Success Metrics & Milestones

| Phase | Milestone | Success Criteria |
|-------|-----------|------------------|
| 1 | MVP Ready | 10 PDFs ingested, search functional |
| 2 | Chat Live | Semantic search + chat working for 50 articles |
| 3 | Analytics Beta | Recommendations engine running, showing insights |
| 4 | CARE Ready | Community approvals implemented, 100% attributed |
| 5 | Production | 50k articles, 50+ concurrent users, 99.5% uptime |

---

## Next Steps

1. **Backend Setup** (Week 1): Initialize FastAPI, PostgreSQL, pgvector
2. **Frontend Foundation** (Week 1): React project setup, basic components
3. **PDF Ingestion** (Week 2): Implement upload and metadata extraction
4. **Embedding Service** (Week 2): Integrate SPECTER2, batch processing
5. **Search API** (Week 3): Vector similarity search endpoints
6. **Chat Interface** (Week 4): WebSocket, LLM integration
7. **URL Submission** (Week 4): CrossRef API integration
8. **Recommendations** (Week 6): Similarity clustering algorithm
9. **Trends Dashboard** (Week 7): Analytics queries and UI
10. **CARE & Monitoring** (Week 8): Community workflows, publication monitoring
11. **Testing & Deployment** (Week 10): End-to-end validation, containerization
12. **Production Launch** (Week 11): Deploy, documentation, onboarding

---

## Conclusion

This plan provides a clear pathway from MVP to production for the ethnobotany vectorized database system. The architecture prioritizes:

1. **Cost-effectiveness**: Open-source models + databases + user-provided LLM keys
2. **Scalability**: From 50k to 50M articles with minimal architectural changes
3. **User value**: Semantic search + recommendations + trend insights
4. **Ethical governance**: CARE principles integrated from the start
5. **Flexibility**: Multi-LLM support, multiple ingestion methods

The 11-week implementation timeline is achievable with a lean team of 2-3 engineers.
