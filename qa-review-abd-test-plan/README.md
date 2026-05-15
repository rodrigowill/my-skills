# qa-review-and-test-plan

Skill especialista em **revisão de User Story (Descrição, Regras, Critérios de Aceite)** e **plano de testes funcionais em Gherkin** para agentes de código. Conduz perguntas e sugestões com aprovação explícita, atualiza a história e entrega cenários agrupados (happy path, alternativos, borda/erro), com tags de automação e pirâmide de testes (Unit, Integration, E2E), usando *pairwise* quando aplicável, além de riscos e casos implícitos.

## Quando usar

Invoque esta skill sempre que precisar:
- Revisar uma User Story do ponto de vista de QA antes ou durante o refinamento.
- Gerar ou validar um plano de testes funcionais alinhado à história.
- Produzir cenários Gherkin (Given / When / Then) a partir de uma história estruturada.
- Classificar cenários para automação e nível da pirâmide de testes.

## Gatilhos típicos

- "Revisão de QA da user story..."
- "Plano de testes para esta história..."
- "Gera cenários Gherkin a partir da US..."
- "Critérios de aceite e cenários de teste..."
- Uso explícito: *"use a skill qa-review-and-test-plan"*

## Como funciona

A skill segue um **fluxo consultivo obrigatório**:

1. **Análise** — lê a User Story completa e verifica se há informação suficiente para testes relevantes.
2. **Perguntas** — esclarece ambiguidades antes de propor cenários.
3. **Sugestões** — melhorias em Descrição, Regras e Critérios de Aceite; aguarda aceite ou rejeição **explícitos** de cada sugestão.
4. **Confirmações** — só avança após respostas a todas as perguntas e decisão sobre todas as sugestões; confirma explicitamente que recebeu tudo antes de criar testes.
5. **Entrega** — história atualizada (mesmo formato do texto recebido), cenários Gherkin em três grupos (Happy Path, Fluxos Alternativos, Casos de Borda / Erro), tags de automação e pirâmide no título do cenário (ex.: `Scenario 8: Filtro aplicado com sucesso (unit)`), lista de riscos adicionais e casos não explícitos na US.
6. **Arquivo** — gera um `.md` com título, US atualizada, cenários com tags, riscos adicionais e casos não explícitos.

O agente responde em **português simples e direto** no chat e **não** inclui emojis nem testes não funcionais (performance, segurança, acessibilidade, usabilidade, etc.), conforme o escopo da skill.

## Estrutura do documento gerado (.md)

```markdown
# [Título]

## User Story atualizada
## Cenários Gherkin (com tags Unit / Integration / E2E)
## Riscos adicionais
## Casos não explícitos na User Story
```

## Idioma e tom de voz

- Respostas no chat em **português simples e direto**.
- `SKILL.md` pode estar em inglês; o comportamento conversacional segue o pedido de português.
- Sem emojis ou caracteres estilizados nas entregas da skill.

## Estrutura de arquivos

```
qa-review-and-test-plan/
├── SKILL.md    # Instruções completas da skill
└── README.md   # Este arquivo
```

## Boas práticas

- Forneça o **texto completo** da User Story (colar no chat ou anexar arquivo).
- Responda a **todas** as perguntas e decida explicitamente sobre **cada** sugestão antes de pedir os cenários.
- Se a história for insuficiente, a skill deve interromper e pedir revisão em vez de inventar requisitos.
- Cenários são **somente funcionais**; use *pairwise* para reduzir redundância sem perder cobertura dos critérios.

## Limitações

- **Não** cobre testes não funcionais (performance, segurança, acessibilidade, usabilidade).
- **Não** substitui *exploratory testing*, automação implementada ou revisão de código.
- Depende de integração/autenticação apenas se você trouxer o corpo da história por MCP/API; colar o texto continua sendo fluxo válido e frequentemente o mais simples.

## Licença e autoria

- Autor: Rodrigo Soares
- Data de criação: 2026-05-01
- Versão: 1.0.0
