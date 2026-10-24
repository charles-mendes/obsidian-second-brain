---
tags:
  - ia
  - tooling
---

# Ferramentas de Análise de Código

Voltar para [[AI Tooling - Visão Geral]] · Relacionado: [[Testes e CI-CD com IA]] · [[AI Coding Assistants]]

Ferramentas que ajudam a **navegar em repositórios grandes, localizar padrões e fazer alterações estruturais** com precisão. Para agentes, elas resolvem dois problemas:

1. **Encontrar** o que importa sem ler o repositório inteiro (economiza contexto).
2. **Mudar em escala** de forma determinística, em vez de o LLM editar 300 arquivos um a um (lento, caro e sujeito a erro).

> [!tip] Regra de ouro
> Para mudanças mecânicas em muitos arquivos, peça para o agente **escrever a regra/receita** (ast-grep, OpenRewrite) e rodar a ferramenta — não para editar arquivo por arquivo. Você revisa uma regra de 10 linhas em vez de 300 diffs.

## Busca textual × busca estrutural

| Tipo | Ferramenta | Entende | Exemplo |
| --- | --- | --- | --- |
| **Texto** | ripgrep (`rg`) | Caracteres e regex | Achar a string `LegacyDateUtils` |
| **Estrutura (AST)** | ast-grep, Semgrep | A árvore sintática do código | Achar chamadas a `format(...)` **só** da classe `LegacyDateUtils`, ignorando comentários e strings |
| **Semântica** | LSP (IDE), OpenRewrite | Tipos, imports, hierarquia | Achar todos os usos de um método, inclusive via herança |

**AST** (*Abstract Syntax Tree*) é a representação do código como árvore (classe → método → chamada → argumentos). Buscar na AST evita falsos positivos de regex.

## ripgrep

Rápido e respeita o `.gitignore`. É o que a maioria dos agentes usa por baixo para buscar.

```bash
rg "LegacyDateUtils" --type java            # onde é usado
rg -l "@Transactional" src/main             # só nomes de arquivos
rg "new RestTemplate\(" -C 2                # com 2 linhas de contexto
```

## ast-grep

Busca e reescrita estrutural com padrões que parecem código. `$A` captura uma expressão; `$$$` captura uma lista de argumentos.

```bash
# Encontrar chamadas ao método legado
ast-grep run --lang java --pattern 'LegacyDateUtils.format($DATA, $FORMATO)'

# Reescrever para a API nova (mostra o diff; use -U para aplicar em tudo)
ast-grep run --lang java \
  --pattern 'LegacyDateUtils.format($DATA, $FORMATO)' \
  --rewrite 'DateTimeFormatter.ofPattern($FORMATO).format($DATA)'
```

Regras em YAML podem ficar versionadas no repositório e rodar no CI (`ast-grep scan`):

```yaml
# rules/no-field-injection.yml
id: no-field-injection
language: java
severity: warning
message: Use injeção por construtor em vez de @Autowired em campo
rule:
  kind: field_declaration          # um campo da classe...
  has:
    stopBy: end
    kind: marker_annotation        # ...que tenha a anotação @Autowired
    regex: ^@Autowired$
```

> [!tip] Descobrindo os nomes dos nós
> O [playground do ast-grep](https://ast-grep.github.io/playground.html) mostra a AST de um trecho de código. É o jeito mais rápido de escrever (ou pedir para o agente escrever) uma regra.

## Semgrep

Parecido com o ast-grep, mas com foco em **segurança e qualidade**, com milhares de regras prontas (OWASP, Spring, React).

```bash
semgrep scan --config auto          # regras recomendadas para as linguagens do projeto
semgrep scan --config p/java        # pacote de regras Java
```

Regra própria:

```yaml
rules:
  - id: sql-concatenado
    languages: [java]
    severity: ERROR
    message: Possível SQL injection. Use parâmetros (?) ou JPA.
    pattern: $JDBC.query("..." + $X, ...)
```

Uso com agentes: rodar Semgrep no CI e pedir "corrija os achados do Semgrep deste PR".

## OpenRewrite (refatoração em massa no Java)

Para o ecossistema Java, é a ferramenta mais poderosa: **receitas** prontas que entendem tipos e fazem migrações inteiras.

```bash
# Migrar para Spring Boot 3 (javax → jakarta, propriedades renomeadas, APIs removidas...)
./mvnw -U org.openrewrite.maven:rewrite-maven-plugin:run \
  -Drewrite.recipeArtifactCoordinates=org.openrewrite.recipe:rewrite-spring:RELEASE \
  -Drewrite.activeRecipes=org.openrewrite.java.spring.boot3.UpgradeSpringBoot_3_3
```

Receitas úteis: migração de Java 8/11/17 → 21, JUnit 4 → 5, Spring Boot 2 → 3, remover imports não usados, padronizar formatação.

> [!example] Combinação ideal
> OpenRewrite faz 90% da migração de forma determinística → o agente resolve os 10% restantes (casos específicos que a receita não cobre) guiado pelos erros de compilação e testes.

## Linters e regras de arquitetura

| Ferramenta | Stack | Para quê |
| --- | --- | --- |
| **ESLint** | React/TS | Erros e padrões (regras de hooks do React, imports, acessibilidade) |
| **Prettier** / **Spotless** | JS / Java | Formatação automática (o agente não precisa se preocupar com estilo) |
| **Checkstyle, PMD, SpotBugs** | Java | Estilo, más práticas e bugs comuns |
| **Error Prone** | Java | Bugs detectados em tempo de compilação |
| **ArchUnit** | Java | Regras de arquitetura como **testes** |

### ArchUnit: arquitetura que o agente não consegue quebrar

```java
@AnalyzeClasses(packages = "com.loja")
class ArquiteturaTest {

    @ArchTest
    static final ArchRule controllerNaoAcessaRepository =
            noClasses().that().haveSimpleNameEndingWith("Controller")
                    .should().dependOnClassesThat().haveSimpleNameEndingWith("Repository");

    @ArchTest
    static final ArchRule entidadesNaoVazamParaApi =
            noClasses().that().haveSimpleNameEndingWith("Controller")
                    .should().dependOnClassesThat().areAnnotatedWith(jakarta.persistence.Entity.class);
}
```

As regras do `CLAUDE.md` ("Controller nunca acessa Repository") são **pedidos**; o ArchUnit as transforma em **garantias** — se o agente violar, o teste falha e ele mesmo corrige.

### ESLint (flat config)

```js
// eslint.config.js
import js from "@eslint/js";
import tseslint from "typescript-eslint";
import reactHooks from "eslint-plugin-react-hooks";

export default tseslint.config(
  js.configs.recommended,
  ...tseslint.configs.recommended,
  {
    plugins: { "react-hooks": reactHooks },
    rules: {
      "react-hooks/rules-of-hooks": "error",
      "react-hooks/exhaustive-deps": "warn",
    },
  }
);
```

## Quando usar cada uma

| Situação | Ferramenta |
| --- | --- |
| "Onde isso é usado?" | ripgrep ou LSP |
| Trocar um padrão de código em centenas de arquivos | ast-grep |
| Migração de framework/versão no Java | OpenRewrite |
| Garantir regras de segurança no CI | Semgrep / CodeQL |
| Garantir regras de arquitetura | ArchUnit |
| Estilo e erros comuns | ESLint, Checkstyle, Prettier, Spotless |

## Exercícios

1. Use `ast-grep` para listar todos os `@Autowired` em campo de um projeto seu e peça ao agente uma regra de rewrite para injeção por construtor.
2. Rode `semgrep scan --config auto` e peça ao agente para corrigir os achados mais graves.
3. Adicione o `ArquiteturaTest` acima a um projeto e veja se o código atual já viola alguma regra.
4. Rode uma receita do OpenRewrite (ex.: JUnit 4 → 5) em um projeto legado.
