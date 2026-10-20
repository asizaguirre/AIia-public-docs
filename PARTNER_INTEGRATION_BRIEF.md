# AlEx | Brief de integração para parceiros de delivery

## Português

### Convite à colaboração

O AlEx busca conversas técnicas com plataformas de delivery e provedores do
ecossistema — incluindo iFood, Uber, 99 e outros — para definir um caminho
oficial de interoperabilidade que beneficie negócios e preserve as regras de
cada plataforma.

Este documento é uma proposta de colaboração, não um anúncio de parceria. As
marcas são citadas apenas para indicar categorias de interesse. Não há
endosso, autorização, contrato ou integração homologada implícitos.

### O que estamos construindo

O produto reúne uma interface independente para negócios, gestão de pedidos e
agentes assistivos. A aplicação separa a experiência do usuário das APIs e dos
serviços de dados e está sendo preparada para operar com escopos isolados por
negócio.

O objetivo é permitir que um negócio acompanhe suas operações em um único
ambiente, mantendo a plataforma parceira como autoridade sobre seus próprios
recursos, políticas, credenciais e estados de pedido.

### Capacidades e estado atual

| Capacidade | Estado atual |
|---|---|
| Interface web responsiva, preparação PWA e shells desktop/mobile | Implementada no frontend; requer validação nos dispositivos e ambientes finais. |
| Cadastro de negócio e fluxo de aprovação administrativa | Implementados no frontend e conectados às APIs disponíveis no projeto. |
| Gestão interna de pedidos e entrega própria | Fluxos internos presentes; não representam pedidos originados em marketplaces. |
| Onboarding e configuração para delivery | Estrutura de cadastro/configuração presente; não executa integração oficial. |
| Proteção de configuração sensível | Há mecanismos de configuração cifrada; operação, rotação e recuperação ainda precisam de validação de produção. |
| Registro de agentes e vínculo a escopo de negócio | Base de gestão presente; não significa provisionamento ou execução automática de workflows n8n externos. |
| Adaptadores oficiais para provedores de delivery | **Não implementados nem homologados.** |

O projeto ainda não está declarado pronto para produção. Consulte o código e
as notas técnicas atuais antes de depender de qualquer capacidade em um
ambiente comercial.

### Escopo proposto para descoberta técnica

O escopo deve ser definido com cada parceiro, usando somente APIs e permissões
oficialmente autorizadas. Tópicos possíveis:

- autenticação, consentimento do lojista e emissão/rotação de credenciais;
- disponibilidade de sandbox e processo de homologação;
- consulta e atualização de catálogo, preços e disponibilidade, se permitidas;
- recebimento de pedidos e consulta/atualização de estados;
- cancelamento, suporte, falhas de entrega e limites de responsabilidade;
- webhooks, idempotência, ordenação, repetição e reconciliação de eventos;
- limites de taxa, observabilidade, privacidade e retenção de dados;
- critérios de segurança, suporte e operação para lançamento gradual.

Esses itens são perguntas para descoberta, não funcionalidades nem contratos
de API já suportados pelo AlEx.

### Princípios de integração

1. Usar somente APIs, eventos e credenciais oficialmente autorizados.
2. Respeitar o consentimento do negócio, os escopos mínimos e o isolamento
   entre negócios.
3. Tratar pedidos e valores com regras determinísticas; IA não autoriza
   transações, altera preços nem substitui validações do parceiro.
4. Projetar para idempotência, auditoria, tratamento explícito de falhas e
   reconciliação.
5. Começar em sandbox com dados sintéticos e avançar por etapas de homologação
   e liberação controlada.
6. Não armazenar segredos em documentação, repositórios públicos, logs ou
   ferramentas de atendimento.

### Próximos passos sugeridos

1. Identificar o canal oficial de parceria e a documentação para
   desenvolvedores.
2. Confirmar permissões, requisitos de marca, sandbox, certificações e critérios
   de homologação.
3. Selecionar um caso de uso limitado e seus fluxos permitidos.
4. Elaborar um contrato técnico com estados, eventos, segurança e suporte.
5. Implementar e testar o adaptador após aprovação do parceiro.
6. Fazer revisão operacional e de segurança antes de qualquer piloto real.

Para iniciar uma conversa, use os canais oficiais do parceiro. Para questões
públicas sobre o projeto, abra uma issue neste repositório. Não envie tokens,
dados de clientes ou documentos confidenciais em issues públicas.

---

## English

### Invitation to collaborate

AlEx is seeking technical discussions with delivery platforms and ecosystem
providers—including iFood, Uber, 99, and others—to define an official
interoperability path that benefits businesses while respecting each
platform's rules.

This document is a collaboration proposal, not a partnership announcement.
Brands are mentioned only to identify areas of interest. No endorsement,
authorization, contract, or approved integration is implied.

### What we are building

The product brings together an independent business interface, order
management, and assistive agents. The application separates the user
experience from APIs and data services and is being prepared to operate with
business-scoped data.

The goal is to let a business follow its operations in one environment while
keeping the partner platform authoritative over its own resources, policies,
credentials, and order states.

### Capabilities and current status

| Capability | Current status |
|---|---|
| Responsive web interface, PWA groundwork, and desktop/mobile shells | Implemented in the frontend; validation on target devices and environments is still required. |
| Business registration and administrator approval flow | Implemented in the frontend and connected to APIs available in the project. |
| Internal order management and own-delivery workflow | Internal workflows are present; they do not represent marketplace-originated orders. |
| Delivery onboarding and configuration | Registration/configuration groundwork is present; it does not perform an official integration. |
| Protection of sensitive configuration | Encryption mechanisms exist; production operation, rotation, and recovery still need validation. |
| Agent records linked to a business scope | Management groundwork exists; this does not provision or automatically run external n8n workflows. |
| Official delivery-provider adapters | **Not implemented or approved.** |

The project is not declared production-ready. Review the current code and
technical notes before relying on any capability in a commercial environment.

### Proposed technical discovery scope

The scope should be defined with each partner, using only officially
authorized APIs and permissions. Potential topics include:

- authentication, merchant consent, and credential issuance/rotation;
- sandbox availability and the approval process;
- catalog, price, and availability reads or updates, where permitted;
- order intake and order-state queries or updates;
- cancellations, support, delivery failures, and responsibility boundaries;
- webhooks, idempotency, ordering, retries, and event reconciliation;
- rate limits, observability, privacy, and data retention;
- security, support, and operational criteria for a gradual launch.

These are discovery questions, not features or API contracts currently
supported by AlEx.

### Integration principles

1. Use only officially authorized APIs, events, and credentials.
2. Respect business consent, least-privilege scopes, and separation between
   businesses.
3. Handle orders and monetary values through deterministic rules; AI does not
   authorize transactions, change prices, or replace partner validation.
4. Design for idempotency, auditing, explicit failure handling, and
   reconciliation.
5. Start in a sandbox with synthetic data and progress through approval and
   controlled release stages.
6. Never store secrets in documentation, public repositories, logs, or support
   tools.

### Suggested next steps

1. Identify the partner's official partnership channel and developer
   documentation.
2. Confirm permissions, brand requirements, sandbox access, certifications,
   and approval criteria.
3. Select a limited use case and its permitted workflows.
4. Define a technical contract covering states, events, security, and support.
5. Implement and test the adapter after partner approval.
6. Complete operational and security reviews before any live pilot.

To start a conversation, use the partner's official channels. For public
project questions, open an issue in this repository. Do not post tokens,
customer data, or confidential documents in public issues.
