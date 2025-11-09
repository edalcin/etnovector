# Feature Specification: Ethnobotany Scientific Articles Vectorized Database (Updated with Docling RAG Architecture)

**Feature Branch**: `001-docling-rag-update`
**Created**: 2025-11-09
**Status**: Draft
**Input**: User description: "considere o projeto https://github.com/coleam00/ottomator-agents/tree/main/docling-rag-agent como exemplo para este projeto e atualize a especificação."

**Reference Architecture**: [Docling RAG Agent](https://github.com/coleam00/ottomator-agents/tree/main/docling-rag-agent)

## User Scenarios & Testing

### User Story 1 - Multi-Format Document Ingestion with Automatic Processing (Priority: P1)

As a **researcher or librarian**, I want to submit scientific articles in multiple formats (PDF, Word, PowerPoint, Excel, HTML, Markdown, text) via file upload or URL submission so that the knowledge base automatically processes diverse document types without manual format conversion.

**Why this priority**: This is the foundation of the entire system and directly addresses the most common user pain point - dealing with documents in various formats. The docling-rag-agent demonstrates that multi-format support via Docling enables ingestion from diverse sources (journal PDFs, conference presentations, supplementary materials, research notes) without requiring users to standardize formats manually. This is the core MVP that makes the system immediately usable.

**Independent Test**: Can be fully tested by ingesting documents in each supported format (PDF, DOCX, PPTX, XLSX, HTML, MD, TXT), verifying metadata extraction works correctly for each type, validating that embeddings are created uniformly regardless of source format, and confirming that search retrieves content from all document types. Delivers immediate value: a multi-format knowledge base with semantic search.

**Acceptance Scenarios**:

1. **Given** a scientific article PDF with extractable text, **When** user uploads it via web interface, **Then** system uses Docling for format detection and conversion to Markdown, extracts metadata (title, authors, year, abstract, DOI), chunks content into semantic segments (default 1000 tokens), generates embeddings, and stores in PostgreSQL with pgvector without retaining original file
2. **Given** an article URL (DOI, arXiv, ResearchGate, institutional repository), **When** user submits the URL, **Then** system fetches content, detects format automatically, processes through Docling pipeline, extracts metadata via APIs (CrossRef, arXiv, PubMed) when available, and stores embeddings with link to original source
3. **Given** a PowerPoint presentation from an ethnobotany conference, **When** user uploads PPTX file, **Then** system extracts slide content, converts to searchable text via Docling, creates embeddings for presentation content, and indexes it alongside journal articles
4. **Given** Excel supplementary data files with plant species listings, **When** user uploads XLSX files, **Then** system extracts tabular data, converts to structured text format, and makes it semantically searchable
5. **Given** duplicate article detection (by DOI or content similarity), **When** user submits an article already in system, **Then** system prevents duplication and alerts user with existing entry details
6. **Given** large or complex documents (up to 50MB, multi-page PDFs), **When** user uploads them, **Then** system processes asynchronously with progress tracking, uses configurable chunking strategy (default 1000 tokens), and provides status notifications when processing completes

---

### User Story 2 - Intelligent RAG-Powered Chat Interface with Multi-Provider LLM Support (Priority: P1)

As a **researcher**, I want to search the knowledge base using natural language queries through a conversational chat interface that uses retrieval-augmented generation (RAG) to provide contextual answers grounded in indexed articles, with the ability to choose between Claude, Gemini, or ChatGPT as my reasoning engine.

**Why this priority**: This is the key differentiator that transforms a static vector database into an intelligent research assistant. The docling-rag-agent architecture demonstrates how tool-calling LLM agents can orchestrate vector search as a callable tool (`search_knowledge_base`), enabling multi-turn conversations with context maintenance and answer grounding through source citations. This directly addresses researcher needs for exploratory investigation and literature synthesis.

**Independent Test**: Can be fully tested by running natural language queries through chat interface, validating that responses are grounded in retrieved article content (with source citations), confirming that follow-up questions maintain conversation context, verifying that switching LLM providers (Claude/Gemini/ChatGPT) works seamlessly with user-provided API keys, and checking that streaming responses provide real-time feedback.

**Acceptance Scenarios**:

1. **Given** a populated vector database with ethnobotany articles, **When** user asks "What plants are used for inflammation treatment in Amazonian communities?", **Then** system generates query embedding, performs cosine similarity search via pgvector `<=>` operator, retrieves top-k relevant chunks (default k=5, threshold 0.7), and LLM synthesizes answer with inline source citations (title, file path, similarity score)
2. **Given** user asks follow-up question "What are their preparation methods?", **When** system processes it, **Then** conversation history is maintained across turns, context from previous query is preserved, and system refines search based on conversational context to retrieve preparation-specific content
3. **Given** ambiguous query terminology, **When** user searches with colloquial terms (e.g., "natural antibiotics from plants"), **Then** vector embeddings capture semantic intent, system returns relevant articles about antimicrobial plant properties, and LLM explains terminology mapping in response
4. **Given** user preference for specific LLM provider, **When** user configures API key for Claude, Gemini, or ChatGPT in settings, **Then** system uses selected provider for query processing, supports provider-specific parameters (model version, temperature), and switches providers seamlessly without re-indexing
5. **Given** token-by-token streaming enabled, **When** LLM generates response, **Then** user sees real-time answer generation (not blocked waiting for full response), improving perceived responsiveness for complex synthesis tasks
6. **Given** insufficient relevant results (all below similarity threshold), **When** query retrieves no good matches, **Then** system explains lack of relevant sources transparently and suggests query refinement strategies

---

### User Story 3 - Semantic Article Recommendation with Explainability (Priority: P2)

As a **researcher**, I want to discover related articles based on semantic similarity to a selected article, with explanations of why each recommendation was suggested, so that I can find complementary research from different methodologies, regions, or disciplinary perspectives that study related ethnobotanical phenomena.

**Why this priority**: Enhances research discovery by surfacing non-obvious connections between studies through vector similarity. The docling-rag-agent pattern of returning similarity scores alongside results enables transparent recommendation ranking and user trust in suggestions.

**Independent Test**: Can be fully tested by selecting an article and validating that recommended articles have high cosine similarity but are not duplicates, verifying that recommendations span different regions/methodologies while maintaining topical relevance, and confirming that similarity scores and explanations are provided for each suggestion.

**Acceptance Scenarios**:

1. **Given** a selected article about anti-inflammatory plants in Brazilian Cerrado, **When** user requests recommendations, **Then** system computes embedding similarity to all other articles, returns top 10 matches ranked by cosine similarity score, and provides explanation for each (e.g., "Similar focus on anti-inflammatory properties (score: 0.89)" or "Studies related plant species in different biome (score: 0.82)")
2. **Given** articles with different research methodologies (ethnographic vs. laboratory analysis), **When** recommendation engine analyzes them, **Then** it highlights conceptual connections across methodologies (e.g., "Both investigate antimicrobial properties of *Passiflora* species" even if one is field study and one is phytochemical analysis)
3. **Given** recommendation results, **When** user views them, **Then** system displays article metadata, similarity score, key shared concepts extracted from embeddings, and links to original sources for further reading
4. **Given** user feedback on recommendations (helpful/not helpful), **When** researcher rates suggestions, **Then** system logs feedback for future model refinement without requiring immediate retraining

---

### User Story 4 - Trend Analysis Dashboard with Research Gap Identification (Priority: P2)

As a **research coordinator or funding agency analyst**, I want to analyze which plants, regions, and medicinal properties are most/least studied in the indexed literature, with visual dashboards showing temporal trends and geographic distribution, so that I can identify research gaps and guide strategic research priorities.

**Why this priority**: Enables evidence-based research planning and demonstrates landscape of ethnobotanical knowledge to stakeholders. Valuable for grant applications, curriculum development, and identifying under-investigated traditional knowledge systems.

**Independent Test**: Can be fully tested by generating trend reports from indexed articles, validating plant frequency counts against source data, verifying regional distribution accuracy, confirming temporal trend calculations (studies per year), and checking that identified gaps correspond to actual underrepresented areas in the dataset.

**Acceptance Scenarios**:

1. **Given** indexed articles with extracted metadata (plants, regions, properties, years), **When** user generates trend report, **Then** system displays: top 20 most-studied plants (ranked by article count), medicinal properties by study frequency, regional coverage (map visualization), temporal trends (articles per year), and gaps identified via threshold analysis (plants/regions with <5 studies)
2. **Given** comparison by region, **When** user selects two regions (e.g., Brazilian Amazon vs. Atlantic Forest), **Then** system shows which plants studied in both (shared knowledge), region-specific plants, property differences, and publication timeline comparison
3. **Given** filter by medicinal property, **When** user searches "anti-parasitic plants", **Then** system aggregates all articles mentioning anti-parasitic use, clusters by plant family and region, highlights under-researched combinations (e.g., "anti-parasitic plants in Pantanal biome: only 2 studies"), and exports results as CSV or visualization
4. **Given** temporal analysis, **When** user views publication trends over time, **Then** system identifies emerging research areas (properties with accelerating publication rates), declining topics, and stable research domains

---

### User Story 5 - CARE Data Principles Implementation with Community Governance (Priority: P1)

As a **community liaison, Indigenous knowledge holder, or ethics officer**, I want the system to implement Collective Benefit, Authority to Decide, Responsibility, and Ethics (CARE) principles for traditional knowledge, with transparent data governance, community approval workflows, and benefit-sharing documentation, so that traditional communities maintain sovereignty over their knowledge and participate in research governance.

**Why this priority**: Non-negotiable ethical requirement for legitimacy when working with Indigenous and traditional knowledge. The system must be designed from the ground up to respect community rights and knowledge sovereignty, not retrofitted later. This establishes trust with communities and academic institutions.

**Independent Test**: Can be fully tested by verifying that: (1) articles mentioning traditional knowledge are flagged and linked to source communities, (2) community representatives can review and approve/dispute content about their knowledge, (3) usage tracking logs all access to community knowledge, (4) benefit-sharing agreements are recorded and accessible, and (5) API access terms enforce CARE principles.

**Acceptance Scenarios**:

1. **Given** article discussing traditional knowledge from specific Indigenous community (e.g., Guarani medicinal plant use), **When** system ingests it, **Then** metadata extraction identifies community attribution, flags article as containing traditional knowledge, links to community profile (if available), and records approval status as "pending" until community review
2. **Given** community representative accessing system, **When** they view articles about their traditional knowledge, **Then** they can: validate accuracy of ethnobotanical information, add contextual annotations (e.g., seasonal usage, preparation restrictions, cultural significance), dispute misrepresentations, and approve/request removal of sensitive content
3. **Given** external researcher querying traditional knowledge via API, **When** system serves results, **Then** each response includes: community attribution, citation requirements, usage terms specifying CARE compliance, and notification is logged for community benefit-sharing tracking
4. **Given** commercial use of research derived from system, **When** usage tracking detects it (via user declarations or publication monitoring), **Then** system ensures communities are notified, benefit-sharing agreements are in place or initiated, and compliance is documented in audit trail
5. **Given** community requests knowledge removal or access restriction, **When** they submit governance request, **Then** system implements restriction within 48 hours, notifies administrators, and logs decision in governance audit trail

---

### User Story 6 - Automated Publication Monitoring via Journal APIs (Priority: P3)

As a **system administrator or librarian**, I want the system to automatically monitor scientific journal APIs (PubMed, CrossRef, arXiv, BioRxiv) and ethnobotany-specific sources for new publications matching configured criteria, ingesting them automatically so that the knowledge base stays current without manual submission for every new article.

**Why this priority**: Ensures long-term sustainability and currency of the knowledge base. While valuable, can be implemented after core ingestion and search features are stable. Automation reduces maintenance burden significantly.

**Independent Test**: Can be fully tested by configuring API monitoring with specific search criteria (e.g., "ethnobotany" OR "medicinal plants"), triggering scheduled publication checks, validating that new articles are discovered and added to database, verifying metadata extraction from API feeds, and confirming duplicate detection prevents redundant ingestion.

**Acceptance Scenarios**:

1. **Given** configured monitoring rules for PubMed (e.g., query: "(ethnobotany[Title/Abstract]) AND (Brazil[Affiliation])"), **When** system runs daily scheduled check, **Then** it discovers new articles published in last 24 hours, fetches metadata via PubMed API, downloads abstracts (or full text if open access), and queues for Docling processing
2. **Given** CrossRef API monitoring for ethnobotany journals, **When** system detects new DOI registrations, **Then** it fetches article metadata, attempts to retrieve full text via open access links or institutional access, processes through ingestion pipeline, and adds to database with source tracking
3. **Given** duplicate detection via DOI matching, **When** monitored API returns article already in system, **Then** system skips re-ingestion, logs skip event, and optionally checks for updated versions or metadata corrections
4. **Given** API rate limits (e.g., PubMed 3 requests/second), **When** system performs bulk monitoring, **Then** it respects rate limits, implements exponential backoff on errors, and logs API call statistics for monitoring

---

### Edge Cases

- What happens when Docling fails to convert a document format (corrupted files, encrypted PDFs, scanned images without OCR, unsupported language scripts)?
- How does the system handle articles discussing plants by Indigenous names that lack scientific binomial nomenclature or have multiple regional naming variations?
- What occurs when a researcher queries in Portuguese but relevant articles are in English, and scientific terminology doesn't translate directly (e.g., "plantas medicinais" vs. "medicinal plants" with different semantic nuances)?
- How does the system handle article updates, corrections, or retractions in monitored journals?
- What happens when embedding generation fails for specific article types (extremely long documents exceeding context windows, highly technical mathematical content, non-textual data tables)?
- How does the system manage LLM API failures, rate limits, or cost overruns when user provides limited-budget API keys?
- What occurs when multiple users submit the same article simultaneously via different routes (URL vs. PDF upload)?
- How does the system handle articles behind paywalls when URL is provided but content cannot be fetched automatically?

## Requirements

### Functional Requirements

- **FR-001**: System MUST support document ingestion in multiple formats: PDF, Word (DOCX), PowerPoint (PPTX), Excel (XLSX), HTML, Markdown, and plain text via Docling automatic format detection
- **FR-002**: System MUST allow authenticated users to submit documents via: (a) file upload with drag-and-drop interface, or (b) URL submission (journal link, DOI, arXiv, institutional repository)
- **FR-003**: System MUST automatically extract metadata for articles including: title, authors, year, publication venue, abstract, DOI, and custom ethnobotany fields (plants mentioned, regions, medicinal properties)
- **FR-004**: System MUST support metadata extraction via: (a) Docling text extraction from uploaded files, (b) API calls to CrossRef, arXiv, PubMed for URL submissions, or (c) manual entry form with validation
- **FR-005**: System MUST convert all ingested documents to normalized Markdown format via Docling pipeline before embedding generation
- **FR-006**: System MUST implement semantic chunking with configurable token limits (default 1000 tokens per chunk) to handle long documents exceeding embedding model context windows
- **FR-007**: System MUST generate vector embeddings using OpenAI text-embedding-3-small model (1536 dimensions) for semantic search capability
- **FR-008**: System MUST store embeddings in PostgreSQL with pgvector extension, supporting cosine similarity search via `<=>` operator
- **FR-009**: System MUST store document metadata and chunks in PostgreSQL with tables: `documents` (original metadata), `chunks` (text segments with embeddings), and custom SQL function `match_chunks()` for vector search
- **FR-010**: System MUST NOT retain full-text copies of uploaded documents; only metadata, chunks, and embeddings are stored, with links to original sources via URL/DOI
- **FR-011**: System MUST provide chat-based conversational interface using PydanticAI agent framework with tool-calling architecture
- **FR-012**: System MUST implement `search_knowledge_base` tool callable by LLM agents, accepting query string and result limit (default 5), returning chunks with content, similarity scores, source titles, and file paths
- **FR-013**: System MUST support multi-turn conversations with context maintenance across user queries within a session
- **FR-014**: System MUST integrate with multiple LLM providers (OpenAI GPT-4o-mini default, with support for Claude and Gemini via user-provided API keys and provider abstraction layer)
- **FR-015**: System MUST implement token-by-token streaming responses for real-time user feedback during answer generation
- **FR-016**: System MUST ground all answers in retrieved source content with inline citations (document title, chunk location, similarity score)
- **FR-017**: System MUST provide semantic recommendation engine that computes cosine similarity between article embeddings and returns top-k similar articles with scores and explanations
- **FR-018**: System MUST implement CARE data principles with: (a) community attribution tracking, (b) approval workflow for traditional knowledge content, (c) usage audit trail, (d) benefit-sharing documentation
- **FR-019**: System MUST allow community representatives to annotate, validate, dispute, or request removal of content related to their traditional knowledge
- **FR-020**: System MUST provide trend analysis dashboard showing: plant study frequency, regional distribution, medicinal property categories, temporal trends, and research gap identification
- **FR-021**: System MUST support automatic publication monitoring via APIs (PubMed, CrossRef, arXiv) with configurable search criteria and scheduled execution
- **FR-022**: System MUST implement duplicate detection via DOI matching and content similarity thresholds to prevent redundant ingestion
- **FR-023**: System MUST support asynchronous document processing with progress tracking for large files (up to 50MB)
- **FR-024**: System MUST provide user authentication and role-based access control (administrator, researcher, community liaison, public viewer)
- **FR-025**: System MUST support bilingual interface (Portuguese and English) with content search capabilities across both languages
- **FR-026**: System MUST provide export functionality for search results in CSV, JSON, and BibTeX formats
- **FR-027**: System MUST implement connection pooling for PostgreSQL (minimum 2, maximum 10 async connections) for performance optimization
- **FR-028**: System MUST support Docker containerization with docker-compose for deployment, including PostgreSQL + pgvector container and application container
- **FR-029**: System MUST implement error handling with user-friendly messages for: document processing failures, API rate limits, embedding generation errors, and database connection issues
- **FR-030**: System MUST log all data access, modifications, and CARE-related governance events for audit trails

### Key Entities

- **Document**: Original article metadata (document_id, title, authors, year, abstract, doi, source_url, file_path, upload_timestamp, processing_status, format_type)
- **Chunk**: Text segment from document (chunk_id, document_id, chunk_text, chunk_embedding [vector(1536)], chunk_index, token_count, created_at)
- **Community**: Traditional knowledge source (community_id, name, region, language, knowledge_domains, contact_info, governance_status)
- **CommunityApproval**: Knowledge validation record (approval_id, community_id, document_id, approval_status [pending/approved/disputed/restricted], reviewer_notes, decision_date)
- **Annotation**: User/community contextual notes (annotation_id, document_id, user_id, annotation_text, annotation_type [validation/context/dispute/correction], created_at)
- **SearchQuery**: User interaction tracking (query_id, user_id, query_text, timestamp, results_count, selected_results, session_id)
- **LLMConfiguration**: User's API settings (config_id, user_id, provider [openai/anthropic/google], encrypted_api_key, model_preference, temperature, max_tokens)
- **MonitoringRule**: Publication monitoring configuration (rule_id, api_source [pubmed/crossref/arxiv], search_query, schedule_cron, last_run, status)
- **AuditLog**: CARE compliance tracking (log_id, event_type [access/modification/governance], user_id, document_id, community_id, timestamp, details)

## Success Criteria

### Measurable Outcomes

- **SC-001**: System successfully ingests documents in all supported formats (PDF, DOCX, PPTX, XLSX, HTML, MD, TXT) with 90%+ success rate for well-formed files
- **SC-002**: Document processing completes in under 5 minutes for typical scientific articles (10-50 pages PDF or equivalent) including Docling conversion, chunking, and embedding generation
- **SC-003**: URL-based metadata fetch and embedding completes in under 2 minutes for articles with API-accessible metadata (CrossRef, PubMed, arXiv)
- **SC-004**: Semantic search returns relevant articles with 85%+ precision (assessed by researcher validation) for natural language queries across 100 test queries
- **SC-005**: Vector similarity search via pgvector executes in under 200ms for queries against database of 50,000 articles with 1536-dimensional embeddings
- **SC-006**: Chat interface provides token-by-token streaming responses with first token arriving within 1 second of query submission
- **SC-007**: Recommendation engine identifies semantically related articles with 70%+ user-rated relevance for suggestions outside obvious keyword matches
- **SC-008**: Trend analysis correctly identifies research gaps (plants/regions with <5 studies) with 95%+ accuracy validated against manual dataset review
- **SC-009**: CARE compliance audit shows 100% attribution of traditional knowledge with documented community approval status for all flagged content
- **SC-010**: Publication monitoring discovers 90%+ of new ethnobotany publications within 7 days via configured API sources (PubMed, CrossRef)
- **SC-011**: System maintains 99.5% uptime in production with Docker deployment and handles concurrent access by 50+ users without performance degradation
- **SC-012**: Duplicate detection prevents redundant ingestion with 99%+ accuracy using DOI matching and content similarity thresholds
- **SC-013**: Chat interface usable by non-technical researchers with 70%+ successful task completion on first interaction (measured via user testing)
- **SC-014**: System deployment via docker-compose completes in under 30 minutes on standard server (8GB RAM, 4 cores) with automated database initialization
- **SC-015**: Connection pooling maintains stable performance under load with average query latency under 500ms for 95th percentile of requests

## Assumptions

1. **Docling integration**: Docling library will successfully convert 90%+ of scientific articles in supported formats to Markdown with acceptable quality; OCR for scanned PDFs is out of scope for MVP
2. **Embedding model**: OpenAI text-embedding-3-small provides sufficient quality for ethnobotany semantic search at acceptable cost (~$0.02 per 1M tokens); migration to open-source models (SPECTER2, SciBERT) can be implemented later if cost becomes prohibitive
3. **Database choice**: PostgreSQL with pgvector extension provides adequate performance for 50,000 articles with 1536-dimensional embeddings; migration to specialized vector databases (Qdrant, Weaviate) deferred until scale requires it
4. **LLM API availability**: OpenAI API (GPT-4o-mini default) is primary provider with costs borne by users via their API keys; Claude and Gemini support added via provider abstraction
5. **PydanticAI framework**: Agent framework provides stable tool-calling and streaming capabilities; framework updates will maintain backward compatibility
6. **Journal API access**: PubMed, CrossRef, arXiv APIs remain freely available with current rate limits (3 req/s for PubMed); monitoring implemented within rate limit constraints
7. **CARE principles implementation**: Community representatives will participate in governance design and approval workflow validation during development
8. **Document formats**: Scientific articles will primarily be well-formed digital documents (not scanned images); low-quality scans requiring OCR are edge cases handled manually
9. **Language support**: Primary languages are Portuguese and English; embedding model handles both adequately; expansion to Indigenous languages deferred
10. **Deployment environment**: Docker and docker-compose are available on target deployment servers; PostgreSQL 15+ with pgvector extension can be containerized successfully

## Constraints and Limitations

- **Data Retention Strategy**: System stores only metadata, text chunks, and embeddings - not full-text article copies. Original sources accessed via DOI/URL links. This minimizes storage (50,000 articles ~5GB vs. ~500GB for full PDFs), respects copyright, and reduces infrastructure costs while maintaining full search functionality.
- **Embedding Model Lock-In**: Switching embedding models requires re-processing entire corpus; migration strategy includes: (1) incremental re-embedding with model versioning, (2) parallel index maintenance during transition, (3) A/B testing for quality validation
- **LLM API Dependency**: System requires external LLM API access (user-provided keys); offline operation not supported in MVP; self-hosted LLM integration can be added later
- **Docling Processing Limitations**: Some complex document layouts (multi-column scientific papers, embedded figures with captions, supplementary materials) may not convert perfectly to Markdown; manual review may be needed for critical documents
- **Vector Search Performance**: Query performance degrades with corpus size; above 100,000 articles, migration to specialized vector database (Qdrant with HNSW indexing) may be required
- **Community Governance Scalability**: Manual approval workflow for traditional knowledge assumes hundreds of communities, not thousands; automated governance tools needed at larger scale

## Out of Scope for Initial Implementation

- OCR processing for scanned PDF images (only digital text extraction via Docling)
- Self-hosted LLM deployment (all inference via external APIs)
- Audio/video content ingestion beyond basic transcription (future: Whisper integration for ethnobotany interviews/oral histories)
- Real-time collaborative annotation (MVP supports asynchronous annotations only)
- Mobile application (web interface responsive but native apps out of scope)
- Integration with citation management tools (Zotero, Mendeley, EndNote) - can be added via export functionality
- Institutional repository harvesting via OAI-PMH (can be added as additional monitoring source)
- Multi-language translation of articles (users can provide translations; auto-translate out of scope)
- Advanced visualizations (network graphs of plant-property relationships, geographic heat maps) - dashboard provides basic charts only
- Version control for article updates (system tracks latest version only; full revision history out of scope)
