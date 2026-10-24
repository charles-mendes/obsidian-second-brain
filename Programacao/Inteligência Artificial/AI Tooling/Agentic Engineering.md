---
tags:
  - ia
  - tooling
---

# Agentic Engineering

Voltar para [[AI Tooling - Visão Geral]] · Relacionado: [[AI Coding Assistants]] · [[Spec-Driven Development]]

**Agentic engineering** é a disciplina de **projetar o ambiente e o processo** em que agentes de IA trabalham: que contexto recebem, que ferramentas podem usar, como são verificados e onde o humano entra. Não é "escrever prompts melhores" — é engenharia de sistema, onde o agente é um componente.

> [!info] Mudança de papel
> Você passa de **quem escreve cada linha** para **quem define o problema, prepara o ambiente, revisa e decide**. Parecido com liderar um time: o resultado depende de quão claras são as instruções e de quão bons são os mecanismos de verificação.

## O que é um agente

Um agente é um LLM rodando em **loop** com acesso a ferramentas:

```
enquanto não terminou:
    pensar no próximo passo
    chamar uma ferramenta (ler, buscar, editar, rodar teste, consultar MCP)
    observar o resultado
```

Três ingredientes determinam a qualidade: **modelo** (você escolhe pouco), **contexto** (você controla muito) e **ferramentas/verificação** (você controla muito).

## Context engineering

A habilidade mais importante. A context window é finita e **o que entra nela define o que sai**.

| Fonte de contexto | Quando é carregada | Exemplo |
| --- | --- | --- |
| **Rules / memória** | Sempre | `CLAUDE.md`, `AGENTS.md`, `.cursor/rules` |
| **Skills** | Quando a tarefa combina com a descrição da skill | Skill de "criar migration Flyway" |
| **Specs** | Quando você referencia | `openspec/changes/add-checkout/` |
| **Código** | O agente busca sob demanda | Classes relevantes |
| **MCP** | Quando o agente chama a tool | Issue do GitHub, schema do banco |
| **Histórico** | Acumula na sessão | Tudo que já foi dito e lido |

Princípios:

- **Menos e mais relevante** > mais. Contexto irrelevante piora a resposta (ver [[Fundamentos de LLM#Tokens e context window]]).
- **Carregamento progressivo**: regras curtas sempre; detalhes só quando necessários (é para isso que skills existem).
- **Uma tarefa por sessão**; comece do zero quando mudar de assunto.
- **Escreva para o agente o que você escreveria para um dev novo no time** — sem o óbvio, com as armadilhas.

## Os blocos de construção

### Rules (memória do projeto)

Instruções que valem para **todas** as tarefas. Ver o exemplo de `CLAUDE.md` em [[AI Coding Assistants#Exemplo de CLAUDE.md para Spring Boot + React]].

No Cursor, regras em `.cursor/rules/*.mdc` podem ser aplicadas só a certos arquivos:

```markdown
---
description: Convenções de componentes React
globs: web/src/**/*.tsx
alwaysApply: false
---

- Componentes funcionais com props tipadas (`type Props = {...}`).
- Estado de servidor com React Query; nunca `useEffect` + `fetch`.
- Testes com Testing Library ao lado do componente (`Componente.test.tsx`).
```

### Skills

Uma **skill** é uma pasta com um `SKILL.md` (instruções) e, opcionalmente, scripts e arquivos de referência. O agente vê só o **nome e a descrição** de cada skill; o conteúdo completo é carregado **apenas quando a tarefa combina** — por isso dá para ter muitas skills sem lotar o contexto.

`.claude/skills/novo-endpoint-spring/SKILL.md`:

```markdown
---
name: novo-endpoint-spring
description: Cria um endpoint REST no backend Spring Boot seguindo o padrão do projeto. Use quando o usuário pedir um novo endpoint, rota ou API no backend.
---

# Novo endpoint Spring Boot

1. Leia `pedido/PedidoController.java` e `pedido/PedidoService.java` como referência de padrão.
2. Crie DTOs de entrada/saída como `record` no pacote da feature, com Bean Validation.
3. Controller → Service → Repository. Regras de negócio só no Service.
4. Erros de negócio: lance exceções de `com.loja.shared.exception`; o `GlobalExceptionHandler` converte para ProblemDetail.
5. Testes obrigatórios:
   - `@WebMvcTest` cobrindo sucesso, validação (400) e erros de negócio.
   - Teste unitário do Service com Mockito.
6. Se precisar de tabela nova, crie migration Flyway `V<próximo número>__descricao.sql`.
7. Rode `./mvnw verify` e só finalize com tudo verde.
```

Ideias de skills para seu contexto: revisar PR Java, criar migration Flyway, criar componente React com teste, investigar erro do Sentry, atualizar dependência do Spring Boot.

> [!tip] A descrição é o gatilho
> O agente decide usar a skill pela **descrição**. Escreva o que ela faz **e quando usar** ("Use quando...").

### Comandos personalizados (slash commands)

Prompts reutilizáveis que **você** dispara. Em `.claude/commands/revisar.md`:

```markdown
Revise as mudanças não commitadas (`git diff`) como um engenheiro Java sênior:
- Bugs e casos de borda não tratados
- Violações das regras do CLAUDE.md
- Testes faltando ou fracos
Liste os problemas por severidade. Não edite nada.
```

Uso: `/revisar`. Diferença para skill: **comando você chama**; **skill o agente escolhe** sozinho.

### Hooks

Scripts que rodam **automaticamente** em eventos do agente (antes/depois de usar uma ferramenta, ao terminar, ao receber um prompt). Diferente de uma regra, um hook **sempre** executa — é uma garantia, não um pedido.

`.claude/settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "./mvnw -q spotless:apply" }
        ]
      }
    ]
  }
}
```

Usos comuns:

| Evento | Uso |
| --- | --- |
| Depois de editar (`PostToolUse`) | Formatar o código, rodar lint no arquivo |
| Antes de rodar comando (`PreToolUse`) | Bloquear comandos perigosos (`rm -rf`, `git push --force`, acesso a `.env`) |
| Ao terminar (`Stop`) | Rodar testes; se falharem, o agente continua corrigindo |
| Notificação | Avisar no desktop/Slack quando o agente precisa de você |

### Subagentes

Agentes especializados que o agente principal chama, cada um com **contexto próprio**, instruções e ferramentas limitadas. Vantagens: não poluem o contexto principal e podem rodar em paralelo.

`.claude/agents/revisor-seguranca.md`:

```markdown
---
name: revisor-seguranca
description: Revisa mudanças em busca de problemas de segurança (injeção, autenticação, exposição de dados). Use após implementar endpoints ou mudanças de autenticação.
tools: Read, Grep, Glob
---

Você é um especialista em segurança de aplicações Spring Boot.
Analise o diff e reporte: SQL/JPQL injection, endpoints sem autorização,
dados sensíveis em logs ou respostas, validação de entrada ausente.
Não edite arquivos; apenas reporte com arquivo e linha.
```

Repare em `tools: Read, Grep, Glob`: o revisor **não pode editar** — princípio do menor privilégio.

## Resumo: qual bloco usar

| Preciso que... | Use |
| --- | --- |
| O agente saiba sempre as regras do projeto | Rules (`CLAUDE.md` / `AGENTS.md`) |
| O agente saiba fazer uma tarefa específica quando ela aparecer | Skill |
| Eu dispare um prompt padronizado | Slash command |
| Algo aconteça **sempre**, sem depender do modelo | Hook |
| Uma tarefa rode isolada, com papel e permissões próprios | Subagente |
| O agente acesse um sistema externo | MCP |
| O agente saiba o que construir | Spec ([[Spec-Driven Development]]) |
| O agente saiba se acertou | Testes, tipos, lint, CI ([[Testes e CI-CD com IA]]) |

## Padrões de workflow

| Padrão | Como funciona | Quando usar |
| --- | --- | --- |
| **Explorar → Planejar → Implementar → Verificar** | Plan mode primeiro, depois execução | Padrão para quase tudo |
| **TDD com agente** | Testes primeiro, depois implementação | Regras de negócio claras |
| **Agente escritor + agente revisor** | Um implementa, outro (contexto limpo) revisa | Código crítico |
| **Agentes em paralelo** | Várias tarefas independentes, cada uma em sua branch/worktree (`git worktree`) ou agente na nuvem | Backlog de tarefas pequenas |
| **Headless / CI** | `claude -p "..."` em scripts e pipelines | Tarefas recorrentes (triagem de issues, atualização de docs) |

## Human in the loop: onde você não pode sair

- **Aprovar a spec e o plano** — decisões de produto e arquitetura são suas.
- **Revisar o diff** antes do merge — responsabilidade pelo código continua sua.
- **Ações irreversíveis**: deploy, migrations em produção, apagar dados, `push --force`.
- **Permissões**: comece restrito (agente pede confirmação) e libere aos poucos os comandos seguros (`./mvnw test`, `npm test`).

## Como medir se está funcionando

Como em [[Fundamentos de LLM#Mensurabilidade]], sem medir você não sabe se melhorou.

- % de tarefas que o agente conclui **sem retrabalho**.
- Tempo do pedido até o PR aprovado.
- Comentários de review por PR gerado por agente.
- Bugs em produção originados de código gerado.
- Custo (tokens/assinatura) por tarefa.

Quando o agente erra algo repetidamente, **não corrija só o código — corrija o sistema**: adicione uma regra, uma skill, um teste ou um hook para que o erro não se repita.

## Exercícios

1. Crie a skill `novo-endpoint-spring` adaptada ao seu projeto e teste pedindo "cria um endpoint para listar categorias".
2. Configure um hook que formata o código após cada edição.
3. Crie o subagente `revisor-seguranca` e peça para ele revisar a última feature.
4. Rode duas tarefas em paralelo usando `git worktree` e compare o tempo com fazê-las em sequência.
