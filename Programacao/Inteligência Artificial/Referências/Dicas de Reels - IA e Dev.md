---
tags:
  - referencia
  - ia
  - tooling
---

# Dicas de Reels - IA e Dev

Voltar para [[Mapa - Inteligência Artificial]]

Ferramentas e dicas de IA e programação tiradas dos reels salvos na coleção *evoluir* do Instagram. Lista completa: [Reels Salvos - Evoluir](obsidian://open?vault=Empreendedorismo&file=Reels%20Salvos%20-%20Evoluir).

## Economizar tokens e dar contexto ao agente

[@matheusgomes](https://www.instagram.com/reel/DbzUERLMRtz/) mostra o Claude Code rodando prompts por mais de 20 horas e indica quatro repositórios. Basta buscar o nome + "GitHub" e pedir ao Claude Code para instalar:

| Repositório | O que faz (segundo o vídeo) |
| --- | --- |
| **Caveman** | Comprime os prompts: no exemplo, de 69 para 19 tokens |
| **RTK** | Reduz o consumo de tokens (o autor fala em até 90%) |
| **Ponytail** | Mais uma opção de economia de tokens; o autor recomenda comparar os três no seu repositório |
| **Graphify** | Gera um **grafo do sistema** (arquivos, funções, imports/exports, classes e onde cada função é usada). Assim o agente sabe o que mais é afetado quando muda um arquivo e não quebra quem depende dele |

> [!tip] Relação com o vault
> O Graphify resolve o mesmo problema de *context engineering* descrito em [[Agentic Engineering]] e [[Ferramentas de Análise de Código]].

## MCP na prática

- **Conectar um MCP no Claude** ([@techysabrina](https://www.instagram.com/reel/DdsisRfhwgG/)): *Customize → Connectors → Add custom connector* e colar a URL do servidor MCP (no exemplo, o da Higgsfield, para gerar vídeos). Ver [[MCP - Model Context Protocol]].
- **Agentes que substituem um time de vendas ou produzem vídeos** ([@oalanicolas](https://www.instagram.com/reel/C31Chp5iHBO/)): ver [Ideias de Negócio e Renda Extra](obsidian://open?vault=Empreendedorismo&file=Ideias%20de%20Neg%C3%B3cio%20e%20Renda%20Extra).

## Onde estudar de graça

- **7 canais de IA no YouTube** ([@danielsocrates.seo](https://www.instagram.com/reel/DeytvUURIoz/)), separados por momento: começando no Claude Code, além do básico, agentes de IA para o negócio, como a IA funciona por dentro e SEO para Google e ChatGPT. Os nomes só são enviados para quem comenta "CANAIS". Dica do vídeo: em vídeos em inglês, use *engrenagem → Legendas → Traduzir automaticamente → Português*. "O YouTube não é rede social, é buscador."
- **Escola Virtual de Governo (EV.G)** ([@jornadatop](https://www.instagram.com/reel/CzEZJlxOA78/)): mais de 540 cursos gratuitos **com certificado**, incluindo programação, IA, marketing, direito digital, Libras e liderança.
- **Internet Archive** ([@wilson_fernando](https://www.instagram.com/reel/CwxL1vurFbx/)): biblioteca sem fins lucrativos com livros (ex.: *O Poder do Agora*, *As Armas da Persuasão*, *A Arte da Guerra*) e o histórico de mais de 800 bilhões de páginas da web.
- **Desafios de lógica usados por empresas como Meta e Uber** ([@pasquadev](https://www.instagram.com/reel/CyMX-djgBoa/)): permite resolver em qualquer linguagem, comparar soluções e fazer testes técnicos de empresas. O nome do site não ficou claro no áudio; vale conferir no vídeo.

## Carreira

- **Teste de carga antes de produção** ([@ogustavojacob](https://www.instagram.com/reel/DdFIA3WMNCj/), anúncio de pós-graduação): um sistema que caiu com 300 usuários simultâneos porque não teve teste de carga, de arquitetura (ex.: ArchUnit), testes de integração com Testcontainers nem benchmark de performance. "Na minha máquina funciona" não vale. Ver [[Testes e CI-CD com IA]].
- **Vaga internacional** ([@coderscompany](https://www.instagram.com/reel/De0D7u1AMGC/), anúncio): devs plenos e seniores podem ganhar 3 a 5 vezes mais trabalhando de casa para fora. As barreiras costumam ser o inglês e a síndrome do impostor, não a parte técnica.
- **Opinião:** o futuro do front-end é depender de menos bibliotecas JavaScript, e quem não aprender back-end vai ficar para trás. — [@manodeyvin](https://www.instagram.com/reel/DUIhS5Jji6o/)

## Segurança

- Alerta sobre ataques em massa usando IA: há vazamento de dados pessoais em vários países, então desconfie de onde você coloca seus dados. — [@piratimnha](https://www.instagram.com/reel/De1c8-PxA4I/) (vídeo sensacionalista, sem fontes)
