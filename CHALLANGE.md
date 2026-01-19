# Desafio técnico: Centralização de cobrança

## Contexto
Estamos em processo de centralizar nossa operação de cobrança, migrando diferentes produtos e fluxos para um único gateway: Pagar.me. Hoje, também utilizamos o ASAAS, o que gera complexidade para os clientes e para o time financeiro.

## Seu desafio
Como Engenheiro de Software, você foi convidado a propor uma solução técnica que nos permita:

     Unificar os fluxos de cobrança em uma plataforma própria
     Usar exclusivamente a Pagar.me como gateway
     Permitir que produtos distintos (PMS, Motor de Reservas, Channel) compartilhem a mesma base de pagamento
     Manter compatibilidade com modelos de cobrança distintos (recorrente e avulso)


## O que esperamos da sua entrega
Você pode escolher como deseja entregar: código, arquitetura comentada, diagrama ou documentação técnica.

*No mínimo:*

     Desenhe e explique como estruturaria a API (REST ou GraphQL)
     Modele os principais domínios (pagamento, cliente, produto, status, tipo de cobrança etc.)
     Simule a lógica de integração com o Pagar.me (não precisa integrar de fato)
     Explique como migraria clientes ASAAS → Pagar.me sem perda de controle
     Sugira caminhos para escalar a solução e facilitar integração futura com o PMS real


### Bônus (opcional, mas bem-vindo)

     Como lidaria com múltiplos produtos numa mesma fatura
     Como versionaria e estruturaria rotas da API pensando em futuro acoplamento com outros gateways
     Como deixaria isso pronto para ser um produto independente


## O que vamos avaliar

     Clareza e organização do raciocínio
    Capacidade de modelagem de domínios.
     Capacidade de tomar boas decisões técnicas
     Visão de produto e pragmatismo
     Código limpo (se decidir codar) ou comunicação bem estruturada


## Entrega

     Repositório público ou arquivo compartilhado (README explicando as decisões)
     Linguagem/stack livre, mas temos apreço por Rails, Node.js e boas práticas
     Você decide o quanto quer codar. Queremos ver sua capacidade de estruturar, decidir e explicar


## Por que isso importa?
Padronizar e escalar a operação de pagamentos é um passo estratégico para qualquer empresa SaaS.
Essa é uma oportunidade de criar algo com impacto direto: mais eficiência, melhor experiência e base sólida para crescer com tranquilidade.

    Boa sorte! Estamos torcendo para ver o seu talento em ação. 💜


