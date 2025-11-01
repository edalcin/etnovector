# EtnoVector: Base de Dados Vetorizada de Artigos Científicos sobre Etnobotânica

Base de dados semântica de artigos científicos sobre etnobotânica (uso tradicional de plantas por comunidades brasileiras) com busca inteligente por IA e sistema de recomendações.

**Status**: Em Planejamento • **Versão**: 0.1.0 (MVP em Desenvolvimento)

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

### 1. Ingestão de Artigos (PDF + URL)

- **Upload de PDF**: Envie artigos científicos em PDF
- **Submissão por URL**: Envie links para artigos (DOI, links de journals, arXiv, ResearchGate)
- **Extração Automática de Metadados**: Sistema extrai automaticamente título, autores, ano, resumo
- **Sem Retenção de PDF**: Metadados + embeddings armazenados, links para artigo original mantidos
- **Detecção de Duplicatas**: Previne artigos duplicados por DOI ou similaridade de título

### 2. Busca Semântica Inteligente (Chat)

```
Usuário: "Quais plantas usadas por comunidades amazônicas têm propriedades anti-parasitárias validadas cientificamente?"

EtnoVector:
1. Busca pela similaridade semântica do significado
2. Retorna artigos relevantes mesmo com terminologia diferente
3. Opcionalmente: LLM sintetiza os achados em resposta conversacional
```

**Características**:
- Busca em português e inglês
- Integração com Claude, Gemini ou ChatGPT (escolha do usuário)
- Mantém contexto de conversa para refinamento iterativo
- Mostra quais artigos foram usados para a resposta

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

### Componentes do Sistema

```
┌─────────────────────────────────────────────────────────────────┐
│                  Interface Web (React)                          │
│           Upload de PDFs • Busca Semântica • Chat               │
│              Dashboard de Tendências • Gerenciamento            │
└──────────────────────────┬──────────────────────────────────────┘
                           │ REST/WebSocket
                           │
┌──────────────────────────▼──────────────────────────────────────┐
│           Backend API (FastAPI/Python)                          │
│  ┌──────────────┬──────────────┬──────────────────────────────┐ │
│  │ Ingestão de  │   Busca &    │   Recomendações &           │ │
│  │ Artigos      │ Interface Chat  Análise de Tendências      │ │
│  │ - Extrator PDF                                            │ │
│  │ - Busca URLs    ┌──────────────────────────┐             │ │
│  │ - Metadados     │  SPECTER2 (Embeddings)   │             │ │
│  └────────┬────────┤ Modelo de IA p/ semântica│─────┬───────┘ │
│           │        └──────────────────────────┘     │         │
└───────────┼────────────────────────────────────────┬┼─────────┘
            │                                        ││
    ┌───────▼────────────┐              ┌────────────▼▼──────────┐
    │  PostgreSQL 15+    │              │  PostgreSQL pgvector   │
    │  (Metadados)       │              │  (Embeddings de 768dim)│
    │ - Título, autores  │              │                        │
    │ - Resumo, palavras │              │ Busca semântica <50ms  │
    │ - Comunidades      │              │ para 50k artigos       │
    │ - Aprovações CARE  │              │                        │
    └────────────────────┘              └────────────────────────┘
```

### Stack Tecnológico

| Componente | Tecnologia | Por quê? |
|-----------|-----------|---------|
| **Embeddings** | SPECTER2 (Hugging Face) | Otimizado para artigos científicos, task-adaptive |
| **Banco de Dados SQL** | PostgreSQL 15+ | ACID, full-text search, pgvector para vetores |
| **Banco Vetorial** | pgvector (MVP) / Qdrant (Produção) | Eficiente, open-source, custo-benefício |
| **Backend API** | FastAPI (Python) | Async-first, moderno, excelente documentação |
| **Frontend** | React 18 + TypeScript | Tipo-seguro, componentes reutilizáveis |
| **LLM Integration** | LiteLLM + Claude/Gemini/ChatGPT | Multi-provider, switch automático de APIs |
| **Container** | Docker + Docker Compose | Reproduzível, fácil desenvolvimento |
| **PDF Processing** | PyPDF2, pdfplumber | Extração de texto e metadados |

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

- [Especificação de Funcionalidades](./specs/001-ethnobotany-vector-db/spec.md)
- [Plano de Implementação](./specs/001-ethnobotany-vector-db/plan.md)
- [Modelo de Dados](./specs/001-ethnobotany-vector-db/data-model.md)
- [API Specification](./specs/001-ethnobotany-vector-db/contracts/api-specification.md)
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

Este projeto foi inspirado por:

1. **Sistemas RAG (Retrieval-Augmented Generation)** - Combina bancos vetoriais com LLMs
2. **Princípios CARE** - Para dados indígenas (Collective Benefit, Authority, Responsibility, Ethics)
3. **Etnobotânica Colaborativa** - Envolvimento direto de comunidades
4. **Open-Science** - Acesso público ao conhecimento

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

- **Comunidades Indígenas**: Pelo conhecimento ancestral sobre plantas
- **Pesquisadores**: Pelos artigos que tornam este conhecimento visível
- **Allenai**: Por SPECTER2 (modelo de embedding científico)
- **PostgreSQL**: Pela confiabilidade e pgvector
- **LiteLLM**: Pela abstração multi-LLM

---

**Última Atualização**: 1º de Novembro de 2025
**Versão**: 0.1.0 (Planejamento Detalhado Completo)

---

## 📞 Suporte

Precisa de ajuda?

- 📚 Leia a [Documentação](./docs/)
- 🐛 Reporte bugs em [Issues](https://github.com/edalcin/etnovector/issues)
- 💬 Participe das [Discussions](https://github.com/edalcin/etnovector/discussions)
- 📧 Contato: [configurar email]

---

**EtnoVector**: Conectando Conhecimento Tradicional com Inovação Científica 🌿🔬
