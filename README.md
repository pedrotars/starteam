# Squad Template

Um scaffold reutilizável para tocar projetos de software com uma **squad de 8 agentes** especialistas coordenados por um `orchestrator`, com portões de aprovação humana (tollbooths) em cada transição de fase.

Destilado da execução real de um MVP fullstack (21 tasks, 5 ondas, do PRD ao deploy). Os aprendizados de **processo** dessa execução estão embutidos nas memórias dos agentes e no [PLAYBOOK.md](PLAYBOOK.md).

## A squad

| Agente | Papel | Entrega |
|--------|-------|---------|
| `orchestrator` | Regente — coordena, não executa | Dispatch + enforcement de tollbooths |
| `product` | O quê e por quê | Especificação de Produto (PRD) |
| `ux` | Como o usuário interage | Fluxos, estados, microcopy, a11y |
| `architect` | Como o sistema funciona | Spec técnica + ADRs |
| `task-manager` | Quebra e prioriza | Tasks executáveis no board |
| `engineer` | Implementa em TDD | Código + testes |
| `qa` | Revisa contra a spec | Veredito + bugs por severidade |
| `tech-writer` | Documenta pós-merge | README, docs de API, onboarding |

## Fluxo (resumido)

```
Intenção → [product → ux] → 🚦T1 → [architect] → 🚦T2 → [task-manager] → 🚦T3 → [engineer → qa] → 🚦merge → [tech-writer]
```

Os 🚦 são **tollbooths intransponíveis sem aprovação humana explícita**. Detalhes no PLAYBOOK.

## Como usar este template

1. Copie esta pasta para o seu novo projeto (ou use como base do repo).
2. Abra o Claude Code **de dentro da pasta** (senão os agentes da squad não aparecem — ver `CLAUDE.md`).
3. Preencha a primeira frase de `CLAUDE.md` (o que é o projeto).
4. Descreva sua ideia/problema ao `orchestrator`. Ele captura a intenção crua no épico `EP-01`, dispara a squad e segura nos tollbooths.
5. Aprove cada tollbooth via merge do PR correspondente.

## Estrutura

```
.claude/agents/            ← os 8 agentes
.claude/agent-memory/      ← memória persistente por agente
tasks/                     ← o board (FS é a fonte da verdade)
  _TEMPLATE.md
  FOLLOW-UPS.md
docs/brief.md              ← decisões macro + hipóteses
docs/arch/adr/             ← ADRs
CLAUDE.md                  ← orientação para qualquer sessão
PLAYBOOK.md                ← manual de processo da squad
```
