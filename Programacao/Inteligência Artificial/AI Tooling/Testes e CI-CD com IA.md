---
tags:
  - ia
  - tooling
---

# Testes e CI/CD com IA

Voltar para [[AI Tooling - Visão Geral]] · Relacionado: [[Ferramentas de Análise de Código]] · [[Agentic Engineering#Hooks]]

Testes e CI são o **sistema de verificação** do agente. Sem eles, cada linha gerada precisa de revisão manual minuciosa. Com eles, o agente descobre sozinho a maioria dos próprios erros.

> [!info] Mudança de mentalidade
> Antes da IA, testes protegiam contra regressões. Com agentes, testes também são **a especificação executável** e o **sinal de feedback** que deixa o agente trabalhar sozinho. Quanto melhor sua suíte, mais autonomia você pode dar.

## A pirâmide de verificação

Do sinal mais rápido para o mais lento — o agente deve passar pelos rápidos primeiro.

| Camada | Tempo | Backend (Java/Spring) | Frontend (React) |
| --- | --- | --- | --- |
| Tipos / compilador | segundos | `javac` via `./mvnw compile` | `tsc --noEmit` |
| Lint / formatação | segundos | Spotless, Checkstyle | ESLint, Prettier |
| Testes unitários | segundos | JUnit 5 + Mockito + AssertJ | Vitest/Jest + Testing Library |
| Testes de fatia (slice) | segundos | `@WebMvcTest`, `@DataJpaTest` | Componentes com Testing Library |
| Integração | minutos | `@SpringBootTest` + Testcontainers | MSW (mock de API) |
| End-to-end | minutos | — | Playwright |
| Qualidade / segurança | minutos | SonarQube, JaCoCo, Semgrep, Dependabot | SonarQube, `npm audit` |

## Testes no backend (Spring Boot)

### Unitário do Service

```java
@ExtendWith(MockitoExtension.class)
class CheckoutServiceTest {

    @Mock EstoqueService estoqueService;
    @Mock PedidoRepository pedidoRepository;
    @InjectMocks CheckoutService checkoutService;

    @Test
    void deveFalharQuandoEstoqueInsuficiente() {
        var request = new CheckoutRequest(1L, List.of(new ItemRequest(10L, 3)));
        when(estoqueService.disponivel(10L)).thenReturn(2);

        assertThatThrownBy(() -> checkoutService.finalizar(request))
                .isInstanceOf(EstoqueInsuficienteException.class);

        verify(pedidoRepository, never()).save(any());
    }
}
```

### Fatia do Controller

```java
@WebMvcTest(CheckoutController.class)
class CheckoutControllerTest {

    @Autowired MockMvc mockMvc;
    @MockitoBean CheckoutService checkoutService;

    @Test
    void deveRetornar409QuandoEstoqueInsuficiente() throws Exception {
        when(checkoutService.finalizar(any())).thenThrow(new EstoqueInsuficienteException(10L));

        mockMvc.perform(post("/api/checkout")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content("""
                                {"clienteId": 1, "itens": [{"produtoId": 10, "quantidade": 3}]}
                                """))
                .andExpect(status().isConflict());
    }
}
```

> [!note] `@MockitoBean` é do Spring Boot 3.4+. Em versões anteriores, use `@MockBean`.

### Integração com Testcontainers

Banco **real** em container, em vez de H2 — pega problemas de SQL, migrations e constraints que mocks escondem.

```java
@SpringBootTest
@Testcontainers
class CheckoutIntegrationTest {

    @Container
    @ServiceConnection
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16");

    @Autowired CheckoutService checkoutService;

    @Test
    void deveReservarEstoqueAoFinalizar() {
        // arrange com dados reais, act, assert no banco
    }
}
```

## Testes no frontend (React)

```tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { CheckoutButton } from "./CheckoutButton";

test("mostra erro quando o estoque é insuficiente", async () => {
  const onCheckout = vi.fn().mockRejectedValue(new Error("Estoque insuficiente"));
  render(<CheckoutButton onCheckout={onCheckout} />);

  await userEvent.click(screen.getByRole("button", { name: /finalizar/i }));

  expect(await screen.findByText(/estoque insuficiente/i)).toBeInTheDocument();
});
```

Testing Library incentiva testar **como o usuário usa** (por papel e texto), não detalhes de implementação — o que torna os testes mais resistentes às refatorações que o agente fizer.

## Padrões de uso com agentes

| Padrão | Como funciona |
| --- | --- |
| **TDD com agente** | Você (ou o agente, a partir da spec) escreve os testes primeiro; confirma que falham; o agente implementa até passarem. O melhor sinal de "pronto" que existe |
| **Testes a partir da spec** | Cada cenário Dado/Quando/Então de [[Spec-Driven Development]] vira um teste |
| **Reproduzir antes de corrigir** | Para bug: "escreva um teste que reproduz o bug, confirme que falha, depois corrija" |
| **Critério de pronto** | No `CLAUDE.md`: "rode `./mvnw verify` antes de dizer que terminou" |
| **Proteger testes** | Regra: "não altere testes existentes sem pedir". Agentes às vezes "corrigem" o teste em vez do código |
| **Revisar asserts** | Testes gerados por IA podem ser fracos (só testam que não lança exceção). Confira se os asserts verificam o comportamento |

> [!warning] Cobertura alta ≠ testes bons
> O agente consegue gerar 90% de cobertura com testes que não verificam nada. Use **mutation testing** (PIT no Java, Stryker no JS) para medir se os testes realmente pegam bugs.

## CI/CD com GitHub Actions

O CI é a verificação **final e imparcial**: roda em ambiente limpo, para todo PR, feito por humano ou agente.

`.github/workflows/ci.yml`:

```yaml
name: CI

on:
  pull_request:
  push:
    branches: [main]

jobs:
  backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: "21"
          cache: maven
      - name: Build, testes e verificações
        run: ./mvnw -B verify   # Testcontainers funciona no runner ubuntu (Docker disponível)

  frontend:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: web
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "22"
          cache: npm
          cache-dependency-path: web/package-lock.json
      - run: npm ci
      - run: npm run lint
      - run: npx tsc --noEmit
      - run: npm test -- --run
```

### Integrando o agente ao CI

- **Agente lê o CI**: com o MCP do GitHub ou o CLI `gh` (`gh run view --log-failed`), peça "o CI do meu PR falhou, descubra por quê e corrija".
- **Agente no CI**: Claude Code tem uma GitHub Action (`anthropics/claude-code-action`) que responde a menções `@claude` em issues e PRs; o Copilot coding agent recebe uma issue e abre um PR; o Cursor tem o Bugbot para review automático de PRs.
- **Review automático**: um agente revisa o diff de cada PR e comenta problemas. Não substitui o review humano, mas pega o óbvio antes.

## Qualidade: SonarQube e afins

| Ferramenta | O que verifica |
| --- | --- |
| **SonarQube / SonarCloud** | Bugs, code smells, duplicação, vulnerabilidades, cobertura. O **quality gate** bloqueia o merge se o código novo piorar |
| **JaCoCo** | Cobertura de testes no Java (alimenta o Sonar) |
| **PIT** | Mutation testing no Java |
| **Dependabot / Renovate** | Atualização de dependências |
| **Semgrep / CodeQL** | Análise de segurança — ver [[Ferramentas de Análise de Código]] |

> [!tip] Quality gate em código novo
> Configure o Sonar para exigir qualidade no **código novo** (ex.: cobertura ≥ 80% nas linhas alteradas). Isso pega exatamente o código que o agente acabou de gerar, sem travar o time por causa do legado.

## Exercícios

1. Garanta que `./mvnw verify` e `npm test` rodam limpos em um projeto seu. Esse é o pré-requisito de tudo.
2. Crie o workflow acima e abra um PR com um teste quebrado de propósito para ver o CI falhar.
3. Faça um bug fix com o padrão "reproduzir antes de corrigir" usando o agente.
4. Rode PIT no módulo onde o agente escreveu testes e veja quantos mutantes sobrevivem.
