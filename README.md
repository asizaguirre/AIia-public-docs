# AlEx Grid — Inteligência Artificial para o seu Negócio de Delivery

<div align="center">

**A plataforma que coloca IA nativa, automação e integração multicanal na mão de qualquer lojista.**

*Sem mensalidade por chamada de API. Sem dados seus na nuvem de terceiros. Sem lock-in.*

[![Versão](https://img.shields.io/badge/versão-2.4-indigo?style=flat-square)](https://github.com/asizaguirre/AIia)
[![Licença](https://img.shields.io/badge/licença-proprietária-slate?style=flat-square)](https://github.com/asizaguirre/AIia)
[![Status](https://img.shields.io/badge/status-operacional-emerald?style=flat-square)](https://github.com/asizaguirre/AIia)

</div>

---

## Por que o AlEx Grid existe

Cada restaurante no Brasil paga entre **30–40% de comissão** nas plataformas de delivery. Faz isso porque não tem alternativa — sem tecnologia própria, sem dados de clientes, sem controle.

O AlEx Grid muda esse jogo.

Com ele, o lojista tem seu próprio cardápio online, seu próprio assistente de IA atendendo 24/7, seus pedidos centralizados e suas integrações com iFood, Rappi e Uber Eats operando no menor custo possível — tudo em uma plataforma que **ele controla, hospedada onde ele quiser**.

---

## O que é o AlEx Grid

Uma **plataforma completa de gestão de delivery com IA nativa**, composta por:

| Componente | Função |
|:---|:---|
| **Painel do lojista** | Gestão de cardápio, pedidos em tempo real, integrações e relatórios |
| **Cardápio público** | Página própria da loja com carrinho, checkout e assistente de IA |
| **Assistente LLM** | Responde clientes em linguagem natural, 24/7, sobre produtos e entrega |
| **Motor de agentes** | n8n + Ollama: automação de atendimento, relatórios e workflows customizados |
| **Hub de delivery** | Integração com iFood, Rappi, Uber Eats, 99Food e Keeta em um painel |
| **Otimizador de frete** | Compara entregadores em tempo real e seleciona o mais barato |
| **RAG empresarial** | Base de conhecimento privada que alimenta todos os agentes de IA |

---

## Funcionalidades em produção

### 🤖 Assistente de Cardápio com IA Real

O cliente acessa o cardápio da loja, faz uma pergunta — *"Tem opção sem glúten?"*, *"Qual o frete para meu bairro?"*, *"Aceitam PIX?"* — e recebe uma resposta em linguagem natural, imediatamente.

Não é um chatbot de respostas fixas. É um LLM (Qwen 2.5) rodando **na própria infraestrutura da loja**, treinado com o catálogo real de produtos.

```
Cliente: "O que vocês têm de vegetariano?"
Assistente: "Temos o Bowl de Quinoa com Legumes (R$ 32,90) e a
             Pizza Margherita sem anchovas (R$ 48,90). Ambos são
             preparados sem proteína animal. Posso ajudar com
             mais alguma coisa?"
```

### 📦 Kanban de Pedidos ao Vivo

Todos os pedidos — do cardápio próprio, do iFood, do Rappi — em um único painel kanban. Drag-and-drop para mover entre status, atualização automática a cada 15 segundos, filtro por plataforma.

### 🛒 Cardápio Público com Link Próprio

Cada loja tem um link único (`/menu/nome-da-loja`) que funciona como um mini-ecommerce: catálogo com fotos, carrinho, cálculo de frete em tempo real, formulário de pedido e assistente de IA flutuante.

### 🔌 Hub de Integrações com 5 Plataformas

Painel unificado para configurar iFood, Rappi, Uber Eats, 99Food e Keeta. Inclui guia passo a passo de credenciais, links diretos para portais dev de cada parceiro e instruções de sandbox para testes sem custo.

### 🚀 Deploy em 1 Comando

```bash
./scripts/deploy.sh start    # inicia tudo
./scripts/deploy.sh tunnel   # URL HTTPS pública via ngrok
./scripts/deploy.sh aws      # scripts prontos para EC2 Free Tier
```

---

## Para quem é

| Perfil | Como o AlEx Grid ajuda |
|:---|:---|
| **Restaurante / dark kitchen** | Cardápio próprio + assistente IA + integração iFood sem intermediário |
| **Rede / franquia** | Arquitetura multi-tenant: cada loja isolada, gestão centralizada |
| **Integrador / SaaS** | White-label pronto: troque a marca, hospede, monetize |
| **Parceiro de tecnologia** | API aberta para delivery, agentes e RAG — integre ao seu ecossistema |
| **Investidor / acelerador** | Veja a [arquitetura técnica](./ARQUITETURA_REFERENCIA.md) e o [modelo de segurança](./SEGURANCA.md) |

---

## Diferenciais técnicos

### IA sem custo variável
O LLM roda localmente (Ollama). **Zero custo por chamada de API** — ao contrário de soluções baseadas em OpenAI/Anthropic, onde cada pergunta de cliente gera custo. Com o AlEx Grid, quanto mais você usa, mais vale.

### Dados que ficam com você
Nenhuma pergunta de cliente, nenhum produto do catálogo, nenhuma conversa de atendimento sai para servidores externos de IA. **Privacidade por arquitetura**, não por política de uso.

### Isolamento criptográfico por loja
Cada negócio tem um token HMAC-SHA256 único. É matematicamente impossível para um tenant acessar dados de outro — não é uma regra de negócio, é uma garantia criptográfica.

### Escala sem reescrita
Da pizza de bairro à rede de franquias: a arquitetura multi-tenant suporta dezenas de negócios no mesmo servidor ou milhares em um cluster distribuído, sem alterar uma linha de código de produto.

---

## Documentação técnica

| Documento | Conteúdo |
|:---|:---|
| [Modelo de Arquitetura](./ARQUITETURA_REFERENCIA.md) | Diagramas completos, CQRS, fluxos de autenticação, topologias de deploy |
| [Mecanismos de IA e n8n](./N8N_AI_WORKFLOWS.md) | 6 workflows ativos, fluxo LLM, RAG, orquestração |
| [Segurança Multi-Tenant](./SEGURANCA.md) | Criptografia AES-256-GCM, HMAC, isolamento, análise de ameaças |
| [Brief para Parceiros](./PARTNER_INTEGRATION_BRIEF.md) | Proposta de integração com plataformas de delivery |
| [Guia Técnico Completo](./GUIA_TECNICO.md) | Referência de operação para times técnicos |
| [Repositório principal](https://github.com/asizaguirre/AIia) | Código-fonte completo |

---

## Próximos passos

**Se você é um lojista:**
Entre em contato para uma demonstração ao vivo do assistente no seu cardápio.

**Se você é uma plataforma de delivery:**
Leia o [Brief de Integração](./PARTNER_INTEGRATION_BRIEF.md) e nos escreva para uma conversa técnica.

**Se você é um investidor ou parceiro:**
A plataforma está operacional. Podemos fazer uma sessão técnica ao vivo.

---

<div align="center">

**Contato:** asizaguirre@gmail.com

**Repositório:** [github.com/asizaguirre/AIia](https://github.com/asizaguirre/AIia)

---

*AlEx Grid v2.4 — Powered by [Alam Izaguirre](mailto:asizaguirre@gmail.com) (A.I.) · 2026*

</div>
