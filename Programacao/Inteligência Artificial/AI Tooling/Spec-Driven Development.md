---
tags:
  - ia
  - tooling
---

# Spec-Driven Development

Voltar para [[AI Tooling - Visão Geral]] · Relacionado: [[Agentic Engineering]] · [[Workflow Prático - API Spring Boot com IA]]

**Spec-driven development (SDD)** é escrever **especificação → plano → tarefas** antes de deixar o agente implementar. A spec vira a fonte da verdade: o agente implementa a partir dela, e você revisa contra ela.

> [!info] Por que isso importa com IA
> O agente é ótimo em **como** fazer, mas péssimo em adivinhar **o quê** você quer. Prompts soltos ("faz o checkout") deixam decisões importantes implícitas — e o agente decide por você, muitas vezes errado. A spec força essas decisões a acontecerem **antes**, por escrito, onde são baratas de mudar.

## Vibe coding × spec-driven

| | Vibe coding | Spec-driven |
| --- | --- | --- |
| Entrada | Prompt solto | Requisitos + critérios de aceitação |
| Decisões | Implícitas, tomadas pelo agente | Explícitas, revisadas por você |
| Revisão | "Parece funcionar?" | "Atende a cada cenário da spec?" |
| Bom para | Protótipos, scripts descartáveis | Features reais, código que vai para produção |
| Histórico | Perdido no chat | Versionado no repositório junto com o código |

## As etapas

1. **Specify** — *o quê e por quê*: problema, usuários, requisitos, cenários de aceitação. Sem falar de tecnologia.
2. **Plan** — *como*: arquitetura, stack, modelo de dados, contratos de API, riscos.
3. **Tasks** — quebrar o plano em tarefas pequenas, ordenadas e verificáveis.
4. **Implement** — o agente executa as tarefas uma a uma, rodando testes.
5. **Verify / Archive** — conferir contra a spec e atualizar a documentação viva do sistema.

> [!tip] Você revisa em cada etapa
> O ganho está em **revisar a spec e o plano** (poucos minutos de leitura) em vez de revisar 2.000 linhas de código que resolvem o problema errado.

## Como escrever bons requisitos

Use cenários no formato **Dado / Quando / Então** (Given/When/Then), que viram testes diretamente.

```markdown
### Requisito: Finalizar pedido
O sistema DEVE permitir que um cliente autenticado finalize um pedido com itens no carrinho.

#### Cenário: Checkout com sucesso
- DADO um carrinho com 2 itens disponíveis em estoque
- QUANDO o cliente envia POST /api/checkout com um endereço válido
- ENTÃO o pedido é criado com status AGUARDANDO_PAGAMENTO
- E o estoque dos itens é reservado
- E a resposta é 201 com o ID do pedido

#### Cenário: Estoque insuficiente
- DADO um carrinho com um item cujo estoque é 0
- QUANDO o cliente finaliza o pedido
- ENTÃO a resposta é 409 com ProblemDetail indicando o produto
- E nenhum estoque é reservado
```

Boas práticas:

- **DEVE / NÃO DEVE** (MUST / SHALL) deixam claro o que é obrigatório.
- **Cenários de erro** são os mais importantes — é onde o agente mais inventa.
- **Fora de escopo** explícito ("não inclui cupom de desconto") evita que o agente faça a mais.
- Marque dúvidas como `[PRECISA ESCLARECER]` em vez de chutar.

## OpenSpec

[OpenSpec](https://github.com/Fission-AI/OpenSpec) é uma ferramenta leve, focada em **mudanças em projetos existentes** (*brownfield*). Funciona com Claude Code, Cursor, Copilot e outros.

```bash
npm install -g @fission-ai/openspec
openspec init      # cria a pasta openspec/ e configura os comandos no seu agente
```

Estrutura:

```
openspec/
├── project.md                 # contexto do projeto (stack, convenções)
├── specs/                     # FONTE DA VERDADE: como o sistema funciona hoje
│   └── checkout/spec.md
└── changes/                   # propostas de mudança em andamento
    └── add-checkout-pix/
        ├── proposal.md        # por que e o que muda
        ├── design.md          # decisões técnicas (opcional)
        ├── tasks.md           # checklist de implementação
        └── specs/checkout/spec.md   # DELTA: requisitos adicionados/alterados/removidos
```

O conceito central é o **delta**: a mudança descreve só o que muda, com seções `## ADDED Requirements`, `## MODIFIED Requirements` e `## REMOVED Requirements`. Ao terminar, você **arquiva** a mudança e o delta é aplicado em `openspec/specs/`, que continua refletindo o sistema real.

Fluxo:

1. **Propor**: no agente, peça para criar uma proposta (ex.: `/openspec:proposal Adicionar pagamento via PIX no checkout`). Ele gera `proposal.md`, `tasks.md` e os deltas.
2. **Revisar e ajustar** a proposta até concordar. `openspec validate <change>` checa o formato.
3. **Aplicar**: `/openspec:apply` — o agente implementa as tarefas e marca o checklist.
4. **Arquivar**: `/openspec:archive` ou `openspec archive <change>` — atualiza as specs.

> [!note] Comandos mudam entre versões
> O nome dos comandos de barra muda com as versões do OpenSpec. Depois do `openspec init`, confira os comandos instalados no seu agente e `openspec --help`.

## Spec Kit (GitHub)

[Spec Kit](https://github.com/github/spec-kit) é o toolkit do GitHub para SDD. É mais estruturado e completo, ótimo para **features novas grandes** ou projetos do zero.

```bash
uvx --from git+https://github.com/github/spec-kit.git specify init meu-projeto
```

Comandos no agente:

| Comando | Gera | O que faz |
| --- | --- | --- |
| `/speckit.constitution` | `.specify/memory/constitution.md` | Princípios do projeto (ex.: "testes antes do código", "sem dependências novas sem justificativa") |
| `/speckit.specify` | `specs/001-checkout/spec.md` | Requisitos e histórias de usuário, sem tecnologia |
| `/speckit.clarify` | atualiza a spec | O agente faz perguntas sobre pontos ambíguos |
| `/speckit.plan` | `plan.md`, `data-model.md`, `contracts/`, `research.md` | Plano técnico com a stack que você informar |
| `/speckit.tasks` | `tasks.md` | Tarefas ordenadas, marcando as que podem rodar em paralelo |
| `/speckit.analyze` | relatório | Checa consistência entre spec, plano e tarefas |
| `/speckit.implement` | código | Executa as tarefas |

## Kiro e outros

- **Kiro** (AWS): IDE com SDD embutido; gera `requirements.md` (no formato EARS: *"WHEN ... THE SYSTEM SHALL ..."*), `design.md` e `tasks.md`.
- **Plan mode** do Claude Code/Cursor: uma versão "leve" de SDD — o plano existe, mas normalmente não fica versionado.
- **Sem ferramenta nenhuma**: um `docs/specs/checkout.md` escrito à mão e passado ao agente já captura boa parte do benefício.

## OpenSpec × Spec Kit

| | OpenSpec | Spec Kit |
| --- | --- | --- |
| Foco | Mudanças em sistemas existentes | Features novas / projetos do zero |
| Peso | Leve, poucos arquivos | Mais etapas e artefatos |
| Ideia central | Deltas sobre specs vivas | Constituição + spec → plano → tarefas |
| Instalação | npm | uv (Python) |
| Quando escolher | Dia a dia em um projeto Spring Boot já existente | Feature grande, time querendo processo bem definido |

## Armadilhas

- **Spec gigante** que ninguém lê: prefira várias mudanças pequenas.
- **Spec que vira documentação morta**: se o código mudar sem a spec, ela perde valor. Por isso o OpenSpec arquiva deltas.
- **Pular a revisão**: gerar a spec com IA e aprovar sem ler elimina todo o benefício.
- **Burocracia para tarefa trivial**: corrigir um typo não precisa de spec.

## Exercícios

1. Escreva à mão a spec do checkout acima com mais 3 cenários de erro (cliente não autenticado, carrinho vazio, endereço inválido).
2. Instale o OpenSpec em um projeto Spring Boot seu e faça uma mudança pequena do início ao arquivamento.
3. Faça a mesma feature com Spec Kit e compare: qual gerou um plano mais útil? Qual foi mais rápido?
