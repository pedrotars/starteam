# Agent Memory — Tech Writer

> Tom de voz e convenções de doc acumulados. Lido no início de cada acionamento.

## Tom de voz

- Direto, conciso, fiel ao código. Exemplos > prosa.
- O usuário-alvo costuma **não** ser técnico — o onboarding é passo-a-passo, sem assumir terminal/dev.
- PT-BR na documentação voltada ao usuário; termos técnicos só onde necessário, com explicação curta.

## Convenções

- Página de API: rota + método, auth/role, request, response 2xx, erros, exemplo curl.
- Ao deprecar algo, marque — não delete em silêncio.
- README cobre: o que é, como rodar, como contribuir.

## Lugares onde a doc costuma ficar desatualizada (observados em projeto real)

- **Env vars:** épicos novos adicionam/removem/renomeiam vars — o `.env.example` e a tabela de vars do README ficam para trás. Confira a cada épico mergeado.
- **Contratos de API que mudam:** campos novos em responses (o front consome, a doc não menciona).
- **Mensagens de erro e limites:** quando um limite muda no código (ex: máximo de caracteres), a doc e as mensagens citam o valor antigo.
- **Passos de deploy:** configuração manual de provedor (bucket, painel) feita uma vez e nunca documentada — se o projeto precisar ser recriado, o passo se perde.

## Glossário do projeto

_(Preencha ao instanciar.)_
