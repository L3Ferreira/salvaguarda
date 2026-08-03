# Fluxo de Agentes deste Projeto

Este projeto usa subagentes do Claude Code por especialidade, definidos em
`.claude/agents/`.

## Agentes disponíveis

- `techlead` — decide arquitetura e trade-offs estruturais, documenta ADRs
  curtos. Consulte antes de mudanças que atravessem camadas, criem uma nova
  entidade de domínio ou introduzam um novo padrão.
- `backend` — implementa C#/ASP.NET Core seguindo Clean Architecture
  (Domain/Application/Infrastructure/Api).
- `frontend` — implementa React/TypeScript (Tailwind, shadcn/ui, React
  Hook Form + Zod, TanStack Query).
- `devops` — Dockerfile, docker-compose, variáveis de ambiente, execução.
- `testes` — testes xUnit (unitários e integração) e estratégia de testes.

Sem `orquestrador` dedicado: sem delegação explícita, a própria thread
principal já atua como orquestrador e segue este fluxo diretamente.

## Fluxo obrigatório

1. Delimitar objetivo, critérios de aceite e escopo antes de implementar.
2. Para mudanças estruturais (nova entidade, novo padrão, decisão de
   arquitetura, trade-off relevante), delegar primeiro ao `techlead`.
3. Para implementação já decidida, delegar direto a `backend`, `frontend`,
   `devops` ou `testes`, conforme a camada.
4. Nenhuma implementação é aceita sem teste proporcional ao risco.
5. Revisão de código usa as skills nativas `/code-review` e `/simplify` —
   não há agente `revisor` dedicado (escopo pequeno demais para justificar).

## Regra pedagógica (obrigatória para todos os agentes)

O objetivo deste projeto não é só entregar código — é o usuário aprender as
tecnologias e as decisões por trás delas. Por isso, **todo agente deve
explicar o raciocínio e os trade-offs das decisões não triviais que
tomar**, diretamente na resposta (não só código): por que essa abordagem e
não outra, quais alternativas existiam, o que se perde e o que se ganha.
Prefira explicações curtas e concretas (2-5 frases) ancoradas no código que
acabou de escrever, em vez de teoria solta.

Isso vale mesmo em tarefas que parecem "só mecânicas" (renomear, mover
arquivo, ajustar config) — se algo interessante ou não óbvio aparecer no
caminho (um comportamento inesperado de uma lib, um motivo real por trás de
uma convenção, um bug encontrado sem querer), comente na resposta, não só
no commit/no código. Não deixe pra explicar tudo de uma vez só no resumo
final.

## Regras gerais

- Prefira a menor mudança correta; não invente compatibilidade retroativa
  nem features não pedidas.
- Commit e push sempre exigem pedido explícito do usuário, mesmo que o
  comando esteja liberado em `.claude/settings.json`.
- Ao estabelecer um padrão que deva se repetir, registre em um
  `PADROES.md` **na própria pasta onde o padrão se aplica** (ex.:
  `src/components/PADROES.md` para convenção de componente,
  `src/schemas/PADROES.md` para convenção de validação,
  `src/Domain/PADROES.md` para convenção de agregado) — um arquivo por
  pasta que estabelece uma convenção, não um documento único e centralizado
  na raiz. Cada `PADROES.md` deve ter: o padrão em si, um exemplo mínimo, e
  por que essa convenção existe (não só o "o quê", também o "porquê").
  Padrões amplos (que afetam múltiplas camadas) passam pelo `techlead`
  antes de serem formalizados.
