---
tags:
  - ia
  - tooling
---

# Workflow Prático - API Spring Boot com IA

Voltar para [[AI Tooling - Visão Geral]]

Exemplo ponta a ponta juntando todas as camadas do tooling em uma feature real: **checkout de pedidos** em uma loja com backend Spring Boot e frontend React.

```
0. Preparar o terreno → 1. Spec (OpenSpec) → 2. Plano → 3. Implementação (Claude Code)
→ 4. Contexto externo (MCP) → 5. Verificação local (testes, hooks) → 6. PR + CI (GitHub Actions) → 7. Review → 8. Arquivar a spec
```

Você não precisa usar todas as etapas sempre. Para um bug simples, pule direto para a 3 com o padrão "reproduzir antes de corrigir".

## 0. Preparar o terreno (uma vez por projeto)

| Item | Por quê | Nota |
| --- | --- | --- |
| `CLAUDE.md` / `AGENTS.md` com comandos e regras | O agente conhece o projeto | [[AI Coding Assistants]] |
| `./mvnw verify` e `npm test` rodando limpos | O agente tem como se verificar | [[Testes e CI-CD com IA]] |
| Testcontainers + ArchUnit | Integração real e arquitetura garantida | [[Ferramentas de Análise de Código]] |
| CI no GitHub Actions | Verificação imparcial em todo PR | [[Testes e CI-CD com IA#CI/CD com GitHub Actions]] |
| MCP do GitHub e Postgres de dev (somente leitura) | Contexto externo | [[MCP - Model Context Protocol]] |
| `openspec init` | Specs versionadas | [[Spec-Driven Development]] |
| Skill `novo-endpoint-spring` + hook de formatação | Padronização | [[Agentic Engineering]] |

## 1. Especificar

No Claude Code:

> "Crie uma proposta OpenSpec para o checkout: o cliente autenticado finaliza o carrinho, o sistema valida estoque, reserva os itens e cria o pedido com status AGUARDANDO_PAGAMENTO. Pagamento fica fora do escopo. Faça perguntas se algo estiver ambíguo."

O agente gera `openspec/changes/add-checkout/` com `proposal.md`, `tasks.md` e o delta da spec. **Você revisa** e ajusta, principalmente os cenários de erro:

```markdown
## ADDED Requirements

### Requirement: Finalizar pedido
O sistema DEVE criar um pedido a partir do carrinho do cliente autenticado.

#### Scenario: Checkout com sucesso
- WHEN o cliente envia POST /api/checkout com carrinho válido
- THEN o pedido é criado com status AGUARDANDO_PAGAMENTO
- AND o estoque de cada item é reservado
- AND a resposta é 201 com o ID do pedido

#### Scenario: Estoque insuficiente
- WHEN algum item não tem estoque suficiente
- THEN a resposta é 409 com ProblemDetail listando os produtos
- AND nada é reservado (operação atômica)

#### Scenario: Carrinho vazio
- WHEN o carrinho não tem itens
- THEN a resposta é 422
```

## 2. Planejar

Em **plan mode**:

> "Leia a proposta `add-checkout` e os módulos `carrinho` e `produto`. Proponha o design: entidades, migration, endpoints, transações e concorrência na reserva de estoque. Não edite nada."

Pontos que você deve checar no plano (é aqui que sua experiência pesa):

- A reserva de estoque é **atômica** (`@Transactional`) e trata **concorrência** (lock otimista com `@Version` ou `UPDATE ... WHERE estoque >= ?`)?
- O endpoint é **idempotente** se o cliente clicar duas vezes (chave de idempotência)?
- A migration Flyway é nova, sem alterar as existentes?

Ajuste o plano e salve as decisões em `design.md`.

## 3. Implementar

> "Implemente as tarefas de `openspec/changes/add-checkout/tasks.md` uma por vez, seguindo a skill de novo endpoint. Comece escrevendo os testes de cada cenário da spec, confirme que falham e depois implemente. Marque cada tarefa ao concluir."

O que o agente vai produzir, mais ou menos:

```java
@RestController
@RequestMapping("/api/checkout")
class CheckoutController {

    private final CheckoutService checkoutService;

    CheckoutController(CheckoutService checkoutService) {
        this.checkoutService = checkoutService;
    }

    @PostMapping
    ResponseEntity<CheckoutResponse> finalizar(@AuthenticationPrincipal Jwt jwt,
                                               @Valid @RequestBody CheckoutRequest request) {
        var pedido = checkoutService.finalizar(jwt.getSubject(), request);
        return ResponseEntity.created(URI.create("/api/pedidos/" + pedido.id())).body(pedido);
    }
}
```

```java
@Service
class CheckoutService {

    @Transactional
    public CheckoutResponse finalizar(String clienteId, CheckoutRequest request) {
        var carrinho = carrinhoService.buscar(clienteId);
        if (carrinho.itens().isEmpty()) {
            throw new CarrinhoVazioException(clienteId);
        }
        estoqueService.reservar(carrinho.itens());   // lança EstoqueInsuficienteException
        var pedido = pedidoRepository.save(Pedido.aguardandoPagamento(clienteId, carrinho.itens()));
        return CheckoutResponse.from(pedido);
    }
}
```

Seu papel aqui: acompanhar, interromper se ele sair do plano e **não deixar a sessão crescer demais** (uma sessão por grupo de tarefas).

## 4. Usar contexto externo (MCP)

Durante a implementação, o MCP entra quando o agente precisa de algo fora do repositório:

- **Postgres (dev, somente leitura)**: "confira no banco de dev como está a tabela `estoque` e se existem produtos com estoque negativo".
- **GitHub**: "leia a issue #87, que tem os requisitos do time de produto para o checkout".
- **Context7**: "confira na documentação atual como configurar `ProblemDetail` no Spring Boot 3".

## 5. Verificar localmente

- O **hook** formata o código a cada edição.
- O agente roda `./mvnw verify` (unitários, `@WebMvcTest`, Testcontainers, ArchUnit) até ficar verde.
- No frontend: componente `CheckoutButton` com teste de Testing Library para o erro 409.
- Peça uma revisão com contexto limpo: rode o subagente `revisor-seguranca` ou o comando `/revisar`.

## 6. Abrir o PR e deixar o CI validar

> "Crie uma branch `feat/checkout`, faça commits pequenos e abra um PR descrevendo a mudança e linkando a proposta OpenSpec."

O agente usa `git` e `gh pr create` (ou o MCP do GitHub). O GitHub Actions roda backend, frontend, lint e Sonar. Se falhar:

> "O CI do PR falhou. Veja os logs (`gh run view --log-failed`), descubra a causa e corrija."

## 7. Review humano

Revise contra a **spec**, não contra o seu gosto:

- [ ] Cada cenário da spec tem um teste correspondente?
- [ ] Os testes verificam comportamento (asserts reais), não só "não lança exceção"?
- [ ] Concorrência e transação estão corretas?
- [ ] Nada fora do escopo foi adicionado?
- [ ] Nenhum teste existente foi alterado para passar?
- [ ] Sem segredos, logs com dados sensíveis ou endpoints sem autorização?

## 8. Arquivar e melhorar o sistema

- `openspec archive add-checkout` → a spec de `checkout` passa a refletir o sistema real.
- **Retrospectiva rápida**: o que o agente errou? Vire isso em regra no `CLAUDE.md`, skill, teste ou hook. É assim que o processo melhora a cada feature.

## Versão enxuta (dia a dia)

Nem toda tarefa precisa do processo completo.

| Tamanho da tarefa | Fluxo |
| --- | --- |
| Typo, ajuste pequeno | Pedir direto ao agente → revisar diff |
| Bug | Teste que reproduz → corrigir → CI |
| Feature pequena | Plan mode → implementar com testes → PR |
| Feature média/grande | Fluxo completo com spec |
| Migração/refatoração em massa | OpenRewrite/ast-grep → agente resolve o resto → CI |
