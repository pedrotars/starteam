# Product Agent — Memória Persistente

> Padrões e métodos acumulados. Lido no início de cada acionamento.
> Semeada com **métodos** genéricos (sem personas/domínio específico de projeto).

## Persona recorrente

### Pedro (admin / dono do produto)
- Perfil não-técnico: não opera terminal, não quer gerenciar infra.
- Prefere fluxos rápidos e diretos (ver, agir, confirmar — sem burocracia).
- Restrição de custo costuma ser rígida — trate budget como hard constraint com threshold de alerta.

_(Adicione as personas específicas do projeto ao instanciar.)_

## Padrões de Critério de Aceite aceitos pelo time

- CAs de segurança especificam o que é visível na UI ("nunca aparece em texto plano") **e** o que é verificável no banco ("armazenado encriptado").
- CAs de fluxo com ramificação cobrem **todos** os caminhos.
- CAs de cadência/tempo incluem **tolerância numérica** (ex: "± 5 minutos").
- CAs de comportamento de IA são verificáveis por **inspeção de artefato**, não por resultado subjetivo.

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

## Métricas que funcionaram

- Custo mensal como hard constraint com threshold de alerta + threshold de investigação obrigatória.
- Proxy de qualidade por taxa de aprovação sem edição substantiva (quando há loop humano de aprovação).
