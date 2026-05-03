---
name: playwright-test-plan-to-code
description: >-
  Converte planos de teste redigidos em Gherkin (cenários, tags, outline com exemplos,
  qualquer domínio) em suites Playwright (@playwright/test): specs, fixtures, auth,
  projetos e2e/integration/API conforme o plano ou convenção do repositório. Inclui
  práticas de QA (rastreabilidade, dados, flakiness, independência) e uso pontual do
  Playwright MCP para exploração. Use ao automatizar cenários BDD em Playwright, gerar
  specs a partir de plano Gherkin, ou ao mencionar plano de teste + Playwright.
disable-model-invocation: true
---

# Playwright: plano Gherkin (genérico) → automação

## Objetivo

Ler **qualquer plano** que descreva cenários em estilo **Gherkin** (estrutura Dado–Quando–Então ou Given–When–Then, com ou sem `Funcionalidade`/`Feature`, com ou sem tabelas `Exemplos`/`Examples`) e produzir **testes executáveis** com [@playwright/test](https://playwright.dev/docs/intro), alinhados ao repositório alvo. Complementa skills que **geram** o plano (ex.: `test-scenarios-generation`, `qa-review-and-test-plan`); esta skill foca em **implementação e qualidade da suíte**.

Referências oficiais: [Playwright Test — intro](https://playwright.dev/docs/intro), [Playwright MCP](https://playwright.dev/mcp/introduction).

---

## 1. O que conta como “plano Gherkin” (entrada genérica)

O agente deve aceitar variações comuns, sem exigir sintaxe Cucumber perfeita:

| Elemento | Variações aceitas |
|----------|-------------------|
| Cenário | `Cenário:`, `Scenario:`, `Cenário N:`, título na mesma linha ou na seguinte |
| Passos | `Dado`/`Given`, `Quando`/`When`, `Então`/`Then`, `E`/`And`/`Mas`/`But` |
| Tags | Qualquer `@tag` no cenário ou bloco (ex.: `@smoke`, `@regressão`); usar como metadado Playwright |
| Outline | `Esquema do Cenário` / `Scenario Outline` + tabela `Exemplos` / `Examples` com cabeçalho e linhas |
| Tipo de teste | Texto no título como `(e2e)`, `(integration)`, `(api)` **ou** inferir por tag/pasta/convenção do projeto; se ambíguo, **perguntar uma vez** ou registrar suposição no topo do spec |

**Não assumir** domínio (papéis de negócio, entidades específicas). Extrair do texto apenas: atores, estados, ações, resultados esperados e dados tabulares — mapear para a aplicação sob teste.

---

## 2. Princípios de QA na automação

1. **Rastreabilidade**: título do `test` ou `test.describe` deve conter **identificador do cenário** do plano (número ou nome estável) para relatório e defeitos.
2. **Independência**: cada teste deve poder rodar isolado; usar `test.beforeEach` / fixtures para pré-condições, evitar ordem implícita entre arquivos salvo plano explícito e documentado.
3. **Dados**: preferir factories, API de setup ou seed idempotente; **nunca** credenciais ou PII reais no código — `process.env` / secrets de CI.
4. **Asserções observáveis**: cada `Então`/`E` deve corresponder a estado verificável (UI, rede, storage); evitar “deve funcionar” sem critério mensurável.
5. **Ambiguidade do plano**: se o plano não define como detectar sucesso/falha (ex.: “bloqueado” sem UI vs API), **perguntar** ou implementar a opção mais segura e **documentar** em comentário no spec.
6. **Flakiness**: auto-wait do Playwright; evitar `waitForTimeout`; estabilizar com [locators](https://playwright.dev/docs/locators) priorizando `getByRole`, `getByLabel`, `getByTestId`; repetir falhas com trace em CI.
7. **Pirâmide / escopo**: respeitar o que o plano chama de e2e (fluxo browser) vs integration (UI+backend, menos camadas, etc.); se o plano não distingue, seguir **padrão do repositório** ou um único projeto com comentário de escopo.

---

## 3. Mapeamento Gherkin → Playwright

| Gherkin | Playwright (padrão) |
|---------|---------------------|
| Tag `@x` | `test.describe(..., { tag: ['@x'] })` e/ou filtro CI `grep` / projetos |
| `Dado` / `Given` | `test.beforeEach`, fixture, `storageState`, login programático ou `test.step` inicial |
| `Quando` / `When` | `test.step` + ações (`page.goto`, interações, `request` para API) |
| `Então` / `Then`, `E` / `And` | `test.step` + `expect` ([assertions](https://playwright.dev/docs/test-assertions)) |
| Outline + tabela | `test.describe.each(linhas)` ou describe por linha; **não** usar `test.use()` dentro do callback de `test()` |
| Múltiplos `Quando` no mesmo cenário | Ordem do plano; sub-steps com `test.step` nomeados como no plano |

**Autenticação / contexto** (genérico): o plano pode falar em “usuário logado”, “perfil X”, “token válido”. Implementar com o que o projeto já usa: [armazenamento de estado](https://playwright.dev/docs/auth), login no `beforeEach`, ou cabeçalhos em `APIRequestContext` — sem impor um único modelo.

---

## 4. Fluxo de trabalho do agente

### 4.1 Normalizar o plano

- Extrair lista de cenários (id + título + tags + passos + tabelas).
- Detectar idioma dos passos para espelhar em `test.step` (melhora leitura para QA).
- Identificar **dados variáveis** (colunas da tabela, placeholders `<nome>`) → parâmetros de teste.

### 4.2 Alinhar ao repositório

- Se já existir `playwright.config.ts`, **estender** o padrão (pastas, `projects`, `baseURL`, reporters).
- Se não existir, seguir [intro Playwright](https://playwright.dev/docs/intro) (`npm init playwright@latest` ou equivalente) e então adicionar specs.

### 4.3 Estrutura de arquivos (sugestão neutra)

```
playwright.config.ts
e2e/                    # ou tests/, nome que o time usar
  fixtures/             # setup compartilhado, não obrigatório
  pages/                # Page Object / camadas de abstração, opcional
  specs/                # *.spec.ts — um arquivo por feature ou por plano
```

Dividir arquivos por **área funcional** ou **documento de plano**, não por tecnologia interna. Evitar um único arquivo gigante sem necessidade.

### 4.4 Projetos e tipos de teste

- **e2e**: `testDir` ou tag/projeto dedicado; testes que exigem browser e fluxo completo.
- **integration** (quando o plano pedir): pode ser mesma pasta com tag, ou `testMatch` separado, ou `request` + browser parcial — **definir no comentário** do spec o que “integration” significa naquele repo.

### 4.5 Implementação e verificação

- Implementar cenário a cenário; rodar `npx playwright test` (ou yarn/pnpm do projeto).
- Falhas: [trace / HTML report](https://playwright.dev/docs/intro); corrigir locators ou dados, não “endurecer” com sleeps.

---

## 5. Playwright MCP (uso pontual)

| Uso | Quando |
|-----|--------|
| Explorar UI desconhecida, árvore de acessibilidade, refs | [Playwright MCP](https://playwright.dev/mcp/introduction) |
| Gerar locators para colar no spec | Ferramentas do MCP ou codegen do projeto |
| Suíte versionada em CI | Código em `.spec.ts`; MCP **não** substitui arquivos commitados |

MCP costuma ser **headed**; CI **headless** — validar sempre com runner local/CI.

---

## 6. Templates mínimos (TypeScript)

**Cenário único** (tags e passos genéricos):

```typescript
import { test, expect } from '@playwright/test';

test.describe('ID-001 Título do cenário @smoke', { tag: ['@smoke'] }, () => {
  test('fluxo conforme plano', async ({ page }) => {
    await test.step('Dado: pré-condições do plano', async () => {
      // ex.: await page.goto('/'); login se necessário
    });
    await test.step('Quando: ação principal', async () => {
      // interações
    });
    await test.step('Então: resultado esperado', async () => {
      await expect(page.getByRole('heading')).toBeVisible();
    });
  });
});
```

**Outline / tabela** (`test.use` apenas no nível do `describe`):

```typescript
const linhas = [
  { colA: 'valor1', colB: 'esperado1' },
  { colA: 'valor2', colB: 'esperado2' },
] as const;

test.describe.each(linhas)('ID-010 Outline — colA=$colA', (row) => {
  // test.use({ storageState: row.authFile }) // se aplicável, por linha no describe.each

  test('executa passos do outline', async ({ page }) => {
    await test.step('Quando aplico dados da linha', async () => {
      // usar row.colA, row.colB
    });
    await test.step('Então verifico colB', async () => {
      await expect(page.getByText(row.colB)).toBeVisible();
    });
  });
});
```

Se cada linha exigir **auth ou storage diferentes**, preferir **um `test.describe` por grupo** com `test.use` no topo, ou **projetos** no config — nunca `test.use` dentro do corpo de `test()`.

---

## 7. Checklist de qualidade (Definition of Done da suíte)

- [ ] Todo cenário do plano com mapeamento 1:1 ou justificativa explícita (skip com issue).
- [ ] Tags do plano refletidas em `tag` ou nome, permitindo `grep`/projetos.
- [ ] Passos `Então`/`E` com asserts explícitos; negativos com critério observável.
- [ ] Sem segredos no repositório; dados sensíveis via env.
- [ ] Sem `waitForTimeout` salvo exceção documentada.
- [ ] Testes independentes ou dependência documentada + ordem controlada (`serial` se inevitável).
- [ ] Nomes/locators alinhados à acessibilidade e estabilidade do app sob teste.

---

## 8. Idioma e convenções

- Texto de `test.step` pode seguir o **idioma do plano** para auditoria com QA/STLC.
- Identificadores de código (variáveis, arquivos): convenção do repositório.

---

## Resumo operacional

1. **Parse** genérico: cenários, tags, passos, tabelas, placeholders — sem supor domínio de negócio.  
2. **QA**: rastreabilidade, independência, dados seguros, asserts claros, tratar ambiguidade.  
3. **Playwright**: describe/step/expect, auth conforme o app, outline com `describe.each` ou projetos.  
4. **MCP** só como apoio à descoberta; **fonte da verdade** = specs + config no git.  
5. **Docs**: [Playwright intro](https://playwright.dev/docs/intro), [Playwright MCP](https://playwright.dev/mcp/introduction).
