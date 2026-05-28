# Go Conventions

> Carregado quando: escrevendo Go neste repo.
> Referenciado por: `AGENTS.md#core-rules`, `code-standards.md`.

<naming>
- **Pacotes**: lowercase, sem underscore, sem plural (`workspace`, não `workspaces`).
- **Arquivos**: snake_case quando multi-palavra (`worktree_create.go`).
- **Exported**: PascalCase. **Unexported**: camelCase.
- **Acrônimos**: mantenha o case (`HTTPClient`, `parseURL`, `httpClient`).
- **Interfaces curtas** com sufixo `-er` (`Runner`, `Reader`). Maiores ganham nome de papel (`GitAdapter`).
- **Erros**: variáveis `ErrXxx`; tipos `XxxError`.
</naming>

<error-handling>
- Sempre `fmt.Errorf("operação %q: %w", arg, err)` para wrap.
- Sentinel errors em `internal/<pkg>/errors.go`: `var ErrNotFound = errors.New("workspace not found")`.
- Use `errors.Is` / `errors.As` em verificações; nunca compare string de erro.
- Erros em CLI saem por `stderr`; exit code via `os.Exit(1)` apenas em `main`.
</error-handling>

<concurrency>
- Toda função que faz I/O ou pode bloquear: primeiro parâmetro = `ctx context.Context`.
- Nunca armazene `context.Context` em struct (exceção: `Request` types).
- Goroutines: sempre com mecanismo de cancelamento (`ctx`) **e** sincronização explícita (`sync.WaitGroup`, `errgroup.Group`).
- Channels: documente owner (quem fecha). Regra: quem envia, fecha.
- Prefira `sync.Mutex` a channels para proteger estado simples; channels para sinalização/handoff.
</concurrency>

<generics>
- Use quando elimina duplicação real entre tipos não relacionados.
- Não invente type constraints elaboradas; comece com `any` ou `comparable`.
- Se uma versão não-genérica é mais clara, prefira essa.
</generics>

<testing>
- Table-driven é o default:
  ```go
  tests := []struct {
      name string
      in   Input
      want Output
      err  error
  }{ ... }
  for _, tt := range tests {
      t.Run(tt.name, func(t *testing.T) { ... })
  }
  ```
- `t.Parallel()` por default, **exceto** quando o teste toca filesystem/env compartilhado.
- Helpers de teste em `testhelper_test.go` no mesmo pacote.
- Sem `init()` em testes — use `TestMain` se realmente precisar.
- Fixtures em `testdata/` (ignorado por `go build` por convenção).
</testing>

<imports>
- Ordem: stdlib, blank line, third-party, blank line, internos. `goimports` cuida disso.
- Sem `import .` (dot imports).
- Sem aliases salvo colisão real.
</imports>
