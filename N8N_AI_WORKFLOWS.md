# Mecanismos de IA e Workflows n8n — AlEx Grid v2.4

<div align="center">

*Como a inteligência artificial é orquestrada dentro da plataforma*

**Powered by Alam Izaguirre (A.I.)**

</div>

---

## A IA do AlEx Grid não é um chatbot

É uma orquestração completa de modelos de linguagem, base de conhecimento semântica e automação de processos — rodando integralmente na sua infraestrutura.

```
Pergunta do cliente
       │
       ▼
  Contexto real       ──► Ollama LLM  ──►  Resposta natural
  (catálogo + loja)        local              em < 5 segundos
```

Não há prompt fixo. Não há respostas pré-cadastradas. Cada resposta é gerada com base no catálogo real da loja, nas políticas cadastradas e no contexto da conversa.

---

## Infraestrutura de IA

```
┌────────────────────────────────────────────────────────────────┐
│                   STACK DE IA DO BACK-NODE                      │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Ollama (container Docker)                               │   │
│  │                                                          │   │
│  │  qwen2.5:3b ──────── geração de texto e chat            │   │
│  │  nomic-embed-text ── vetores de 768 dimensões            │   │
│  │                                                          │   │
│  │  keep_alive = -1 → modelo sempre na memória             │   │
│  │  GPU NVIDIA auto-detectada → fallback CPU se ausente    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                           │                                      │
│  ┌────────────────────────▼─────────────────────────────────┐  │
│  │  Nginx Gateway (http://gateway:11434)                     │  │
│  │  Balanceia entre Ollama primary (GPU) e helpers (CPU)    │  │
│  └─────────────────────────────────────────────────────────┘   │
│                           │                                      │
│  ┌────────────────────────▼─────────────────────────────────┐  │
│  │  n8n (workflow engine interno)                            │  │
│  │  Porta 5678 — somente loopback (nunca exposto)           │  │
│  │                                                          │  │
│  │  6 workflows ativos, chamados via webhooks internos      │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────┘
```

---

## Os 6 Workflows em Produção

### 1. 💬 Assistente de Cardápio Público
**Webhook:** `POST /webhook/menu-chat`

O mais impactante para o cliente final. Responde qualquer pergunta sobre produtos, preços, entrega e pagamento em linguagem natural.

**Fluxo:**
```
1. Recebe: { message, catalog[], menu_title, distance_km }
2. Monta prompt com contexto completo da loja
3. Chama Ollama qwen2.5:3b
4. Retorna: { answer, agent_name, powered_by_ai: true }
```

**Exemplo de pergunta → resposta:**
```
Entrada:  "Tem algo sem lactose?"
Saída:    "Sim! Nossa Pizza Napolitana (R$ 42,90) é feita com
           azeite no lugar de queijo. O Risoto de Cogumelos
           (R$ 38,00) também não leva laticínios. Deseja
           adicionar algum desses ao pedido?"
```

---

### 2. 🧠 Alia — Agente RAG e Parametrização
**Webhook:** `POST /webhook/agent-chat`

Alia é a especialista em conhecimento organizacional. Consulta a base RAG da empresa para responder com dados precisos, não com "achismos".

**Fluxo:**
```
1. Recebe: { message, email, agent }
2. Busca documentos relevantes no Elasticsearch
   (BM25 léxico + kNN semântico em combinação)
3. Injeta contexto dos documentos no prompt
4. Chama Ollama com contexto enriquecido
5. Se detecta necessidade de criar componente/tela:
   └── Emite JSON estruturado → pipeline Alex
```

---

### 3. ⚙️ Alex — Pipeline de Engenharia de Software
**Webhook:** `POST /webhook/alex-pipeline`

Alex implementa o que Alia planeja. Gera código, cria páginas de produto, otimiza workflows.

**Fluxo:**
```
1. Recebe plano estruturado de Alia
2. Analisa o plano com Ollama (contexto técnico)
3. Executa ações: criar página, gerar HTML, criar componente
4. Retorna assets criados + próximos passos
```

---

### 4. 📄 Product Page Builder — Agente CQRS
**Webhook:** `POST /webhook/product-page-agent`

Transforma uma instrução em linguagem natural em uma página de produto completa e publicada.

**Fluxo:**
```
1. Entrada: "Crie a página do meu bolo de chocolate,
             R$45, WhatsApp 11999999999"
2. Ollama gera: { action: "create", slug, title, content }
3. PageCommandService → PostgreSQL (persistência)
4. PageProjectionPublisher → Elasticsearch (busca)
5. Retorna: { publicUrl: "/menu/bolo-chocolate", ... }
```

**Resultado:** uma página profissional com foto, preço, descrição, botão WhatsApp e chatbot embutido — criada em segundos, sem saber programar.

---

### 5. 📚 RAG Training — Ingestão Contínua
**Webhook:** `POST /webhook/rag-training`

Mantém a base de conhecimento da empresa sempre atualizada.

**Fluxo:**
```
1. Recebe documento (texto, PDF extraído, FAQ)
2. Divide em chunks (parágrafos)
3. Gera embedding de 768 dimensões por chunk
   usando nomic-embed-text
4. Indexa no Elasticsearch com metadata do tenant
5. Base fica disponível para todos os agentes
```

---

### 6. 🔧 AlEx Grid Webhook Test
**Webhook:** `POST /webhook/alex-grid-n8n-test`

Diagnóstico de conectividade. Confirma que o n8n, a API e o Ollama estão se comunicando corretamente.

---

## Configuração em 1 Comando

```bash
bash back-node/bin/setup-n8n-workflows
```

O script faz tudo automaticamente:
- Cria o usuário admin do n8n
- Importa todos os workflows de `shared/n8n-workflows/`
- Obtém o `versionId` de cada workflow
- Ativa todos os webhooks
- Exibe o status final

---

## Performance Real

| Métrica | Valor medido |
|:---|:---:|
| Tempo de resposta (saudação simples) | ~3–5 segundos |
| Tempo de resposta (consulta de cardápio) | ~4–8 segundos |
| Cold start (modelo não carregado) | ~8–12 segundos |
| Com `keep_alive=-1` (modelo sempre na memória) | ~2–5 segundos |
| Respostas simultâneas (OLLAMA_NUM_PARALLEL=2) | 2 sem bloqueio |

**Hardware de referência dos testes:** CPU Intel i7-10th gen, 32 GB RAM, sem GPU. Com GPU NVIDIA, os tempos caem ~60%.

---

## Variáveis de configuração

```bash
# back-node/.env — configurações relevantes para IA
OLLAMA_URL=http://gateway:11434        # Ollama via gateway interno
OLLAMA_MODEL=qwen2.5:3b               # Modelo de chat
OLLAMA_CONTEXT_LENGTH=4096            # Contexto máximo em tokens
OLLAMA_NUM_PARALLEL=1                 # Respostas simultâneas
OLLAMA_KEEP_ALIVE=5m                  # Tempo em memória (use -1 para sempre)

N8N_ENABLED=false                     # n8n como fallback de /api/chat
N8N_MENU_ASSISTANT_WEBHOOK_URL=http://n8n:5678/webhook/menu-chat
N8N_PRODUCT_PAGE_WEBHOOK_URL=http://n8n:5678/webhook/product-page-agent
N8N_RAG_WEBHOOK_URL=http://n8n:5678/webhook/rag-training
```

---

<div align="center">

*Workflows de IA — AlEx Grid v2.4 · Powered by Alam Izaguirre (A.I.)*

[Arquitetura](./ARQUITETURA_REFERENCIA.md) · [Segurança](./SEGURANCA.md) · [Repositório](https://github.com/asizaguirre/AIia)

</div>
