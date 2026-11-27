# Skills de Inglês (Claude Code)

Duas skills do Claude Code para estudar inglês (nível A1-A2, falante de português).

| Skill | Comando | Para que serve |
|---|---|---|
| **english-drill** | `/english-drill [tópico] [nº]` | Exercícios de gramática, uma questão por vez, com correção em português e caderno de erros |
| **lang-tutor** | `/lang-tutor english portuguese beginner` | Modo tutor: corrige o seu inglês em cada mensagem enquanto você usa o Claude Code normalmente |

## Estrutura

```
Skills/
├── README.md
├── english-drill/
│   └── SKILL.md              # skill própria (criada em 2026-11-22)
└── lang-tutor/
    ├── SKILL.md              # ajustada para usar só o guia de inglês
    ├── LICENSE.md            # MIT, autor original: samhamilton
    └── languages/
        ├── _common.md        # formatos de feedback e tabelas de nível
        └── english.md        # guia específico de inglês
```

## english-drill

Tópicos: `present-past`, `irregular`, `story`, `do-does-did`, `articles`, `prepositions`, `mix`, `review`.

A skill tem um **perfil de erros** (seção 0 do `SKILL.md`) montado a partir do caderno: os padrões que mais se repetem entram em todas as sessões. Um erro só é marcado como dominado depois de 2 acertos em dias diferentes. No fim de cada sessão a própria skill atualiza essa tabela (contagens e padrões novos).

```
/english-drill irregular 10
/english-drill mix
/english-drill review      # revisa os erros abertos do caderno
```

Os erros ficam salvos em `Análise de Inglês/Caderno de Erros.md`.

## lang-tutor

```
/lang-tutor english portuguese beginner
```

Depois de ativado, vale para a sessão inteira. Escreva em inglês para receber correção; escreva em português para receber tradução e explicação.

Original: <https://github.com/hamsamilton/lang-tutor> (só os arquivos de inglês foram mantidos aqui).

## Instalação

O Claude Code lê as skills de `~/.claude/skills/`. Esta pasta é o backup versionado no vault. Para instalar ou atualizar:

```bash
cp -r Skills/english-drill Skills/lang-tutor ~/.claude/skills/
```
