# Agent Memory — Architect

> Padrões, decisões e gotchas acumulados. Lido no início de cada acionamento.
> Semeada com **convenções e método** genéricos, validados em 5 épicos reais. A stack canônica é definida ao instanciar o projeto.

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
- **Endpoint de listagem retorna o que a UI precisa exibir:** ao especificar uma tela de lista, confira que o contrato do endpoint inclui cada dado que o CA manda mostrar — senão nasce um CA inimplementável no front.

## Convenções de tipagem

- Sem `any`/`Any` injustificado. `unknown` + type guard quando o tipo não é conhecido.
- Tipos discriminados por um campo `status` em vez de múltiplos booleans.
- Campos computados (ex: `valor_final = editado ?? original`) não precisam ser persistidos.
- Tipos/enums compartilhados entre camadas (router/service, back/front) vivem num módulo central — duplicação diverge na primeira mudança.

## Método de ADR

Toda decisão arquitetural relevante vira ADR (`docs/arch/adr/_ADR_TEMPLATE.md`): Status / Contexto / Decisão / Alternativas consideradas (com o porquê de cada rejeição) / Consequências. **Documente o que foi descartado e por quê** — salva o time de redebater. Liste os descartados nesta MEMORY conforme acumular.

- **Dependência externa arriscada (API não-oficial, provedor instável) → plano B documentado no mesmo ADR.** Quando o plano B foi necessário de verdade, tê-lo por escrito economizou um ciclo inteiro de decisão.
- **Trade-offs aceitos ficam registrados** (ex: heurística de matching com falso positivo improvável) — para não rediscutir a cada review.
- **Deprecar, não deletar:** artefato substituído (prompt antigo, rota antiga) é mantido e marcado como deprecado para compatibilidade histórica — remoção silenciosa quebra rastreabilidade.

## Padrões de segurança (universais)

- Segredos/keys **nunca** em response body nem em log. GET retorna só `{ configurado: bool, updated_at }`.
- Validação de input nos boundaries (lib de schema: Zod/Pydantic/etc.) — sobre o valor pós-normalização.
- CORS: só o domínio do front em produção — sem wildcard.
- Segredos em env var, nunca commitados. Documente quais env vars são obrigatórias.
- **CSP por tipo de recurso:** especifique as origens externas de `connect-src`, `img-src`, `media-src` via env var — e revise o CSP sempre que uma feature passar a carregar um novo tipo de recurso externo.
- **BaaS com service-role key:** o RLS/segurança do provedor é bypassado por design — **toda autorização vive no middleware da API**. Registre essa decisão em ADR para ninguém assumir o contrário.
- **Contratos de integração externa incluem limites:** timeout por chamada, tamanho máximo de download, comportamento ao atingir rate limit (pausar o ciclo, não derrubar o worker).

## Padrões de teste (recomendados)

- Unit: cobertura mínima alvo em `services/` (ex: 70%).
- Integração: contra **banco real local** (Docker), nunca contra produção.
- E2E: cobrir os fluxos críticos da UX.
- **Nunca mockar o banco.** Mocks só para APIs externas (HTTP, LLM, provedores).
- Isole o contrato de cada API externa num único módulo/função que recebe o JSON cru e retorna tipos internos — o resto do sistema nunca vê o shape do provider.
- Invariantes que atravessam duas operações (ex: "cancelado não executa") entram na estratégia de testes como cenário sequencial explícito.

## Performance / dados

- **Índice desenhado para a query quente:** o worker/dispatcher que roda a cada minuto merece índice (parcial, se o filtro for por status) — não deixe a query mais frequente do sistema sem índice.
- Batch insert em loops de coleta; upsert atômico (`ON CONFLICT`) em vez de SELECT-then-INSERT.
- Limites do free tier do provedor documentados com mitigação (pausa por inatividade, teto de storage, pooling de conexões via pooler — não conexão direta).

## Gotchas e decisões descartadas

_(Acumule aqui: "avaliamos X e descartamos porque Y". Ex: filas/workers pesados sem ganho pro volume; BaaS auth vs. auth própria; embeddings vs. heurística simples. Sempre com o trade-off registrado.)_

## Estimativa de custo

_(Mantenha uma tabela de custo mensal estimado por item quando o projeto tiver restrição de budget. O usuário costuma ter budget rígido.)_

- Inclua um **cenário pessimista** (ex: 2× o item mais volátil) e confira que ainda cabe no teto.
- **Breaking changes de env vars entre épicos** (adicionar/remover/renomear) são parte da spec — liste explicitamente para a task de deploy.
