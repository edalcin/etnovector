# Research Document: Scientific Article Embedding Models

**Research Date**: 2025-11-01
**Updated**: 2025-11-09 (docling-rag-agent uses OpenAI embeddings)
**Feature**: 001-ethnobotany-vector-db
**Purpose**: Evaluate embedding models for scientific articles

**Reference Architecture Update**: [docling-rag-agent](https://github.com/coleam00/ottomator-agents/tree/main/docling-rag-agent) uses **OpenAI text-embedding-3-small** (1536 dimensions) in production with excellent results. Consider this as proven alternative to open-source models.

---

## Executive Summary

**docling-rag-agent Choice**: **OpenAI text-embedding-3-small** (1536d, ~$0.02/1M tokens)
- Production-proven in RAG systems
- High quality, cost-effective
- Simple API integration
- **Recommended for MVP**: Start with OpenAI, migrate to open-source if needed

**Open-Source Alternatives** (original research):

- **SPECTER2**: Purpose-built for scientific documents, trained on 6M triplets across 23 fields of study, task-adaptive
- **SciBERT**: Solid foundation trained on 1.14M scientific papers, stable and well-documented
- **Sentence-Transformers**: General-purpose models, good ecosystem

---

## Detailed Model Evaluation

### 1. SPECTER2 (Recommended for this project)

**Model Name**: `specter-v2` (Allenai)

**Key Characteristics**:
- **Training Data**: 6M triplets spanning 23 fields of study (10X more than original SPECTER)
- **Input**: Full scientific document (abstract + title, up to 512 tokens)
- **Output Vector Dimension**: 768
- **Embedding Type**: Task-adaptive (can generate different embeddings based on task)
- **License**: Apache 2.0 (Open Source)
- **Availability**: Hugging Face hub, GitHub (allenai/specter)

**Strengths**:
- ✅ Specifically designed for scientific documents
- ✅ Citation network-aware: papers close in citation graph are close in embedding space
- ✅ **Task-adaptive capability**: Can generate embeddings tuned for different tasks (retrieval, recommendation, clustering)
- ✅ Covers 23 diverse fields including Botany, Medicine, Biology (perfect for ethnobotany which bridges these domains)
- ✅ Trained on larger dataset (6M vs original SPECTER's 600k)
- ✅ Superior performance on scientific document retrieval benchmarks
- ✅ Can process entire abstracts + titles without truncation for most papers

**Limitations**:
- ❌ Requires title + abstract (full text not fully utilized)
- ❌ Vector dimension 768 requires reasonable computational resources
- ❌ Less widely deployed than general models (smaller community)

**Performance Metrics**:
- Outperforms original SPECTER on document ranking tasks
- Achieves state-of-the-art on scientific document retrieval benchmarks
- Strong performance on cross-domain tasks (multiple fields)

**Cost**: FREE (self-hosted open source)

**Recommendation**: **USE THIS** for the ethnobotany system. The task-adaptive nature is perfect for supporting different use cases (recommendations, semantic search, trend clustering).

---

### 2. SciBERT

**Model Name**: `scibert-scivocab-uncased` (AllenAI)

**Key Characteristics**:
- **Training Data**: 1.14M scientific papers from Semantic Scholar corpus, 3.1B tokens
- **Input**: Text up to 512 tokens
- **Output Vector Dimension**: 768
- **License**: Apache 2.0 (Open Source)
- **Availability**: Hugging Face hub, GitHub

**Strengths**:
- ✅ Specifically pre-trained on scientific text (scientific vocabulary)
- ✅ Well-documented with strong academic citations
- ✅ Lightweight and stable
- ✅ Good for text classification on scientific documents
- ✅ Effective for scientific NLP tasks beyond just embeddings

**Limitations**:
- ❌ Not specifically optimized for document-level similarity (was trained on general BERT objective)
- ❌ Token limit of 512 means longer abstracts are truncated
- ❌ Doesn't leverage citation network information
- ❌ Less sophisticated than SPECTER2 for document-level tasks

**Performance Metrics**:
- Good for document classification
- Reasonable performance on retrieval tasks (but inferior to SPECTER2)
- Strong on scientific NLP tasks

**Use Case**: Better for article classification, keyword extraction, and NLP preprocessing rather than pure semantic similarity.

**Cost**: FREE (self-hosted open source)

**Recommendation**: Consider as **secondary option** if SPECTER2 proves insufficient or if you need additional NLP capabilities beyond embeddings.

---

### 3. Sentence-Transformers (General Purpose)

**Model Examples**:
- `all-mpnet-base-v2` (17.8M downloads)
- `paraphrase-multilingual-mpnet-base-v2` (5.39M downloads)
- `all-MiniLM-L6-v2` (extremely lightweight)

**Key Characteristics**:
- **Training Data**: General text corpora (not scientific-specific)
- **Vector Dimension**: 384-768 depending on model
- **License**: Apache 2.0
- **Availability**: Sentence-transformers library (pip installable)
- **Ecosystem**: 127+ models available

**Strengths**:
- ✅ Extremely mature and well-documented
- ✅ Multiple size options (from 23M to 435M parameters)
- ✅ Multilingual variants available
- ✅ Easy integration with `sentence-transformers` Python library
- ✅ Can be fine-tuned on domain-specific data
- ✅ Good if you want to fine-tune on ethnobotany articles

**Limitations**:
- ❌ Not specialized for scientific papers
- ❌ Smaller vector dimensions require tradeoff with quality
- ❌ General-purpose training means less understanding of scientific terminology
- ❌ Citation network information not leveraged

**Performance Metrics**:
- MTEB leaderboard shows good performance on general retrieval
- Performance on scientific tasks: Moderate (significantly worse than SPECTER2)

**Fine-tuning Potential**: ⭐⭐⭐ Can be fine-tuned on ethnobotany article pairs if you collect similarity annotations

**Cost**: FREE (self-hosted open source)

**Recommendation**: Consider as **fallback option** or if fine-tuning budget available. Not recommended for primary embeddings without fine-tuning.

---

### 4. Other Contenders (Not Recommended)

**ColBERT**: Better for dense passage retrieval but adds complexity with late interaction architecture
**BGE Models**: Good general embeddings but not scientific-specific
**Gritlm**: Instruction-tuned but overkill for document embeddings
**OpenAI/Cohere Embeddings**: Excellent but commercial (cost increases with scale; $0.02 per million tokens for Ada, higher for better models)

---

## Comparison Table

| Aspect | SPECTER2 | SciBERT | Sentence-Transformers | OpenAI Ada |
|--------|----------|---------|----------------------|-----------|
| **Scientific Focus** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐ | ⭐⭐⭐⭐ |
| **Document Similarity** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Cost** | Free | Free | Free | $0.02/1M tokens |
| **Model Size** | 235M | 110M | 33-435M | Proprietary |
| **Ease of Use** | Medium | Easy | Very Easy | Very Easy |
| **Customization** | Medium | Medium | High | None |
| **Vector Dimension** | 768 | 768 | 384-768 | 1536 |
| **Task Adaptability** | ⭐⭐⭐⭐⭐ | ⭐ | ⭐⭐ | ⭐⭐⭐ |
| **Community** | Growing | Established | Very Large | Largest |

---

## Implementation Recommendations

### For Initial MVP (Ethnobotany Vector DB)

**Primary Model**: SPECTER2
- Download from Hugging Face: `specter2-v2` or equivalent
- Use with Python transformers library
- Install: `pip install transformers torch`
- Inference: ~0.5-2 seconds per article abstract

**Processing Pipeline**:
```
PDF Upload
  → Extract Title + Abstract
  → Tokenize (max 512 tokens)
  → Generate SPECTER2 Embedding
  → Store 768-dimensional vector
```

### Deployment Considerations

**Memory Requirements**:
- SPECTER2: ~900MB GPU memory or ~2GB CPU memory
- SciBERT: ~700MB GPU memory or ~1.5GB CPU memory
- Sentence-Transformers (all-mpnet): ~500MB GPU memory or ~1GB CPU memory

**Inference Speed**:
- SPECTER2 with GPU: ~50-100 articles/second
- SPECTER2 with CPU: ~2-5 articles/second
- Batch processing is recommended (process 32-64 articles at once)

**Docker Image**: ~2-3GB (includes base image + model weights)

---

## Cost Analysis (for 50k articles)

**SPECTER2 (Recommended)**:
- Model: Free (open source)
- Storage (768-dim vectors): ~150MB (50k × 3KB per vector)
- Compute: One-time cost for GPU instance to process articles, or ~$0.50-2.00 per 1000 articles on cloud GPU
- Total Initial Cost: ~$25-100 to process entire corpus
- Ongoing Cost: $0 (embeddings computed once, not per-query)

**OpenAI Embeddings API** (for comparison):
- Cost: $0.02 per 1M tokens
- 50k articles × 500 tokens avg = 25M tokens
- Cost: ~$0.50 for initial processing
- But recurring cost if you re-embed, and API rate limits
- Proprietary, locked into OpenAI

**Recommendation**: SPECTER2 wins on cost and control for this scale.

---

## Migration Path

If SPECTER2 proves insufficient:

1. **Phase 1 (Current)**: Deploy SPECTER2, collect search quality feedback
2. **Phase 2**: Fine-tune SPECTER2 on ethnobotany article pairs using positive/negative examples from user searches
3. **Phase 3 (If needed)**: Switch to OpenAI embeddings with same architecture (simple swap)
4. **Phase 4**: Hybrid approach combining multiple embedding models

---

## Final Recommendation

**Use SPECTER2 for the ethnobotany vector database**

**Rationale**:
1. ✅ Specifically designed for scientific documents
2. ✅ Task-adaptive capability supports future features
3. ✅ Covers 23 fields including botany, medicine, biology
4. ✅ Free open-source eliminates API costs
5. ✅ Citation network awareness improves serendipitous discovery (recommendations)
6. ✅ Proven performance on scientific document tasks
7. ✅ Docker-deployable without external dependencies
8. ✅ Supports fine-tuning if needed later

**Next Steps**:
- Confirm SPECTER2 availability on Hugging Face
- Test on sample ethnobotany abstracts to validate quality
- Benchmark against SciBERT if time permits
- Plan GPU instance for batch processing of 50k articles
