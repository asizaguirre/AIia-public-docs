# Segurança e Isolamento Multi-Tenant — AlEx Grid v2.4

<div align="center">

*Porque "confiança" não é uma estratégia de segurança*

**Powered by Alam Izaguirre (A.I.)**

</div>

---

## Princípios de Design

O AlEx Grid foi projetado do zero com quatro garantias inegociáveis:

> **1. Isolamento matemático** — não é uma regra de negócio que impede um tenant de ver dados de outro. É impossibilidade criptográfica.

> **2. Segredos zero-knowledge** — nem o operador da plataforma consegue ler as credenciais de um tenant sem o PIN do proprietário.

> **3. IA local-first** — dados de clientes e catálogo nunca saem para APIs externas de IA.

> **4. Defesa em camadas** — se uma proteção falhar, as outras permanecem intactas.

---

## Cadeia de Confiança Criptográfica

```
Data-Node                   Back-Node               Proprietário do negócio
    │                           │                           │
    │  gera Ed25519 keypair      │                           │
    │  assina data-conf.enc      │                           │
    │  ──────────────────────►  │                           │
    │                           │  verifica assinatura      │
    │                           │  valida fingerprint       │
    │                           │  (canal independente)     │
    │                           │  decifra AES-256-GCM      │
    │                           │  conecta ao cluster       │
    │                           │                           │
    │                           │  gera token HMAC          │
    │                           │  cifra back-conf.enc      │
    │                           │  ──────────────────────►  │
    │                           │                           │  decifra no browser
    │                           │                           │  (WebCrypto API)
    │                           │                           │  sessão em memória
    │                           │                           │  nunca em disco
```

---

## Envelope `data-conf` — Anatomia

```json
{
  "magic": "ALEXDATA",
  "version": "2.0.0",
  "cluster_id": "dn-cluster-{16 hex chars}",

  "kdf": {
    "algorithm": "PBKDF2-HMAC-SHA256",
    "iterations": 600000,
    "salt_b64": "{16 bytes aleatórios em Base64}"
  },

  "cipher": {
    "algorithm": "AES-256-GCM",
    "nonce_b64": "{12 bytes aleatórios em Base64}"
  },

  "payload_b64": "{conteúdo cifrado — endpoints PostgreSQL + Elasticsearch}",

  "integrity_sha256": "{SHA-256 do payload cifrado}",

  "signature": {
    "algorithm": "Ed25519",
    "public_key_pem": "{chave pública do Data-Node}",
    "signature_b64": "{SHA-256(magic+version+cluster_id+salt+nonce+payload)}",
    "fingerprint_hint": "Valide por canal independente: SHA-256(public_key_pem)[:16].upper()"
  }
}
```

**Por que 600.000 iterações PBKDF2?**
O NIST SP 800-132 (2023) recomenda este mínimo para proteger segredos de média entropia contra ataques de força bruta modernos. Com GPU high-end, testar 1 bilhão de senhas/segundo levaria ~16 minutos para 8 caracteres aleatórios — impraticável em cenários reais.

**Por que validação de fingerprint por canal independente?**
Protege contra ataques MITM onde um atacante substitui a chave pública no envelope. O fingerprint é calculado como `SHA-256(public_key_pem)[:16].upper()` e validado através de um canal diferente (e-mail seguro, presença física, SMS).

---

## Isolamento Multi-Tenant

### Camada de banco de dados

Todas as tabelas sensíveis têm `tenant_id` como componente obrigatório:

```sql
-- Qualquer query de agente sempre filtra por tenant
SELECT agent_id, agent_name, role
FROM tenant_n8n_agents
WHERE tenant_id = ?
ORDER BY created_at ASC;

-- Pedidos totalmente isolados
SELECT * FROM tenant_delivery_orders
WHERE tenant_id = ? AND order_id = ?;

-- Catálogo por tenant — impossível listar outro tenant
SELECT * FROM tenant_delivery_catalog_items
WHERE tenant_id = ? AND available = TRUE;
```

Não existe query cross-tenant no código. O isolamento é estrutural.

### Camada HTTP — Validação de Assinatura

```
Toda requisição tenant passa por ApiAuthenticationFilter antes
de chegar ao controller:

  1. Extrai: X-Tenant-Id, X-Nonce, X-Timestamp, X-Signature

  2. Valida timestamp:
     │now - X-Timestamp│ < 30 segundos
     → Proteção anti-replay por janela temporal

  3. Valida nonce:
     UUID único por requisição
     → Proteção anti-replay por unicidade

  4. Busca tenant_secret no PostgreSQL por tenant_id

  5. Recalcula: HMAC-SHA256(request_body, tenant_secret)

  6. Compara usando tempo constante:
     hmac.compare_digest(expected, received)
     → Protege contra timing attacks

  7. Se válido: injeta TenantAuthentication no request attribute
     → Disponível em todos os controllers como parâmetro typed
```

### Busca RAG — Isolamento por tenant no Elasticsearch

```json
{
  "knn": {
    "field": "embedding",
    "query_vector": [...],
    "num_candidates": 50,
    "k": 10,
    "filter": { "term": { "tenant_id": "tnt_xyz" } }
  },
  "query": {
    "bool": {
      "must": [
        { "term": { "tenant_id": "tnt_xyz" } },
        { "match": { "content": "pizza" } }
      ]
    }
  }
}
```

Um tenant nunca vê documentos de outro no RAG — mesmo na busca vetorial semântica.

---

## Autenticação de Usuário (Google OAuth)

```
Por que não senha própria?

Senhas próprias = responsabilidade de armazenar hashes,
                  gerenciar resets, defender brute-force,
                  lidar com vazamentos.

Google OAuth = delegação de autenticação para um sistema
               com equipe de segurança de classe mundial.
               Zero senha armazenada no AlEx Grid.
               MFA automático se o usuário tiver no Google.
```

**Fluxo técnico:**
```
1. Browser carrega Google GSI (Identity Services) dinamicamente
2. Usuário clica em "Continuar com Google"
3. Google retorna JWT assinado com chaves públicas rotativas
4. Back-Node valida: assinatura + audiência + expiração
5. Email extraído → verificado contra ADMIN_EMAILS para admins
6. Sessão criada com JWT próprio (expira em 7 dias)
```

---

## Gestão de Segredos

| Segredo | Localização | Proteção |
|:---|:---|:---|
| Passphrase do `data-conf` | Apenas na memória do operador | Jamais versionada ou transmitida |
| `data_node_ed25519.key` | `data-node/keys/` (chmod 600) | `.gitignore`d |
| `back_node_ed25519.key` | `back-node/keys/` (chmod 600) | `.gitignore`d |
| `tenant_credentials.key` | `back-node/keys/` (chmod 600) | 32 bytes aleatórios (OpenSSL) |
| `JWT_SECRET` | `back-node/.env` (chmod 600) | `openssl rand -hex 32` |
| Token HMAC por tenant | PostgreSQL | 256 bits aleatórios (SecureRandom) |
| Credenciais iFood/Uber | PostgreSQL | AES-GCM cifrado |
| `back-conf` decifrado | Memória RAM do browser | WebCrypto API — sem persistência |

**Regra de ouro:** se um arquivo contém segredo, ele é `.gitignore`d e recebe `chmod 600`. Violações desta regra são tratadas como incidentes de segurança.

---

## O que o Nginx garante

```nginx
# Headers de segurança adicionados em todas as respostas:
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "no-referrer-when-downgrade" always;
add_header Access-Control-Allow-Origin '*' always;   # CORS explícito
```

**Portas expostas em produção:**
- `3001` — Gateway API (HTTPS em produção, HTTP internamente)
- `22` — SSH (acesso ao servidor)

**Portas NUNCA expostas:**
- PostgreSQL (5432) — rede Docker interna
- Elasticsearch (9200) — rede Docker interna
- Ollama (11434) — rede Docker interna
- n8n (5678) — somente `127.0.0.1`

---

## Análise de Ameaças

| Ameaça | Impacto | Mitigação |
|:---|:---:|:---|
| Roubo do `data-conf.enc` | 🔴 Alto | AES-256-GCM: sem passphrase, dados irrecuperáveis |
| Substituição de chave pública | 🔴 Alto | Fingerprint Ed25519 validado por canal independente |
| Replay de requisição autenticada | 🟡 Médio | Nonce único + janela de 30s |
| Cross-tenant data access | 🔴 Alto | tenant_id obrigatório em todas as queries + filtro HTTP |
| Timing attack no HMAC | 🟡 Médio | `hmac.compare_digest()` tempo constante |
| Exfiltração via LLM | 🔴 Alto | Ollama local — zero chamadas para APIs externas |
| Acesso ao n8n pelo exterior | 🟡 Médio | Porta 5678 vinculada ao loopback |
| Credenciais de integração expostas | 🔴 Alto | AES-GCM no PostgreSQL — nunca em texto claro |
| Sessão browser comprometida | 🟡 Médio | back-conf em memória — sem localStorage |
| Força bruta no `data-conf` | 🔴 Alto | PBKDF2 600k iterações — tempo proibitivo |

---

## Auditoria e Rastreabilidade

Todos os eventos de segurança relevantes são logados:
- Autenticação de tenant (sucesso/falha + motivo)
- Criação, aprovação e rejeição de negócios
- Operações CQRS em páginas de produto
- Falhas de integração com parceiros (`IFOOD_UPSTREAM_FAILURE`)
- Tentativas de acesso com assinatura inválida

Os logs ficam no stdout dos containers e podem ser integrados a qualquer stack de observabilidade (ELK, CloudWatch, Datadog, Grafana Loki).

---

## Rotação de Chaves

Para procedimentos de rotação de `JWT_SECRET`, chaves Ed25519 e `tenant_credentials.key`, consulte:
[`configs/security/KEY_ROTATION.md`](https://github.com/asizaguirre/AIia/blob/main/configs/security/KEY_ROTATION.md)

---

<div align="center">

*Segurança AlEx Grid v2.4 · Powered by Alam Izaguirre (A.I.)*

Dúvidas de segurança: **asizaguirre@gmail.com**

[Arquitetura](./ARQUITETURA_REFERENCIA.md) · [Workflows de IA](./N8N_AI_WORKFLOWS.md) · [Repositório](https://github.com/asizaguirre/AIia)

</div>
