
## Visão Geral

Produto SaaS que recebe vídeo, áudio ou texto e devolve um vídeo editado automaticamente (cortes, zoom, legendas, B-roll, música), com um roadmap incremental de 4 fases. Este plano detalha **o que construir, em que ordem, com quais critérios de saída em cada fase**.

---

## Fase 0 — Validação e Fundação (2–3 semanas)

Antes de codar qualquer pipeline de IA, validar o problema e montar a base técnica.

### Objetivos

- Confirmar que existe demanda real (não só "seria legal")
- Montar infraestrutura mínima que suporte todas as fases seguintes

### Tarefas

1. **Validação de mercado**
    
    - Entrevistar 10–15 criadores de conteúdo (YouTube/TikTok) sobre dor atual com edição
    - Mapear concorrência direta (CapCut, Opus Clip, Descript, Captions, Vizard) — o que cada um faz bem/mal
    - Definir ICP (perfil de cliente ideal): criador solo? agência? empresa?
2. **Fundação técnica**
    
    - Repositório monorepo (Turborepo ou Nx) com frontend + backend + workers separados
    - Setup PostgreSQL (schema inicial: users, projects, jobs, assets)
    - Setup Redis + BullMQ (fila de processamento)
    - Setup S3 ou Cloudflare R2 (upload/storage de vídeo)
    - Pipeline de CI/CD básico (deploy automático em staging)
    - Definir custo estimado por minuto de vídeo processado (Whisper + LLM + TTS + render) — isso vai orientar precificação desde o início

### Critério de saída

- Infraestrutura de upload → fila → worker "vazio" funcionando ponta a ponta (mesmo sem IA ainda)
- Modelo de custo por vídeo documentado

---

## Fase 1 — MVP: Corte de Silêncio + Legendas (4–6 semanas)

O núcleo mínimo que já entrega valor sozinho.

### Escopo

- Upload de vídeo (mp4)
- Transcrição via Whisper
- Detecção e remoção automática de silêncios/pausas longas
- Geração de legendas estilizadas (timestamp preciso)
- Exportação em MP4 (16:9 e 9:16)
- **Tela de revisão**: usuário vê a timeline gerada e pode desfazer cortes específicos antes de exportar (isso é diferencial — não deixar para depois)

### Stack desta fase

- Frontend: Next.js + Tailwind, player de vídeo com timeline editável
- Backend: NestJS/Express, endpoints de upload e status de job
- Worker: processa fila, chama Whisper, roda FFmpeg
- Render: FFmpeg puro (sem Remotion ainda — manter simples)

### Tarefas técnicas

1. Endpoint de upload com progresso (chunked upload para arquivos grandes)
2. Worker de transcrição (Whisper via API ou self-hosted)
3. Algoritmo de detecção de silêncio (threshold configurável, ex: >500ms)
4. Geração de timeline JSON (cortes propostos) — formato interno reutilizável nas próximas fases
5. UI de revisão: lista de cortes com play/desfazer por corte
6. Renderização final via FFmpeg a partir da timeline aprovada
7. Sistema de notificação (job concluído) — websocket ou polling

### Critério de saída

- Usuário sobe vídeo → recebe vídeo cortado com legendas em menos de X minutos (definir SLA, ex: 2x a duração do vídeo)
- Testar com 20–30 usuários beta reais, medir: tempo de processamento, taxa de correção manual, NPS

---

## Fase 2 — Zoom Automático, Jump Cuts e Templates (6–8 semanas)

### Escopo

- Detecção facial (OpenCV/MediaPipe) para zoom dinâmico
- Jump cuts automáticos em repetições e muletas de fala ("é...", "hum...", "tipo...")
- Templates de estilo de edição (ex: "podcast", "tutorial", "vlog") com presets de zoom/corte/legenda
- Exportação com música de fundo (biblioteca licenciada integrada)

### Tarefas técnicas

1. Pipeline de detecção facial frame a frame → coordenadas de centro + score de expressão
2. Regra de decisão de zoom (ex: zoom in se expressão > threshold, zoom out em troca de assunto)
3. Detector de muletas de fala em português (dicionário + modelo de classificação leve) — atenção: isso precisa ser adaptável por idioma se for expandir mercado depois
4. Sistema de templates: estrutura de dados que define presets (velocidade de corte, agressividade de zoom, estilo de legenda)
5. Biblioteca de música com licenciamento resolvido (Epidemic Sound API, Artlist, ou similar)
6. **Score de confiança por decisão de edição** — se a IA não tem certeza (ex: pausa pode ser proposital), não corta automaticamente, marca para revisão

### Critério de saída

- Redução mensurável de edição manual pós-processamento (medir taxa de correção do usuário na tela de revisão)
- Templates testados com pelo menos 3 nichos de conteúdo diferentes

---

## Fase 3 — Geração de Conteúdo (Áudio → Vídeo e Texto → Vídeo) (8–10 semanas)

Essa é a fase que expande o produto além de "editor" para "gerador".

### Escopo A — Áudio → Vídeo

- Transcrição do áudio
- Busca de imagens relacionadas (Pexels/Unsplash API) ou geração via IA
- Efeito Ken Burns (zoom lento) + transições
- Legendas + exportação

### Escopo B — Texto → Vídeo

- Roteiro gerado ou fornecido pelo usuário
- TTS (ElevenLabs ou OpenAI TTS) com seleção de voz
- Geração de imagens por IA (Flux, Imagen, GPT Image) sincronizada com o roteiro
- Animação leve das imagens (Remotion entra aqui)
- Legendas palavra por palavra (estilo karaokê)

### Tarefas técnicas

1. Integração com API de busca de imagens de banco + fallback de geração por IA
2. Pipeline de matching texto→imagem (quando o roteiro menciona "Roma", buscar/gerar imagem relacionada)
3. Integração TTS com controle de velocidade/tom
4. Setup Remotion para animações programáticas (Ken Burns, transições, movimento de texto)
5. Sistema de B-roll automático: matching de palavras-chave → banco de imagens/vídeos por categoria

### Critério de saída

- Usuário digita um texto de 200 palavras → recebe vídeo completo em menos de 5 minutos
- Qualidade de imagem/matching validada por amostragem humana (não só automática)

---

## Fase 4 — Agentes de Estilo e Personalização Avançada (contínuo)

### Escopo

- Agentes especializados por formato (YouTube longo, TikTok, podcast, curso, Reels)
- Aprendizado de preferências do usuário (ex: sempre remove certos tipos de pausa, sempre usa a mesma fonte de legenda)
- Analytics de performance do vídeo gerado (se integrado com YouTube/TikTok API) para ajustar decisões de edição no próximo vídeo

### Critério de saída

- Métrica de retenção de usuários mês a mês
- Taxa de reuso da plataforma sem intervenção manual

---

## Considerações Transversais (aplicam-se a todas as fases)

|Tema|Ação recomendada|
|---|---|
|**Custo de IA por vídeo**|Monitorar desde a Fase 0; considerar cache de resultados intermediários (não retranscrever se o usuário só ajustar corte)|
|**Direitos de imagem/voz**|Termos de uso claros sobre clonagem de voz e uso de imagem da própria pessoa; nunca usar dados do usuário para treinar modelos sem opt-in explícito|
|**Compute pesado**|Workers com GPU dedicada para detecção facial, geração de imagem e renderização — separar do backend principal|
|**Feedback em tempo real**|Websocket para status de job desde a Fase 1|
|**Multi-idioma**|Se expansão internacional for objetivo, isolar a lógica de detecção de muletas de fala por idioma desde a Fase 2|
|**Qualidade / confiança**|Manter score de confiança e opção de revisão manual em toda decisão automática — é o que constrói confiança do usuário no produto|

---

## Cronograma Resumido

|Fase|Duração estimada|Entrega principal|
|---|---|---|
|Fase 0|2–3 semanas|Infraestrutura + validação de mercado|
|Fase 1 (MVP)|4–6 semanas|Corte de silêncio + legendas + revisão manual|
|Fase 2|6–8 semanas|Zoom automático + jump cuts + templates|
|Fase 3|8–10 semanas|Áudio→vídeo e Texto→vídeo completos|
|Fase 4|Contínuo|Agentes de estilo + personalização|

**Total até produto completo (Fases 0–3): ~5–6 meses** com equipe pequena focada (2–3 devs full-stack + 1 dedicado a pipeline de IA).

---

## Próximos Passos Imediatos

1. Definir orçamento disponível para APIs de IA (Whisper, LLM, TTS, geração de imagem) nos primeiros 3 meses
2. Decidir entre self-hosted (Whisper local, mais barato em escala) vs API (mais rápido de lançar)
3. Rodar as entrevistas de validação da Fase 0 antes de escrever a primeira linha de código do MVP
4. Montar o schema de dados inicial (projects, jobs, timeline JSON) — esse formato de timeline será reutilizado em todas as fases seguintes, vale investir tempo em desenhá-lo bem desde já