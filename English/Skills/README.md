# Skills de Inglês (Claude Code)

Duas skills do Claude Code para estudar inglês (nível A1-A2, falante de português).

| Skill | Comando | Para que serve |
|---|---|---|
| **english-drill** | `/english-drill [tópico] [nº]` | Exercícios de gramática, uma questão por vez, com correção em português, revisão espaçada e caderno de erros |
| **lang-tutor** | `/lang-tutor english portuguese beginner` | Modo tutor: corrige o seu inglês em cada mensagem enquanto você usa o Claude Code normalmente |

## Estrutura

```
Skills/
├── README.md
├── english-drill/
│   ├── SKILL.md                  # fluxo, formato de correção, alertas e regras (não muda sozinho)
│   └── references/
│       ├── falsos-cognatos.md    # tabela de falsos cognatos para brasileiro
│       └── topicos/              # um arquivo por tópico (regra, erros típicos, exemplos)
│           ├── present-past.md  irregular.md  story.md  do-does-did.md  articles.md
│           ├── be-have.md  plurals.md  prepositions.md  phrasal.md
│           └── continuous.md  future.md  past-continuous.md  questions.md  comparatives.md  pronouns.md
└── lang-tutor/
    ├── SKILL.md                  # ajustada para usar só o guia de inglês
    ├── LICENSE.md                # MIT, autor original: samhamilton
    └── languages/
        ├── _common.md            # formatos de feedback e tabelas de nível
        └── english.md            # guia específico de inglês
```

Dados do aluno (fora da skill, em `Análise de Inglês/`):

| Arquivo | O que guarda |
|---|---|
| `Caderno de Erros.md` | Histórico de sessões e cada erro com tag (`#p1`…), palavra-chave, caixa e data de revisão. Itens dominados vão para `## Dominados` |
| `Perfil de Erros.md` | Padrões de erro do aluno, contagens recalculadas a partir do caderno e candidatos a padrão novo |

## english-drill

Tópicos: `present-past`, `irregular`, `story`, `do-does-did`, `articles`, `be-have`, `plurals`, `prepositions`, `phrasal`, `continuous`, `future`, `past-continuous`, `questions`, `comparatives`, `pronouns`.
Modos: `mix`, `review`, `diary`.

```
/english-drill irregular 10
/english-drill mix
/english-drill review      # só os erros com revisão vencida
/english-drill diary       # escreva 3-5 frases sobre ontem e receba a correção completa
```

Como funciona:
- **Aquecimento**: toda sessão começa com até 2 erros vencidos do caderno.
- **Revisão espaçada em caixas**: acertou um item vencido → sobe de caixa (revisa em 1, 3, 7, 14 dias); depois da caixa 4 → dominado. Errou → volta para a caixa 1.
- **Dificuldade adaptativa**: 3 acertos seguidos sobe o nível; 2 erros seguidos desce e reexplica.
- **Alertas** em toda correção: tipo de verbo irregular (de família, só no passado, totalmente irregular) e falsos cognatos.
- No fim, a skill atualiza o Caderno e o Perfil (nunca o próprio `SKILL.md`).

## lang-tutor

```
/lang-tutor english portuguese beginner
```

Depois de ativado, vale para a sessão inteira. Escreva em inglês para receber correção; escreva em português para receber tradução e explicação.

Original: <https://github.com/hamsamilton/lang-tutor> (só os arquivos de inglês foram mantidos aqui).

## Instalação

O Claude Code lê as skills de `~/.claude/skills/`. Lá, `english-drill` e `lang-tutor` são **atalhos (junction)** para as pastas deste vault, então qualquer edição aqui vale na hora. Não renomeie nem mova esta pasta.

Para recriar os atalhos numa máquina nova (no Claude Code, com `!` na frente):

```
! powershell -NoProfile -Command '$v="C:\Users\Charles Mendes\Documents\English Obsidian\English\Skills"; $s="$env:USERPROFILE\.claude\skills"; foreach($n in "english-drill","lang-tutor"){ if(Test-Path "$s\$n"){ Remove-Item -Recurse -Force "$s\$n" }; New-Item -ItemType Junction -Path "$s\$n" -Target "$v\$n" }'
```
