# Salvaguarda

Metodologia e biblioteca de templates para criar projetos novos dentro de
um workspace multi-projeto, cada um com sua própria configuração de
subagentes do Claude Code — sem depender de configuração compartilhada na
raiz do workspace.

## Por que isso existe

A abordagem centralizada (agentes definidos direto na raiz do workspace,
pensando em reaproveitá-los entre projetos) não funciona de forma
confiável: o Claude Code descobre `.claude/` a partir do diretório onde a
sessão é aberta (o cwd), **sem subir para pastas pai**. Assim, os agentes
só apareciam quando a sessão era aberta a partir da raiz do workspace —
abrir uma sessão direto dentro de um projeto (o fluxo mais natural quando
você já está trabalhando nele) fazia os agentes sumirem silenciosamente.

A solução: cada projeto recebe sua **própria cópia materializada** dos
agentes (`<projeto>/.claude/agents/*.md`, `<projeto>/.claude/settings.json`,
`<projeto>/AGENTS.md`, `<projeto>/CLAUDE.md`), copiada a partir dos
templates aqui em `salvaguarda/templates/`. Cada projeto fica
autossuficiente, funciona não importa de onde a sessão é aberta, e pode
inclusive ser publicado com essa configuração dentro do próprio
repositório (não há nada nos templates que mencione o workspace ou outros
projetos — são genéricos de propósito).

## Trade-off aceito: duplicação

Melhorar um agente aqui em `salvaguarda/templates/` **não atualiza**
projetos já criados — cada um já tem sua cópia própria, que só muda se você
editá-la diretamente naquele projeto ou recopiar o template manualmente.
Isso é proposital: os templates são a fonte canônica para **novos**
projetos, não um link vivo. Mesmo modelo de qualquer scaffold/gerador
(`create-react-app`, `dotnet new`, etc.).

## Estrutura

```
salvaguarda/
  README.md              <- este arquivo
  templates/
    AGENTS.md             <- fluxo de agentes + regra pedagógica (genérico)
    CLAUDE.md              <- aponta para AGENTS.md
    settings.json           <- permissões de bash
    agents/
      techlead.md
      backend.md
      frontend.md
      devops.md
      testes.md
```

## Como criar um projeto novo

1. Criar a pasta do projeto na raiz do workspace: `<workspace>/<nome-do-projeto>/`.
2. Copiar `salvaguarda/templates/AGENTS.md` e `CLAUDE.md` para a raiz do
   projeto.
3. Copiar `salvaguarda/templates/settings.json` para
   `<projeto>/.claude/settings.json`.
4. Copiar `salvaguarda/templates/agents/*.md` para
   `<projeto>/.claude/agents/`.
5. Ajustar os agentes copiados para as particularidades reais do projeto
   (stack específica, convenções de nomenclatura já decididas, gotchas já
   descobertos) — os templates são o ponto de partida, não o resultado
   final. Ao aprender algo específico do projeto que vale preservar (ex.:
   um bug de uma lib que sempre se repete), registre no `.md` do agente
   responsável ou em um `PADROES.md` na pasta relevante.
6. `git init` dentro da pasta do projeto — cada projeto é seu próprio
   repositório, independente e publicável.

## Workflow do dia a dia

1. **Projeto novo** → seguir "Como criar um projeto novo" acima.
   **Projeto existente** → abrir a sessão direto dentro dele; os agentes já
   estão materializados ali, sem depender desta pasta.
2. **Delegação**: mudança estrutural (nova entidade, novo padrão, decisão
   de arquitetura) passa pelo `techlead` primeiro, que documenta o ADR.
   Implementação já decidida vai direto pro agente da camada (`backend`,
   `frontend`, `devops`, `testes`).
3. **Registrar no momento em que aparece**: um gotcha de lib, uma
   convenção que vale repetir, um trade-off não óbvio — registre assim que
   descobrir, no `PADROES.md` da pasta ou no `.md` do agente responsável.
   Não depender de lembrar depois nem deixar só numa resposta de texto que
   some da conversa.
4. **Backport pro template** (o passo mais fácil de esquecer): se o
   aprendizado do passo 3 é específico daquele projeto, fica só lá. Se é
   algo que qualquer projeto futuro também enfrentaria (ex.: um bug de uma
   lib usada por padrão, uma convenção de stack), traga de volta pra
   `salvaguarda/templates/` — é o único jeito dos templates melhorarem com
   o tempo, já que a cópia em cada projeto não se atualiza sozinha (ver
   "Trade-off aceito: duplicação" acima). Sem esse passo, o mesmo erro se
   repete no próximo projeto.

## Padrão de stack já validado nesta metodologia

- **Backend**: C#/.NET, Clean Architecture (Domain/Application/
  Infrastructure/Api), xUnit + FluentAssertions + Moq (unitários),
  Testcontainers (integração), Docker.
- **Frontend**: Vite + React + TypeScript, Tailwind CSS v4, shadcn/ui (CLI
  real), React Hook Form + Zod, TanStack Query.

Esses são os padrões *default* dos templates `backend.md`/`frontend.md` —
mudam apenas se o `techlead` do projeto documentar uma decisão diferente em
ADR.
