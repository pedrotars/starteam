# Orchestrator Memory

> Padrões, decisões de processo e gotchas acumulados. Lido no início de cada acionamento.
> Semeada com aprendizados de **processo** de um MVP fullstack real (genéricos — sem domínio específico).

## Sobre o usuário (Pedro)

- **Não é técnico** — precisa de passo-a-passo claro para qualquer coisa de infra/dev. Evitar jargão sem explicar.
- **Comunica em PT-BR.**
- **Prefere PRs a commits diretos** na `main`. Toda alteração de repo vai via branch + PR para revisão dele.
- **Único ponto de contato é o `orchestrator`** — Pedro não conversa com os outros agentes diretamente.
- Restrição de custo costuma ser rígida em projetos pessoais — confirme o budget cedo e trate como hard constraint.

## Gotcha crítico — disponibilidade dos agentes

A squad (`product`, `ux`, `architect`, `task-manager`, `engineer`, `qa`, `tech-writer`) só fica visível para a Agent tool quando o Claude Code é aberto **de dentro** da pasta do projeto (ou subpasta). Aberto da home, só os agentes globais aparecem. → Se precisar acionar a squad de verdade, avise o Pedro pra abrir o Claude Code dentro do projeto. De fora, dá pra preparar arquivos, mas não delegar aos especialistas.

## Lições de processo (custaram caro — não repetir)

- **PRs empilhados em branch não-mergeada NÃO funcionam.** Só o primeiro (base `main`) chega na `main`; o resto fica em limbo. Toda branch sai de `main` atualizada. Per-task contra `main`, OU branch de integração por onda com um PR só. (PLAYBOOK §2)
- **`product` → `ux` é SEQUENCIAL**, não paralelo. Escrevem no mesmo task file; o `ux` depende do PRD. Paralelo = conflito de escrita. (PLAYBOOK §3)
- **`MEMORY.md` editado fica non-staged.** O `orchestrator` e o `qa` editam memória depois do commit principal. Commite-a **antes** do próximo `git checkout` ou ela se perde.
- **Não decida no primeiro pedido.** Tentação de responder como product+architect+engineer e escolher stack. Capture a intenção crua, dispare a squad. (PLAYBOOK §7)

## Padrões que funcionaram bem

- **QA acha 2–7 ressalvas Medium/Low por task.** Mande o `engineer` corrigir as baratas/de-fundação no mesmo PR; empurre pra follow-up só o que depende de task futura. Qualidade alta sem travar.
- **Loop de dev:** orchestrator move TODO→IN_PROGRESS + marca tollbooth → engineer implementa em TDD e commita (não abre PR, não move DONE) → qa revisa e escreve Review → orchestrator manda corrigir bugs baratos → move DONE, commita bookkeeping+memórias, abre PR. Mergear PR = checkpoint humano. UMA task em IN_PROGRESS por vez.
- **Cada tollbooth = um PR.** kickoff=PR, PM/UX=PR, Arch=PR, quebra-tasks=PR, cada task de dev=PR. Mergear = aprovar.
- **Follow-ups cross-cutting se perdem** (ex: um campo que precisa de fix em vários modelos). Registre em `tasks/FOLLOW-UPS.md` e relembre o engineer na próxima task relevante.

## Convenção de branches

```
orchestrator/EP-NN-slug      product/EP-NN-prd-ux       architect/EP-NN-spec-tecnica
task-manager/EP-NN-quebra    engineer/NNN-task-slug     (ou engineer/EP-NN-onda-K)
```

## Padrão de captura de intenção

Ao receber uma ideia/problema: criar `tasks/IN_PROGRESS/EP-NN-slug.md` com **"Intenção do usuário (input cru)"** no topo (objetivo / restrições declaradas / diagnósticos preliminares dele / o que NÃO é decisão ainda), placeholders nas seções dos especialistas, e disparar `product` → `ux`. Parar no Tollbooth 1.

---

## Estado do projeto atual

_(Preencha quando instanciar. Épicos em voo, tollbooth corrente, decisões recentes.)_
