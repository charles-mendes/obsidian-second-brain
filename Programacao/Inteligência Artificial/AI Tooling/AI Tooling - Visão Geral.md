---
tags:
  - ia
  - tooling
---

# AI Tooling - Visão Geral

Voltar para [[Mapa - Inteligência Artificial]] · Relacionado: [[Roadmap AI Engineer]] · [[Fundamentos de LLM]]

**AI Tooling** é o conjunto de ferramentas, integrações e automações que você usa para desenvolver software **com** IA — com mais produtividade, qualidade e autonomia. O ponto central: o modelo sozinho é só um gerador de texto. Quem transforma isso em um processo confiável de engenharia é o *tooling* ao redor dele.

> [!info] AI Tooling × AI Engineering
> - **AI Engineering** ([[Roadmap AI Engineer]]) = construir **produtos** que usam IA (RAG, agentes no produto, evals).
> - **AI Tooling** (esta trilha) = usar IA para **construir software** melhor e mais rápido (agentes de código, MCP, specs, CI).
>
> As duas se reforçam: os conceitos de [[Fundamentos de LLM]] (contexto, tool calling, evals) valem para ambas.

## As camadas do tooling

Pense no tooling como camadas. Cada uma resolve um problema diferente do agente de código.

| # | Camada | Problema que resolve | Ferramentas | Nota |
| --- | --- | --- | --- | --- |
| 1 | **AI Coding Assistants** | Quem lê o código, raciocina e edita | Claude Code, Cursor, GitHub Copilot | [[AI Coding Assistants]] |
| 2 | **MCP** | Como o agente acessa sistemas externos (GitHub, banco, docs, Sentry) | Servidores MCP | [[MCP - Model Context Protocol]] |
| 3 | **Spec-driven development** | O agente sabe **o quê** construir antes de construir | OpenSpec, Spec Kit, Kiro | [[Spec-Driven Development]] |
| 4 | **Testes e CI/CD** | Como saber se o que o agente fez está certo | JUnit, Testcontainers, Jest/Vitest, GitHub Actions, SonarQube | [[Testes e CI-CD com IA]] |
| 5 | **Análise de código** | Como achar e mudar padrões em escala, com precisão | ripgrep, ast-grep, Semgrep, ESLint, OpenRewrite, ArchUnit | [[Ferramentas de Análise de Código]] |
| 6 | **Agentic engineering** | Como orquestrar tudo: regras, skills, hooks, subagentes, workflows | CLAUDE.md, AGENTS.md, skills, hooks | [[Agentic Engineering]] |

Exemplo ponta a ponta com Java/Spring Boot + React: [[Workflow Prático - API Spring Boot com IA]].

## A ideia mais importante: o loop de feedback

Um agente de código é tão bom quanto os **sinais de verificação** que ele recebe. O ciclo é sempre:

```
Contexto (spec, regras, código) → Agente planeja → Agente edita → Verificação (compilador, testes, lint, CI) → Agente corrige → ... → Revisão humana
```

- Sem **contexto**, o agente inventa (alucina APIs, ignora padrões do projeto).
- Sem **verificação**, você não sabe se ele acertou — e vira revisor de código gerado às cegas.
- Quanto mais rápido e automático o sinal de verificação, mais autonomia você pode dar ao agente.

> [!tip] Regra prática
> Antes de pedir mais autonomia para o agente, pergunte: *"Como ele vai saber sozinho que errou?"* Se a resposta for "não vai", invista primeiro em testes, tipos, lint e CI.

## Tooling, skills, agentes, workflow e fine-tuning

| Termo | Significado | Exemplo |
| --- | --- | --- |
| **Tooling** | Conjunto de ferramentas e integrações | Claude Code + MCP + testes + GitHub Actions |
| **Tool** (ferramenta do agente) | Uma função que o modelo pode pedir para executar | `read_file`, `run_tests`, `query_database` |
| **Skill** | Instruções reutilizáveis (e scripts opcionais) para uma tarefa | Skill para revisar código Java seguindo o padrão do time |
| **Rules / memória do projeto** | Instruções sempre carregadas sobre o projeto | `CLAUDE.md`, `AGENTS.md`, `.cursor/rules` |
| **Agent** | Sistema que raciocina em loop e executa etapas usando tools | Agente que investiga um bug, reproduz, corrige e roda os testes |
| **Subagent** | Agente especializado chamado por outro, com contexto próprio | Subagente "revisor de segurança" |
| **Hook** | Script executado automaticamente em um evento do agente | Rodar o formatter após cada edição |
| **Workflow** | Sequência organizada de etapas | Spec → plano → implementação → testes → PR → review |
| **Fine-tuning** | Ajustar os pesos de um modelo com exemplos | Treinar um modelo para um formato de saída fixo |

> [!warning] Erro comum
> Achar que precisa de fine-tuning para o agente "entender seu projeto". Quase sempre o que falta é **contexto** (rules, specs, docs via MCP) e **verificação** (testes). Fine-tuning é a última opção — ver [[Fundamentos de LLM#Fine-tuning]].

## Níveis de maturidade

Use como régua para saber onde você está e o próximo passo.

| Nível | Como você trabalha | Próximo passo |
| --- | --- | --- |
| 0 | Copia e cola código do ChatGPT | Usar um assistente dentro do editor |
| 1 | Autocomplete e chat no editor | Usar modo agente para tarefas pequenas |
| 2 | Agente edita vários arquivos, você revisa tudo | Criar `CLAUDE.md`/`AGENTS.md` e exigir que ele rode os testes |
| 3 | Agente com regras do projeto + testes + MCP | Escrever specs antes de implementar |
| 4 | Spec → tarefas → implementação → CI verde → PR | Skills, hooks e subagentes para padronizar |
| 5 | Agentes rodando em paralelo/na nuvem, abrindo PRs que você revisa | Medir: taxa de PR aceito, retrabalho, bugs em produção |

## Plano de estudo sugerido

| Semana | Tema | Prática |
| --- | --- | --- |
| 1 | [[AI Coding Assistants]] | Criar o `CLAUDE.md`/`AGENTS.md` de um projeto Spring Boot seu e implementar um endpoint com o agente |
| 2 | [[Testes e CI-CD com IA]] | Garantir `./mvnw verify` e `npm test` rodando, e criar um workflow do GitHub Actions |
| 3 | [[MCP - Model Context Protocol]] | Conectar MCP do GitHub e de um Postgres local **somente leitura** |
| 4 | [[Spec-Driven Development]] | Fazer uma feature inteira com OpenSpec ou Spec Kit |
| 5 | [[Ferramentas de Análise de Código]] | Fazer uma refatoração em massa com ast-grep ou OpenRewrite |
| 6 | [[Agentic Engineering]] | Criar 1 skill, 1 hook e 1 subagente para seu projeto |
| 7 | [[Workflow Prático - API Spring Boot com IA]] | Juntar tudo em uma feature real, do requisito ao PR |

## Recomendação

Para evoluir como engenheiro de software na era da IA, estude principalmente **AI tooling**, **agentic engineering** e **spec-driven development**. Esses conhecimentos ajudam a projetar um **processo de desenvolvimento confiável**, e não apenas a escrever prompts melhores. Os fundamentos de engenharia (testes, arquitetura, revisão de código) ficam **mais** importantes, não menos: eles são o que permite confiar no que o agente produz.
