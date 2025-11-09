# Research Document: Vector and SQL Database Selection

**Research Date**: 2025-11-01
**Updated**: 2025-11-09 (docling-rag-agent uses PostgreSQL + pgvector)
**Feature**: 001-ethnobotany-vector-db
**Purpose**: Evaluate databases for ethnobotany system

**Reference Architecture Update**: [docling-rag-agent](https://github.com/coleam00/ottomator-agents/tree/main/docling-rag-agent) uses **PostgreSQL 15+ with pgvector extension only** - no separate vector database. Proven to handle production workloads efficiently with:
- Single database for metadata + vectors (simplified ops)
- `<=>` cosine distance operator for similarity search
- Custom `match_chunks()` function with threshold filtering
- asyncpg connection pooling (2-10 connections)
- **Recommended for MVP and Production**: Start with pgvector, only add specialized vector DB if performance requires it

---

## Executive Summary

**docling-rag-agent Choice**: **PostgreSQL + pgvector only**
- Single database simplicity
- Production-proven performance
- ACID guarantees + vector search
- **Recommended approach**: Follow reference implementation

**Original Research** (still valid for future scaling):

- **For metadata + full-text search**: PostgreSQL 15+ with pgvector (single database, minimal ops)
- **For semantic search at scale**: Qdrant (cost-effective open-source) OR Weaviate (feature-rich)
- **Hybrid approach** (if needed later): PostgreSQL (primary) + Qdrant (vector search)

---

## Part 1: Vector Databases

### 1. Qdrant (Recommended for this project)

**Type**: Open-source vector database, self-hosted or managed

**Deployment Options**:
- Self-hosted (Docker, Kubernetes)
- Managed cloud service (Qdrant Cloud)
- Hybrid (self-hosted with cloud option)

**Key Characteristics**:
- **Vector Capacity**: 0 to billions of vectors
- **Latency**: 20-50ms p50 for typical queries
- **Vector Dimension**: Supports 768+ (perfect for SPECTER2)
- **Scalability**: Horizontal scaling with sharding
- **Language**: Rust-based (high performance)
- **License**: AGPL 3.0 (commercial license available)
- **API**: REST + gRPC

**Strengths**:
- ✅ Excellent cost-to-performance ratio
- ✅ Self-hosted option (no vendor lock-in)
- ✅ Active development and community
- ✅ Payload filtering (store metadata with vectors)
- ✅ Sub-50ms query latency
- ✅ Point-in-time snapshots and disaster recovery
- ✅ Docker-native (easy containerization)
- ✅ **Estimated cost: $9/month for 50k vectors (Qdrant Cloud)**

**Limitations**:
- ❌ Smaller ecosystem than Pinecone
- ❌ Self-hosting requires operational expertise
- ❌ AGPL license requires careful review for commercial use

**Pricing (Qdrant Cloud)**:
- Free tier: 1GB storage
- Starter: $9/month (~50k vectors)
- Professional: $99/month (~500k vectors)
- Enterprise: Custom pricing

**Performance**:
- Milvus-comparable latency, better than Weaviate for large-scale
- Excellent on MTEB benchmarks
- Native support for 768-dim vectors

**Recommendation**: **PRIMARY CHOICE** for vector search due to cost, performance, and self-hosting flexibility.

---

### 2. Weaviate (Alternative)

**Type**: Open-source vector database, self-hosted or managed

**Key Characteristics**:
- **Vector Capacity**: Billions
- **Latency**: 50-100ms p50 (slower than Qdrant)
- **Languages**: GraphQL, REST, Python client
- **License**: BSL 1.1 (source-available, not pure open source)
- **Ecosystem**: Large community, many integrations

**Strengths**:
- ✅ Feature-rich (built-in generative search, hybrid search)
- ✅ GraphQL API (powerful queries)
- ✅ Hybrid search (vector + keyword) built-in
- ✅ Multi-tenancy support
- ✅ Active development
- ✅ Good documentation

**Limitations**:
- ❌ Slower latency than Qdrant (Weaviate somewhat higher)
- ❌ Higher operational complexity
- ❌ Storage-based pricing (can be expensive at scale)
- ❌ BSL license (not fully open source)
- ❌ Heavier resource footprint

**Pricing (Weaviate Cloud)**:
- Free tier: 50MB storage
- Base: $150/month (50GB storage)
- Significantly more expensive than Qdrant for same capacity

**Recommendation**: Consider if you need built-in hybrid search or advanced features, but **Qdrant is more cost-effective**.

---

### 3. Milvus (High-Performance Option)

**Type**: Open-source vector database, self-hosted only

**Key Characteristics**:
- **Vector Capacity**: Billions with sharding
- **Latency**: <10ms p50 (fastest)
- **GPU Support**: Native GPU acceleration
- **License**: AGPL 3.0 (open source)
- **Infrastructure**: Requires Kubernetes for production

**Strengths**:
- ✅ Best-in-class latency performance
- ✅ GPU acceleration for extreme scale
- ✅ Excellent for billion+ vector scenarios
- ✅ Cloud version (Zilliz Cloud) available

**Limitations**:
- ❌ Complex operational requirements (Kubernetes)
- ❌ Steeper learning curve
- ❌ Overkill for 50k articles
- ❌ Higher resource consumption

**Pricing**:
- Self-hosted: Free (operational costs only)
- Zilliz Cloud: $60/month starter tier

**Recommendation**: **NOT RECOMMENDED** for initial MVP. Too much infrastructure complexity for 50k articles. Qdrant provides 80% of performance with 20% of operational burden.

---

### 4. Pinecone (Managed Solution)

**Type**: Fully managed vector database (SaaS only)

**Key Characteristics**:
- **Latency**: 20-50ms p50
- **Scalability**: Automatic (unlimited)
- **API**: REST, Python SDK
- **No Infrastructure**: Fully managed
- **License**: Proprietary

**Strengths**:
- ✅ Zero operational burden
- ✅ Automatic scaling
- ✅ Reliable SLAs
- ✅ Good for teams without DevOps
- ✅ Strong community

**Limitations**:
- ❌ **Expensive**: $0.10 per 1M vectors/month + API costs
- ❌ Vendor lock-in (proprietary)
- ❌ Free tier very limited (1M vectors)
- ❌ Less control over data

**Estimated Cost for 50k articles**:
- With 768-dim vectors: ~$5-15/month storage
- Plus API request costs (0.001-0.01 per request)
- Estimated total: **$20-50/month**

**Recommendation**: Only if team prefers zero operations and budget is not constrained. Qdrant offers better value.

---

## Part 2: SQL Databases

### 1. PostgreSQL + pgvector (Recommended)

**Type**: Relational database + vector extension

**Key Characteristics**:
- **Base**: PostgreSQL 15+
- **Extension**: pgvector (open source, Apache 2.0)
- **Vector Support**: Storage + similarity search via pgvector
- **Text Search**: Full-text search built-in
- **License**: PostgreSQL License (permissive open source)
- **Availability**: Self-hosted or managed services (AWS RDS, Azure Database, DigitalOcean, Render, etc.)

**Strengths**:
- ✅ Single database for both metadata + vectors + text search
- ✅ Powerful query capabilities (combine vector + keyword + structured)
- ✅ ACID transactions (data integrity)
- ✅ Mature ecosystem (30+ years)
- ✅ Excellent documentation
- ✅ No vendor lock-in
- ✅ pgvector well-maintained (GitHub: pgvector/pgvector)
- ✅ Hybrid search in single query (filter by metadata, then vector similarity)
- ✅ Perfect for articles metadata: title, authors, year, DOI, etc. + full-text search

**Limitations**:
- ❌ Slower vector search than dedicated vector DBs (20-100ms vs <20ms)
- ❌ pgvector performance degrades with billions of vectors
- ❌ Requires tuning for optimal performance
- ❌ Vector operations are secondary (not primary purpose)

**Typical Performance**:
- Indexed pgvector: ~50-200ms for similarity search on 50k vectors
- Full-text search on 50k articles: <100ms
- Combined query (filter + vector): 100-300ms

**Setup**:
```bash
# In PostgreSQL 15+
CREATE EXTENSION pgvector;
CREATE TABLE articles (
  id SERIAL PRIMARY KEY,
  title TEXT NOT NULL,
  authors TEXT[],
  year INTEGER,
  abstract TEXT,
  doi TEXT UNIQUE,
  embedding vector(768),
  INDEXED BY (embedding)
);
```

**Cost** (Self-hosted Docker):
- Free (open source)
- Storage: Standard disk costs (~100MB for 50k articles + metadata)

**Cost** (Managed Service):
- AWS RDS: $15-30/month (db.t3.small)
- DigitalOcean: $15/month
- Render: $7-20/month depending on storage
- Azure Database: $20-50/month

**Recommendation**: **PRIMARY DATABASE for all metadata and full-text search**. Use alongside Qdrant for specialized vector operations if needed.

---

### 2. MongoDB (Not Recommended)

**Type**: Document database with vector search (new in 2024)

**Why Not**:
- ❌ Vector support is new and not production-hardened
- ❌ Less suitable for structured article metadata
- ❌ No full-text search as good as PostgreSQL
- ❌ Operational complexity
- ✓ Only consider if you already use MongoDB

---

## Part 3: Recommended Architecture

### Option A: PostgreSQL + pgvector (RECOMMENDED FOR MVP)

**Architecture**:
```
┌─────────────────────┐
│   Upload Interface  │
│    (Web UI/API)     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Backend Service    │
│  (Python/Node.js)   │
└──────────┬──────────┘
           │
           ├─────────────────────────┐
           │                         │
           ▼                         ▼
┌────────────────────────┐   ┌──────────────────┐
│   PostgreSQL 15+       │   │ SPECTER2 Model   │
│  (Article Metadata)    │   │  (GPU or CPU)    │
│  (Full-text Search)    │   └──────────────────┘
│  (pgvector Extension)  │          │
│  (768-dim Embeddings)  │◄─────────┘
└────────────────────────┘
```

**Strengths**:
- ✅ Single database (minimal ops)
- ✅ Hybrid search capabilities
- ✅ Cost-effective (<$20/month)
- ✅ Full control over data
- ✅ Docker-deployable
- ✅ Perfect for 50k articles

**Latency**: ~100-300ms for combined queries (acceptable for MVP)

**When to use**: Initial MVP, small to mid-scale (up to 500k articles)

**Cost**: $0-30/month depending on hosting

---

### Option B: PostgreSQL + Qdrant (RECOMMENDED FOR PRODUCTION)

**Architecture**:
```
┌─────────────────────┐
│   Upload Interface  │
│    (Web UI/API)     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Backend Service    │
│  (Python/Node.js)   │
└────┬────────────┬───┘
     │            │
     ▼            ▼
┌──────────────┐ ┌──────────────┐
│PostgreSQL    │ │   Qdrant     │
│(Metadata)    │ │  (Vectors)   │
│(Full-text)   │ │  (Search)    │
└──────────────┘ └──────────────┘
```

**Strengths**:
- ✅ Best-of-breed: PostgreSQL for metadata, Qdrant for vectors
- ✅ Dedicated vector DB performance (<50ms queries)
- ✅ Both open-source, no vendor lock-in
- ✅ Excellent scalability (Qdrant scales to billions)
- ✅ Docker-Compose deployable
- ✅ Cost-effective ($9-15/month if using Qdrant Cloud)

**Latency**: ~50-100ms for search (faster than pgvector alone)

**When to use**: Production deployment, high-volume searches, future scaling

**Cost**: $0-25/month (self-hosted) or $10-25/month (Qdrant Cloud + PostgreSQL)

---

### Option C: PostgreSQL + Pinecone (Not Recommended)

**Architecture**: PostgreSQL + Pinecone (managed vector DB)

**Why Not**:
- ❌ Significantly more expensive ($50+/month vs $15)
- ❌ Vendor lock-in
- ❌ Less control over data
- ✓ Only if team has budget and wants zero operations

**Cost**: $30-100/month

---

## Decision Matrix

| Aspect | PostgreSQL + pgvector | PostgreSQL + Qdrant | Pinecone |
|--------|----------------------|---------------------|----------|
| **Cost (50k articles)** | $5-30/month | $10-25/month | $50-100/month |
| **Operational Burden** | Low | Medium | None |
| **Vector Search Latency** | 50-200ms | 20-50ms | 20-50ms |
| **Full-text Search** | Excellent | Excellent | None (requires separate DB) |
| **Metadata Query** | Excellent | Excellent | Requires separate DB |
| **Scalability** | Up to 500k vectors | Up to billions | Unlimited |
| **Vendor Lock-in** | None | None | High |
| **Self-hosting** | Yes | Yes | No |
| **Learning Curve** | Low | Medium | Low |

---

## Recommendation for Ethnobotany System

### Phase 1 (MVP): PostgreSQL + pgvector
- Simple, cost-effective, all-in-one
- Good for validating system before scaling
- Sub-1 second queries acceptable for MVP

### Phase 2 (Production): Add Qdrant
- Separate vector DB for better performance
- Keep PostgreSQL for metadata
- Both open-source, cost-effective
- Docker-Compose makes deployment easy

### Phase 3 (Hyper-scale): Consider Milvus
- Only if you have 10M+ vectors
- Requires Kubernetes expertise
- For this use case, unlikely needed

---

## Database Schema Outline (PostgreSQL)

```sql
-- Core articles table
CREATE TABLE articles (
  id SERIAL PRIMARY KEY,
  title TEXT NOT NULL,
  authors TEXT[],
  year INTEGER,
  publication_venue TEXT,
  abstract TEXT,
  doi TEXT UNIQUE,
  url TEXT,
  pdf_url TEXT,
  embedding vector(768),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  extraction_status VARCHAR(20) DEFAULT 'pending'
);

-- Create indexes
CREATE INDEX ON articles USING GIN(authors);
CREATE INDEX ON articles USING GIST(embedding);
CREATE INDEX ON articles(year);
CREATE INDEX ON articles(doi);

-- Full-text search
ALTER TABLE articles ADD COLUMN search_vector tsvector;
CREATE INDEX articles_search_idx ON articles USING GIN(search_vector);

-- Metadata for CARE principles
CREATE TABLE community_metadata (
  article_id INTEGER REFERENCES articles(id),
  community_name TEXT,
  approval_status VARCHAR(20),
  notes TEXT
);

-- User interactions (for recommendations)
CREATE TABLE search_queries (
  id SERIAL PRIMARY KEY,
  user_id INTEGER,
  query_text TEXT,
  timestamp TIMESTAMP DEFAULT NOW(),
  results_count INTEGER
);

-- Community annotations
CREATE TABLE annotations (
  id SERIAL PRIMARY KEY,
  article_id INTEGER REFERENCES articles(id),
  user_id INTEGER,
  annotation_text TEXT,
  annotation_type VARCHAR(50),
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## Deployment Considerations

### Docker Setup (Recommended)

```yaml
version: '3.8'
services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: ethnobotany
      POSTGRES_PASSWORD: secure_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  # Optional for production
  qdrant:
    image: qdrant/qdrant
    ports:
      - "6333:6333"
    volumes:
      - qdrant_data:/qdrant/storage
```

### Backup & Disaster Recovery

- **PostgreSQL**: pg_dump for backups, point-in-time recovery
- **Qdrant**: Snapshot functionality, automated backups in Qdrant Cloud

---

## Cost Summary

### Total Monthly Cost (50k articles, production)

| Component | Cost |
|-----------|------|
| PostgreSQL (managed) | $15-25 |
| Qdrant Vector DB | $9 (free self-hosted) |
| Docker hosting | $5-20 |
| **Total** | **$29-54** (or $5-20 if self-hosted) |

---

## Final Recommendation

**For ethnobotany system**:
1. **Start with**: PostgreSQL + pgvector (MVP)
2. **Add when needed**: Qdrant for dedicated vector search
3. **Avoid**: Pinecone (expensive), Milvus (overkill for initial scale)
4. **Avoid**: MongoDB, unproven vector features

Both PostgreSQL and Qdrant are open-source, cost-effective, and deployable in Docker, aligning with the project's architectural goals.
