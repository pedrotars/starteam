# Agent Memory — QA

> Padrões de qualidade e bugs recorrentes. Lido no início de cada acionamento.
> Semeada com **padrões de bug universais** (a regra de prevenção, não o log do bug específico).

## Higiene de memória

Guarde a **regra de prevenção reutilizável**, não o relato de cada bug de cada task (isso vive no task file). Pode periodicamente. (PLAYBOOK §9)

## Comandos do projeto

_(Preencha ao instanciar: testes, lint, format, typecheck, build, boot check, migrations.)_

## Padrões de bug universais (transcendem stack — verifique sempre)

### Segurança / auth
- **Timing side-channel no login:** `if user is None or not verify(...)` não chama o hash quando o user não existe → diferença de tempo detectável. Sempre chamar verify com hash dummy quando user é None.
- **Anti-enumeração com OR fraco:** `assert "a" not in msg or "b" not in msg` nunca pega vazamento de um só campo. Use **AND**.
- **Anti-leak por dict key vs. raw text:** verificar `"campo" not in dict` é mais fraco que `"<prefixo-do-segredo>" not in resp.text`. Em endpoints que servem dados sensíveis, cheque o raw text.
- **Secret default inseguro** (JWT_SECRET, senha de seed): guard que bloqueia em produção/staging quando o valor é o default.
- **Rota criada só para teste** indo pra produção: prefira fixture condicional por ambiente.

### Dados / banco
- **`updated_at` que nunca muda:** `server_default=now()` sem `onupdate`/trigger fica igual ao `created_at` pra sempre. Verifique em todo modelo que expõe `updated_at`. **(Follow-up cross-cutting clássico — costuma ser esquecido em alguns modelos.)**
- **Validação de input cru vs. normalizado:** valide o valor **pós-transformação** (trim/normalize), não o cru — senão entra lixo no banco.
- **Timestamp naive vs. timezone-aware** em bind param para coluna com timezone: comparação silenciosamente errada. Passe datetime com tzinfo explícito.
- **Recovery de worker com campo errado de tempo:** usar campo de negócio (`approved_at`) em vez de infra (`updated_at`) no `WHERE ... < now() - INTERVAL` quebra a janela de tolerância.

### Frontend
- **CSP `connect-src 'self'`** em SPA com backend separado bloqueia cross-origin em produção — torne dinâmico via env var.
- **Touch target < 44px** em nav/botões; **`aria-current` ausente** em nav; **`aria-describedby` ausente** em erro de form.
- **Estado restaurado errado no catch:** salve o estado anterior antes da ação async e restaure-o no catch, em vez de fixar um estado.
- **Contrato front/back desalinhado:** campos que o back não retorna chegam `undefined` em runtime. Audite o shape real do endpoint, não o tipo legado do front.

### Pipelines de texto / LLM
- **Guard de output vazio** após `.strip()`: `len > max` não pega `len == 0`. Sempre `if not out: return None`.

## Convenções de teste aceitas

- Testes de **comportamento**, não de implementação (status + body, não internals).
- `asyncio_mode = auto` (quando aplicável) — sem decorator em cada teste.
- Integração com **banco real** via Docker; fixture de migração `scope="session"`; nunca mockar banco.
- Mocks só para APIs externas. Evite `side_effect=[...]` por ordem de chamada (frágil) — prefira mock condicional.

## Áreas frágeis do projeto atual

_(Acumule aqui conforme revisar: pontos que merecem revisão extra a cada task.)_
