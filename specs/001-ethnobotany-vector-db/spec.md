# Feature Specification: Ethnobotany Scientific Articles Vectorized Database

**Feature Branch**: `001-ethnobotany-vector-db`
**Created**: 2025-11-01
**Status**: Draft
**Input**: User description: Vectorized database of scientific articles on ethnobotany (traditional plant use in Brazilian communities)

## User Scenarios & Testing

### User Story 1 - Ingest and Index Scientific Articles (Priority: P1)

As a **researcher or librarian**, I want to submit scientific articles for cataloging via PDF upload or article URL so that our ethnobotany knowledge base grows and becomes searchable without requiring the system to retain article files.

**Why this priority**: This is the foundation of the entire system. Without indexed articles, no search or analysis features can function. This is the core MVP - a functional knowledge base. Supporting both upload (for existing PDFs) and URL (for persistent access) maximizes accessibility.

**Independent Test**: Can be fully tested by ingesting an article via either upload or URL, verifying metadata extraction, SQL storage, and vector embedding creation. Delivers immediate value: a single searchable article in the system.

**Acceptance Scenarios**:

1. **Given** a valid PDF file with scientific article content, **When** user uploads it via the web interface, **Then** the system extracts metadata (title, authors, year, abstract, DOI), stores metadata + embeddings in database (without retaining PDF), and creates vector embeddings for semantic search
2. **Given** an article URL (journal link, DOI, arXiv, ResearchGate), **When** user submits the URL, **Then** the system fetches metadata from the source (via APIs or web scraping), extracts content, stores metadata + embeddings, and links to original source without storing the full PDF
3. **Given** duplicate article detection is enabled, **When** user submits an article already in the system (by DOI or title match), **Then** the system prevents duplication and alerts the user
4. **Given** a malformed or non-readable PDF, **When** user attempts to upload it, **Then** the system displays clear error message and suggests remediation (or manual metadata entry)
5. **Given** a URL that cannot be accessed or lacks required metadata, **When** user submits it, **Then** the system allows manual metadata entry with article URL reference
6. **Given** large PDF files (up to 50MB), **When** user uploads them, **Then** the system processes them asynchronously without blocking the interface

---

### User Story 2 - Intelligent Chat-Based Search (Priority: P1)

As a **researcher**, I want to search for articles using natural language queries like "plants used for inflammation in Amazonian communities" so that I can find semantically relevant papers even if they don't contain my exact search terms.

**Why this priority**: This is the key user-facing feature that differentiates this system from traditional keyword search. Directly addresses the core value proposition of vector embeddings.

**Independent Test**: Can be fully tested by running semantic queries and validating that returned articles are semantically relevant to the query intent, even with paraphrased language or different terminology.

**Acceptance Scenarios**:

1. **Given** a populated vector database with articles, **When** user submits query "anti-parasitic properties in traditional medicine", **Then** system returns articles discussing parasitic diseases and relevant plants, ranked by semantic relevance
2. **Given** user query with ambiguous terminology, **When** system processes it, **Then** it handles multiple interpretations and returns broad relevant results with confidence scores
3. **Given** a chat interface, **When** user provides follow-up questions, **Then** system maintains context to refine searches and improve result relevance
4. **Given** integration with LLM APIs (Claude, Gemini, ChatGPT), **When** user toggles agent mode, **Then** system uses selected LLM to summarize and synthesize article findings in response to queries

---

### User Story 3 - Article Recommendation System (Priority: P2)

As a **researcher**, I want to discover related articles beyond obvious keyword matches so that I can find complementary research from different regions or methodologies studying plants with similar properties.

**Why this priority**: Enhances research discovery by making non-obvious connections between studies. High value for literature reviews and identifying research patterns.

**Independent Test**: Can be fully tested by selecting an article and validating that recommended articles are semantically similar but not obvious duplicates, connecting different research domains meaningfully.

**Acceptance Scenarios**:

1. **Given** a selected article about plant X used in region A, **When** user requests recommendations, **Then** system returns articles studying plant X in other regions or studying plants with similar properties in region A
2. **Given** articles with different methodologies or research focuses, **When** recommendation engine analyzes them, **Then** it highlights conceptual similarities (e.g., "both study antimicrobial properties" even if in different plants/regions)
3. **Given** recommendation results, **When** user views them, **Then** each recommendation includes explanation of why it was suggested (semantic relationship, shared properties, regional connection, etc.)

---

### User Story 4 - Trend Analysis and Research Gaps (Priority: P2)

As a **research coordinator**, I want to analyze which plants are being studied most frequently, identify research gaps by region or property type, and detect patterns across communities so that I can guide research priorities and identify under-investigated areas.

**Why this priority**: Enables strategic research planning and demonstrates research landscape to stakeholders. Valuable for grant applications and research direction.

**Independent Test**: Can be fully tested by generating trend reports (plant frequency counts, regional distribution, property categories), validating accuracy against source data, and verifying that identified gaps correspond to actual underrepresented areas.

**Acceptance Scenarios**:

1. **Given** a dataset of articles, **When** user generates trend report, **Then** system shows: plants by study frequency (ranked), properties studied across regions, temporal trends (which plants studied more in recent years), gaps in coverage
2. **Given** comparison by region, **When** user selects two regions, **Then** system displays which plants studied in both (shared knowledge), which are region-specific, and property differences
3. **Given** search for "plants studied for property X", **When** user filters by region or time period, **Then** system identifies patterns and highlights under-researched areas (e.g., "anti-inflammatory plants in Northeast Brazil only studied in 2 papers")

---

### User Story 5 - CARE Data Principles Compliance (Priority: P1)

As a **community liaison and ethics officer**, I want the system to respect Collective Benefit, Authority to Decide, Responsibility, and Ethics principles for Indigenous and traditional knowledge so that communities maintain sovereignty over their knowledge and benefit from its use.

**Why this priority**: Non-negotiable requirement for legitimacy and ethical operation. Foundational to the entire system's acceptance by traditional communities and academic institutions.

**Independent Test**: Can be fully tested by verifying data governance policies, community access controls, benefit-sharing mechanisms, and attribution/citation standards are implemented and enforced.

**Acceptance Scenarios**:

1. **Given** articles discussing traditional knowledge from specific communities, **When** system processes them, **Then** it clearly identifies knowledge origin, attribution to communities, and records community approval status
2. **Given** community representatives as system users, **When** they access knowledge about their traditions, **Then** they can view, validate, dispute, or annotate information with contextual notes
3. **Given** commercial or research use of system-derived knowledge, **When** system tracks it, **Then** it ensures communities are notified and benefit-sharing agreements are in place (or documented as needed)
4. **Given** API access to article data, **When** external users query system, **Then** usage terms clearly state CARE principles and licensing restrictions regarding Indigenous knowledge

---

### User Story 6 - Automatic Publication Monitoring and Integration (Priority: P3)

As a **system administrator**, I want the system to periodically monitor scientific journals and publication APIs for new ethnobotany-related articles so that the database stays current with minimal manual effort.

**Why this priority**: Ensures long-term sustainability and relevance. While valuable, can be implemented after core system is functional.

**Independent Test**: Can be fully tested by configuring API monitoring, triggering publication checks, validating metadata extraction from API feeds, and verifying articles are added to database automatically.

**Acceptance Scenarios**:

1. **Given** configured journal API connections, **When** system runs scheduled publication checks, **Then** it discovers new articles matching ethnobotany criteria and adds them to the database
2. **Given** an article discovered via API, **When** it's processed, **Then** metadata extraction works correctly and content is vectorized using same embedding model
3. **Given** duplicate detection, **When** an article already in system is discovered via API, **Then** system skips it rather than creating duplicates

---

### Edge Cases

- What happens when PDF extraction fails (scanned images without OCR, corrupted files, non-English papers with special characters)?
- How does the system handle articles discussing plants by indigenous/traditional names that don't have scientific names?
- What occurs when a researcher queries in Portuguese (Brazilian focus) but some articles are in English, and terminology doesn't map directly?
- How does the system handle updates to articles (new versions, corrections, retractions)?
- What happens when embedding model has limitations or fails for specific article types (very long documents, highly technical papers)?

## Requirements

### Functional Requirements

- **FR-001**: System MUST allow authenticated users to submit scientific articles via: (a) PDF file upload, or (b) article URL (journal link, DOI, arXiv, ResearchGate)
- **FR-002**: System MUST automatically extract or fetch metadata for articles (title, authors, year, publication venue, abstract, DOI)
- **FR-003**: System MUST support metadata extraction from: (a) PDF files via text extraction, (b) URL via APIs (CrossRef, arXiv, PubMed), or (c) manual entry
- **FR-004**: System MUST store article metadata in SQL database with full-text search capability, linking to original sources via URL/DOI without retaining full-text PDF copies
- **FR-005**: System MUST create semantic embeddings for article content using open-source, scientific-article-optimized embedding model (SPECTER2)
- **FR-005a**: System MUST support embedding generation from either extracted PDF text or fetched metadata+abstract from URL sources
- **FR-006**: System MUST provide chat-based semantic search interface for natural language queries
- **FR-007**: System MUST support integration with multiple LLM APIs (Claude, Gemini, ChatGPT) with user-managed API keys
- **FR-008**: System MUST generate article recommendations based on semantic similarity
- **FR-009**: System MUST provide trend analysis dashboard showing plant study frequency, regional patterns, and research gaps
- **FR-010**: System MUST implement CARE data principles (Collective Benefit, Authority to Decide, Responsibility, Ethics)
- **FR-011**: System MUST allow traditional communities and researchers to annotate articles with contextual information and indigenous knowledge
- **FR-012**: System MUST support automatic monitoring of scientific journal APIs for new publications
- **FR-013**: System MUST support batch operations (import multiple articles, bulk update metadata)
- **FR-014**: System MUST provide user authentication and role-based access control (administrator, researcher, community liaison)
- **FR-015**: System MUST handle articles in Portuguese and English, with support for expansion to other languages
- **FR-016**: System MUST provide export functionality for search results (CSV, JSON, BibTeX formats)
- **FR-017**: System MUST log all data access and modifications for audit trails and CARE compliance
- **FR-018**: System MUST support containerization via Docker for easy deployment

### Key Entities

- **Article**: Scientific publication record (title, authors, year, abstract, full text, DOI, publication venue, metadata extraction status)
- **Embedding**: Vector representation of article content (article_id, embedding_vector, model_used, created_timestamp)
- **Community**: Traditional community or research group metadata (name, region, knowledge domains, approval status for knowledge use)
- **CommunityApproval**: Record of community validation/approval for articles discussing their knowledge (community_id, article_id, status, notes)
- **SearchQuery**: User search interactions (user_id, query_text, timestamp, results_count, selected_results for usage tracking)
- **Annotation**: User/community notes on articles (article_id, user_id, annotation_text, type: clarification/validation/dispute/context)
- **LLMConfiguration**: User's API key configuration for different LLM providers (user_id, provider, encrypted_api_key, model_preference)

## Success Criteria

### Measurable Outcomes

- **SC-001**: System supports ingestion of up to 50,000 scientific articles with sub-second query response times
- **SC-002**: Semantic search returns relevant articles with 85%+ precision (assessed by researcher validation) for natural language queries
- **SC-003**: Recommendation engine identifies related articles not obvious from keyword matching, with 70%+ user satisfaction for relevance
- **SC-004**: Trend analysis correctly identifies research gaps (regions/plants with <5 studies) with 95% accuracy
- **SC-005**: Article ingestion completes in under 5 minutes for: (a) PDF upload/indexing (10-50 pages), or (b) URL submission with metadata fetch
- **SC-005a**: URL-based submission (metadata fetch + embedding) completes in under 2 minutes for articles with available API metadata
- **SC-006**: API monitoring discovers 90%+ of new ethnobotany publications within 7 days of publication
- **SC-007**: System maintains 99.5% uptime in production deployment
- **SC-008**: CARE compliance audit shows 100% attribution of traditional knowledge with documented community approvals
- **SC-009**: Search interface usable by researchers without technical background (70%+ complete searches on first interaction)
- **SC-010**: Embedding model produces semantically meaningful results for scientific terminology and plant property descriptions
- **SC-011**: System deployment via Docker completes in under 30 minutes with standard configuration
- **SC-012**: Database supports concurrent access by 50+ researchers without performance degradation
- **SC-013**: Metadata extraction succeeds for 95%+ of PDFs and 90%+ of URL submissions without manual correction

## Assumptions

1. **Metadata extraction**: PDFs contain extractable text (not purely scanned images); if OCR needed, it will be implemented separately
2. **Embedding model selection**: Open-source models from Hugging Face (like SciBERT, SPECTER, or similar) will provide sufficient quality for scientific article embeddings
3. **Vector database choice**: Open-source options (Milvus, Weaviate, Qdrant) or managed services (Pinecone free tier) will be sufficient for initial scale (50k articles)
4. **SQL database**: PostgreSQL as the SQL choice for metadata, with full-text search capabilities via pgvector extension
5. **API availability**: Journal APIs (PubMed, CrossRef, arXiv) will be available and not rate-limited prohibitively
6. **LLM APIs**: Users will provide their own API keys for Claude, Gemini, ChatGPT usage (cost borne by users)
7. **CARE principles**: Implementation assumes participation of community representatives in design and governance
8. **Deployment**: Docker containerization will be feasible with current tech stack choices
9. **Scale limitation**: Initial system assumes single-region deployment (Brazil); multi-region scaling addressed in future phases
10. **Article language**: Primary languages English and Portuguese; expansion to other languages deferred

## Constraints and Limitations

- **Data Retention Strategy**: System will link to original sources for full article text rather than storing copies. This minimizes storage costs, reduces infrastructure burden, and respects publisher copyright while maintaining full functionality through embeddings and metadata. Links to articles will be maintained via DOI or journal URL references.
- System is read-optimized for search queries; article update/deletion operations are secondary concerns
- Community approval workflow may require manual review for complex cases (automated binary approval only for simple cases)

## Out of Scope

- Institutional repository integration or harvesting from university systems (can be added later via API connectors)
- Printed article digitization (only digital PDFs accepted)
- Article translation to Portuguese from other languages (users can provide translations)
- Research collaboration features (version control, commenting, manuscript co-writing)
- Integration with citation management tools (Zotero, Mendeley) - can be added via export functionality
