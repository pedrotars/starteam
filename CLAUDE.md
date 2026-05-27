# CLAUDE.md

> Lido automaticamente por qualquer sessão do Claude Code aberta nesta pasta.
> Orienta tanto o **orchestrator** quanto uma sessão avulsa (`claude`) sobre como este projeto opera.

## O que é este projeto

_(Preencha quando instanciar o template. Uma frase: o que estamos construindo e para quem.)_

Este repositório usa um **modelo de squad de 8 agentes** coordenados por um `orchestrator`. O fluxo, os portões (tollbooths) e as regras de processo estão no **[PLAYBOOK.md](PLAYBOOK.md)** — leia-o antes de agir em qualquer pedido não-trivial.

## Regra de ouro

**Não decida produto, design ou arquitetura no vácuo.** Se o pedido do usuário é um épico ou uma feature, ele passa pela squad (`product` → `ux` → `architect` → `task-manager` → `engineer` → `qa` → `tech-writer`), respeitando os tollbooths. Dúvidas pontuais e bugs pequenos podem ser respondidos direto. Veja o PLAYBOOK para a fronteira.

## Onde está o quê

| Caminho | O que é |
|---|---|
| `.claude/agents/` | Definição dos 8 agentes da squad |
| `.claude/agent-memory/<agente>/MEMORY.md` | Memória persistente de cada agente (lida no início de cada acionamento) |
| `tasks/` | O board. O **filesystem é a fonte da verdade** (TODO / IN_PROGRESS / DONE / BLOCKED) |
| `tasks/_TEMPLATE.md` | Template canônico de task file |
| `tasks/FOLLOW-UPS.md` | Follow-ups cross-cutting que não pertencem a uma task única |
| `docs/brief.md` | Resumo executivo + log de decisões macro (D-NN) + hipóteses (H-NN) |
| `docs/arch/adr/` | Architecture Decision Records |
| `PLAYBOOK.md` | Manual de processo da squad (fluxo, tollbooths, lições aprendidas) |

## Gotcha crítico — disponibilidade dos agentes

Os 8 subagentes só ficam visíveis para a Agent tool quando o Claude Code é aberto **de dentro desta pasta** (ou subpasta). Se você abrir da home, só os agentes globais aparecem. Para acionar a squad de verdade, abra o Claude Code dentro do diretório do projeto.

## Convenções que valem para qualquer sessão

- **Idioma:** PT-BR na comunicação com o usuário e na documentação. Código e nomes técnicos em inglês.
- **Commits/PRs:** o usuário prefere **PR a commit direto na `main`**. Trabalhe em branch e abra PR. Veja a convenção de branches no PLAYBOOK.
- **Antes de sobrescrever/deletar:** olhe o conteúdo. Se contradiz a descrição, levante a mão em vez de prosseguir.
- **Memória:** ao editar um `MEMORY.md`, **commite antes do próximo `git checkout`** — esses arquivos costumam ficar non-staged e se perdem na troca de branch.
