# Agent Memory — task-manager

> Padrões de quebra e calibração acumulados. Lido no início de cada acionamento.
> Semeada com **método** genérico, calibrado em 2 quebras reais de épico grande (21 e 17 tasks).

## Granularidade

- Uma task = **um PR**. Não cabe num PR → grande demais. Não faz nada sozinha → pequena demais.
- Num MVP fullstack de porte médio-grande, ~20 tasks (mix S/M, nenhuma L) cobriu do PRD ao deploy.
- Num épico de **evolução** de porte grande (novos pipelines + frontend extenso), 17 tasks (10 S, 6 M, 1 L) em 6 ondas.

## Estimativas calibradas

- **S** (< meio dia): scaffold, migrations DDL, endpoint CRUD com testes, módulo isolado, job/dispatcher, **componente frontend individual com estados bem definidos**, **configuração de storage/infra**.
- **M** (meio dia–2 dias): worker com lógica de negócio, tela com múltiplos estados/componentes, suíte E2E multi-cenário, deploy + docs.
- **L** (> 2 dias): **alerta — quebre.** Exceção validada: quando os sub-componentes compartilham a mesma função/estado central (quebrar criaria um 2º PR dependente de estado incompleto do 1º), mantenha L com a nota **"considerar quebra em 2 PRs — decisão do engineer ao começar"**.

## Padrão de ondas (por dependência, não lista plana)

**MVP do zero (5 ondas):**
1. **Fundação** — scaffold (back/front), migrations, auth. Sem lógica de negócio.
2. **Segurança e configuração** — cripto de segredos, CRUD de config. Dependem do auth.
3. **Pipeline de backend** — cadeia sequencial de lógica de negócio.
4. **Frontend** — telas, paralelas ao pipeline mas dependentes do auth/scaffold front.
5. **Fechamento** — E2E → deploy/infra → onboarding.

**Épico de evolução (6 ondas, produto já no ar):**
1. **Fundação de dados** — migrations novas + infra de storage (paralelas).
2. **Infra de backend** — scheduler/cadência, coleta, endpoints de config.
3. **Pipelines de geração** — cadeia sequencial.
4. **Aprendizado/exposição de API** — o que o frontend novo vai consumir.
5. **Frontend** — quebrado por componente/feature (várias S), não uma M única.
6. **Fechamento** — E2E → deploy.

## Dependências recorrentes

- **Migrations antes de tudo** que toca o banco.
- **Auth antes de qualquer rota protegida.**
- **Scaffold front antes de qualquer tela.**
- **Cripto de segredos antes do worker que usa o segredo.**
- **Endpoints/tipos novos antes do frontend que os consome** — os tipos do front dependem dos campos novos do back.
- **Storage/infra antes do pipeline que grava nela.**

## Convenções de quebra que funcionaram

- Separar back e front da mesma feature (paralelismo após a fundação comum).
- A feature mais crítica/complexa do épico → granularidade mais fina (mais tasks menores).
- E2E em task própria (depende de todo o front estável).
- **Usuário não-técnico → última task inclui onboarding** (deploy + operação passo-a-passo).
- **Configuração manual de provedor** (bucket, painel externo) = task própria S — é setup manual + SDK, não migração de código.
- **Limpeza/retenção de dados** = task separada do fluxo principal; pode rodar em paralelo ao frontend.
- **Frontend por componente:** várias tasks S (Card, Painel, Tela de config, etc.) em vez de uma M "frontend do épico" — evita PR gigante.

## Gotchas de cobertura de CAs

- Invariantes do modelo de dados (ex: "nada é executado sem aprovação") aparecem em **várias** tasks (migration + backend + E2E). Marque em todas.
- CAs com sub-critérios (ex: agendamento a/b/c/d) se distribuem entre tasks — mapeie qual task cobre qual.
- **CA que toca back e front** (ex: remover uma opção: migration de dados no back + dropdown no front) é marcado **nas duas tasks** — cobrir só um lado deixa o CA pela metade.
- CAs que não são código (ex: "custo < $X/mês") são verificados fora do código — documente na task de deploy como o usuário monitora.

## Quebra do projeto atual

_(Registre aqui a quebra real conforme ela acontecer: nº de tasks, ondas, o que funcionou/errou.)_
