# AlEx | Plataforma independente para operações comerciais

O AlEx está desenvolvendo uma experiência independente para negócios que
precisam organizar presença digital, agentes assistivos e operações de pedidos
em um único fluxo. A arquitetura mantém o frontend desacoplado dos serviços de
dados e de IA, com atenção a isolamento por negócio e execução local ou
distribuída.

## Para plataformas e parceiros de delivery

Buscamos diálogo técnico com plataformas, operadores e provedores de delivery
para avaliar integrações oficiais, seguras e homologadas. O objetivo é reduzir
fricção operacional para negócios, sem substituir os canais, regras comerciais
ou controles de cada parceiro.

**Estado atual, sem ambiguidades:** o sistema inclui fluxo interno de pedidos,
interface de cadastro/configuração de plataformas e preparação de credenciais.
Os adaptadores oficiais para iFood, Uber, 99 e outros provedores ainda não
estão implementados ou homologados. A interface de configuração não significa
que exista sincronização de catálogo, pedidos, status, cancelamentos ou
entregadores com esses serviços. Também não há parceria, endosso ou autorização
implícita de qualquer marca mencionada.

Veja o [brief de integração para parceiros](./PARTNER_INTEGRATION_BRIEF.md)
para a proposta, limites atuais e próximos passos de validação.

## Visão do produto

- **Frontend independente:** aplicação web/PWA e shells para desktop/mobile,
  conectados ao backend por API.
- **Operação por negócio:** identidade do negócio e dados operacionais são
  tratados dentro de escopos próprios; o isolamento deve ser validado em cada
  ambiente antes de uso real.
- **Apoio por IA:** agentes podem apoiar descoberta, consulta e organização de
  informações. Ações críticas continuam sujeitas a regras determinísticas e
  revisão humana.
- **Entrega:** existe um fluxo de entrega própria e uma base de onboarding para
  futuras conexões oficiais. Disponibilidade de cada função depende da
  implementação e homologação do adaptador correspondente.
- **Implantação flexível:** componentes podem ser executados em conjunto ou
  separados. Os exemplos de desenvolvimento não são configurações prontas para
  produção.

## Estado de maturidade

O projeto está em desenvolvimento. A documentação técnica do repositório de
código registra que a solução **ainda não está pronta para produção**: são
necessários endurecimento de rede, testes de isolamento e resiliência,
procedimentos ensaiados de backup/recuperação, monitoramento e homologação das
integrações externas antes de uma operação comercial.

## Como conversar sobre uma integração

Uma avaliação conjunta pode começar por documentação oficial de APIs, sandbox,
regras de credenciais e webhooks, requisitos de homologação, políticas de
privacidade e um fluxo de teste com dados não produtivos. Nenhuma conexão deve
ser ativada sem autorização do provedor e validação de segurança e operação.

Para iniciar uma conversa pública, abra uma discussão ou issue neste
repositório. Não publique credenciais, dados pessoais ou informações
confidenciais em canais públicos.

## Documentos

- [Brief de integração para parceiros](./PARTNER_INTEGRATION_BRIEF.md) —
  proposta de colaboração e matriz factual de capacidades.
- [Referência histórica de arquitetura](./ARQUITETURA_REFERENCIA_HISTORICA.md) —
  material anterior, não é especificação vigente.
- O PDF de arquitetura existente é material conceitual/histórico. Não o use
  como documentação de APIs atuais, integração homologada ou implantação de
  produção.
