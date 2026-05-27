# Agent Memory — task-manager

> Padrões de quebra e calibração acumulados. Lido no início de cada acionamento.
> Semeada com **método** genérico (sem a quebra específica de um projeto).

## Granularidade

- Uma task = **um PR**. Não cabe num PR → grande demais. Não faz nada sozinha → pequena demais.
- Num MVP fullstack de porte médio-grande, ~20 tasks (mix S/M, nenhuma L) cobriu do PRD ao deploy.

## Estimativas calibradas

- **S** (< meio dia): scaffold, migrations DDL, endpoint CRUD com testes, módulo isolado, job/dispatcher.
- **M** (meio dia–2 dias): worker com lógica de negócio, tela com múltiplos estados/componentes, suíte E2E multi-cenário, deploy + docs.
- **L** (> 2 dias): **alerta — quebre.** Feature de front com 6+ componentes novos + fluxos complexos → separe "componentes base" de "integração de tela".

## Padrão de ondas (por dependência, não lista plana)

1. **Fundação** — scaffold (back/front), migrations, auth. Sem lógica de negócio.
2. **Segurança e configuração** — cripto de segredos, CRUD de config. Dependem do auth.
3. **Pipeline de backend** — cadeia sequencial de lógica de negócio.
4. **Frontend** — telas, paralelas ao pipeline mas dependentes do auth/scaffold front.
5. **Fechamento** — E2E → deploy/infra → onboarding.

## Dependências recorrentes

- **Migrations antes de tudo** que toca o banco.
- **Auth antes de qualquer rota protegida.**
- **Scaffold front antes de qualquer tela.**
- **Cripto de segredos antes do worker que usa o segredo.**

## Convenções de quebra que funcionaram

- Separar back e front da mesma feature (paralelismo após a fundação comum).
- A feature mais crítica/complexa do épico → granularidade mais fina (mais tasks menores).
- E2E em task própria (depende de todo o front estável).
- **Usuário não-técnico → última task inclui onboarding** (deploy + operação passo-a-passo).

## Gotchas de cobertura de CAs

- Invariantes do modelo de dados (ex: "nada é executado sem aprovação") aparecem em **várias** tasks (migration + backend + E2E). Marque em todas.
- CAs com sub-critérios (ex: agendamento a/b/c/d) se distribuem entre tasks — mapeie qual task cobre qual.
- CAs que não são código (ex: "custo < $X/mês") são verificados fora do código — documente na task de deploy como o usuário monitora.

## Quebra do projeto atual

_(Registre aqui a quebra real conforme ela acontecer: nº de tasks, ondas, o que funcionou/errou.)_
