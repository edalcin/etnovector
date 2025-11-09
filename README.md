# EtnoVector: Base de Dados Vetorizada de Artigos Científicos sobre Etnobotânica

Base de dados semântica de artigos científicos sobre etnobotânica (uso tradicional de plantas por comunidades brasileiras) com busca inteligente por IA e sistema de recomendações.

**Status**: Em Planejamento • **Versão**: 0.1.0 (MVP em Desenvolvimento)

**Arquitetura de Referência**: [docling-rag-agent](https://github.com/coleam00/ottomator-agents/tree/main/docling-rag-agent) - Sistema RAG com ingestão multi-formato e chat conversacional

---

## 📚 O Projeto

### Visão Geral

EtnoVector é uma plataforma de pesquisa especializada que conecta comunidades tradicionais, pesquisadores acadêmicos e tomadores de decisão através de uma base de dados vetorizada de artigos científicos sobre etnobotânica.

Diferentemente de ferramentas de busca tradicionais (como Google Scholar), EtnoVector usa **embeddings semânticos** (representações matemáticas do significado) para encontrar artigos relacionados mesmo quando usam terminologia diferente. Por exemplo:

- **Busca tradicional** por "plantas para inflamação": encontra apenas artigos com essas palavras
- **EtnoVector** por "plantas para inflamação em comunidades amazônicas": encontra artigos sobre anti-inflamatórios, plantas medicinais regionais, etnofarmacologia, mesmo sem usar exatamente essas palavras

### Benefícios Acadêmicos

1. **Descoberta Semântica**: Encontre artigos relacionados sem precisar conhecer terminologia técnica específica
2. **Análise de Tendências**: Identifique quais plantas estão sendo mais estudadas, lacunas de pesquisa, padrões entre regiões
3. **Síntese Inteligente**: LLM (Claude, Gemini, ChatGPT) sintetiza achados de múltiplos artigos em resposta a perguntas
4. **Recomendações Não-Óbvias**: Descubra conexões entre pesquisas de diferentes regiões que estudam plantas com propriedades similares
5. **Integração com Comunidades**: Respeite e reconheça conhecimento tradicional via princípios CARE

### Benefícios para Comunidades Tradicionais

1. **Soberania de Dados**: Comunidades mantêm controle sobre seu conhecimento tradicional
2. **Reconhecimento**: Atribuição clara de conhecimento à comunidade de origem
3. **Participação**: Comunidades validam e aprovam informações sobre seu conhecimento
4. **Análise de Tendências**: Comunidades veem quais plantas de seu conhecimento estão sendo pesquisadas
5. **Sem Retenção de Dados**: Sistema armazena apenas metadados e embeddings, não retém PDFs (respeita direitos autorais)

---

## 🎯 Capacidades Principais

### 1. Ingestão Multi-Formato de Documentos (Docling)

- **Formatos Suportados**: PDF, Word (DOCX), PowerPoint (PPTX), Excel (XLSX), HTML, Markdown, TXT
- **Processamento Automático**: Detecção automática de formato e conversão via [Docling](https://github.com/DS4SD/docling)
- **Upload de Arquivo**: Envie documentos via drag-and-drop ou seleção
- **Submissão por URL**: Envie links para artigos (DOI, links de journals, arXiv, ResearchGate)
- **Extração Automática de Metadados**: Sistema extrai automaticamente título, autores, ano, resumo
- **Chunking Semântico**: Documentos longos são divididos em segmentos de 1000 tokens para processamento eficiente
- **Sem Retenção de Documentos**: Apenas metadados + embeddings armazenados, links para fonte original mantidos
- **Detecção de Duplicatas**: Previne duplicados por DOI ou similaridade de conteúdo

**Referência Técnica**: Baseado no pipeline de ingestão do [docling-rag-agent](https://github.com/coleam00/ottomator-agents/tree/main/docling-rag-agent) com conversão para Markdown normalizado

### 2. Interface de Chat RAG com Tool-Calling (PydanticAI)

```
Usuário: "Quais plantas usadas por comunidades amazônicas têm propriedades anti-parasitárias validadas cientificamente?"

EtnoVector (via PydanticAI Agent):
1. LLM analisa a pergunta e decide chamar a ferramenta search_knowledge_base
2. Busca vetorial retorna top-k chunks relevantes (similaridade > 0.7)
3. LLM sintetiza resposta com citações inline dos artigos encontrados
4. Retorna resposta conversacional com fontes verificáveis
```

**Características**:
- **Arquitetura de Agentes**: Framework [PydanticAI](https://ai.pydantic.dev/) com tool-calling para orquestração
- **Multi-Provider LLM**: Suporte para Claude, Gemini e ChatGPT (chaves de API fornecidas pelo usuário)
- **Streaming de Respostas**: Feedback token-por-token em tempo real
- **Contexto Multi-Turno**: Mantém histórico de conversa para refinamento iterativo
- **Citações com Scores**: Mostra artigos usados com scores de similaridade
- **Busca Bilíngue**: Funciona em português e inglês

**Referência Técnica**: Arquitetura baseada no agente RAG do [docling-rag-agent](https://github.com/coleam00/ottomator-agents/tree/main/docling-rag-agent) com `search_knowledge_base` como tool callable

### 3. Sistema de Recomendações

Dado um artigo, descubra outros relacionados de formas não-óbvias:
- Mesma planta, diferentes regiões
- Diferentes plantas com propriedades similares
- Diferentes metodologias de pesquisa
- Conexões via citações

Exemplo: "Artigos que estudam propriedades anti-inflamatórias em plantas de diferentes famílias"

### 4. Análise de Tendências e Lacunas de Pesquisa

Dashboard de análise com:
- **Plantas mais estudadas** por região, ano, propriedade
- **Lacunas de pesquisa**: Quais plantas em quais regiões têm <5 estudos
- **Padrões**: Quais propriedades estão crescendo em interesse
- **Comparação regional**: Diferenças entre conhecimento tradicional em diferentes comunidades

### 5. Governança CARE (Collective Benefit, Authority to Decide, Responsibility, Ethics)

Implementação dos princípios CARE para dados indígenas e de conhecimento tradicional:

- **Coletividade**: Comunidades têm voz coletiva nas decisões sobre dados
- **Autoridade**: Comunidades controlam quem pode acessar/usar conhecimento tradicional
- **Responsabilidade**: Rastreamento de uso, transparência, benefits-sharing
- **Ética**: Respeito às tradições, consentimento informado, privacidade

**Implementação**:
- Artigos reconhecem origem do conhecimento tradicional
- Comunidades validam/disputam informações sobre seu conhecimento
- Auditoria completa de acesso e uso
- Relatórios de impacto compartilhados com comunidades

### 6. Monitoramento de Publicações

Sistema automático que:
- Monitora journals científicas e APIs (PubMed, CrossRef, arXiv)
- Busca artigos novos sobre etnobotânica
- Adiciona automaticamente à base de dados
- Notifica pesquisadores sobre publicações novas relevantes

---

## 🏗️ Arquitetura Técnica

**Baseado em**: [docling-rag-agent](https://github.com/coleam00/ottomator-agents/tree/main/docling-rag-agent) - Arquitetura de referência comprovada para RAG com multi-formato

### Componentes do Sistema

```
┌─────────────────────────────────────────────────────────────────┐
│                  Interface Web (React)                          │
│    Upload Multi-Formato • Chat RAG • Recomendações • CARE      │
│              Dashboard de Tendências • Gerenciamento            │
└──────────────────────────┬──────────────────────────────────────┘
                           │ REST API + WebSocket
                           │
┌──────────────────────────▼──────────────────────────────────────┐
│           Backend API (FastAPI/Python)                          │
│  ┌──────────────┬──────────────┬──────────────────────────────┐ │
│  │ Ingestão     │  PydanticAI  │   Recomendações &           │ │
│  │ Multi-Formato│  RAG Agent   │   Análise de Tendências      │ │
│  │              │              │                               │ │
│  │  Docling     │  Tool-Calling│   Similaridade Vetorial      │ │
│  │  Pipeline    │  search_kb() │   Gap Analysis               │ │
│  │  ↓           │  ↓           │   ↓                          │ │
│  │  Markdown    │  PostgreSQL  │   Dashboard                  │ │
│  │  Chunking    │  + pgvector  │   Visualizações              │ │
│  │  Embeddings  │  Streaming   │                              │ │
│  └──────┬───────┴──────┬───────┴──────────────────────────────┘ │
│         │              │                                         │
│    ┌────▼──────────────▼────────────────┐                      │
│    │ OpenAI text-embedding-3-small     │                      │
│    │ 1536 dimensions • ~$0.02/1M tokens│                      │
│    └───────────────────────────────────┘                      │
└───────────┬────────────────────────────────────────────────────┘
            │
    ┌───────▼────────────────────────────────────────┐
    │  PostgreSQL 15+ com pgvector Extension         │
    │                                                 │
    │  Tabela: documents                              │
    │  - Metadados (título, autores, DOI, URL)       │
    │  - Status de processamento                      │
    │  - Community approvals (CARE)                   │
    │                                                 │
    │  Tabela: chunks                                 │
    │  - Texto do segmento                            │
    │  - Embedding vetorial [1536d]                   │
    │  - Referência ao documento original             │
    │                                                 │
    │  Função: match_chunks(query_embedding, limit)   │
    │  - Busca por cosine similarity (<=> operator)   │
    │  - Retorna top-k chunks mais relevantes         │
    │  - Filtragem por threshold (0.7 default)        │
    │                                                 │
    │  Connection Pooling: 2-10 async connections     │
    └─────────────────────────────────────────────────┘
```

### Stack Tecnológico

**Baseado na arquitetura comprovada do [docling-rag-agent](https://github.com/coleam00/ottomator-agents/tree/main/docling-rag-agent)**

| Componente | Tecnologia | Por quê? | Referência |
|-----------|-----------|---------|------------|
| **Document Processing** | [Docling](https://github.com/DS4SD/docling) | Conversão multi-formato (PDF, DOCX, PPTX, XLSX, HTML, MD) para Markdown normalizado | docling-rag-agent |
| **Embeddings** | OpenAI text-embedding-3-small (1536d) | Alta qualidade, custo-efetivo (~$0.02/1M tokens), usado em produção | docling-rag-agent |
| **Banco de Dados** | PostgreSQL 15+ com pgvector | Single database para metadados + vetores, ACID, cosine similarity via `<=>` operator | docling-rag-agent |
| **Vector Search** | pgvector extension | Busca eficiente com cosine distance, function `match_chunks()`, threshold filtering | docling-rag-agent |
| **Agent Framework** | [PydanticAI](https://ai.pydantic.dev/) | Tool-calling architecture, streaming responses, type-safe, multi-provider LLM | docling-rag-agent |
| **LLM Providers** | OpenAI GPT-4o-mini (default) + Claude + Gemini | Multi-provider com user-provided API keys, provider abstraction layer | docling-rag-agent |
| **Backend API** | FastAPI (Python) + asyncpg | Async-first, connection pooling (2-10 connections), high performance | docling-rag-agent |
| **Database Client** | asyncpg | Async PostgreSQL driver, efficient connection pooling | docling-rag-agent |
| **Container** | Docker + Docker Compose | Reproduzível, PostgreSQL + pgvector container + app container | docling-rag-agent |
| **Package Manager** | UV | Fast Python package management (opcional, pode usar pip) | docling-rag-agent |
| **Frontend** | React 18 + TypeScript | Tipo-seguro, componentes reutilizáveis, chat interface com streaming | Planejado |

**Extensões ao docling-rag-agent**:
- **Etnobotânica Features**: Metadados customizados (plantas, regiões, propriedades medicinais)
- **CARE Principles**: Community approval workflows, audit logging, benefit-sharing tracking
- **Trend Analysis**: Dashboard de análise de gaps e padrões de pesquisa
- **Publication Monitoring**: APIs de journals (PubMed, CrossRef, arXiv) para ingestão automática

### Dados Não Retidos

⚠️ **IMPORTANTE**: Este sistema responde princípios de retenção de dados mínima:

- ✅ Armazenado: Metadados (título, autores, resumo, DOI)
- ✅ Armazenado: Embeddings (representação vetorial para busca)
- ✅ Armazenado: Links para artigo original (DOI, URL)
- ❌ **NÃO armazenado**: PDF completo dos artigos
- ❌ **NÃO armazenado**: Texto completo extraído de PDFs
- ✅ Mantido: Link persistente para acesso ao original via DOI/URL

Esta abordagem:
- Respeita direitos autorais
- Minimiza armazenamento (economiza custos)
- Mantém flexibilidade para atualizações
- Garante acesso permanente ao original

---

## 🚀 Como Começar

### Pré-requisitos

- **Docker e Docker Compose** (recomendado para desenvolvimento)
- **Python 3.11+** (se desenvolvimento local)
- **Node.js 18+** (se desenvolvimento do frontend)
- **Git**

### Instalação Rápida (Docker)

```bash
# Clone o repositório
git clone https://github.com/edalcin/etnovector.git
cd etnovector

# Copie arquivo de configuração
cp .env.example .env

# Inicie containers
docker-compose up -d

# Inicialize base de dados
docker-compose exec backend python -m alembic upgrade head

# Acesse a aplicação
# Frontend: http://localhost:3000
# API Docs: http://localhost:8000/docs
```

### Desenvolvimento Local

```bash
# Backend
cd backend
python -m venv venv
source venv/bin/activate  # ou `venv\Scripts\activate` no Windows
pip install -r requirements.txt
python -m uvicorn app.main:app --reload

# Frontend (em outro terminal)
cd frontend
npm install
npm start
```

---

## 📖 Documentação

### Para Pesquisadores/Usuários

- [Guia de Uso](./docs/user-guide-pt.md) - Como usar a plataforma
- [FAQ](./docs/faq-pt.md) - Perguntas frequentes
- [Princípios CARE](./docs/care-principles-pt.md) - Explicação dos princípios éticos

### Para Desenvolvedores

- [Especificação de Funcionalidades](./spec.md)
- [Plano de Implementação](./plan.md)
- [Modelo de Dados](./data-model.md)
- [API Specification](./api-specification.md)
- [Pesquisa: Embedding Models](./research/research-embedding-models.md)
- [Pesquisa: Databases](./research/research-databases.md)
- [Pesquisa: LLM APIs](./research/research-llm-apis.md)
- [Guia de Contribuição](./CONTRIBUTING.md)
- [CLAUDE.md](./CLAUDE.md) - Guia para Claude Code

### Para Administradores/Operações

- [Guia de Deployment](./docs/deployment-pt.md)
- [Estratégia de Backup](./docs/backup-recovery-pt.md)
- [Monitoramento](./docs/monitoring-pt.md)

---

## 📊 Status do Projeto

### Fases de Desenvolvimento

- [x] **Fase 0**: Especificação e Pesquisa Tecnológica
  - ✅ Especificação de funcionalidades (6 user stories)
  - ✅ Pesquisa de modelos de embedding (SPECTER2)
  - ✅ Pesquisa de bancos de dados (PostgreSQL + pgvector)
  - ✅ Pesquisa de LLM APIs (Claude, Gemini, ChatGPT)

- [ ] **Fase 1**: Foundation (Semanas 1-3)
  - [ ] Setup Docker e projeto estrutura
  - [ ] Schema PostgreSQL com pgvector
  - [ ] Upload de PDF + extração de metadados
  - [ ] Geração de embeddings com SPECTER2

- [ ] **Fase 2**: Search & Chat (Semanas 4-5)
  - [ ] Busca semântica
  - [ ] Interface de chat com LLM
  - [ ] Submissão por URL
  - [ ] WebSocket streaming

- [ ] **Fase 3**: Recommendations & Analytics (Semanas 6-7)
  - [ ] Motor de recomendações
  - [ ] Dashboard de tendências
  - [ ] Análise de lacunas

- [ ] **Fase 4**: CARE & Monitoring (Semanas 8-9)
  - [ ] Governança de comunidades
  - [ ] Monitoramento de publicações
  - [ ] Auditoria e compliance

- [ ] **Fase 5**: Polish & Launch (Semanas 10-11)
  - [ ] Testes integração e carga
  - [ ] Documentação completa
  - [ ] Deploy em produção

### Cronograma

**MVP (Mínimo Viável)**: Semanas 1-5 (busca semântica + chat funcional)
**Produção**: Semanas 1-11 (todas features)

---

## 🤝 Contribuindo

Contribuições são bem-vindas! Por favor:

1. Leia [CONTRIBUTING.md](./CONTRIBUTING.md)
2. Crie discussão sobre mudanças maiores em Issues
3. Faça commits descritivos em português quando possível
4. Envie PR com descrição clara

**Área de Focus Atual**:
- Desenvolvimento do backend FastAPI
- Integração com SPECTER2
- Setup PostgreSQL + pgvector

---

## 📝 Licença

Este projeto é distribuído sob a licença **MIT** (veja [LICENSE](./LICENSE)).

Porém, note que:
- Os modelos de embedding (SPECTER2) têm licença Apache 2.0
- Dados de artigos (metadados) são vinculados a obras originais (respeitando direitos autorais)
- Conhecimento tradicional é regido pelos princípios CARE

---

## 👥 Comunidade & Contato

### Contribuidores Principais

- **Pesquisa & Especificação**: Claude Code + Community Input
- **Arquitetura**: Baseada em padrões de RAG (Retrieval-Augmented Generation) com ênfase em ética

### Parceiros Esperados

- 🌿 Comunidades indígenas e tradicionais (Kayapó, Asháninka, etc.)
- 🏫 Instituições acadêmicas (Universidades brasileiras)
- 🔬 Programas de etnobotânica e etnofarmacologia
- 📚 Periódicos científicos (integração de APIs)

### Contato

- **Issues & Feedback**: GitHub Issues
- **Discussões**: GitHub Discussions
- **Email**: [configurar contato]

---

## 🔬 Pesquisa & Inspiração

### Arquitetura de Referência Técnica

Este projeto é baseado diretamente na arquitetura do **[docling-rag-agent](https://github.com/coleam00/ottomator-agents/tree/main/docling-rag-agent)**, um sistema RAG comprovado em produção que implementa:

- ✅ **Ingestão multi-formato via Docling** (PDF, DOCX, PPTX, XLSX, HTML, MD, TXT)
- ✅ **PostgreSQL + pgvector** para armazenamento unificado de metadados e embeddings
- ✅ **PydanticAI agent framework** com tool-calling architecture
- ✅ **OpenAI embeddings** (text-embedding-3-small, 1536 dimensões)
- ✅ **Streaming responses** token-por-token
- ✅ **Connection pooling** (asyncpg) para performance
- ✅ **Docker containerization** para deploy reproduzível

**Por que usar docling-rag-agent como base?**
1. **Arquitetura comprovada em produção** - Não reinventar a roda
2. **Stack moderno e eficiente** - FastAPI async, PydanticAI, pgvector
3. **Multi-formato out-of-the-box** - Docling suporta 7+ formatos automaticamente
4. **Código de referência disponível** - Implementação clara para seguir
5. **Best practices embutidas** - Chunking, pooling, streaming, error handling

### Inspirações Adicionais

1. **Princípios CARE** - Para dados indígenas (Collective Benefit, Authority, Responsibility, Ethics)
2. **Etnobotânica Colaborativa** - Envolvimento direto de comunidades tradicionais
3. **Open-Science** - Acesso público ao conhecimento científico
4. **RAG Systems** - Combina retrieval de documentos com geração de LLMs

---

## 🛠️ Roadmap Futuro

### Curto Prazo (3-6 meses)
- [ ] MVP com busca e chat funcionais
- [ ] Integração com 2-3 APIs de journals
- [ ] Comunidade piloto de 5 grupos

### Médio Prazo (6-12 meses)
- [ ] Escala para 50k artigos
- [ ] Suporte multilíngue (português, inglês, espanhol)
- [ ] Mobile app
- [ ] Integração com plataformas acadêmicas (ResearchGate, Academia.edu)

### Longo Prazo (1-2 anos)
- [ ] Cobertura global de etnobotânica
- [ ] Marketplace de dados com benefit-sharing automático
- [ ] Integração com repositórios institucionais
- [ ] Análise de impacto econômico do conhecimento

---

## 📌 Notas Importantes

### Segurança de Dados

- Chaves de API (Claude, Gemini, ChatGPT) são **criptografadas** e **específicas por usuário**
- Senhas são **hashed** com bcrypt
- Comunicação é **HTTPS** em produção
- Auditoria de acesso a conhecimento tradicional é **loggada**

### Conformidade Legal

- Projeto segue princípios CARE para dados indígenas
- Metadados de artigos respeitam direitos autorais (não armazena texto completo)
- Comunidades têm direito de retirar/corrigir informações
- Transparência em usos comerciais (benefit-sharing)

### Sustentabilidade

- Stack escolhido é **open-source** (sem vendor lock-in)
- Custos de infraestrutura estimados em **$30-60/mês** para 50k artigos
- Modelo de sustentabilidade: doações, grants acadêmicos, parcerias institucionais

---

## 🙏 Agradecimentos

- **[docling-rag-agent](https://github.com/coleam00/ottomator-agents/tree/main/docling-rag-agent)**: Arquitetura de referência técnica que serviu de base para este projeto
- **Comunidades Indígenas**: Pelo conhecimento ancestral sobre plantas
- **Pesquisadores**: Pelos artigos que tornam este conhecimento visível
- **[Docling (DS4SD)](https://github.com/DS4SD/docling)**: Biblioteca multi-formato para conversão de documentos
- **[PydanticAI](https://ai.pydantic.dev/)**: Framework moderno de agentes com tool-calling
- **PostgreSQL & pgvector**: Pela confiabilidade e busca vetorial eficiente
- **OpenAI**: Por text-embedding-3-small e APIs de LLM

---

**Última Atualização**: 9 de Novembro de 2025
**Versão**: 0.2.0 (Arquitetura baseada em docling-rag-agent)

---

## 📞 Suporte

Precisa de ajuda?

- 📚 Leia a [Documentação](./docs/)
- 🐛 Reporte bugs em [Issues](https://github.com/edalcin/etnovector/issues)
- 💬 Participe das [Discussions](https://github.com/edalcin/etnovector/discussions)
- 📧 Contato: [configurar email]

---

**EtnoVector**: Conectando Conhecimento Tradicional com Inovação Científica 🌿🔬
