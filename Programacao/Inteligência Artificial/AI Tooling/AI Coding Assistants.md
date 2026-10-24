---
tags:
  - ia
  - tooling
---

# AI Coding Assistants

Voltar para [[AI Tooling - Visão Geral]] · Relacionado: [[Agentic Engineering]] · [[MCP - Model Context Protocol]]

Ferramentas que **entendem o código, implementam funcionalidades e corrigem bugs**. Por baixo, todas fazem a mesma coisa: um LLM com acesso a *tools* (ler arquivo, buscar, editar, rodar comando) rodando em loop — o mesmo mecanismo de [[Fundamentos de LLM#Tool calling / function calling|tool calling]].

## Os três modos de uso

| Modo | O que faz | Quando usar |
| --- | --- | --- |
| **Autocomplete** | Sugere a próxima linha/bloco enquanto você digita | Código repetitivo, boilerplate, testes parecidos |
| **Chat** | Responde perguntas sobre o código, sugere trechos | Entender código legado, tirar dúvidas, explorar ideias |
| **Agente** | Lê o repositório, planeja, edita vários arquivos, roda comandos e testes | Implementar features, corrigir bugs, refatorar |

O salto de produtividade real está no **modo agente** — e é nele que o resto do tooling (specs, testes, MCP) faz diferença.

## Principais ferramentas

| Ferramenta | Onde roda | Destaques | Arquivo de regras do projeto |
| --- | --- | --- | --- |
| **Claude Code** | Terminal (CLI), extensão de VS Code/JetBrains, GitHub Actions | Agente forte em tarefas longas; skills, hooks, subagentes, modo headless (`claude -p`) | `CLAUDE.md` |
| **Cursor** | Editor próprio (fork do VS Code) e CLI | Autocomplete muito bom, modo agente e plan, agentes na nuvem, Bugbot para review de PR | `.cursor/rules/*.mdc` e `AGENTS.md` |
| **GitHub Copilot** | VS Code, JetBrains, GitHub.com | Integração nativa com GitHub; *coding agent* que recebe uma issue e abre um PR | `.github/copilot-instructions.md` e `AGENTS.md` |
| Outros | — | OpenAI Codex, Gemini CLI, Windsurf, JetBrains Junie, Aider | Geralmente `AGENTS.md` |

> [!tip] `AGENTS.md`
> `AGENTS.md` está virando o padrão aberto de "README para agentes", lido por várias ferramentas. Uma estratégia simples: escrever o conteúdo em `AGENTS.md` e, no `CLAUDE.md`, apenas referenciá-lo (ou manter os dois iguais).

## Como um agente de código funciona por dentro

1. **Recebe** seu pedido + system prompt da ferramenta + arquivo de regras (`CLAUDE.md` etc.).
2. **Explora**: busca arquivos (grep/glob), lê classes relevantes.
3. **Planeja**: decide quais arquivos mudar (no *plan mode*, ele para aqui e mostra o plano).
4. **Edita**: aplica mudanças nos arquivos.
5. **Verifica**: roda compilador, testes, lint.
6. **Corrige** com base nos erros e repete até terminar.

O limite principal é a **context window**: tudo que ele lê, os outputs de comandos e o histórico ocupam espaço. Sessões muito longas degradam a qualidade — por isso existem comandos como `/clear` e `/compact` no Claude Code, e subagentes que exploram em contexto separado.

## Claude Code na prática

| Recurso | Para que serve |
| --- | --- |
| `/init` | Gera um `CLAUDE.md` inicial analisando o repositório |
| `CLAUDE.md` | Memória do projeto: comandos, arquitetura, convenções (carregado sempre) |
| **Plan mode** (`Shift+Tab`) | O agente só analisa e propõe um plano, sem editar |
| `/clear`, `/compact` | Limpar ou resumir o contexto da sessão |
| `.claude/commands/*.md` | Comandos personalizados (`/revisar-pr`, `/novo-endpoint`) |
| `.claude/skills/` | Skills — ver [[Agentic Engineering#Skills]] |
| `.claude/agents/` | Subagentes especializados |
| `.claude/settings.json` | Permissões (o que pode rodar sem pedir) e hooks |
| `claude mcp add ...` | Conectar servidores MCP — ver [[MCP - Model Context Protocol]] |
| `claude -p "..."` | Modo headless, para scripts e CI |

### Exemplo de CLAUDE.md para Spring Boot + React

```markdown
# Projeto: Loja

## Stack
- Backend: Java 21, Spring Boot 3, Spring Data JPA, PostgreSQL, Flyway
- Frontend: React 18 + TypeScript + Vite, React Query, Vitest + Testing Library

## Comandos
- Build e testes backend: `./mvnw verify`
- Apenas um teste: `./mvnw test -Dtest=PedidoServiceTest`
- Frontend: `cd web && npm run lint && npm test`

## Arquitetura (backend)
- Pacotes por feature: `com.loja.pedido`, `com.loja.produto`
- Camadas: `Controller` → `Service` → `Repository`. Controller nunca acessa Repository.
- DTOs são `record`s; nunca expor entidades JPA na API.
- Validação com Bean Validation (`@Valid`, `@NotNull`) nos DTOs de entrada.
- Erros tratados no `GlobalExceptionHandler` (`@RestControllerAdvice`), formato ProblemDetail.

## Regras
- Toda feature nova precisa de teste: unitário no Service e `@WebMvcTest` no Controller.
- Testes de integração com Testcontainers (não usar H2).
- Migrations novas em `src/main/resources/db/migration`, nunca editar migrations existentes.
- Rode `./mvnw verify` antes de dizer que terminou.
```

> [!tip] O que colocar no arquivo de regras
> Coisas que o agente **não descobre sozinho** ou erra com frequência: comandos exatos, decisões de arquitetura, "nunca faça X". Não precisa explicar o que Spring Boot é. Mantenha curto — cada linha consome contexto em toda sessão.

## Como pedir bem para um agente

Um bom pedido tem **objetivo, contexto, restrições e critério de pronto**.

> [!example] Pedido fraco
> "Cria um endpoint de pedidos."

> [!example] Pedido bom
> "Implemente `POST /api/pedidos` no módulo `pedido`.
> - Entrada: `clienteId`, lista de itens (`produtoId`, `quantidade` > 0).
> - Regras: produto inexistente → 404; estoque insuficiente → 409 com ProblemDetail.
> - Siga o padrão de `ProdutoController` e `ProdutoService`.
> - Crie testes `@WebMvcTest` para o controller e unitários para o service.
> - Pronto quando `./mvnw verify` passar. Mostre o plano antes de editar."

Boas práticas:

- **Aponte exemplos** do próprio repositório ("siga o padrão de X"). Isso vale mais que descrever o padrão.
- **Peça o plano primeiro** em tarefas médias/grandes (plan mode).
- **Tarefas pequenas e verificáveis** > uma tarefa gigante.
- **Uma sessão por tarefa**: contexto limpo dá resultado melhor.
- **Revise o diff**, não só o resultado. Você continua responsável pelo código.
- Se o agente errou duas vezes a mesma coisa, **pare, limpe o contexto e reformule** — ou adicione a regra no `CLAUDE.md`.

## Onde os agentes erram mais

| Problema | Como mitigar |
| --- | --- |
| Inventa APIs/métodos que não existem | Tipos + compilador + testes; MCP de documentação (ex.: Context7) |
| Ignora padrões do projeto | Regras no `CLAUDE.md` + apontar arquivo de exemplo |
| "Corrige" o teste em vez do código | Regra explícita: "nunca altere testes existentes para fazê-los passar" |
| Mudanças grandes demais | Pedir plano, limitar escopo, commits pequenos |
| Diz que terminou sem testar | Critério de pronto explícito + hook que roda testes ([[Agentic Engineering#Hooks]]) |
| Segurança (segredos, comandos destrutivos) | Permissões restritas, nunca dar credenciais de produção |

## Exercícios

1. Rode `/init` em um projeto seu e **edite** o `CLAUDE.md` gerado com as regras reais do time.
2. Peça para o agente implementar um endpoint REST simples usando o modelo de "pedido bom" acima. Compare com o "pedido fraco".
3. Use o plan mode para uma refatoração e avalie o plano antes de aprovar.
