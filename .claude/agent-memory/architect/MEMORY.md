# Agent Memory — Architect

> Padrões, decisões e gotchas acumulados. Lido no início de cada acionamento.
> Semeada com **convenções e método** genéricos. A stack canônica é definida ao instanciar o projeto.

## Stack canônica do projeto

_(Preencha ao instanciar. Tabela camada → tecnologia. Justifique cada escolha no ADR-001.)_

| Camada | Tecnologia |
|---|---|
| Backend | _(a definir)_ |
| Frontend | _(a definir)_ |
| Banco de dados | _(a definir)_ |
| Hosting | _(a definir)_ |
| ... | ... |

## Convenções de API (recomendadas — ajuste ao projeto)

- Prefixo versionado: `/api/v1/`.
- Auth: Bearer token em rotas pós-login. 401 = sem token; 403 = token válido sem permissão.
- **Formato de erro canônico** (defina cedo, use em todos os endpoints): `{ "error": "<code>", "message": "<str>", "fields"?: [{"field","message"}] }`. Inconsistência aqui vira bug recorrente no front — alinhe back e front no mesmo shape.
- Paginação: `{ items, total, page, page_size }` com teto de page_size.
- Roles explícitos com fronteira clara de permissão.

## Convenções de tipagem

- Sem `any`/`Any` injustificado. `unknown` + type guard quando o tipo não é conhecido.
- Tipos discriminados por um campo `status` em vez de múltiplos booleans.
- Campos computados (ex: `valor_final = editado ?? original`) não precisam ser persistidos.

## Método de ADR

Toda decisão arquitetural relevante vira ADR (`docs/arch/adr/_ADR_TEMPLATE.md`): Status / Contexto / Decisão / Alternativas consideradas (com o porquê de cada rejeição) / Consequências. **Documente o que foi descartado e por quê** — salva o time de redebater. Liste os descartados nesta MEMORY conforme acumular.

## Padrões de segurança (universais)

- Segredos/keys **nunca** em response body nem em log. GET retorna só `{ configurado: bool, updated_at }`.
- Validação de input nos boundaries (lib de schema: Zod/Pydantic/etc.).
- CORS: só o domínio do front em produção — sem wildcard.
- Segredos em env var, nunca commitados. Documente quais env vars são obrigatórias.

## Padrões de teste (recomendados)

- Unit: cobertura mínima alvo em `services/` (ex: 70%).
- Integração: contra **banco real local** (Docker), nunca contra produção.
- E2E: cobrir os fluxos críticos da UX.
- **Nunca mockar o banco.** Mocks só para APIs externas (HTTP, LLM, provedores).
- Isole o contrato de cada API externa num único módulo/função que recebe o JSON cru e retorna tipos internos — o resto do sistema nunca vê o shape do provider.

## Gotchas e decisões descartadas

_(Acumule aqui: "avaliamos X e descartamos porque Y". Ex: filas/workers pesados sem ganho pro volume; BaaS auth vs. auth própria; embeddings vs. heurística simples. Sempre com o trade-off registrado.)_

## Estimativa de custo

_(Mantenha uma tabela de custo mensal estimado por item quando o projeto tiver restrição de budget. O usuário costuma ter budget rígido.)_
