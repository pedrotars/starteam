# Squad Playbook

> O manual de processo da squad. Destilado da execução real de um MVP fullstack do PRD ao deploy.
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
- **(b) Branch de integração por onda:** construa a onda inteira numa única branch baseada em `main` e abra **um PR só** para a onda (base `main`). Use quando ele pedir "toca a onda inteira".

**Nunca** empilhe PRs com base numa feature branch não-mergeada.

### Convenção de branches
```
orchestrator/EP-NN-slug      ← kickoff e bookkeeping do épico
product/EP-NN-prd-ux         ← PM + UX
architect/EP-NN-spec-tecnica ← spec técnica + ADRs
task-manager/EP-NN-quebra    ← quebra em tasks
engineer/NNN-task-slug       ← uma task de dev (ou engineer/EP-NN-onda-K p/ estratégia b)
```

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

> **Padrão observado:** o QA quase sempre acha 2–7 ressalvas Medium/Low por task. Conserte as baratas/de-fundação no mesmo PR; empurre pra follow-up só o que depende de algo futuro. Mantém qualidade alta sem travar o fluxo.

---

## 5. Ondas de execução

Quebre o épico em **ondas** por dependência, não numa lista plana. Padrão que funcionou para um fullstack:

1. **Fundação** — scaffold (back/front), migrations, auth. Sem lógica de negócio.
2. **Segurança e configuração** — cripto de segredos, CRUD de configuração. Dependem do auth.
3. **Pipeline de backend** — a cadeia sequencial de lógica de negócio (cada etapa depende da anterior).
4. **Frontend** — telas, paralelas ao pipeline mas dependentes do auth/scaffold front.
5. **Fechamento** — testes E2E (dependem do front estável) → deploy/infra → onboarding.

Dependências recorrentes: **migrations antes de tudo**; **auth antes de qualquer rota protegida**; **scaffold front antes de qualquer tela**; **cripto antes do worker que usa o segredo**.

---

## 6. Estimativas (calibradas)

- **S** (< meio dia): scaffold, migrations DDL, endpoint CRUD com testes, módulo isolado, job/dispatcher.
- **M** (meio dia–2 dias): workers com lógica de negócio, telas com múltiplos estados/componentes, suíte E2E multi-cenário, deploy + docs.
- **L** (> 2 dias): **alerta — quebre mais.** Se uma feature de front tem 6+ componentes novos + fluxos complexos, separe "componentes base" de "integração de tela". Numa execução de 21 tasks, nenhuma precisou ser L.

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
- [ ] Lint + typecheck + format limpos.
- [ ] Sem segredo/PII em log ou em response body.
- [ ] Validação de input nos boundaries.
- [ ] Review do `qa` escrita no task file com veredito.
- [ ] Bugs baratos corrigidos no PR; follow-ups registrados.
- [ ] Task movida para `DONE`; memórias relevantes atualizadas e **commitadas**.
- [ ] PR aberto contra `main` (nunca empilhado).
