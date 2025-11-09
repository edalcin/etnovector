# Data Model: Ethnobotany Vectorized Database

**Created**: 2025-11-01
**Updated**: 2025-11-09 (Aligned with docling-rag-agent schema)
**Reference Architecture**: [docling-rag-agent](https://github.com/coleam00/ottomator-agents/tree/main/docling-rag-agent) - PostgreSQL + pgvector schema

**Documentation**:
  - [Feature Specification](./spec.md)
  - [Updated Specification](./specs/001-docling-rag-update/spec.md)
  - [Implementation Plan](./plan.md)
  - [API Specification](./api-specification.md)

**Schema Reference**: Based on docling-rag-agent `documents` and `chunks` tables with ethnobotany-specific extensions

---

## Entity Relationship Diagram

```
Articles
  ├─ 1:1 → Embeddings
  ├─ 1:N → Article_Communities
  ├─ 1:N → Annotations
  └─ 1:N → Search_Queries

Communities
  └─ 1:N → Article_Communities

Users
  ├─ 1:1 → User_Settings
  ├─ 1:N → Search_Queries
  └─ 1:N → Annotations
```

---

## Core Entities

### 1. Articles

**Purpose**: Core entity storing article metadata and references to original sources

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | BIGINT | PK, auto-increment | Unique identifier |
| doi | VARCHAR(255) | UNIQUE, nullable | Digital Object Identifier (e.g., "10.1234/example") |
| title | TEXT | NOT NULL | Article title |
| authors | TEXT[] | NOT NULL | Array of author names (indexed for searching) |
| year | INTEGER | nullable | Publication year |
| publication_venue | TEXT | nullable | Journal, conference, or publisher name |
| abstract | TEXT | nullable | Article abstract (used for embedding) |
| keywords | TEXT[] | nullable | Author-provided or extracted keywords |
| language | VARCHAR(10) | DEFAULT 'en' | 'en', 'pt', 'es', etc. |
| source_url | TEXT | nullable | Link to original article (journal, DOI, arXiv, ResearchGate) |
| source_type | VARCHAR(50) | nullable | 'doi_link', 'journal_url', 'arxiv', 'researchgate', 'institutional' |
| ingestion_method | VARCHAR(20) | NOT NULL | 'pdf_upload' or 'url_fetch' |
| metadata_confidence | NUMERIC(3,2) | 0.00-1.00 | Confidence score for extracted metadata (0=manual, 1=perfect) |
| embedding_status | VARCHAR(20) | DEFAULT 'pending' | 'pending', 'processing', 'completed', 'failed' |
| created_at | TIMESTAMP | DEFAULT NOW() | Creation timestamp |
| updated_at | TIMESTAMP | DEFAULT NOW() | Last modification timestamp |
| created_by | VARCHAR(255) | nullable | User ID who submitted article |

**Indexes**:
```sql
CREATE INDEX idx_articles_doi ON articles(doi);
CREATE INDEX idx_articles_year ON articles(year);
CREATE INDEX idx_articles_venue ON articles(publication_venue);
CREATE INDEX idx_articles_language ON articles(language);
CREATE INDEX idx_articles_created ON articles(created_at DESC);
CREATE INDEX idx_articles_search ON articles USING GIN(search_vector);
```

**Constraints**:
- Either `doi` OR `source_url` must be present
- `title` is mandatory
- `authors` array must not be empty

---

### 2. Embeddings

**Purpose**: Stores vector embeddings for semantic search

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | BIGINT | PK, auto-increment | Unique identifier |
| article_id | BIGINT | FK→Articles, UNIQUE | Reference to article |
| embedding | vector(768) | NOT NULL | SPECTER2 embedding (768 dimensions) |
| model_version | VARCHAR(50) | NOT NULL | Model identifier (e.g., "specter2-v2") |
| generated_from | VARCHAR(50) | NOT NULL | 'abstract', 'full_text', 'title_abstract' |
| token_count | INTEGER | nullable | Number of tokens used for embedding |
| created_at | TIMESTAMP | DEFAULT NOW() | Generation timestamp |

**Indexes**:
```sql
CREATE INDEX idx_embeddings_article ON embeddings(article_id);
CREATE INDEX idx_embeddings_similarity ON embeddings USING HNSW (embedding vector_cosine_ops);
```

**Notes**:
- One embedding per article (1:1 relationship)
- HNSW index enables efficient similarity search (<50ms for 50k vectors)
- `generated_from` tracks what input was used (important for quality tracking)

---

### 3. Communities

**Purpose**: Track traditional communities and their knowledge domains

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | SERIAL | PK, auto-increment | Unique identifier |
| name | VARCHAR(255) | NOT NULL, UNIQUE | Community name (e.g., "Kayapó", "Asháninka") |
| region | VARCHAR(255) | nullable | Geographic region (e.g., "Xingu Basin", "Peruvian Amazon") |
| country | VARCHAR(100) | nullable | Country (Brazil, Peru, Colombia, etc.) |
| research_domain | VARCHAR(255) | nullable | Focus area (ethnobotany, ethnopharmacology, ethnomedicine) |
| approval_status | VARCHAR(20) | NOT NULL, DEFAULT 'pending' | 'pending', 'approved', 'declined', 'archived' |
| contact_email | VARCHAR(255) | nullable | Community representative contact |
| approval_notes | TEXT | nullable | Notes on approval decision |
| created_at | TIMESTAMP | DEFAULT NOW() | Registration timestamp |
| updated_at | TIMESTAMP | DEFAULT NOW() | Last update timestamp |

**Indexes**:
```sql
CREATE INDEX idx_communities_name ON communities(name);
CREATE INDEX idx_communities_region ON communities(region);
CREATE INDEX idx_communities_status ON communities(approval_status);
```

---

### 4. Article_Communities

**Purpose**: Implements CARE principles by tracking which communities are associated with article knowledge

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | BIGSERIAL | PK, auto-increment | Unique identifier |
| article_id | BIGINT | FK→Articles, NOT NULL | Reference to article |
| community_id | INTEGER | FK→Communities, NOT NULL | Reference to community |
| knowledge_origin | TEXT | nullable | Description of traditional knowledge discussed |
| validation_status | VARCHAR(20) | DEFAULT 'pending' | 'pending', 'approved', 'disputed', 'retracted' |
| validated_by | VARCHAR(255) | nullable | Name of community representative who validated |
| validated_at | TIMESTAMP | nullable | Timestamp of validation |
| notes | TEXT | nullable | Community notes or corrections |

**Constraints**:
```sql
UNIQUE(article_id, community_id)  -- One association per community per article
```

**Indexes**:
```sql
CREATE INDEX idx_article_communities_article ON article_communities(article_id);
CREATE INDEX idx_article_communities_community ON article_communities(community_id);
CREATE INDEX idx_article_communities_status ON article_communities(validation_status);
```

---

### 5. User_Settings

**Purpose**: Stores user preferences and encrypted LLM API keys

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| user_id | VARCHAR(255) | PK | Unique user identifier (from auth system) |
| preferred_llm | VARCHAR(50) | DEFAULT 'claude' | 'claude', 'gemini', or 'chatgpt' |
| api_keys_encrypted | JSONB | nullable | Encrypted JSON: {claude: enc_key, gemini: enc_key, chatgpt: enc_key} |
| language_preference | VARCHAR(10) | DEFAULT 'pt' | 'pt' (Portuguese) or 'en' (English) |
| theme | VARCHAR(20) | DEFAULT 'light' | 'light' or 'dark' UI theme |
| max_monthly_api_spend | NUMERIC(10,2) | nullable | User's self-imposed API spending limit |
| notifications_enabled | BOOLEAN | DEFAULT true | Receive updates on new articles |
| created_at | TIMESTAMP | DEFAULT NOW() | Account creation |
| updated_at | TIMESTAMP | DEFAULT NOW() | Last modification |

**Notes**:
- API keys are encrypted at rest using AES-256
- Only user can decrypt their own keys (key derivation from password)
- Monthly spend tracking helps prevent surprises

---

### 6. Search_Queries

**Purpose**: Tracks user search interactions for analytics and improving recommendations

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | BIGSERIAL | PK, auto-increment | Unique identifier |
| user_id | VARCHAR(255) | FK→User_Settings, nullable | Who performed search |
| query_text | TEXT | NOT NULL | Original search query |
| query_embedding | vector(768) | nullable | Embedding of query (for analytics) |
| results_count | INTEGER | nullable | Number of results returned |
| selected_articles | BIGINT[] | nullable | Articles user clicked/opened |
| llm_provider | VARCHAR(50) | nullable | Which LLM was used for synthesis |
| response_time_ms | INTEGER | nullable | Query response latency |
| created_at | TIMESTAMP | DEFAULT NOW() | Query timestamp |

**Indexes**:
```sql
CREATE INDEX idx_search_queries_user ON search_queries(user_id);
CREATE INDEX idx_search_queries_created ON search_queries(created_at DESC);
```

**Purpose**: Enable analytics like:
- Most popular search terms
- Query latency tracking
- User behavior analysis
- Recommendation improvements

---

### 7. Annotations

**Purpose**: Enables community and researcher feedback/corrections on articles

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| id | BIGSERIAL | PK, auto-increment | Unique identifier |
| article_id | BIGINT | FK→Articles, NOT NULL | Which article is being annotated |
| user_id | VARCHAR(255) | FK→User_Settings, NOT NULL | Who created annotation |
| annotation_type | VARCHAR(50) | NOT NULL | 'correction', 'clarification', 'dispute', 'confirmation' |
| text | TEXT | NOT NULL | Annotation content |
| evidence_url | TEXT | nullable | Link supporting the annotation |
| status | VARCHAR(20) | DEFAULT 'pending' | 'pending', 'reviewed', 'approved', 'rejected' |
| reviewed_by | VARCHAR(255) | nullable | Reviewer (admin or community lead) |
| created_at | TIMESTAMP | DEFAULT NOW() | Annotation timestamp |
| reviewed_at | TIMESTAMP | nullable | Review timestamp |

**Indexes**:
```sql
CREATE INDEX idx_annotations_article ON annotations(article_id);
CREATE INDEX idx_annotations_user ON annotations(user_id);
CREATE INDEX idx_annotations_type ON annotations(annotation_type);
CREATE INDEX idx_annotations_status ON annotations(status);
```

**Annotation Types**:
- **correction**: Factual error in metadata or abstract
- **clarification**: Provides additional context
- **dispute**: Challenges accuracy or appropriateness
- **confirmation**: Validates information (especially from communities)

---

## Data Validation Rules

### Articles
```
- title: 1-500 characters, required
- authors: minimum 1, each 1-200 characters
- year: 1900-current year, or NULL
- abstract: max 5000 characters
- doi: matches regex /^10\.\d{4,}/
- source_url: valid URL format
- language: ISO 639-1 code (en, pt, es, fr, de)
```

### Embeddings
```
- embedding: exactly 768 dimensions (float32)
- model_version: matches /^specter2-v\d+$/
- article_id: must reference existing article
```

### Communities
```
- name: 1-200 characters, unique
- country: ISO 3166-1 alpha-2 code
- approval_status: one of ['pending', 'approved', 'declined', 'archived']
```

### User_Settings
```
- preferred_llm: one of ['claude', 'gemini', 'chatgpt']
- language_preference: one of ['pt', 'en']
- max_monthly_api_spend: >= 0
```

---

## Data Lifecycle

### Article Ingestion Workflow

```
1. Submission (user uploads PDF or provides URL)
   ↓
2. Metadata Extraction
   - PDF: Extract via pdfplumber
   - URL: Fetch via CrossRef/arXiv API
   - Status: 'processing'
   ↓
3. Duplicate Detection
   - Check DOI, title, author+year combinations
   - If found: Notify user, don't proceed
   ↓
4. Metadata Enrichment
   - CrossRef API lookup if DOI present
   - Language detection
   - Create fulltext search vector
   - Status: 'pending_embedding'
   ↓
5. Embedding Generation
   - Input: abstract + title (or full text if available)
   - Process via SPECTER2
   - Store in embeddings table
   - Status: 'completed'
   ↓
6. Community Tagging
   - Identify associated communities (manual or automatic)
   - Create article_communities records
   - Wait for community validation
   ↓
7. Live
   - Article searchable
   - Included in recommendations
   - Available for CARE approval
```

### Data Retention Policy

- **Metadata**: Retained indefinitely
- **Embeddings**: Retained indefinitely
- **Full-text PDFs**: NOT retained (only metadata links)
- **Search queries**: Retained for 90 days (for analytics)
- **Annotations**: Retained indefinitely (with approval trail)
- **Backups**: Daily automated backups, 30-day retention

---

## Search Indexes Summary

| Table | Index Name | Type | Columns | Purpose |
|-------|-----------|------|---------|---------|
| articles | search_vector | GIN (text search) | title, abstract | Full-text search |
| articles | idx_articles_year | BTREE | year | Temporal filtering |
| articles | idx_articles_doi | BTREE | doi | Duplicate detection |
| embeddings | idx_embeddings_similarity | HNSW (vector) | embedding | Semantic similarity |
| search_queries | idx_search_queries_created | BTREE | created_at | Time-based analytics |
| article_communities | idx_article_communities_status | BTREE | validation_status | CARE workflow |

---

## Example Queries

### Find semantically similar articles
```sql
SELECT articles.*
FROM articles
JOIN embeddings ON articles.id = embeddings.article_id
ORDER BY embeddings.embedding <-> (SELECT embedding FROM embeddings WHERE article_id = 123)
LIMIT 10;
```

### Find articles by community knowledge
```sql
SELECT a.*, ac.validation_status
FROM articles a
JOIN article_communities ac ON a.id = ac.article_id
JOIN communities c ON ac.community_id = c.id
WHERE c.name = 'Kayapó' AND ac.validation_status = 'approved'
ORDER BY a.created_at DESC;
```

### Trend analysis: Plants studied by year
```sql
SELECT a.keywords[1] as plant, a.year, COUNT(*) as study_count
FROM articles a
WHERE a.keywords IS NOT NULL
GROUP BY a.keywords[1], a.year
ORDER BY a.year DESC, study_count DESC;
```

### Full-text + semantic hybrid search
```sql
SELECT a.*,
       ts_rank(a.search_vector, query) as text_rank,
       1 - (e.embedding <-> query_embedding) as semantic_score
FROM articles a
JOIN embeddings e ON a.id = e.article_id
WHERE a.search_vector @@ to_tsquery('english', 'inflammation')
  AND e.embedding <-> (SELECT embedding FROM embeddings WHERE article_id = 123) < 0.3
ORDER BY text_rank DESC, semantic_score DESC;
```

---

## Considerations for Scalability

### Indexing Strategy at 50k+ Articles
- HNSW vector index becomes critical (maintains <50ms search)
- Partitioning by year for search_queries (for fast rollover)
- Archive annotations older than 1 year (separate cold storage)

### At 500k+ Articles
- Consider PostgreSQL partitioning by year
- Shard embeddings across multiple databases if needed
- Move to dedicated vector database (Qdrant) alongside PostgreSQL

### At 5M+ Articles
- Separate read replicas for search queries
- Write-optimized primary database for ingestion
- Denormalized analytics tables for trend queries
