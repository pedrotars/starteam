# Squad Playbook — v2

> O manual de processo da squad. Destilado da execução real de um produto fullstack do PRD ao deploy
> **e além** — 5 épicos, 60+ tasks e dezenas de reviews de QA. A v2 incorpora os erros e acertos
> acumulados na operação contínua (não só no MVP inicial), com ênfase em segurança, testes e eficiência.
> O `orchestrator` lê isto no início de cada épico. Os especialistas leem as seções que lhes dizem respeito.

---

## 1. O fluxo e os tollbooths

```
Intenção do usuário (input cru)
        ↓
   [product] → [ux]          ← SEQUENCIAL, não paralelo (ver §3)
        ↓
🚦 Tollbooth 1 — Aprovação PM/UX (humano)
        ↓
    [architect]              ← ADRs + spec técnica
        ↓
🚦 Tollbooth 2 — Aprovação Architect (humano)
        ↓
   [task-manager]            ← quebra o épico em tasks no board
        ↓
🚦 Tollbooth 3 — Pronto para Dev (humano)
        ↓
   [engineer] → [qa]         ← TDD; uma task por vez em IN_PROGRESS
        ↓
🚦 Checkpoint humano antes do merge (cada task = 1 PR)
        ↓
   [tech-writer]             ← pós-merge
```

**Os tollbooths são intransponíveis sem aprovação humana explícita.** "Manda bala" não pula tollbooth — peça o OK do tollbooth corrente. Cada tollbooth é materializado como um **PR**; mergear o PR = a aprovação humana.

---

## 2. A regra que mais doeu: disciplina de branch/PR

> **Lição custosa:** PRs empilhados com base em branch não-mergeada **não funcionam**. Abrir #6(task2→task1), #7(task3→task2)… faz com que, ao mergear, só o primeiro (base `main`) chegue na `main`; os demais mergeiam nas branches intermediárias e ficam em limbo. A `main` fica só com a primeira task.

**Regra:** toda branch de dev sai de uma **`main` atualizada**. Duas estratégias permitidas:

- **(a) Per-task PR contra `main`:** espere o usuário mergear a task anterior antes de abrir a próxima branch. Use quando ele está revisando em tempo real.
- **(b) Branch de integração por onda:** construa a onda inteira numa única branch baseada em `main` e abra **um PR só** para a onda (base `main`). Use quando ele pedir "toca a onda inteira". **(Validada repetidamente em produção — 4 ondas consecutivas sem incidente de branch.)**

**Nunca** empilhe PRs com base numa feature branch não-mergeada.

### Convenção de branches
```
orchestrator/EP-NN-slug      ← kickoff e bookkeeping do épico
product/EP-NN-prd-ux         ← PM + UX
architect/EP-NN-spec-tecnica ← spec técnica + ADRs
task-manager/EP-NN-quebra    ← quebra em tasks
engineer/NNN-task-slug       ← uma task de dev (ou engineer/EP-NN-onda-K p/ estratégia b)
hotfix/slug                  ← correção pós-merge (CI quebrado, ressalva de QA tardia)
```

### Fix-pass pós-merge
Ressalvas do QA descobertas **depois** do merge (ou bugs de CI) não reabrem a task: viram um **PR de fix-pass dedicado** em `hotfix/slug` (ou `engineer/EP-NN-fix-pass`), referenciando os BUG-IDs no commit. Pequeno, focado, mergeável rápido.

### Gotcha do MEMORY.md non-staged
O `orchestrator` e o `qa` costumam editar `MEMORY.md` **depois** do commit principal, deixando-o non-staged. **Commite o MEMORY.md antes do próximo `git checkout`** ou ele se perde na troca de branch.

---

## 3. Acionamento dos agentes — ordem e paralelismo

- **`product` → `ux` é SEQUENCIAL.** Ambos escrevem no MESMO task file e o `ux` depende do PRD pronto. Rodar em paralelo gera conflito de escrita. (Corrige a intuição de "dispara os dois juntos".)
- **`engineer` e `qa` são sequenciais** por task: engineer implementa, depois qa revisa.
- **Backend e frontend da mesma feature** podem ser tasks paralelas **depois** que a fundação comum (auth, migrations, scaffold) estiver pronta.
- **Uma task em IN_PROGRESS por dev por vez.** Board defasado é board mentiroso — mova o estado na hora.

---

## 4. O loop de dev (estabelecido e validado)

1. `orchestrator` move a task `TODO → IN_PROGRESS` (git mv) e marca o Tollbooth 3.
2. `engineer` implementa em **TDD** na branch da task. Commita. **Não abre PR, não move para DONE.**
3. `qa` revisa e escreve a seção "Review" no task file.
4. Se o `qa` achar bug **barato ou de fundação** → `orchestrator` manda o `engineer` corrigir **no mesmo PR**.
5. Bugs que dependem de **task futura** → viram follow-up registrado **na task futura** (ou em `tasks/FOLLOW-UPS.md` se forem cross-cutting).
6. `orchestrator` move para `DONE`, commita bookkeeping + memórias, abre PR.
7. Mergear o PR = checkpoint humano.

> **Padrão observado:** o QA quase sempre acha 2–7 ressalvas Medium/Low por task. Conserte as baratas/de-fundação no mesmo PR; empurre pra follow-up só o que depende de algo futuro. Mantém qualidade alta sem travar o fluxo. Ressalvas descobertas pós-merge → fix-pass (§2).

---

## 5. Ondas de execução

Quebre o épico em **ondas** por dependência, não numa lista plana. Padrão que funcionou para um fullstack:

1. **Fundação** — scaffold (back/front), migrations, auth. Sem lógica de negócio.
2. **Segurança e configuração** — cripto de segredos, CRUD de configuração. Dependem do auth.
3. **Pipeline de backend** — a cadeia sequencial de lógica de negócio (cada etapa depende da anterior).
4. **Frontend** — telas, paralelas ao pipeline mas dependentes do auth/scaffold front.
5. **Fechamento** — testes E2E (dependem do front estável) → deploy/infra → onboarding.

Para **épicos de evolução** (produto já no ar), o padrão validado tem 6 ondas: fundação de dados (migrations + infra de storage) → infra de backend → pipelines de geração → exposição de API → frontend → fechamento (E2E + deploy).

Dependências recorrentes: **migrations antes de tudo**; **auth antes de qualquer rota protegida**; **scaffold front antes de qualquer tela**; **cripto antes do worker que usa o segredo**; **endpoints/tipos novos antes do frontend que os consome**.

Padrões adicionais validados:
- **Configuração manual de provedor** (bucket de storage, dashboard externo) = task própria S, separada das migrations — é setup manual + SDK, não código de migração.
- **Frontend quebrado por componente/feature** (várias S) em vez de uma task M "frontend do épico" — reduz o risco de PR gigante.
- **Limpeza/retenção de dados** = task separada do fluxo principal; pode rodar em paralelo ao frontend.

---

## 6. Estimativas (calibradas em 2 épicos grandes)

- **S** (< meio dia): scaffold, migrations DDL, endpoint CRUD com testes, módulo isolado, job/dispatcher, componente frontend individual com estados bem definidos, configuração de storage/infra.
- **M** (meio dia–2 dias): workers com lógica de negócio, telas com múltiplos estados/componentes, suíte E2E multi-cenário, deploy + docs.
- **L** (> 2 dias): **alerta — quebre.** Exceção validada: quando os sub-componentes compartilham a mesma função/estado central (quebrar criaria um 2º PR dependente de estado incompleto do 1º), a task pode ficar L com a nota **"considerar quebra em 2 PRs — decisão do engineer ao começar"**.

Granularidade certa: **uma task = um PR**. Não cabe num PR → grande demais. Não faz nada sozinha → pequena demais.

---

## 7. Capturando a intenção do usuário (anti-armadilha)

> **Lição:** na primeira interação é tentador responder como product + architect + engineer de uma vez e já escolher stack. Isso contamina o trabalho dos especialistas e foi corrigido pelo usuário. **Capture, não decida.**

Quando o usuário descreve uma ideia:
1. **NÃO** decida produto, design ou stack.
2. Crie `tasks/IN_PROGRESS/EP-NN-slug.md` com uma seção **"Intenção do usuário (input cru)"** no topo, antes das seções dos especialistas.
3. Separe: o que ele **quer** (objetivo) · **restrições declaradas** (budget, cadência, multi-user) · **diagnósticos preliminares** dele (com aviso de que o `architect` pode questionar) · lista do que **NÃO é decisão ainda**.
4. Deixe as seções dos especialistas com placeholders `(a ser preenchida por X)`.
5. Dispare `product`, depois `ux`. Pare no Tollbooth 1.

---

## 8. Decisões e hipóteses (brief.md)

- **Decisões macro** viram linhas `D-NN` na tabela do `docs/brief.md`, com origem (qual tollbooth/épico) e status.
- **Hipóteses** viram `H-NN`: o que estamos assumindo + **como validar** + status. Quando o usuário resolve uma hipótese por decisão, marque-a **Decidida (D-YY)** — não delete, para rastreabilidade.
- Atualize a persona afetada e a `MEMORY.md` quando uma hipótese fechar.

---

## 9. Memória: o que guardar e o que NÃO guardar

> **Lição:** as memórias do `engineer` e do `qa` viraram um despejo de cada bug de cada task e incharam para dezenas de KB. Isso dilui o sinal.

**Vai na `MEMORY.md` do agente** (durável, relido sempre):
- Padrões que a squad acordou (não re-decidir toda vez).
- Convenções de stack/código/teste do projeto.
- Gotchas recorrentes e "já tentamos X, não funcionou porque Y".
- **Bugs que viram um padrão** (regra de prevenção reutilizável) — não o log do bug específico.

**NÃO vai na memória** (vive no task file):
- Estado de uma task específica.
- Log de execução / lista de commits de uma task.
- O relato de um bug pontual que não vira regra.

Releia e **poda** a memória periodicamente. Memória que ninguém relê é peso morto.

---

## 10. Usuário não-técnico

Quando o destinatário não é técnico (caso comum aqui):
- A última task do épico inclui um **documento de onboarding** passo-a-passo (deploy, operação, reset de senha).
- Microcopy e mensagens de erro sem jargão. Termos técnicos só onde o contexto exige, com explicação curta.
- Nas respostas ao usuário: passo-a-passo claro para qualquer coisa de infra, sem assumir terminal/dev.

---

## 11. Definition of Done (checklist por task)

- [ ] Todos os Critérios de Aceite da spec atendidos e **observáveis**.
- [ ] Testes (unit/integração/E2E conforme a estratégia do architect) passando localmente.
- [ ] Lint **completo** + typecheck + format limpos (lint e format-check são alvos distintos — rode ambos).
- [ ] Sem segredo/PII em log ou em response body.
- [ ] Validação de input nos boundaries — sobre o valor **pós-normalização**, gateando **todos** os caminhos de ação.
- [ ] Toda chamada externa (HTTP, download) com **timeout** e, para downloads, **limite de tamanho**.
- [ ] Mensagens de erro refletem o **comportamento real** (limites, valores) — não a versão antiga da spec.
- [ ] Review do `qa` escrita no task file com veredito.
- [ ] Bugs baratos corrigidos no PR; follow-ups registrados.
- [ ] Task movida para `DONE`; memórias relevantes atualizadas e **commitadas**.
- [ ] PR aberto contra `main` (nunca empilhado).

---

## 12. Baseline de segurança da squad

> Segurança não é uma fase — é responsabilidade distribuída. Cada agente tem a sua parte; o `qa` é o último filtro, não o primeiro.

**Por fase:**
- **`product`:** CAs de segurança especificam o visível na UI **e** o verificável no banco ("nunca aparece em texto plano" + "armazenado encriptado").
- **`ux`:** segredos nunca exibidos (status apenas); campos de segredo write-only (`type="password"`, sem autocomplete); permissões refletidas nos menus; validação client-side desabilita **todos** os caminhos de submissão, não só o botão primário.
- **`architect`:** formato de erro canônico desde o início; segredos só em env vars; CORS sem wildcard; dependência externa arriscada → **plano B documentado no ADR**; contratos de download/integração externa incluem timeout e tamanho máximo; se usar BaaS com service-role key, **toda autorização vive no middleware** (RLS bypassado por design — registre em ADR).
- **`engineer`:** guards contra secret default em produção (JWT_SECRET, senha de seed); verify de senha com hash dummy quando o usuário não existe (anti timing side-channel); validação pós-normalização; nunca logar segredo/token/conteúdo sensível.
- **`qa`:** checklist universal da sua MEMORY — inclui anti-leak por raw text (não só dict key), anti-enumeração com AND, rate-limit com interrupção real do loop, rotas de teste gateadas por ambiente.

**Itens que mais escaparam em produção real (verificar sempre):**
1. Download de mídia/arquivo externo **sem limite de tamanho** e fora do try → resource exhaustion + crash não tratado.
2. Rate limit de API externa **checado mas não respeitado** — ler o header sem `break` no loop é log-only, não proteção.
3. Script de seed com **senha default hardcoded** — ambiente que esquece a env var cria admin com senha pública.
4. Mensagem de erro 422 divergente do limite real aplicado — confunde o usuário e a depuração.
5. Rota criada só para exercitar testes indo para produção.

---

## 13. Higiene de testes e CI

Regras que custaram builds quebrados e falsos verdes:

- **Testes ocos:** assert dentro de bloco condicional (`if x: assert ...`) passa silenciosamente quando a condição é falsa. Asserts são **incondicionais**; se o cenário pode não ocorrer, o teste está mal desenhado.
- **Invariantes de duas operações** (ex: "item cancelado não é executado") exigem **um teste que executa as duas em sequência** — testar cancelamento e execução separadamente não cobre o invariante.
- **Teardown na ordem dos FKs:** delete filhos antes de pais no teardown de integração — localmente pode passar, no CI quebra.
- **`TZ=UTC` no script de teste** + horários com margem — datas locais sem sufixo `Z` são interpretadas no fuso da máquina e quebram em CI.
- **Nenhum teste depende de rede** ou de corpus/dados baixáveis — fallback embutido para dados externos opcionais.
- **Sem path absoluto em config de teste/build** (`path.resolve(__dirname)`, nunca `/Users/...`) — quebra em CI e em qualquer outra máquina.
- **Lint completo no CI:** `lint` e `format --check` são alvos distintos; rodar só um deixa o outro quebrar no PR seguinte.
- **Env vars em fixtures:** `monkeypatch`/context manager, nunca mutação permanente do ambiente do processo.
- **Engines/conexões de teste:** fixture com escopo de módulo + `dispose()` no teardown — engines presos a event loop antigo geram erros opacos.
