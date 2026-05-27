# Agent Memory — UX

> Padrões de design e gotchas acumulados. Lido no início de cada acionamento.
> Semeada com padrões **universais** de a11y/microcopy (tokens específicos do projeto entram ao instanciar).

## Abordagem geral

- **Mobile-first.** Base de referência ~375px. Especifique mobile primeiro.
- Breakpoints típicos: mobile 0–767 · tablet 768–1023 · desktop 1024+.
- Alvo de toque mínimo: **44×44px**. Altura mínima de botão no mobile: ~48px.

## Microcopy — padrões aprovados

### Tom de voz
- Direto e humano, sem jargão. O usuário-alvo costuma **não** ser técnico — evitar "API", "endpoint", "token" na UI principal.
- Exceção: telas que exigem o termo técnico (ex: configurar chave de API) — use com explicação curta.
- **Acentuação correta sempre** (ã, ç, í, é). Erro recorrente do engineer é omitir acentos — especifique os textos com acento.

### Mensagens de erro
- Estrutura: **[o que deu errado]. [o que fazer.]** Ex: "Não foi possível salvar. Tente novamente."
- Credenciais: nunca indique qual campo errou. "E-mail ou senha incorretos."
- Erro de serviço externo: mencione o serviço. "O [serviço] retornou um erro inesperado."

### Toasts
- Sucesso: auto-dismiss ~4s, `aria-live="polite"`.
- Erro: sem auto-dismiss (requer ação), `aria-live="assertive"`.
- Tempo relativo em listas ("há X horas"); timestamp completo só em histórico detalhado.

## Acessibilidade (WCAG 2.1 AA — piso mínimo)

- Contraste: 4.5:1 texto normal, 3:1 texto grande.
- Foco visível: outline 2px, offset 2px, nunca suprimido sem substituto.
- **Nunca comunique status só por cor** — sempre texto/ícone junto.
- Erros inline: `role="alert"` / `aria-live="assertive"` **e** `aria-describedby` no input apontando pro erro.
- Modais: foco entra ao abrir, retorna ao gatilho ao fechar, Escape fecha (a menos que loading bloqueie).
- Campos de senha/segredo: `type="password"`, `autocomplete="new-password"`, write-only.
- Navegação: `aria-current="page"` no item ativo; `role="tab"` + `aria-selected` em tabs.

## Padrões de componente que se repetem

- **ConfirmModal** para ações destrutivas: foco vai pro botão seguro (cancelar); confirmar é `destructive`; botões ≥44px; rótulos específicos ("Cancelar agendamento" / "Manter agendamento", não "Confirmar"/"Cancelar" genéricos).
- **EmptyState** redigido pra não gerar ansiedade — explique o que o sistema fará, não só "nada aqui".
- **Date/time picker**: `<input type="date">` + `<input type="time">` nativos no MVP (evita complexidade de a11y de calendário custom); `min` dinâmico; validação client + server.

## Segurança na UI

- Segredos/keys: nunca exibidos. Status apenas "Configurada" / "Não configurada". Após salvar, o valor nunca volta pela API.
- Separação de permissões reflete na UI: papéis sem acesso não veem o menu correspondente.

---

## Design system do projeto atual

_(Preencha ao instanciar: tokens de cor/espaçamento/tipografia, inventário de componentes e onde vivem.)_
