# Fluxo de Agentes deste Projeto

Este projeto usa subagentes do Claude Code por especialidade, definidos em
`.claude/agents/`.

## Agentes disponíveis

- `techlead` — decide arquitetura e trade-offs estruturais, documenta ADRs
  curtos. Consulte antes de mudanças que atravessem camadas, criem uma nova
  entidade de domínio ou introduzam um novo padrão.
- `backend` — implementa C#/ASP.NET Core seguindo Clean Architecture
  (Domain/Application/Infrastructure/Api).
- `frontend` — implementa Angular/TypeScript (Signals, Tailwind, spartan/ui,
  Reactive Forms + Zod).
- `devops` — Dockerfile, docker-compose, variáveis de ambiente, execução.
- `testes` — testes e estratégia de testes, no escopo definido abaixo
  (xUnit no backend, Jasmine/Karma no frontend, conforme aplicável).

Sem `orquestrador` dedicado: sem delegação explícita, a própria thread
principal já atua como orquestrador e segue este fluxo diretamente.

Nenhum desses agentes é uma instância única/singleton — são papéis
(templates de prompt), não processos persistentes. Nada impede delegar o
**mesmo agente mais de uma vez em paralelo** quando as tarefas são
independentes entre si (ex.: dois endpoints de backend sem dependência um
do outro, ou back e front da mesma feature ao mesmo tempo já com o
contrato de Api combinado). Só serialize quando há dependência real (ex.:
o endpoint B usa uma entidade que o endpoint A ainda vai criar) — não por
achar que "só existe 1 `backend`".

## Escopo de testes deste projeto

`<preencher na criação do projeto — ver salvaguarda/README.md, "Como criar
um projeto novo">`: **backend** | **frontend** | **full** | **nenhum**.

Se "nenhum", o agente `testes` não deveria estar presente em
`.claude/agents/` neste projeto.

## Fluxo obrigatório

1. Delimitar objetivo, critérios de aceite e escopo antes de implementar.
2. Para mudanças estruturais (nova entidade, novo padrão, decisão de
   arquitetura, trade-off relevante), delegar primeiro ao `techlead`.
3. Para implementação já decidida, delegar direto a `backend`, `frontend`,
   `devops` ou `testes`, conforme a camada — em paralelo quando as tarefas
   forem independentes (ver nota acima).
4. Nenhuma implementação é aceita sem teste proporcional ao risco e ao
   escopo de testes definido acima.
5. Revisão de código usa as skills nativas `/code-review` e `/simplify` —
   não há agente `revisor` dedicado (escopo pequeno demais para justificar).

## Regra pedagógica

O objetivo deste projeto não é só entregar código — é o usuário aprender as
tecnologias e as decisões por trás delas. Mas o peso dessa regra varia por
agente, não é uniforme:

- **`techlead` — obrigatória e é o entregável em si**: toda decisão
  arquitetural vem com alternativas reais consideradas, critério de
  escolha explícito e trade-off (o que se ganha, o que se perde), tanto no
  ADR quanto na resposta ao usuário. Não é opcional aqui — é o motivo do
  agente existir.
- **`backend`, `frontend`, `devops`, `testes` (agentes de implementação) —
  leve, por exceção, não por obrigação genérica**: não pare a execução pra
  narrar toda escolha da tarefa (isso tende a virar ruído que o próprio
  agente aprende a ignorar sob pressão de terminar a tarefa concreta). Mas
  se algo realmente saltar aos olhos no caminho — um comportamento
  inesperado de lib, um motivo não óbvio por trás de uma convenção já
  estabelecida, um bug encontrado sem querer — comente na resposta, não só
  no código/commit. Um agente de implementação pode ter uma razão
  project-specific pra pesar mais essa regra (ex.: o `frontend.md` deste
  projeto, porque o usuário está aprendendo Angular do zero) — quando isso
  acontecer, fica documentado no `.md` do próprio agente, não aqui.

Quando qualquer agente explicar algo, prefira explicações curtas e
concretas (2-5 frases) ancoradas no código que acabou de escrever, em vez
de teoria solta. Não deixe pra explicar tudo de uma vez só no resumo
final.

## Convenção de nomenclatura (idioma)

Entidades e enums de domínio têm nome em PT-BR; repositórios, serviços,
DTOs e controllers mantêm o sufixo técnico em inglês com o nome de domínio
traduzido (ex.: `IPedidoRepository`, `PedidoService`, `PedidoDto`,
`PedidosController` para uma entidade `Pedido`). Propriedades/campos e o
contrato JSON da Api continuam em inglês — só o "substantivo" do domínio é
traduzido, o padrão arquitetural (`Repository`/`Service`/`DTO`/
`Controller`) permanece reconhecível pra qualquer dev .NET. Documente a
entidade e suas invariantes num ADR do `techlead` quando ela for criada.

## Docker como ponto de partida, não polimento final

O projeto deve rodar via `docker compose up` desde cedo — ideal é isso
funcionar logo depois do esqueleto inicial compilar, antes mesmo de todas
as features estarem prontas, não como uma etapa final de "empacotamento".
Isso importa por dois motivos: (1) ambiente de desenvolvimento e ambiente
de produção ficam parecidos desde o início, evitando o clássico "funciona
na minha máquina"; (2) adiar Docker pro final costuma esconder problema de
configuração (variável de ambiente esquecida, path relativo que só
funciona fora de container) até já ter muito código em cima — mais caro de
corrigir do que se aparecesse no dia 1. Ver `devops.md` para o que isso
significa na prática.

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
