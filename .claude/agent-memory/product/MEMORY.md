# Product Agent — Memória Persistente

> Padrões e métodos acumulados. Lido no início de cada acionamento.
> Semeada com **métodos** genéricos validados em 5 épicos reais (sem personas/domínio específico de projeto).

## Persona recorrente

### Dono do produto (admin)
- _(Preencha ao instanciar: nível técnico, preferências de fluxo, restrições conhecidas.)_
- Exemplo: perfil não-técnico → passo-a-passo claro, sem jargão; restrição de custo rígida → trate budget como hard constraint.

_(Adicione as personas específicas do projeto ao instanciar.)_

## Padrões de Critério de Aceite aceitos pelo time

- CAs de segurança especificam o que é visível na UI ("nunca aparece em texto plano") **e** o que é verificável no banco ("armazenado encriptado").
- CAs de fluxo com ramificação cobrem **todos** os caminhos.
- CAs de cadência/tempo incluem **tolerância numérica** (ex: "± 5 minutos").
- CAs de comportamento de IA são verificáveis por **inspeção de artefato**, não por resultado subjetivo.
- **CAs de exibição declaram a fonte do dado:** se o CA manda mostrar um dado numa lista, o contrato do endpoint de listagem precisa retorná-lo — sinalize a dependência para o `architect`.
- **CAs que tocam back e front** explicitam os dois lados (ex: "a opção some do formulário **e** os registros existentes são migrados").

## Método: múltiplos caminhos de processamento

Quando o produto tem **mais de um pipeline/caminho** (ex: categorias A e B processadas de formas diferentes), a spec mapeia **explicitamente qual entrada vai para qual caminho** — deixar implícito gera ambiguidade no `architect` e divergência na implementação.

## Método: hipóteses

Toda suposição sobre o usuário/mercado vira `H-NN` com: hipótese · status · como validar / decisão · épico. Quando o usuário resolve uma hipótese por decisão explícita:
1. No task file: troque `> **Hipótese H-XX:**` por `> **H-XX — Decidida (D-YY, data):**` com a decisão e consequências.
2. Na persona afetada: atualize o texto inline.
3. Na MEMORY: marque status "Decidida" + a decisão.
4. **Não remova** a hipótese — registre como decidida, para rastreabilidade.

## Padrões de Fora de Escopo úteis

- Distinguir agendar (futuro) de executar agora — features diferentes.
- Analytics/métricas de engajamento costumam ser pós-MVP em projetos de conteúdo.
- Distinguir multi-conta (da plataforma externa) de multi-usuário (do painel).
- Distinguir **geração** automática de um artefato (entra) de **edição/regeneração** pelo painel (frequentemente pós-escopo) — explicite qual dos dois está no épico.

## Métricas que funcionaram

- Custo mensal como hard constraint com threshold de alerta + threshold de investigação obrigatória.
- Proxy de qualidade por taxa de aprovação sem edição substantiva (quando há loop humano de aprovação).
- **Métricas por segmento/categoria** (ex: taxa de rejeição por categoria) revelam problemas que a média global esconde — defina o alvo por segmento quando houver segmentação.
