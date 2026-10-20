# AlEx | Plataforma independente para operações comerciais

## Português

O AlEx está desenvolvendo uma experiência independente para negócios que
precisam organizar presença digital, agentes assistivos e operações de pedidos
em um único fluxo. A arquitetura mantém o frontend desacoplado dos serviços de
dados e de IA, com atenção ao isolamento por negócio e à execução local ou
distribuída.

### Para plataformas e parceiros de delivery

Buscamos diálogo técnico com plataformas, operadores e provedores de delivery
para avaliar integrações oficiais, seguras e homologadas. O objetivo é reduzir
fricção operacional para negócios, sem substituir os canais, regras comerciais
ou controles de cada parceiro.

**Estado atual:** o sistema inclui fluxo interno de pedidos, interface de
cadastro/configuração de plataformas e preparação de credenciais. Os
adaptadores oficiais para iFood, Uber, 99 e outros provedores ainda não estão
implementados nem homologados. A interface de configuração não significa que
exista sincronização de catálogo, pedidos, status, cancelamentos ou entregadores
com esses serviços. A menção às marcas não implica parceria, endosso ou
autorização.

Leia o [brief de integração para parceiros](./PARTNER_INTEGRATION_BRIEF.md)
para conhecer a proposta, os limites atuais e os próximos passos.

### Visão do produto

- **Frontend independente:** aplicação web/PWA e shells para desktop e mobile,
  conectados ao backend por API.
- **Operação por negócio:** identidade e dados operacionais são tratados em
  escopos próprios; o isolamento precisa ser validado em cada ambiente antes do
  uso real.
- **Apoio por IA:** agentes podem apoiar descoberta, consulta e organização de
  informações. Ações críticas continuam sujeitas a regras determinísticas e
  revisão humana.
- **Entrega:** existe um fluxo de entrega própria e uma base de onboarding para
  futuras conexões oficiais. Cada função externa depende da implementação e
  homologação do adaptador correspondente.
- **Implantação flexível:** componentes podem ser executados em conjunto ou
  separados. Exemplos de desenvolvimento não são configurações prontas para
  produção.

### Maturidade e contato

O projeto está em desenvolvimento e **ainda não está pronto para produção**.
Antes de uma operação comercial, são necessários endurecimento de rede, testes
de isolamento e resiliência, exercícios de backup/recuperação, monitoramento e
homologação de integrações externas.

Uma avaliação conjunta pode começar pela documentação oficial de APIs,
ambientes sandbox, regras de credenciais e webhooks, requisitos de homologação,
privacidade e testes com dados não produtivos. Nenhuma conexão deve ser ativada
sem autorização do provedor e validação de segurança e operação.

Para conversar publicamente sobre o projeto, abra uma discussão ou issue neste
repositório. Não publique credenciais, dados pessoais ou informações
confidenciais.

### Documentos

- [Brief de integração para parceiros](./PARTNER_INTEGRATION_BRIEF.md) —
  proposta de colaboração e estado factual das capacidades.
- [Referência histórica de arquitetura](./ARQUITETURA_REFERENCIA_HISTORICA.md) —
  material anterior; não é a especificação vigente.
- O [PDF de arquitetura](./Modelo%20de%20Arquitetura%20do%20Sistema%20e%20Guia%20de%20Integra%C3%A7%C3%A3o.pdf)
  é conceitual, histórico e está em português. Não deve ser usado como
  documentação de APIs atuais, prova de integração homologada ou guia de
  implantação em produção.

---

## English

AlEx is developing an independent experience for businesses that need to
organize their digital presence, assistive agents, and order operations in a
single workflow. The architecture keeps the frontend decoupled from data and AI
services, with a focus on business-level isolation and local or distributed
deployment.

### For delivery platforms and partners

We welcome technical discussions with delivery platforms, operators, and
ecosystem providers to explore official, secure, and approved integrations. The
goal is to reduce operational friction for businesses without replacing each
partner's channels, commercial rules, or controls.

**Current status:** the system includes an internal order workflow, a platform
registration/configuration interface, and credential-configuration
preparation. Official adapters for iFood, Uber, 99, and other providers have
not been implemented or approved. The configuration interface does not mean
that catalog, order, status, cancellation, or courier data is synchronized with
those services. Mentioning these brands does not imply a partnership,
endorsement, or authorization.

Read the [delivery partner integration brief](./PARTNER_INTEGRATION_BRIEF.md)
for the proposal, current limitations, and suggested next steps.

### Product overview

- **Independent frontend:** a web/PWA application and desktop/mobile shells,
  connected to the backend through APIs.
- **Business-scoped operations:** business identity and operational data are
  handled in separate scopes; isolation must be validated in each environment
  before real-world use.
- **AI assistance:** agents can support discovery, information retrieval, and
  organization. Critical actions remain subject to deterministic rules and
  human review.
- **Delivery:** an own-delivery workflow and onboarding groundwork for future
  official connections are present. Each external capability depends on its
  adapter being implemented and approved.
- **Flexible deployment:** components can run together or separately.
  Development examples are not production-ready configurations.

### Maturity and contact

The project is under development and **is not production-ready**. Network
hardening, isolation and resilience testing, backup/recovery exercises,
monitoring, and approval of external integrations are required before
commercial operation.

A joint evaluation may begin with official API documentation, sandbox
environments, credential and webhook requirements, approval criteria, privacy,
and testing with non-production data. No connection should be enabled without
provider authorization and security and operational validation.

To discuss the project publicly, open a discussion or issue in this
repository. Do not post credentials, personal data, or confidential
information.

### Documents

- [Delivery partner integration brief](./PARTNER_INTEGRATION_BRIEF.md) —
  collaboration proposal and factual capability status.
- [Historical architecture reference](./ARQUITETURA_REFERENCIA_HISTORICA.md) —
  previous material; it is not the current specification.
- The [architecture PDF](./Modelo%20de%20Arquitetura%20do%20Sistema%20e%20Guia%20de%20Integra%C3%A7%C3%A3o.pdf)
  is conceptual, historical, and currently available in Portuguese only. Do
  not use it as current API documentation, proof of an approved integration, or
  a production deployment guide.
