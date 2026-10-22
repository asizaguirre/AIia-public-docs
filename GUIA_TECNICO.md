# AlEx Grid — guia técnico público

Este documento resume a arquitetura e as capacidades atuais do AlEx Grid.
O projeto continua em desenvolvimento; esta documentação não certifica
prontidão para produção nem homologação de integrações externas.

## Arquitetura

O frontend React/TypeScript consome somente a API Spring Boot do Back-Node.
Ele não acessa diretamente PostgreSQL, Elasticsearch ou Ollama. PostgreSQL é
a fonte de verdade para negócios, tenants, agentes e delivery; Elasticsearch
atende busca/RAG; Logstash mantém o pipeline CDC; Ollama fornece inferência
local. Nginx atua como gateway do Back-Node.

Há três perfis de implantação:

| Perfil | Serviços |
|---|---|
| Data-Node | PostgreSQL, Elasticsearch e Logstash/CDC. |
| Back-Node | API, gateway Nginx e Ollama, conectados ao Data-Node. |
| Full-Node | Serviços de dados e aplicação na mesma instalação/rede Docker. |

O menu de deploy oferece diagnóstico local, preparação de imagens e
instalação/operação dos perfis. O diagnóstico é somente leitura e não cria
máquinas virtuais. O frontend pode ser executado em desenvolvimento com Vite
ou publicado como aplicação Web/PWA independente.

## URLs e roteamento

Quando houver proxy público, a porta da URL é a chave do roteamento:
`https://aiex.com:<porta>` termina TLS no proxy de entrada e encaminha para
`http://localhost:<mesma porta>` ou para o serviço Docker publicado nessa
porta. A rede interna pode usar HTTP; TLS e controles de acesso devem ficar no
proxy/gateway de entrada. O frontend usa a API pública; não deve chamar
containers, bancos ou Ollama diretamente.

O nome `aiex.com` é uma referência de configuração, não uma afirmação de
propriedade ou disponibilidade do domínio. DNS, certificado, firewall e
encaminhamento de portas precisam ser configurados pelo operador.

## Inferência local com Ollama

Back-Node e Full-Node mantêm um Ollama primary, com CUDA somente quando a GPU
NVIDIA está disponível ao Docker; caso contrário, o serviço usa CPU. Até dois
helpers CPU-only podem ser preparados em standby. Eles compartilham o volume
de modelos, recebem limites conservadores de CPU/RAM e não são publicados
diretamente para clientes.

Um autoscaler opcional monitora CPU e memória do primary, adiciona helpers
saudáveis ao upstream Nginx sob carga e os remove do pool antes de pará-los
após um período de baixa utilização. O primary não é parado. O endpoint lógico
da API permanece único; o balanceamento não muda o contrato do frontend ou do
Back-Node. O modelo carregado por cada instância ainda consome memória própria,
portanto os limites precisam ser dimensionados e monitorados no host.

Configurações principais incluem `OLLAMA_NUM_PARALLEL` (concorrência por
instância, padrão conservador `1`), limite de modelos carregados, contexto,
memória/CPU por container e limites de scale-out/scale-in. `num_thread` é uma
opção de geração da API (`options.num_thread`), não uma variável global de
thread do Compose. Os helpers desativam CUDA. A fixação de CPU deve usar IDs
verificados no próprio host; não presuma que os P-cores correspondem sempre a
um intervalo fixo de CPUs lógicas.

O procedimento operacional completo e os exemplos de teste estão em
[OLLAMA_AUTOSCALING.md no repositório principal](https://github.com/asizaguirre/AIia/blob/main/docs/OLLAMA_AUTOSCALING.md).

## Delivery e iFood

O produto mantém cadastro empresarial, catálogo de cardápio e pedidos
internos por tenant. Os dados de integração e credenciais iFood são protegidos
no Back-Node. O adaptador iFood atualmente oferece, no contexto de teste:

1. Distributed OAuth e troca do código de autorização por tokens protegidos
   no Back-Node;
2. consulta das lojas autorizadas e seleção validada de uma loja;
3. consulta dos catálogos e publicação explícita do catálogo interno na loja
   autorizada.

A publicação externa é uma ação do operador, não uma sincronização contínua.
Itens removidos do catálogo interno não são apagados automaticamente no
parceiro. O fluxo não implementa ingestão/sincronização geral de pedidos,
estados, cancelamentos ou entregadores. As integrações comerciais com Uber,
99 e outros provedores também não estão implementadas.

Use apenas uma aplicação e uma loja de teste autorizadas pelo iFood. O fluxo
de teste não implica homologação, contrato, parceria, endosso ou autorização
para operação comercial. Consulte a [documentação oficial de autenticação
Distributed](https://developer.ifood.com.br/en-US/docs/food/guides/modules/authentication/distributed)
e os [endpoints oficiais do catálogo](https://developer.ifood.com.br/en-US/docs/food/guides/modules/catalog/endpoints).

## Agentes e n8n

Back-Node e Full-Node incluem uma instância n8n local opcional para teste,
com interface publicada apenas no loopback do host (por padrão,
`127.0.0.1:5678`). Não exponha essa porta diretamente à Internet. O webhook de
exemplo valida conectividade; o chat de agentes continua usando Ollama e RAG,
e não chama automaticamente a URL de webhook registrada no agente.

O vínculo de uma integração de delivery a um agente também não implica
execução ou roteamento automático de workflows.

## Segurança e prontidão

- Não versione `.env`, chaves, envelopes de configuração, credenciais ou
  tokens. Segredos devem ser mantidos no Back-Node e backups devem ter
  custódia segura.
- Valide fingerprints de chaves por canal independente ao importar
  configurações entre nós.
- Mantenha bancos, Elasticsearch, Logstash e Ollama em rede privada; publique
  apenas o gateway necessário.
- Preserve autenticação HMAC, timestamp e nonce nas chamadas tenant.
- Use dados de teste durante desenvolvimento e confirme consentimentos e
  permissões do provedor.
- Revise isolamento entre tenants, backups/restauração, monitoramento,
  políticas de acesso, TLS e recuperação de chaves antes de qualquer piloto.

O projeto não se declara pronto para produção. Consulte o
[checklist de prontidão](https://github.com/asizaguirre/AIia/blob/main/docs/PRODUCTION_READINESS.md)
e os guias de operação do [repositório principal](https://github.com/asizaguirre/AIia)
antes de implantar.
