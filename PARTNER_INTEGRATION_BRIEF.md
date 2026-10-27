# Brief de Integração para Parceiros — AlEx Grid v2.4

<div align="center">

*Proposta técnica de colaboração com plataformas de delivery e provedores do ecossistema*

**Powered by Alam Izaguirre (A.I.)**

---

*Este documento é uma proposta de colaboração técnica. As marcas mencionadas indicam categorias de interesse. Não há endosso, autorização ou contrato implícitos.*

</div>

---

## O problema que resolvemos juntos

Hoje, um lojista que quer estar em 3 plataformas de delivery precisa:

- Cadastrar o mesmo cardápio **3 vezes** em interfaces diferentes
- Atualizar preços e disponibilidade **3 vezes** quando há mudança
- Acompanhar pedidos em **3 apps diferentes** no celular
- Treinar atendentes para **3 fluxos operacionais** distintos
- Pagar taxas para **3 operações de suporte** separadas

O resultado: erros de cardápio, pedidos perdidos, cliente insatisfeito e lojista exausto.

**O AlEx Grid é a camada de gestão unificada** que resolve isso sem substituir nenhuma plataforma — sendo o ponto central de onde o lojista opera todos os canais simultaneamente.

---

## Nossa proposta para plataformas parceiras

Queremos ser um **multiplicador de qualidade** para o ecossistema de cada parceiro:

| Para a plataforma | O que ganhamos juntos |
|:---|:---|
| Cardápio sempre atualizado | Menos pedidos cancelados por item indisponível |
| Resposta mais rápida a pedidos | Melhor NPS do lojista na plataforma |
| Onboarding simplificado | Mais estabelecimentos integrados |
| Dados de catálogo mais ricos | Melhor ranqueamento e conversão |

---

## Estado atual das integrações

### ✅ iFood — Funcional

```
OAuth:
  ├── Fluxo Centralizado (client_credentials)
  │     → token gerado automaticamente com Client ID + Secret
  └── Fluxo Distribuído (user_code)
        → código gerado no portal para confirmação manual

Catálogo:
  ├── Importação do Portal do Parceiro → AlEx Grid
  ├── Publicação do catálogo interno → iFood API
  └── Consulta de categorias e lojas autorizadas

Sandbox disponível em: https://developer.ifood.com.br
```

### ⚙️ Rappi — Configurável

```
Estrutura pronta: credenciais, merchant_id, modo logístico
Adaptador de pedidos: em desenvolvimento
Sandbox: https://developer.rappi.com
```

### ⚙️ Uber Eats / Direct — Configurável

```
OAuth 2.0: fluxo configurável no painel
Uber Direct (entrega própria): em avaliação
Sandbox: https://developer.uber.com/docs/eats
```

### ⚙️ 99Food — Aguardando acesso sandbox

```
Estrutura de credenciais pronta
API em expansão no Brasil
Portal: https://open.99app.com
```

### ⚙️ Keeta — Aguardando documentação API BR

```
Estrutura de credenciais pronta
Documentação em português: limitada
Portal: https://developer.keeta.com
```

### ✅ Entregadores / Logística — Cotação funcional

```
Motoboy Próprio · iFood Entregadores · Rappi Turbo
Loggi · Lalamove · Zap Delivery

→ Comparativo de frete em tempo real
→ Seleção automática do mais barato por distância
→ Links diretos para contratação
```

---

## O que precisamos de cada parceiro

Para uma integração completa e homologada, precisamos de:

| Capacidade | Descrição |
|:---|:---|
| **Acesso ao sandbox** | Ambiente de desenvolvimento com credenciais de teste |
| **Webhook de pedidos** | Notificação em tempo real de novos pedidos |
| **API de atualização de status** | CONFIRMED → PREPARING → READY → DISPATCHED |
| **API de catálogo** | CRUD de produtos, categorias, preços e disponibilidade |
| **Merchant onboarding** | API para cadastro programático de estabelecimentos |
| **Webhook de cancelamento** | Notificação de cancelamentos e estornos |

---

## O que o AlEx Grid oferece em troca

```
Para os lojistas integrados via AlEx Grid:

✓  Catálogo sempre sincronizado (reduz erros de disponibilidade)
✓  Resposta a pedidos em < 30 segundos (IA notifica imediatamente)
✓  Dados de catálogo estruturados e validados
✓  Menor taxa de cancelamento por item indisponível
✓  Melhor experiência do cliente no cardápio próprio
     (que complementa, não concorre com a plataforma)
```

---

## Arquitetura da integração proposta

```
Plataforma Parceira
       │
       │  POST /webhook/pedido-recebido
       │  (assinado com HMAC do parceiro)
       │
       ▼
AlEx Grid Back-Node
  ├── Valida assinatura do webhook do parceiro
  ├── Identifica tenant pelo merchant_id
  ├── Armazena pedido no PostgreSQL (cifrado)
  ├── Atualiza Kanban do lojista em tempo real
  ├── n8n workflow: notifica agente de atendimento
  └── Lojista confirma → PUT /api/parceiro/pedidos/{id}/status
                              { status: "CONFIRMED" }
       │
       ▼
  Plataforma Parceira recebe atualização de status
```

---

## Limitações que declaramos abertamente

Ser transparente faz parte do nosso compromisso:

| Capacidade | Status atual |
|:---|:---|
| Sincronização de pedidos marketplace em produção | ❌ Não implementado |
| Cancelamentos e estornos via API | ❌ Não implementado |
| Rastreamento de entregadores em tempo real | ❌ Aguardando API logística |
| Notificações push para clientes finais | ❌ Fora do escopo atual |
| Homologação comercial de qualquer plataforma | ❌ Não declarada |

---

## Conversa técnica

Se você representa uma plataforma de delivery ou provedor logístico e quer avaliar uma integração, podemos oferecer:

- **Demonstração ao vivo** do fluxo de pedidos e catálogo
- **Acesso ao código** para avaliação técnica (repositório privado)
- **Sessão técnica** de 1h para discutir arquitetura e requisitos
- **Prova de conceito** em ambiente controlado

**Contato:** asizaguirre@gmail.com

**Repositório técnico:** [github.com/asizaguirre/AIia](https://github.com/asizaguirre/AIia)

**Documentação:**
- [Modelo de Arquitetura](./ARQUITETURA_REFERENCIA.md)
- [Segurança e Isolamento](./SEGURANCA.md)
- [Workflows de IA](./N8N_AI_WORKFLOWS.md)

---

<div align="center">

*AlEx Grid v2.4 · Powered by Alam Izaguirre (A.I.) · 2026*

</div>
