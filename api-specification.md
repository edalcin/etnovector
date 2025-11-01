# API Specification: Ethnobotany Vectorized Database

**Created**: 2025-11-01
**Status**: Specification Complete ✅
**Base URL**: `https://api.ethnobotany.local/v1` (MVP) / Production TBD
**Related**:
  - [Feature Specification](./spec.md)
  - [Data Model](./data-model.md)
  - [Implementation Plan](./plan.md)

---

## Authentication

All endpoints (except `/health`) require Bearer token authentication.

```
Authorization: Bearer <jwt_token>
```

---

## Article Management Endpoints

### POST /articles/upload

**Upload PDF article for ingestion**

Request:
```
Content-Type: multipart/form-data

file: <PDF file, max 50MB>
metadata: {optional JSON with title, authors, year, abstract}
```

Response (200 OK):
```json
{
  "id": 12345,
  "doi": null,
  "title": "Traditional Use of Medicinal Plants in the Amazon",
  "authors": ["Silva, J.", "Santos, M."],
  "year": 2023,
  "ingestion_method": "pdf_upload",
  "embedding_status": "processing",
  "created_at": "2025-11-01T10:30:00Z",
  "source_url": null
}
```

Errors:
- 400: File too large, unsupported format
- 409: Duplicate detected (same DOI/title)
- 422: Invalid metadata format

---

### POST /articles/submit-url

**Submit article via URL for cataloging**

Request:
```json
{
  "url": "https://doi.org/10.1234/example",
  "url_type": "doi",
  "manual_metadata": {
    "title": "Optional override",
    "authors": ["Author1", "Author2"],
    "year": 2023,
    "abstract": "Optional abstract override"
  }
}
```

`url_type` values: `doi`, `journal_url`, `arxiv`, `researchgate`, `custom`

Response (200 OK):
```json
{
  "id": 12346,
  "doi": "10.1234/example",
  "title": "Plant Properties in Traditional Medicine",
  "authors": ["Nakamura, K.", "Perez, L."],
  "year": 2022,
  "abstract": "A study of ethnobotanical knowledge...",
  "ingestion_method": "url_fetch",
  "embedding_status": "processing",
  "source_url": "https://doi.org/10.1234/example",
  "source_type": "doi_link",
  "metadata_confidence": 0.92,
  "created_at": "2025-11-01T10:35:00Z"
}
```

---

### GET /articles/{id}

**Retrieve article metadata**

Response (200 OK):
```json
{
  "id": 12345,
  "doi": "10.1234/example",
  "title": "Traditional Use of Medicinal Plants",
  "authors": ["Silva, J.", "Santos, M."],
  "year": 2023,
  "publication_venue": "Journal of Ethnobotany",
  "abstract": "This study examines...",
  "keywords": ["ethnobotany", "Amazon", "medicinal plants"],
  "language": "en",
  "source_url": "https://example.org/article",
  "source_type": "journal_url",
  "ingestion_method": "url_fetch",
  "metadata_confidence": 0.95,
  "embedding_status": "completed",
  "communities": [
    {
      "name": "Kayapó",
      "validation_status": "approved",
      "knowledge_origin": "Traditional use of rainforest plants"
    }
  ],
  "created_at": "2025-11-01T10:30:00Z",
  "created_by": "user123"
}
```

---

### GET /articles

**List articles with filtering and pagination**

Query Parameters:
```
?page=1
&limit=20
&year_from=2000
&year_to=2025
&language=en
&region=Amazonia
&search=inflammation  (full-text search)
&sort_by=created_at|relevance
&sort_order=asc|desc
```

Response (200 OK):
```json
{
  "total": 2543,
  "page": 1,
  "limit": 20,
  "items": [
    {
      "id": 12345,
      "title": "Traditional Use of Medicinal Plants",
      "authors": ["Silva, J.", "Santos, M."],
      "year": 2023,
      "abstract": "This study examines...",
      "source_url": "https://example.org",
      "communities": ["Kayapó", "Xingu"]
    }
    // ... more articles
  ]
}
```

---

## Search Endpoints

### POST /search/semantic

**Semantic search using vector similarity**

Request:
```json
{
  "query": "plants used for inflammation in Amazonian communities",
  "limit": 10,
  "similarity_threshold": 0.7,
  "filters": {
    "year_from": 2000,
    "region": "Amazonia",
    "language": "en"
  }
}
```

Response (200 OK):
```json
{
  "query": "plants used for inflammation",
  "results": [
    {
      "id": 12345,
      "title": "Traditional Use of Medicinal Plants",
      "authors": ["Silva, J."],
      "similarity_score": 0.89,
      "excerpt": "...inflammation remedies used by Kayapó communities...",
      "source_url": "https://example.org"
    },
    {
      "id": 12346,
      "title": "Ethnopharmacology of Amazon Plants",
      "authors": ["Chen, L."],
      "similarity_score": 0.84,
      "excerpt": "...anti-inflammatory properties of rainforest species...",
      "source_url": "https://example.org"
    }
  ],
  "execution_time_ms": 145
}
```

---

### POST /search/hybrid

**Hybrid search combining full-text + semantic**

Request:
```json
{
  "query": "anti-parasitic properties",
  "text_weight": 0.4,
  "semantic_weight": 0.6,
  "limit": 15
}
```

Response: Same format as semantic search with combined scoring.

---

## Chat & Synthesis Endpoints

### WebSocket /ws/chat

**Real-time chat with LLM synthesis**

Client Message:
```json
{
  "type": "message",
  "content": "What plants are studied for anti-inflammatory properties in Amazonia?",
  "llm_provider": "claude",
  "context_articles": [12345, 12346]
}
```

Server Response (streaming):
```
🤖: Based on the search results, several Amazonian plants are studied for anti-inflammatory properties:

1. Copaifera spp. (Copaiba) - Used traditionally...
2. Plectranthus amboinicus - Studies show...

[continues streaming until complete]
```

Response metadata (final message):
```json
{
  "type": "response_complete",
  "articles_cited": [12345, 12346, 12348],
  "sources": [
    {"id": 12345, "title": "Traditional Use...", "doi": "10.1234/ex1"},
    {"id": 12346, "title": "Ethnopharmacology...", "doi": "10.1234/ex2"}
  ],
  "tokens_used": {
    "input": 1250,
    "output": 380
  },
  "cost_usd": 0.0125
}
```

---

## Recommendations Endpoint

### GET /articles/{id}/recommendations

**Get related articles based on semantic similarity**

Query Parameters:
```
?limit=10
&exclude_authors=true  (don't recommend same author)
&cross_regional=true   (recommend from other regions)
&min_similarity=0.7
```

Response (200 OK):
```json
{
  "article_id": 12345,
  "recommendations": [
    {
      "id": 12347,
      "title": "Plant properties across regions",
      "authors": ["Johnson, K."],
      "similarity_score": 0.82,
      "reason": "Studies same plant family (Moraceae) with similar anti-inflammatory focus",
      "region_overlap": "Cross-regional (Cerrado vs Amazon)"
    },
    {
      "id": 12348,
      "title": "Medicinal ethnopharmacology",
      "authors": ["Wang, L."],
      "similarity_score": 0.78,
      "reason": "Same region (Amazonia), different plants with overlapping therapeutic properties"
    }
  ]
}
```

---

## Trends & Analytics Endpoints

### GET /trends/plants

**Get plant study frequency and trends**

Query Parameters:
```
?limit=20
&region=Amazonia
&time_period=last_5_years
&metric=study_count|publication_trend
```

Response (200 OK):
```json
{
  "metric": "study_count",
  "region": "Amazonia",
  "period": "2020-2025",
  "data": [
    {
      "plant_name": "Copaifera spp.",
      "scientific_name": "Copaifera officinalis",
      "study_count": 45,
      "recent_studies": 12,
      "properties_studied": ["anti-inflammatory", "antimicrobial"],
      "trend": "increasing"
    },
    {
      "plant_name": "Plectranthus amboinicus",
      "scientific_name": "Plectranthus amboinicus",
      "study_count": 23,
      "recent_studies": 8,
      "properties_studied": ["digestive", "antimicrobial"],
      "trend": "stable"
    }
  ]
}
```

---

### GET /trends/gaps

**Identify research gaps**

Query Parameters:
```
?region=Northeast_Brazil
&plant_family=Fabaceae
&property=anti-parasitic
```

Response (200 OK):
```json
{
  "gaps": [
    {
      "gap_description": "Anti-parasitic plants in Northeast Brazil poorly studied",
      "region": "Northeast Brazil",
      "plant_families": ["Fabaceae", "Rubiaceae"],
      "properties": ["anti-parasitic"],
      "current_studies": 3,
      "potential_species": ["Alchornea", "Justicia"],
      "recommendation": "Focus on historically used species with <5 publications"
    },
    {
      "gap_description": "Female-predominant traditional knowledge systems underrepresented",
      "region": "All",
      "plant_families": null,
      "properties": null,
      "current_studies": 0,
      "recommendation": "Partner with women's knowledge keepers to document traditional use"
    }
  ]
}
```

---

## CARE Principles Endpoints

### GET /communities

**List communities and approval status**

Response (200 OK):
```json
{
  "communities": [
    {
      "id": 1,
      "name": "Kayapó",
      "region": "Xingu Basin",
      "country": "Brazil",
      "approval_status": "approved",
      "articles_count": 145,
      "articles_approved": 128,
      "contact_email": "kayapo.research@example.org"
    },
    {
      "id": 2,
      "name": "Asháninka",
      "region": "Peruvian Amazon",
      "country": "Peru",
      "approval_status": "pending",
      "articles_count": 32,
      "articles_approved": 0,
      "contact_email": "ashaninka.knowledge@example.org"
    }
  ]
}
```

---

### POST /articles/{id}/communities

**Tag article with associated community knowledge**

Request:
```json
{
  "community_id": 1,
  "knowledge_origin": "Traditional use of Copaifera resin for wound healing",
  "notes": "Knowledge validated by community elder Council"
}
```

Response (200 OK):
```json
{
  "article_id": 12345,
  "community_id": 1,
  "knowledge_origin": "Traditional use of Copaifera resin",
  "validation_status": "pending",
  "created_at": "2025-11-01T11:00:00Z"
}
```

---

### POST /communities/{id}/approve-articles

**Community representative approves knowledge attribution**

Request:
```json
{
  "article_ids": [12345, 12346, 12347],
  "approval_notes": "Approved - knowledge accurately represents Kayapó traditions"
}
```

Response (200 OK):
```json
{
  "community_id": 1,
  "articles_approved": 3,
  "approval_date": "2025-11-01T11:15:00Z",
  "approval_notes": "Approved - knowledge accurately represents Kayapó traditions"
}
```

---

## User Settings Endpoints

### GET /users/settings

**Retrieve user preferences**

Response (200 OK):
```json
{
  "user_id": "user123",
  "preferred_llm": "claude",
  "language_preference": "pt",
  "theme": "dark",
  "notifications_enabled": true,
  "api_keys_configured": {
    "claude": true,
    "gemini": false,
    "chatgpt": true
  },
  "monthly_api_spend": 15.45,
  "monthly_api_budget": 50.00
}
```

---

### POST /users/settings/llm-keys

**Update LLM API keys (encrypted)**

Request:
```json
{
  "provider": "gemini",
  "api_key": "sk_...",
  "test_connection": true
}
```

Response (200 OK):
```json
{
  "provider": "gemini",
  "status": "configured",
  "test_result": "success",
  "last_tested": "2025-11-01T11:20:00Z"
}
```

---

## Health & Monitoring Endpoints

### GET /health

**Health check (no authentication required)**

Response (200 OK):
```json
{
  "status": "healthy",
  "timestamp": "2025-11-01T11:30:00Z",
  "components": {
    "database": "healthy",
    "embeddings_service": "healthy",
    "llm_apis": {
      "claude": "reachable",
      "gemini": "reachable",
      "chatgpt": "reachable"
    },
    "vector_db": "healthy"
  }
}
```

---

### GET /metrics

**System metrics (requires admin role)**

Response (200 OK):
```json
{
  "articles_total": 50000,
  "articles_indexed": 49853,
  "embeddings_pending": 147,
  "avg_query_time_ms": 145,
  "p95_query_time_ms": 380,
  "concurrent_users": 23,
  "uptime_hours": 720,
  "api_calls_today": 12543
}
```

---

## Error Responses

All errors follow this format:

```json
{
  "error": {
    "code": "INVALID_INPUT",
    "message": "The provided DOI is invalid",
    "details": {
      "field": "doi",
      "expected_format": "10.xxxx/xxxx"
    }
  },
  "timestamp": "2025-11-01T11:35:00Z",
  "request_id": "req_abc123xyz"
}
```

Common error codes:
- `INVALID_INPUT`: Validation error
- `UNAUTHORIZED`: Missing/invalid auth
- `FORBIDDEN`: User lacks permission
- `NOT_FOUND`: Resource doesn't exist
- `CONFLICT`: Duplicate resource
- `RATE_LIMITED`: Too many requests
- `INTERNAL_ERROR`: Server error

---

## Rate Limiting

- Unauthenticated: 10 requests/hour
- Authenticated user: 1000 requests/hour
- Embedding generation: 100 articles/hour per user
- Search queries: 500 queries/hour per user

Headers:
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 987
X-RateLimit-Reset: 2025-11-01T12:30:00Z
```

---

## API Versioning

Current version: `v1`

Future versions will be available at `/v2`, etc. with backwards compatibility period.

---

## Webhook Events (Future)

Future implementation for real-time event notifications:
- `article.ingested`
- `article.embedded`
- `community.approved`
- `search.trending_query`
- `recommendation.generated`
