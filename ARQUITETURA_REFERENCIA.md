# Modelo de Arquitetura — AlEx Grid v2.4

<div align="center">

*Documento técnico de referência para times de engenharia, parceiros e investidores*

**Powered by Alam Izaguirre (A.I.)**

</div>

---

## Visão Arquitetural

O AlEx Grid segue uma arquitetura **distribuída em três camadas independentes**, onde cada camada se comunica com a seguinte por contratos criptográficos — nunca por confiança implícita. Cada camada pode ser implantada em máquinas separadas ou colocada em um único host sem alterar a lógica de produto.

```
╔════════════════════════════════════════════════════════════════════╗
║                   CAMADA 1 — APRESENTAÇÃO                          ║
║                                                                    ║
║  ┌─────────────────────────┐  ┌─────────────────────────────────┐ ║
║  │   Painel do Lojista      │  │   Cardápio Público              │ ║
║  │   (autenticado)          │  │   (acesso anônimo)              │ ║
║  │                          │  │                                 │ ║
║  │  📋 Kanban de pedidos    │  │  🛒 Catálogo + Carrinho         │ ║
║  │  🍔 Gestão de catálogo   │  │  💬 Assistente LLM flutuante    │ ║
║  │  🤖 Estúdio de agentes   │  │  🚚 Cálculo de frete ao vivo    │ ║
║  │  🔌 Hub de plataformas   │  │  📱 Mobile-first responsivo     │ ║
║  └─────────────────────────┘  └─────────────────────────────────┘ ║
║                                                                    ║
║  React 18 · TypeScript · Vite 8 · Tailwind CSS 4                  ║
║  PWA (offline-first) · Tauri (desktop) · Capacitor (iOS/Android)  ║
╚══════════════════════════╦═════════════════════════════════════════╝
                           ║
                           ║  HTTPS + HMAC-SHA256
                           ║  (assinatura por tenant)
                           ║
╔══════════════════════════▼═════════════════════════════════════════╗
║                   CAMADA 2 — APLICAÇÃO (BACK-NODE)                 ║
║                                                                    ║
║  ┌──────────────┐  ┌────────────────┐  ┌──────────────────────┐  ║
║  │  API REST     │  │  Ollama LLM    │  │  n8n Workflow Engine  │  ║
║  │  Spring Boot  │  │  qwen2.5:3b   │  │                      │  ║
║  │               │  │  + nomic-embed │  │  6 workflows ativos  │  ║
║  │  Multi-tenant │  │                │  │  Webhooks internos   │  ║
║  │  CQRS pages   │  │  GPU/CPU auto  │  │  IA orquestrada      │  ║
║  │  RAG search   │  │  keep_alive=-1 │  │  127.0.0.1:5678      │  ║
║  └──────────────┘  └────────────────┘  └──────────────────────┘  ║
║                                                                    ║
║  Nginx Gateway (porta 3001) — balanceamento + headers segurança    ║
╚══════════════════════════╦═════════════════════════════════════════╝
                           ║
                           ║  Rede Docker interna (alex_data_net)
                           ║  Resolução por hostname — sem IPs fixos
                           ║
╔══════════════════════════▼═════════════════════════════════════════╗
║                   CAMADA 3 — DADOS (DATA-NODE)                     ║
║                                                                    ║
║  ┌──────────────────────┐    ┌──────────────────────────────────┐ ║
║  │  PostgreSQL 16        │    │  Elasticsearch 8.15              │ ║
║  │  (fonte autoritativa) │    │  (busca semântica + RAG kNN)     │ ║
║  │                       │    │                                  │ ║
║  │  Tenants · Pedidos    │    │  Índice rag_knowledge_base       │ ║
║  │  Catálogos · Agentes  │    │  Embeddings 768-dim              │ ║
║  │  Credenciais cifradas │    │  BM25 + kNN híbrido              │ ║
║  └──────────────────────┘    └──────────────────────────────────┘ ║
║                                                                    ║
║  Logstash — pipeline CDC: PostgreSQL → Elasticsearch (RAG)        ║
╚════════════════════════════════════════════════════════════════════╝
```

---

## Fluxo do Assistente de IA no Cardápio

Este é o fluxo de maior impacto para o cliente final — cada pergunta no cardápio da loja é respondida por IA real:

```
Cliente faz pergunta no cardápio
         │
         ▼
POST /api/public/delivery-menu/{slug}/agent-chat
         │
         ▼
PublicMenuAssistantService.answer()
         │
         ├── findPublicAgent(tenantId)
         │        │
         │        ├── [tem agente configurado]
         │        │        │
         │        │        ▼
         │        │   answerWithLlm()
         │        │        │
         │        │        ├── Monta prompt com:
         │        │        │   • Catálogo completo da loja
         │        │        │   • Descrição e políticas da loja
         │        │        │   • Opções de entrega calculadas
         │        │        │   • Pergunta do cliente
         │        │        │
         │        │        ├── OllamaClient.generate(prompt)
         │        │        │   → http://gateway:11434/api/generate
         │        │        │   → qwen2.5:3b (sempre em memória)
         │        │        │
         │        │        └── Retorna: { answer, powered_by_ai: true }
         │        │
         │        └── [sem agente] → answerRuleBased() [fallback]
         │
         └── Resposta para o cliente em < 5 segundos
```

---

## Padrão CQRS — Páginas de Produto

O módulo de páginas implementa separação total entre leitura e escrita:

```
ESCRITA (Write Side)
──────────────────────────────────────────────────────────────
  Instrução em linguagem natural
    → ProductPageAgentService (Ollama interpreta)
    → Plano JSON: { action, slug, title, content }
    → PageCommandService → PostgreSQL (fonte da verdade)
    → PageProjectionPublisher → Elasticsearch (embedding)

LEITURA (Read Side)
──────────────────────────────────────────────────────────────
  GET /api/pages/{slug}
    → PageQueryService → Elasticsearch (BM25 + kNN)
                      ↘ fallback → PostgreSQL
    → renderPage() → HTML completo com shortcodes
    → Entregue ao visitante via CDN/cache

SHORTCODES SUPORTADOS NO CONTEÚDO
──────────────────────────────────────────────────────────────
  ![alt](url)                    → imagem responsiva
  [video](url)                   → player HTML5
  [youtube](url)                 → embed responsivo
  [whatsapp:55119999](Falar)     → botão WhatsApp direto
  [button:Comprar](url)          → CTA estilizado
  [chatbot:Agente](Descrição)    → widget de chat embutido
  [product:Nome|R$|Desc|img|url] → card de produto
  [pricing]...[/pricing]         → tabela de preços
  [features:título]chips         → badges de diferenciais
```

---

## Fluxo de Autenticação e Provisionamento de Tenant

```
  Usuário
    │
    ├─[1]─ Google OAuth ──► Back-Node verifica credencial Google
    │
    ├─[2]─ Cadastro do negócio ──► status: PENDING (sem token)
    │
    ├─[3]─ Admin aprova no painel ──► token HMAC-SHA256 gerado
    │                                  back-conf emitido (AES-256-GCM)
    │
    ├─[4]─ Proprietário importa back-conf no browser
    │       └── WebCrypto API decifra em memória (nunca em disco)
    │
    └─[5]─ Sessão ativa: cada request assina com HMAC-SHA256
            Headers: X-Tenant-Id · X-Nonce · X-Timestamp · X-Signature
            Validação: ApiAuthenticationFilter → TenantAuthentication
```

---

## Topologias de Implantação

### 🔵 Full-Node — Tudo em 1 Máquina

```
Host único (recomendado: ≥ 8 GB RAM)
├── data_node_postgres        ← PostgreSQL 16
├── data_node_elasticsearch   ← Elasticsearch 8.15
├── data_node_logstash        ← Pipeline CDC → RAG
├── back_node_api             ← API Spring Boot (redes: back + data)
├── back_node_ollama          ← qwen2.5:3b + nomic-embed-text
├── back_node_n8n             ← Orquestração de workflows
└── back_node_gateway         ← Nginx (porta 3001)

Custo estimado: EC2 t3.medium ~$33/mês | VPS 4vCPU/8GB ~$20/mês
```

### 🟢 Distribuído — Escala Horizontal

```
VM 1 (Data-Node)         VM 2 (Back-Node)        Qualquer lugar
│  PostgreSQL              │  Spring Boot API      │  Frontend
│  Elasticsearch   ◄──────│  Ollama               │  (Vite/CDN)
│  Logstash         data-  │  n8n
│                   conf   │  Nginx
│                   seguro │
└──────────────────────────┘

Comunicação interna: rede Docker (alex_data_net)
Hostname DNS: data_node_postgres, data_node_elasticsearch
```

### ☁️ AWS Free Tier

```bash
# Scripts gerados automaticamente:
./scripts/deploy.sh aws

# Inclui:
# - user-data.sh (boot automático EC2)
# - Makefile (make sync HOST=... KEY=... → deploy remoto SSH)
# - Guia de Security Group (portas e regras)

# Instância recomendada: t3.small (2 vCPU, 2 GB RAM)
# Custo estimado Free Tier: R$ 0 por 12 meses
```

---

## Modelos de IA Utilizados

| Modelo | Uso | RAM necessária | Custo por chamada |
|:---|:---|:---:|:---:|
| `qwen2.5:3b` | Chat, agentes, páginas, instruções | ~3 GB | **R$ 0** |
| `nomic-embed-text` | Embeddings RAG (768 dim) | ~1 GB | **R$ 0** |

**Por que local-first?** Soluções SaaS de IA cobram por token. Um restaurante com 500 conversas/dia × 500 tokens cada = 250.000 tokens/dia. No ChatGPT API isso custaria ~R$ 75/dia (~R$ 2.250/mês). No AlEx Grid: **R$ 0,00**.

---

## Stack Tecnológico Completo

| Camada | Tecnologia | Versão |
|:---|:---|:---:|
| Frontend Web | React + TypeScript + Vite | 18 / 5 / 8 |
| Estilização | Tailwind CSS | 4 |
| Desktop nativo | Tauri | 2 |
| Mobile | Capacitor | 8 |
| API Backend | Spring Boot | 3.5 |
| Linguagem backend | Java | 21+ |
| Banco de dados | PostgreSQL | 16 |
| Busca / RAG | Elasticsearch | 8.15 |
| Pipeline CDC | Logstash | 8.x |
| LLM local | Ollama (qwen2.5:3b) | latest |
| Orquestração IA | n8n | 2.44 |
| Gateway / Proxy | Nginx | 1.27 |
| Containerização | Docker + Compose | latest |

---

## Garantias de Arquitetura

| Garantia | Como é implementada |
|:---|:---|
| Um tenant nunca acessa dados de outro | `tenant_id` obrigatório em todas as queries SQL |
| Dados de IA nunca saem para terceiros | Ollama roda localmente; sem chamadas para OpenAI/Anthropic |
| Credenciais nunca em texto claro | AES-256-GCM em repouso, WebCrypto em memória no browser |
| Replay de requisição impossível | Nonce único + janela de timestamp de 30s |
| Operação sem internet | PWA offline-first com IndexedDB cifrado |

---

<div align="center">

*Arquitetura AlEx Grid v2.4 · Powered by Alam Izaguirre (A.I.)*

[Repositório principal](https://github.com/asizaguirre/AIia) · [Segurança](./SEGURANCA.md) · [Workflows de IA](./N8N_AI_WORKFLOWS.md)

</div>
