# Agent Memory — Engineer

> Convenções e gotchas duráveis da stack. Lido no início de cada acionamento.
> Semeada com **princípios** genéricos. As convenções concretas da stack entram ao instanciar.

## Princípio de higiene de memória

Esta memória guarda **convenções e gotchas reutilizáveis** da stack — não um log de bugs por task. Um bug pontual vive no task file; só vira memória se virar uma **regra de prevenção** que se aplica a tasks futuras. Pode periodicamente. (PLAYBOOK §9)

## Stack e ambiente

_(Preencha ao instanciar: versões de runtime, gerenciador de deps, comandos de setup, layout de pastas.)_

## Comandos do projeto

_(Preencha: como rodar testes, lint, format, typecheck, build, dev, migrations, seed.)_

```bash
# testes:
# lint:
# typecheck:
# build:
# dev local:
# migrations:
```

## Convenções de código

- TDD sempre (Red → Green → Refactor). Não pular o Red.
- Sem `any`/`Any` injustificado; se necessário, comentar o porquê na linha.
- Validação só nos boundaries (input externo, API externa). Confiar em código interno.
- Sem comentários óbvios — comentar só o *porquê* não-óbvio.
- Sem abstração prematura — 3 funções similares > 1 abstração forçada.
- Microcopy com **acentuação correta** — checar cada string visível contra a seção Microcopy do épico.

## Padrões reutilizáveis observados (genéricos)

- **Contrato de API externa isolado:** um único módulo recebe o JSON cru do provider e retorna tipos internos. O resto do sistema nunca vê o shape externo. Facilita troca de provider e plano B.
- **Mock de HTTP externo:** use a lib de mock do cliente HTTP do projeto; **nunca** chame a API real em teste.
- **Mock de LLM:** o client é sempre **injetado** como parâmetro; nunca importar o client real em teste. Mock com `ainvoke` async retornando um objeto com `.content` string.
- **Workers/jobs:** cada invocação cria sua própria conexão/sessão de banco para evitar estado preso entre event loops. Sempre `dispose()` do engine no teardown das fixtures.
- **Upsert atômico:** `INSERT ... ON CONFLICT DO UPDATE/NOTHING RETURNING` em vez de SELECT-then-INSERT.
- **Batch insert** em vez de N+1 inserts em loops de coleta.
- **localStorage no SSR/SSG:** nunca acessar fora de `useEffect` em Client Component — quebra no build estático. Use `useState(null)` + `useEffect` pra hidratar.
- **Datas em teste:** fixe `TZ=UTC` no script de teste e use horários com margem — datas locais sem sufixo `Z` são interpretadas no fuso da máquina e quebram em CI.

## Convenções de teste

_(Preencha: framework, organização unit/integração, fixtures base, cobertura alvo.)_

## Env vars do projeto

_(Mantenha uma tabela var → tipo → default → uso, espelhando o `.env.example`.)_
