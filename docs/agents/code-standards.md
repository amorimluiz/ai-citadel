# Code Standards

> Carregado quando: implementando feature, refatorando, code review.
> Referenciado por: `AGENTS.md#core-rules`.

<solid>
Aplicação pragmática — não cite "SOLID" em PRs, **demonstre**.

- **SRP (obrigatório)**: um arquivo, uma razão para mudar. Se um arquivo agrupa parsing + I/O + validação, separe.
- **OCP**: aplique via interfaces **somente** quando houver ≥2 implementações reais (ex.: `GitRunner` se for testar com fake; não invente uma interface "futura").
- **LSP**: subtipos devem cumprir o contrato. Em Go, isso significa: implementações de interface não devem lançar `panic` em métodos que a interface não documenta como tal.
- **ISP**: interfaces pequenas. Prefira `io.Reader` / `io.Writer` ao invés de mega-interfaces. Regra de bolso: ≤4 métodos por interface.
- **DIP**: handlers de comando (Cobra) recebem dependências por construtor; nunca instanciam adapters concretos diretamente.
</solid>

<clean-code>
Padrões de mercado — nada exótico.

- **Nomes**: revelam intenção. `w` é OK em loop curto; `worktreePath` é OK em escopo amplo. Evite `data`, `info`, `manager`, `helper`.
- **Funções**: <30 linhas, um nível de abstração, early returns. Se precisa de comentário interno explicando uma seção → extraia função.
- **Argumentos**: ≤3. Mais que isso → struct de opções (`type CreateOpts struct { ... }`).
- **Erros**: sempre com contexto via `fmt.Errorf("create worktree %q: %w", path, err)`. Nunca `_ = err`.
- **Comentários**: só para o **porquê** não-óbvio (invariante, workaround específico, link para issue). Nunca o **o quê**.
- **Booleans**: nomes afirmativos (`isReady`, não `notReady`). Evite parâmetros booleanos posicionais — use enum/typed constants.
</clean-code>

<design-patterns>
Usar quando resolverem um problema concreto neste projeto.

| Padrão | Onde faz sentido aqui |
|---|---|
| **Adapter** | `internal/git`, `internal/docker` envolvem CLI externa por trás de interface testável |
| **Strategy** | Diferentes estratégias de isolamento (worktree-only vs worktree+compose) |
| **Factory** | Construção de `Workspace` a partir de `config.YAML` |
| **Command** | Comandos Cobra delegam para um `Command` do domínio (separa transporte de regra) |
| **Repository** | Persistência de estado de workspaces (arquivo local) |
| **Options (Functional)** | `New(opts ...Option)` para construtores com muitos parâmetros opcionais |

**Não usar**: Singleton (estado global), AbstractFactory (overkill em Go), Visitor (raramente justificável).
</design-patterns>

<file-size>
- Hard limit: **150 linhas/arquivo `.go`** (excluindo blank lines, incluindo imports).
- Se um arquivo cresce além disso, é sinal de SRP violado → divida por responsabilidade, não por "size".
- Arquivos de teste seguem o mesmo limite; quebre em `foo_create_test.go`, `foo_delete_test.go` etc.
- Exceções: arquivos gerados (`*.pb.go`, `*_gen.go`, mocks gerados).
</file-size>

<testing>
- Use **table-driven tests** como padrão.
- Use o pacote `testing` da stdlib + `testify/require` (apenas se já estiver no `go.mod`).
- Fakes > mocks. Para `GitRunner` / `DockerRunner`, escreva fakes que armazenam chamadas em slice.
- Sem testes de integração que dependam de Docker rodando por default — mover para `//go:build integration`.
</testing>
