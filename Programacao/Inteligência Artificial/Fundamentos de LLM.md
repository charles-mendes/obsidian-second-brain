---
tags:
  - ia
---

# Fundamentos de LLM

Voltar para [[Mapa - Inteligência Artificial]] · Relacionado: [[Roadmap AI Engineer]]

Tópicos anotados de um vídeo:

![[Fundamentos de LLM - tópicos.png|300]]

## Tokens e context window

- **Token** é o pedaço de texto que o modelo processa — em média ~4 caracteres em inglês (em português, um pouco menos eficiente).
- Você paga por **tokens de entrada + tokens de saída**; saída costuma custar mais caro.
- **Context window** é o máximo de tokens que o modelo enxerga de uma vez (prompt + histórico + documentos + resposta). Passou do limite, o conteúdo mais antigo é cortado ou a chamada falha.
- Contexto maior não significa resposta melhor: informação irrelevante piora a qualidade e aumenta o custo.

## System prompts

- Instrução de "nível superior" que define papel, regras, tom e formato da resposta.
- Boas práticas: ser específico, dar exemplos do formato esperado, dizer o que **fazer** (não só o que evitar) e separar instruções de dados (ex.: com tags `<documento>...</documento>`).

## Structured outputs

- Forçar o modelo a responder em um formato fixo, normalmente **JSON validado por um schema**.
- Essencial quando a resposta vai ser lida por código, não por uma pessoa (ex.: a timeline JSON da [Plataforma de Edição de Vídeo por IA](obsidian://open?vault=Empreendedorismo&file=Plataforma%20de%20Edi%C3%A7%C3%A3o%20de%20V%C3%ADdeo%20por%20IA)).

## Embeddings

- Transformam texto em um **vetor de números** que representa o significado. Textos parecidos geram vetores próximos.
- Usos: busca semântica, recomendação, agrupamento, detecção de duplicados.
- Base do **RAG** (*Retrieval-Augmented Generation*): quebrar documentos em trechos → gerar embeddings → salvar em um banco vetorial → na pergunta, buscar os trechos mais próximos e colocá-los no prompt.

## Tool calling / function calling

- O modelo não executa nada sozinho; ele **pede** para chamar uma função que você descreveu (nome, descrição, parâmetros).
- Fluxo: modelo pede a ferramenta → seu código executa → resultado volta para o modelo → modelo responde.
- É o que permite **agentes**: buscar na web, consultar banco, criar arquivo, chamar uma API.

## Fine-tuning

- Treinar mais um pouco um modelo existente com exemplos seus para mudar **comportamento ou formato**.
- Ordem recomendada antes de pensar em fine-tuning: **1. prompt melhor → 2. exemplos no prompt (few-shot) → 3. RAG → 4. fine-tuning**.
- Fine-tuning é bom para estilo e formato consistentes; é ruim para "ensinar fatos novos" (para isso, use RAG).

## Mensurabilidade

Se você não mede, não sabe se uma mudança no prompt melhorou ou piorou o sistema.

| Métrica | O que medir |
| --- | --- |
| **Custo** | Tokens e valor por chamada, por usuário e por funcionalidade |
| **Output** | Formato válido? Latência? Taxa de erro? |
| **Qualidade** | A resposta está correta, completa e útil? |
| **Judges** | Usar um LLM como "juiz" (*LLM-as-a-judge*) para avaliar as respostas de outro em escala, com critérios definidos |

> [!tip] Evals na prática
> Monte um conjunto fixo de casos de teste (entrada + resposta esperada ou critérios). Rode sempre que mudar prompt, modelo ou pipeline, e compare a pontuação com a versão anterior — como testes automatizados de software.
