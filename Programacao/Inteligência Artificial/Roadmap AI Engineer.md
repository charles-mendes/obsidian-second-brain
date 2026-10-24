---
tags:
  - ia
---

# Roadmap AI Engineer

Voltar para [[Mapa - Inteligência Artificial]] · Relacionado: [[Fundamentos de LLM]]

**AI Engineer** é o engenheiro de software que constrói produtos usando modelos de IA prontos (LLMs, modelos de embedding, de voz, de imagem). Diferente do cientista de dados ou do pesquisador de ML, o foco não é treinar modelos do zero, e sim **integrar, avaliar e colocar em produção**.

Tópicos anotados de um vídeo:

![[Roadmap AI Engineer - tópicos.png|300]]

## Trilha

| # | Tópico | O que significa | Como praticar |
| --- | --- | --- | --- |
| 1 | **Software Engineer** | A base continua sendo engenharia de software: APIs, banco de dados, testes, arquitetura | Você já tem essa base como desenvolvedor |
| 2 | **Python (ou TypeScript, Go)** | Python domina o ecossistema de IA; TS e Go são ótimos para o backend do produto | Fazer um script que chama a API de um LLM e trata a resposta |
| 3 | **Deploy e produção** | Colocar o sistema no ar com filas, workers, cache, limites de custo e de taxa | Subir uma API com fila (ex.: BullMQ, Celery) que processa chamadas ao LLM |
| 4 | **Observabilidade** | Registrar prompts, respostas, latência, tokens e custo de cada chamada | Usar uma ferramenta de tracing de LLM (Langfuse, LangSmith, Sentry AI Monitoring) |
| 5 | **Cliente** | Entender o problema do usuário e transformar em requisito técnico | Entrevistar usuários antes de construir (ver [Competências para Construir uma Empresa de Milhões](obsidian://open?vault=Emprendedorismo&file=Compet%C3%AAncias%20para%20Construir%20uma%20Empresa%20de%20Milh%C3%B5es)) |
| 6 | **RAG, vector DB e embeddings** | Dar ao modelo acesso a dados próprios buscando trechos relevantes antes de responder | Criar um chat que responde perguntas sobre as notas deste vault |
| 7 | **Evals** | Testes automatizados para saber se o sistema responde bem | Montar um conjunto de 30 perguntas com respostas esperadas e medir a taxa de acerto |
| 8 | **Qualidade** | Reduzir alucinação, garantir formato e consistência | Usar structured outputs e validação de schema |
| 9 | **Workflows de IA e agentes** | Encadear várias chamadas, ferramentas e decisões | Um agente que pesquisa, resume e salva o resultado |
| 10 | **Fine-tuning e modelos** | Ajustar um modelo para uma tarefa específica quando prompt e RAG não bastam | Fine-tuning de um modelo pequeno para um formato de saída fixo |

Os conceitos técnicos de cada item estão explicados em [[Fundamentos de LLM]].

## Projeto para consolidar

A [Plataforma de Edição de Vídeo por IA](obsidian://open?vault=Emprendedorismo&file=Plataforma%20de%20Edi%C3%A7%C3%A3o%20de%20V%C3%ADdeo%20por%20IA) cobre quase toda a trilha: transcrição (Whisper), LLM para decisões de corte, filas e workers em produção, score de confiança (qualidade), métricas de correção manual (evals) e agentes de estilo (Fase 4).
