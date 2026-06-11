# Agent Memory — Engineer

> Convenções e gotchas duráveis da stack. Lido no início de cada acionamento.
> Semeada com **princípios** genéricos destilados de 5 épicos reais. As convenções concretas da stack entram ao instanciar.

## Princípio de higiene de memória

Esta memória guarda **convenções e gotchas reutilizáveis** da stack — não um log de bugs por task. Um bug pontual vive no task file; só vira memória se virar uma **regra de prevenção** que se aplica a tasks futuras. Pode periodicamente. (PLAYBOOK §9)

## Stack e ambiente

_(Preencha ao instanciar: versões de runtime, gerenciador de deps, comandos de setup, layout de pastas.)_

## Comandos do projeto

_(Preencha: como rodar testes, lint, format, typecheck, build, dev, migrations, seed.)_

```bash
# testes:
# lint (completo — lint E format-check):
# typecheck:
# build:
# dev local:
# migrations:
```

## Convenções de código

- TDD sempre (Red → Green → Refactor). Não pular o Red.
- Sem `any`/`Any` injustificado; se necessário, comentar o porquê na linha.
- Validação só nos boundaries (input externo, API externa) — e sobre o valor **pós-normalização**, não o cru.
- Sem comentários óbvios — comentar só o *porquê* não-óbvio.
- Sem abstração prematura — 3 funções similares > 1 abstração forçada.
- Microcopy com **acentuação correta** — checar cada string visível contra a seção Microcopy do épico.
- Asserts de teste **nunca dentro de condicional** — teste oco passa vazio.

## Fundações que custaram caro descobrir tarde (faça desde o scaffold)

- **Cliente de API do frontend nasce completo:** timeout via `AbortController` (default ~20s) + auto-injeção do token de auth lendo o storage internamente. Retrofitar isso depois custou 3 tasks de correção — callers que passam token manualmente esquecem, e modal com loading-lock sem timeout prende o usuário para sempre.
- **Constantes/labels compartilhados em módulo único** (ex: `lib/categories.ts` com Record de labels + array de options). Definição duplicada em componentes diverge na primeira mudança. O mesmo vale no backend: tipo/Literal compartilhado entre router e service vive num módulo central.
- **Fix de a11y/estilo em componente compartilhado, não por uso:** se o botão X do Dialog precisa de ≥44px, corrija no wrapper `components/ui/dialog.tsx` uma vez — não em cada tela que abre modal.
- **`.eslintrc` (ou equivalente) já no scaffold** — lint que falha interativamente sem config quebra o fluxo de todas as tasks seguintes.

## Padrões reutilizáveis observados (genéricos)

- **Contrato de API externa isolado:** um único módulo recebe o JSON cru do provider e retorna tipos internos. O resto do sistema nunca vê o shape externo. Facilita troca de provider e plano B.
- **Mock de HTTP externo:** use a lib de mock do cliente HTTP do projeto; **nunca** chame a API real em teste.
- **Mock de LLM:** o client é sempre **injetado** como parâmetro; nunca importar o client real em teste. Mock com `ainvoke` async retornando um objeto com `.content` string.
- **Integração por estágios:** worker que vai precisar de um client externo ganha um ponto de injeção (`_get_client()`) que retorna `None` até a task de wiring — a etapa é pulada com log DEBUG e o ciclo completa. Nunca chamar API real sem o wiring explícito.
- **Lazy import de lib pesada/opcional** dentro da função que a usa — evita custo/erro de import em contextos que não a utilizam.
- **Dados externos opcionais (corpus, dicionários):** fallback embutido no código; nada é baixado automaticamente; nenhum teste depende de rede.
- **Workers/jobs:** cada invocação cria sua própria conexão/sessão de banco para evitar estado preso entre event loops. Fixtures de engine com escopo de módulo + `dispose()` no teardown.
- **Upsert atômico:** `INSERT ... ON CONFLICT DO UPDATE/NOTHING RETURNING` em vez de SELECT-then-INSERT.
- **Batch insert** em vez de N+1 inserts em loops de coleta.
- **Downloads externos:** sempre com timeout, dentro do try, e com guard de tamanho máximo (`len(bytes) > MAX → abortar`).
- **Rate limit de API externa:** ler o header da resposta **e interromper o loop** quando abaixo do limiar — checar sem `break` é só log.
- **localStorage no SSR/SSG:** nunca acessar fora de `useEffect` em Client Component — quebra no build estático. Use `useState(null)` + `useEffect` pra hidratar.
- **Estado de UI em ação async:** salve o estado anterior antes de iniciar (`const prev = state`) e restaure no catch — nunca fixe um estado de retorno hardcoded.
- **Derived state:** nunca `setState` no corpo do render — use `useEffect` com a dependência.
- **Lookup O(1) em render:** mapa via `Object.fromEntries` em vez de `find` repetido.
- **Filtragem client-side** é aceitável para listas pequenas (≤ ~200 itens); acima disso, filtro vira query param no backend.
- **Datas em teste:** fixe `TZ=UTC` no script de teste e use horários com margem — datas locais sem sufixo `Z` são interpretadas no fuso da máquina e quebram em CI.
- **Testes de UI com portais:** componentes que renderizam em portal (Dialog/Sheet/Popover) não aparecem no `container` — use `screen.*`.
- **Teardown de integração na ordem dos FKs:** delete filhos antes de pais — local passa, CI quebra.

## Convenções de teste

_(Preencha: framework, organização unit/integração, fixtures base, cobertura alvo.)_

## Env vars do projeto

_(Mantenha uma tabela var → tipo → default → uso, espelhando o `.env.example`. Mudança de env var entre épicos — adicionar/remover/renomear — é breaking change: liste explicitamente na task de deploy.)_
