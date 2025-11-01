# Research Document: LLM API Integration for Chat Interface

**Research Date**: 2025-11-01
**Feature**: 001-ethnobotany-vector-db
**Purpose**: Evaluate LLM APIs for the intelligent chat interface

---

## Executive Summary

**Recommended Approach**: Support all three (Claude, Gemini, ChatGPT) with user-provided API keys

- **Primary LLM**: Claude 3.5 Sonnet (best for scientific analysis, highest reliability)
- **Secondary**: Google Gemini 2.5 (best context window for large queries)
- **Tertiary**: ChatGPT 4o (popular, good fallback)

Each user provides their own API key, eliminating API costs to the platform while maintaining user control.

---

## Part 1: LLM API Comparison

### 1. Claude API (Anthropic) - RECOMMENDED

**Latest Model**: Claude 3.5 Sonnet (Recommended) or Claude 3 Opus (more powerful but slower/expensive)

**Key Characteristics**:
- **Input Cost**: $3.00 per 1M tokens (3.5 Sonnet) or $15 per 1M tokens (Opus)
- **Output Cost**: $15.00 per 1M tokens (3.5 Sonnet) or $75 per 1M tokens (Opus)
- **Context Window**: 200K tokens standard (500K for enterprise)
- **Response Quality**: ⭐⭐⭐⭐⭐ (highest for scientific analysis)
- **Reasoning**: "Extended Thinking" mode for complex queries (added computational cost)
- **Rate Limits**: Up to 100K tokens/minute on standard tier
- **Documentation**: Excellent, well-maintained
- **Reliability**: 99.9%+ uptime SLA

**For Ethnobotany Use Case**:
- **Strengths**:
  - ✅ Superior at understanding scientific terminology
  - ✅ Excellent at synthesizing research findings
  - ✅ Strong at reasoning through botanical/pharmacological properties
  - ✅ Better at handling ambiguity in queries (e.g., plant names in multiple languages)
  - ✅ Extended Thinking useful for complex queries (e.g., "which plants are underresearched")
  - ✅ Good memory of context in multi-turn conversations
  - ✅ Supports vision (can analyze article images if needed future)

**Estimated Cost (per user)**:
- Typical research query (3K tokens input, 500 tokens output): ~$0.01
- Complex synthesis query with Extended Thinking: ~$0.10-0.50
- Average user: ~$1-5/month if using 100-500 queries
- **User cost is manageable if provided with API key**

**Integration Complexity**: Low (official Python library)

**Recommendation**: **Primary LLM for scientific synthesis and deep research**

---

### 2. Google Gemini API - ALTERNATIVE

**Latest Model**: Gemini 2.5 Flash (recommended) or Gemini 2.0 Pro

**Key Characteristics**:
- **Input Cost**: $0.075 per 1M tokens (2.5 Flash) or $1.25 per 1M tokens (2.0 Pro)
- **Output Cost**: $0.30 per 1M tokens (2.5 Flash) or $5.00 per 1M tokens (2.0 Pro)
- **Context Window**: 1M+ tokens (Gemini 2.5 - massive!)
- **Response Quality**: ⭐⭐⭐⭐ (very good for general tasks)
- **Speed**: Very fast (2.5 Flash)
- **Documentation**: Good, improving rapidly
- **Reliability**: 99.5%+ uptime
- **Special Feature**: Extremely long context (great for multi-article analysis)

**For Ethnobotany Use Case**:
- **Strengths**:
  - ✅ **Massive context window (1M tokens)**: Can process entire research corpus at once
  - ✅ Much cheaper than Claude (1/10th the cost)
  - ✅ Fast responses suitable for real-time chat
  - ✅ Good at summarization (useful for condensing multiple articles)
  - ✅ Multimodal (can process images, PDFs if needed)
  - ✅ Google integration (if leveraging Workspace, Scholar)

- **Limitations**:
  - ❌ Slightly lower quality on specialized scientific reasoning (vs Claude)
  - ❌ Less stable than Claude (newer, fewer production deployments)
  - ❌ API is still evolving (breaking changes possible)

**Estimated Cost (per user)**:
- Typical query (3K tokens, 500 output): ~$0.0003
- Even complex queries: <$0.01
- Average user: ~$0.10-1/month
- **Cheapest option by far**

**Recommendation**: **Great secondary option for budget-conscious users or for tasks requiring long context**

---

### 3. ChatGPT API (OpenAI) - FALLBACK

**Latest Model**: GPT-4o (recommended) or GPT-4.5 Turbo

**Key Characteristics**:
- **Input Cost**: $1.25-5.00 per 1M tokens (depending on model)
- **Output Cost**: $10-15 per 1M tokens
- **Context Window**: 128K-200K tokens
- **Response Quality**: ⭐⭐⭐⭐⭐ (excellent overall)
- **Speed**: Medium (slower than Gemini, slightly faster than Claude Opus)
- **Documentation**: Excellent and mature
- **Reliability**: 99.9%+ uptime
- **Most Popular**: Largest user base, most community resources

**For Ethnobotany Use Case**:
- **Strengths**:
  - ✅ Most familiar to users (ChatGPT is ubiquitous)
  - ✅ Excellent all-around performance
  - ✅ Good documentation and libraries
  - ✅ Strong on creative synthesis
  - ✅ Vision capabilities (for article images)

- **Limitations**:
  - ❌ More expensive than Gemini
  - ❌ Slightly lower quality on specialized scientific tasks (vs Claude)
  - ❌ Context window smaller than Gemini

**Estimated Cost (per user)**:
- Typical query (3K input, 500 output): ~$0.02-0.05
- Complex queries: ~$0.05-0.20
- Average user: ~$2-10/month
- **Mid-range pricing**

**Recommendation**: **Good fallback for users already familiar with ChatGPT**

---

## Part 2: Pricing Comparison

### Cost Per User Query

| Scenario | Claude 3.5 Sonnet | Gemini 2.5 Flash | ChatGPT 4o |
|----------|------------------|------------------|-----------|
| Short query (500 input, 200 output) | $0.0015 | $0.00015 | $0.0065 |
| Medium query (3K input, 500 output) | $0.0105 | $0.00105 | $0.0325 |
| Long query (10K input, 2K output) | $0.0330 | $0.00330 | $0.1030 |
| Synthesis (30K input, 3K output) | $0.0945 | $0.00945 | $0.3050 |

### Monthly Cost per User (100 queries, mixed types)

| LLM | Estimated Cost |
|-----|-----------------|
| Claude 3.5 Sonnet | $1-3 |
| Gemini 2.5 Flash | $0.10-0.50 |
| ChatGPT 4o | $3-8 |

**Key Insight**: Users with their own API keys have very manageable costs (<$10/month for heavy usage).

---

## Part 3: Integration Architecture

### User-Provided API Keys Approach (RECOMMENDED)

**Design**:
```
User Registration
  → User adds API keys (Claude, Gemini, ChatGPT)
  → System stores encrypted API keys
  → User selects preferred LLM
  → Chat requests use user's API key

Benefits:
- ✅ No platform API costs (user bears cost)
- ✅ Users have control over spending
- ✅ Multi-provider support (users choose)
- ✅ Scalable (no bottleneck on single API)
- ✅ Privacy (user keys not exposed)
- ✅ Flexibility (swap providers anytime)
```

**Implementation Pattern**:
```python
from anthropic import Anthropic as ClaudeClient
from google.generativeai import GenerativeModel as GeminiClient
from openai import OpenAI as ChatGPTClient

class LLMRouter:
    def __init__(self, user_settings):
        self.user_settings = user_settings  # Contains encrypted API keys
        self.preferred_llm = user_settings['preferred_llm']

    def get_client(self):
        if self.preferred_llm == 'claude':
            return ClaudeClient(api_key=self.user_settings['claude_key'])
        elif self.preferred_llm == 'gemini':
            return GeminiClient(api_key=self.user_settings['gemini_key'])
        elif self.preferred_llm == 'chatgpt':
            return ChatGPTClient(api_key=self.user_settings['openai_key'])

    def query(self, messages, context):
        client = self.get_client()
        # Query implementation specific to chosen LLM
```

---

### Chat Interface Implementation

**Architecture**:
```
User Query (natural language)
  ↓
Retrieve Similar Articles (via Qdrant/pgvector)
  ↓
Build Context (article summaries, metadata)
  ↓
Generate Prompt
  ├─ System prompt (search over ethnobotany articles)
  ├─ Context (retrieved articles)
  ├─ User query
  └─ Conversation history
  ↓
Call LLM API (with user's API key)
  ↓
Stream Response to User
  ↓
Generate Follow-up Suggestions
```

**System Prompt Example**:
```
You are an expert ethnobotany research assistant. You help researchers
find and understand scientific articles about traditional plant use in
Brazilian communities.

You have access to a vector database of {N} ethnobotany articles. When
responding to queries, you:
1. Reference specific articles from the provided search results
2. Highlight key findings and methodologies
3. Suggest related research areas
4. Maintain scientific rigor while being accessible
5. Respect CARE principles - acknowledge traditional knowledge origins
```

---

## Part 4: Implementation Patterns

### 1. LiteLLM Abstraction (RECOMMENDED)

**Library**: `litellm` (unified interface for multiple LLMs)

```python
import litellm

response = litellm.completion(
    model="claude-3-5-sonnet-20241022",  # or gpt-4o, gemini-2.5-flash
    messages=[
        {"role": "system", "content": "You are an ethnobotany expert..."},
        {"role": "user", "content": user_query}
    ],
    temperature=0.7,
    max_tokens=2000
)
```

**Benefits**:
- ✅ Single interface for all LLMs
- ✅ Easy provider switching
- ✅ Built-in error handling and retries
- ✅ Cost tracking per provider
- ✅ Minimal code changes to swap LLMs

**Installation**: `pip install litellm`

---

### 2. Streaming Responses

**User Experience**: Real-time token streaming for better UX

```python
import litellm

# Stream response tokens as they arrive
for chunk in litellm.completion(
    model=user_preferred_model,
    messages=messages,
    stream=True
):
    yield chunk['choices'][0]['delta'].get('content', '')
```

**Implementation**: Send chunks to frontend via WebSocket for real-time display.

---

### 3. Context Window Optimization

Different LLMs have different context limits:

| LLM | Context | Strategy |
|-----|---------|----------|
| Claude 3.5 Sonnet | 200K | Use full capacity, include up to 20-30 articles |
| Gemini 2.5 Flash | 1M | Can include entire corpus if needed |
| ChatGPT 4o | 128K | Limit to 10-15 key articles |

```python
def prepare_context(articles, model_type):
    max_tokens = {
        'claude': 100000,      # Reserve 100K for context
        'gemini': 900000,      # Reserve most of 1M
        'chatgpt': 80000       # Reserve 80K
    }

    articles_to_include = select_articles(
        articles,
        max_tokens=max_tokens[model_type]
    )
    return format_context(articles_to_include)
```

---

## Part 5: API Key Management & Security

### Secure Storage

**Requirements**:
- Encrypt API keys at rest (AES-256)
- Never log API keys
- Use environment variables in development
- Use secrets management in production (e.g., AWS Secrets Manager, HashiCorp Vault)

**Implementation Pattern**:
```python
from cryptography.fernet import Fernet

class SecureKeyStore:
    def __init__(self):
        self.cipher = Fernet(os.getenv('ENCRYPTION_KEY'))

    def store_api_key(self, user_id, provider, api_key):
        encrypted = self.cipher.encrypt(api_key.encode())
        # Store encrypted_key in database

    def get_api_key(self, user_id, provider):
        encrypted = # retrieve from database
        return self.cipher.decrypt(encrypted).decode()
```

### User Consent

- Display cost estimates before making queries
- Show which LLM was used and actual token count
- Allow users to set spending limits per month
- Monthly billing summary of API usage

---

## Part 6: Fallback & Error Handling

### Multi-Provider Fallback Chain

If primary LLM fails:
```
Claude (preferred)
  → timeout/error
  → Gemini (fallback)
    → timeout/error
    → ChatGPT (final fallback)
```

**Implementation**:
```python
async def query_with_fallback(query, user_settings):
    llms = get_llm_priority(user_settings)

    for llm in llms:
        try:
            response = await call_llm(query, llm, timeout=10)
            return response
        except (TimeoutError, RateLimitError) as e:
            continue

    # If all fail, return cached/default response
    return default_response("Sorry, all services temporarily unavailable...")
```

---

## Part 7: Feature Enhancements (Future)

### 1. Extended Thinking (Claude)

For complex queries, use Extended Thinking:
```python
response = litellm.completion(
    model="claude-3-7-sonnet-20250219",
    thinking={
        "type": "enabled",
        "budget_tokens": 5000  # Let Claude think for up to 5k tokens
    },
    messages=messages
)
```

**Use cases**: "What are patterns in ethnobotany research across regions?"

**Cost**: Slightly higher (thinking tokens counted), but better quality

### 2. Vision Capabilities

Both Claude and ChatGPT support image analysis:
```python
response = litellm.completion(
    messages=[
        {"role": "user", "content": [
            {"type": "text", "text": "Analyze this plant image"},
            {"type": "image_url", "image_url": {"url": image_url}}
        ]}
    ]
)
```

**Potential use**: Analyze plant images in article figures

### 3. Function Calling

Allow LLM to call tools:
```python
tools = [
    {
        "name": "search_articles",
        "description": "Search the article database",
        "parameters": {
            "type": "object",
            "properties": {
                "query": {"type": "string"},
                "region": {"type": "string"}
            }
        }
    }
]

# LLM can now call search_articles autonomously
```

---

## Part 8: Cost Monitoring

### Per-User Dashboard

Display to users:
- Queries run this month
- Tokens used (input/output)
- Estimated cost
- Selected LLM
- Provider comparison (what each would have cost)

**Implementation**: Log every API call with token counts
```python
def log_api_usage(user_id, provider, input_tokens, output_tokens):
    usage = {
        'user_id': user_id,
        'provider': provider,
        'input_tokens': input_tokens,
        'output_tokens': output_tokens,
        'cost': calculate_cost(provider, input_tokens, output_tokens),
        'timestamp': now()
    }
    database.save(usage)
```

---

## Recommendation Summary

### Primary Architecture: User-Provided API Keys + Multi-LLM Support

**Implementation Plan**:

1. **Phase 1 (MVP)**: Claude only
   - Simplest implementation
   - Best quality for scientific analysis
   - User cost: ~$1-3/month for typical usage

2. **Phase 2 (Production)**: Add Gemini + ChatGPT
   - LiteLLM for abstraction
   - User selects preferred LLM
   - Fallback chain for reliability

3. **Phase 3 (Advanced)**: Add Extended Thinking, Vision
   - Enhanced capabilities for complex queries
   - Image analysis for article figures

### Cost to Users

| Usage Level | Claude | Gemini | ChatGPT |
|------------|--------|--------|---------|
| Light (10 queries/month) | $0.10 | $0.01 | $0.30 |
| Regular (50 queries/month) | $0.50 | $0.05 | $1.50 |
| Heavy (200 queries/month) | $2.00 | $0.20 | $6.00 |

**Key Decision**: Users bring their own API keys = zero cost to platform, manageable cost to user, maximum flexibility.

---

## Technical Stack for Chat Interface

```python
# Backend
- FastAPI (or similar for REST API)
- LiteLLM (multi-LLM abstraction)
- AsyncIO (for concurrent requests)
- PostgreSQL (message history)
- Redis (caching, session management)

# Frontend
- React / Vue.js (web chat UI)
- WebSocket (real-time streaming)
- TypeScript (type safety)

# Deployment
- Docker (containerization)
- Kubernetes (optional, for scaling)
```

### Sample Chat Backend Endpoint

```python
from fastapi import FastAPI, WebSocket
import litellm

app = FastAPI()

@app.websocket("/ws/chat/{user_id}")
async def websocket_endpoint(websocket: WebSocket, user_id: str):
    await websocket.accept()

    while True:
        data = await websocket.receive_json()
        user_query = data['message']

        # Get user's LLM preference
        user = get_user(user_id)
        llm_model = get_model_name(user.preferred_llm)

        # Retrieve relevant articles
        articles = semantic_search(user_query, limit=10)
        context = format_articles(articles)

        # Build messages
        messages = [
            {"role": "system", "content": SYSTEM_PROMPT},
            {"role": "system", "content": f"Context:\n{context}"},
            {"role": "user", "content": user_query}
        ]

        # Stream response
        async for chunk in litellm.acompletion(
            model=llm_model,
            messages=messages,
            stream=True
        ):
            content = chunk['choices'][0]['delta'].get('content', '')
            await websocket.send_text(content)
```

---

## Final Recommendations

1. **Start with Claude API** for best scientific quality
2. **Add Gemini for cost-conscious users** (1/10th price)
3. **Add ChatGPT for familiarity** (largest user base)
4. **Use LiteLLM for provider abstraction** (easy switching)
5. **Users provide API keys** (zero platform cost, user control)
6. **Implement streaming responses** (better UX)
7. **Monitor costs per user** (transparency)
8. **Plan for Extended Thinking** (future enhancement)

This architecture maximizes flexibility, minimizes platform costs, and gives users choice and control.
