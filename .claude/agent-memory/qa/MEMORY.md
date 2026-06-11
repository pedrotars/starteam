# Agent Memory — QA

> Padrões de qualidade e bugs recorrentes. Lido no início de cada acionamento.
> Semeada com **padrões de bug universais** (a regra de prevenção, não o log do bug específico),
> destilados de dezenas de reviews reais ao longo de 5 épicos.

## Higiene de memória

Guarde a **regra de prevenção reutilizável**, não o relato de cada bug de cada task (isso vive no task file). Pode periodicamente. (PLAYBOOK §9)

## Comandos do projeto

_(Preencha ao instanciar: testes, lint, format, typecheck, build, boot check, migrations.)_

## Padrões de bug universais (transcendem stack — verifique sempre)

### Segurança / auth
- **Timing side-channel no login:** `if user is None or not verify(...)` não chama o hash quando o user não existe → diferença de tempo detectável. Sempre chamar verify com hash dummy quando user é None.
- **Anti-enumeração com OR fraco:** `assert "a" not in msg or "b" not in msg` nunca pega vazamento de um só campo. Use **AND**.
- **Anti-leak por dict key vs. raw text:** verificar `"campo" not in dict` é mais fraco que `"<prefixo-do-segredo>" not in resp.text`. Em endpoints que servem dados sensíveis, cheque o raw text.
- **Secret default inseguro** (JWT_SECRET, senha de seed): guard que bloqueia em produção/staging quando o valor é o default. Scripts de seed com senha default hardcoded criam admin com senha pública quando a env var é esquecida.
- **Rota criada só para teste** indo pra produção: prefira fixture condicional por ambiente.
- **Download externo sem guard de tamanho:** toda busca de mídia/arquivo externo precisa de `len(bytes) > MAX` **e** de estar dentro do try com timeout — sem isso é resource exhaustion + crash não tratado.
- **Rate limit checado mas não respeitado:** ler o header `x-rate-limit-remaining` sem dar `break` no loop é log-only. A proteção real exige interromper o processamento quando `remaining < N`.

### Dados / banco
- **`updated_at` que nunca muda:** `server_default=now()` sem `onupdate`/trigger fica igual ao `created_at` pra sempre. Verifique em todo modelo que expõe `updated_at`. **(Follow-up cross-cutting clássico — costuma ser esquecido em alguns modelos.)**
- **Validação de input cru vs. normalizado:** valide o valor **pós-transformação** (trim/normalize), não o cru — senão entra lixo no banco.
- **Timestamp naive vs. timezone-aware** em bind param para coluna com timezone: comparação silenciosamente errada. Passe datetime com tzinfo explícito.
- **Recovery de worker com campo errado de tempo:** usar campo de negócio (`approved_at`) em vez de infra (`updated_at`) no `WHERE ... < now() - INTERVAL` quebra a janela de tolerância.
- **Overfetch desnecessário:** `LIMIT 20` + `rows[:5]` no código quando a query já está ordenada — alinhe o LIMIT com a necessidade real.

### Frontend
- **CSP `connect-src 'self'`** em SPA com backend separado bloqueia cross-origin em produção — torne dinâmico via env var. **Generalize:** o CSP precisa liberar **cada tipo de recurso externo** que o app carrega (`img-src`, `media-src`, `connect-src`) — revisar o CSP sempre que uma feature passar a carregar um novo tipo de mídia/origem.
- **Touch target < 44px** em nav/botões; **`aria-current` ausente** em nav; **`aria-describedby` ausente** em erro de form.
- **Estado restaurado errado no catch:** salve o estado anterior antes da ação async e restaure-o no catch, em vez de fixar um estado.
- **Contrato front/back desalinhado:** campos que o back não retorna chegam `undefined` em runtime. Audite o shape real do endpoint, não o tipo legado do front.
- **CA de exibição sem dado no endpoint:** quando a spec exige exibir um dado na listagem, verifique se o endpoint de listagem **realmente retorna** esse dado — senão o CA depende de mudança no backend.
- **Validação que gateia só um caminho de ação:** se um estado inválido desabilita o botão primário ("Salvar") mas não os alternativos ("Agendar", "Enviar depois"), o backend recebe o inválido pelo caminho secundário e o erro chega tarde. Toda condição de bloqueio vale para **todos** os botões de submissão.
- **Mensagem de erro divergente do limite real:** a mensagem (422, hint de contador) precisa refletir o limite **efetivamente aplicado** pelo código — quando a spec muda o limite, a mensagem costuma ficar para trás.
- **`setState` durante render** (derived state sync no corpo do componente): anti-pattern que causa render duplo e warnings — derive via `useEffect` com a dependência correta.
- **Input sem `maxLength` quando há limite:** desabilitar o botão não basta — o campo deve impedir/indicar o excesso no momento da digitação (maxLength + contador + erro inline `role="alert"`).

### Pipelines de texto / LLM
- **Guard de output vazio** após `.strip()`: `len > max` não pega `len == 0`. Sempre `if not out: return None`.
- **Persistência de relações sem teste:** quando o CA exige vincular registros (ex: fontes de um item gerado), exija ao menos um teste que asserte a criação das instâncias da relação — sem ele o CA é invisível nos testes.

## Convenções de teste aceitas

- Testes de **comportamento**, não de implementação (status + body, não internals).
- `asyncio_mode = auto` (quando aplicável) — sem decorator em cada teste.
- Integração com **banco real** via Docker; fixture de migração `scope="session"`; nunca mockar banco.
- Mocks só para APIs externas. Evite `side_effect=[...]` por ordem de chamada (frágil) — prefira mock condicional.
- **Testes ocos são reprovação:** assert dentro de bloco condicional (`if x: assert ...`) passa vazio quando a condição é falsa. Asserts incondicionais sempre.
- **Invariante de duas operações** ("cancelado não executa") exige teste que roda as duas em sequência — dois testes isolados não cobrem.
- **Teardown na ordem dos FKs** (filhos antes de pais) — passa local, quebra no CI.
- **`TZ=UTC` no script de teste** + horários com margem — data local sem `Z` é interpretada no fuso da máquina.
- **Sem path absoluto** em config de teste (`path.resolve(__dirname)`) — quebra em CI.
- **Env var em fixture:** `monkeypatch.setenv` / context manager — nunca `os.environ.setdefault` (muta o processo sem cleanup).
- **Lint completo:** `lint` e `format --check` são alvos distintos — rode os dois (ou o alvo `make lint` completo).
- **UI com portais** (Dialog/Sheet/Popover): use `screen.*` (busca no `document.body`), não `container.querySelector`.

## Áreas frágeis do projeto atual

_(Acumule aqui conforme revisar: pontos que merecem revisão extra a cada task.)_
