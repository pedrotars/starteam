---
id: NNN-slug
epic: EP-XX-nome
estimate: S | M | L
depends-on: []
created: YYYY-MM-DD
---

# [NNN] — [Título da task]

**Estado:** TODO | IN_PROGRESS | DONE | BLOCKED
**Tollbooth atual:** PM/UX | Architect | Dev | QA-final

> Para um **épico** (EP-NN), adicione no topo a seção **"Intenção do usuário (input cru)"**
> antes das seções dos especialistas (objetivo / restrições declaradas / diagnósticos
> preliminares do usuário / o que NÃO é decisão ainda). Ver PLAYBOOK §7.

---

## Especificação de Produto
_(preenchido por `product`)_

### Problema
### Personas
### User Stories
### Critérios de Aceite
- CA-01:
- CA-02:
### Métricas de Sucesso
### Fora de Escopo

---

## UX
_(preenchido por `ux` — roda DEPOIS do product)_

### Fluxos
### Inventário de Componentes
### Wireframes
### Estados (default / hover / loading / error / empty / success)
### Microcopy
### Acessibilidade
### Responsividade

> **🚦 Tollbooth 1 — Aprovação PM/UX:** [ ] aprovado por humano em ____

---

## Especificação Técnica
_(preenchido por `architect`)_

### Contratos de API
### Type Definitions
### Modelo de Dados
### Estratégia de Testes
### ADRs Relacionados
### Considerações de Segurança
### Considerações de Performance

> **🚦 Tollbooth 2 — Aprovação Architect:** [ ] aprovado por humano em ____

---

> **🚦 Tollbooth 3 — Pronto para Dev:** [ ] aprovado para entrada em IN_PROGRESS em ____

## Log de Execução
_(preenchido por `engineer`)_

### Commits
### Decisões locais
### Surpresas

---

## Review
_(preenchido por `qa`)_

**Data:** · **Revisor:** qa
**Status final:** Aprovado | Aprovado-com-ressalvas | Reprovado

### Resumo
### Conformidade com Critérios de Aceite
### Bugs encontrados (por severidade)
### Sugestões de melhoria

> **🚦 Checkpoint humano antes do merge:** [ ] aprovado por humano em ____

---

## Definition of Done
_(checklist — ver PLAYBOOK §11)_
- [ ] CAs atendidos e observáveis
- [ ] Testes passando · lint **completo** (lint + format-check)/typecheck limpos
- [ ] Sem segredo/PII em log ou response
- [ ] Validação pós-normalização gateando **todos** os caminhos de ação
- [ ] Chamadas externas com timeout; downloads com limite de tamanho
- [ ] Mensagens de erro refletem o comportamento real (limites/valores)
- [ ] Review do `qa` com veredito
- [ ] Bugs baratos corrigidos no PR; follow-ups registrados
- [ ] Memórias atualizadas e **commitadas**
- [ ] PR aberto contra `main` (nunca empilhado)
