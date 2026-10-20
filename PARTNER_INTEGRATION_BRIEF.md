# AlEx | Brief de integração para parceiros de delivery

## Convite à colaboração

O AlEx está buscando conversas técnicas com plataformas de delivery e
provedores do ecossistema — incluindo iFood, Uber, 99 e outros — para definir
um caminho oficial de interoperabilidade que beneficie negócios e preserve as
regras de cada plataforma.

Esta é uma proposta de colaboração, não um anúncio de parceria. As marcas são
citadas apenas para indicar categorias de interesse. Não existe endosso,
autorização, contrato ou integração homologada implícitos.

## O que estamos construindo

O produto reúne uma interface independente para negócios, gestão de pedidos e
agentes assistivos. A aplicação separa a experiência do usuário das APIs e dos
serviços de dados, e está sendo preparada para operar com escopos isolados por
negócio.

O objetivo de produto é permitir que um negócio acompanhe suas operações em um
único ambiente, mantendo a plataforma parceira como autoridade sobre seus
próprios recursos, políticas, credenciais e estados de pedido.

## Capacidades verificadas no projeto

| Capacidade | Estado atual |
|---|---|
| Interface web responsiva com preparação PWA e shells desktop/mobile | Implementada no frontend; requer validação nos dispositivos e ambientes finais. |
| Cadastro de negócio e fluxo de aprovação administrativa | Implementados no frontend e conectados às APIs disponíveis no projeto. |
| Gestão interna de pedidos e entrega própria | Fluxos internos presentes; não representam pedidos originados em marketplaces. |
| Tela de onboarding e configuração para delivery | Estrutura de cadastro/configuração presente; não executa integração oficial. |
| Armazenamento protegido de configuração sensível | Há mecanismos de configuração cifrada; operação, rotação e recuperação ainda precisam de validação de produção. |
| Registro de agentes e vínculo a escopo de negócio | Base de gestão presente; não significa provisionamento ou execução automática de workflows n8n externos. |
| Adaptadores oficiais para provedores de delivery | **Não implementados nem homologados.** |

Consulte o código e as notas técnicas atuais antes de depender de qualquer
capacidade em um ambiente comercial. O projeto ainda não está declarado pronto
para produção.

## Escopo proposto para descoberta técnica

O escopo concreto deve ser definido com cada parceiro, com APIs e permissões
oficialmente autorizadas. Tópicos possíveis para a conversa:

- autenticação, consentimento do lojista e emissão/rotação de credenciais;
- disponibilidade de sandbox e processo de homologação;
- consulta e atualização de catálogo, preços e disponibilidade, se permitidas;
- recebimento de pedidos e consulta/atualização de estados;
- cancelamento, suporte, falhas de entrega e limites de responsabilidade;
- webhooks, idempotência, ordenação, repetição e reconciliação de eventos;
- limites de taxa, observabilidade, privacidade e retenção de dados;
- critérios de segurança, suporte e operação para lançamento gradual.

Os itens acima são perguntas de descoberta, não funcionalidades ou contratos
de API já suportados pelo AlEx.

## Princípios de integração

1. Usar somente APIs, eventos e credenciais oficialmente autorizados.
2. Respeitar consentimento do negócio, escopos mínimos e isolamento entre
   negócios.
3. Tratar pedidos e valores com regras determinísticas; IA não autoriza
   transações, altera preços nem substitui validações do parceiro.
4. Projetar integrações para idempotência, auditoria, tratamento explícito de
   falhas e reconciliação.
5. Começar em sandbox com dados sintéticos e avançar por etapas de homologação
   e liberação controlada.
6. Não armazenar segredos em documentação, repositórios públicos, logs ou
   ferramentas de atendimento.

## Próximos passos sugeridos

1. Identificar o canal oficial de parceria e documentação para desenvolvedores.
2. Confirmar permissões, requisitos de marca, sandbox, certificações e critérios
   de homologação.
3. Selecionar um caso de uso limitado e seus fluxos permitidos.
4. Elaborar contrato técnico com estados, eventos, segurança e suporte.
5. Implementar e testar o adaptador após aprovação do parceiro.
6. Realizar revisão operacional e de segurança antes de qualquer piloto real.

Para iniciar uma conversa, use os canais oficiais do parceiro e, para questões
públicas sobre o projeto, abra uma issue neste repositório. Não envie tokens,
dados de clientes ou documentos confidenciais em issues públicas.
