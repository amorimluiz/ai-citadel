# CLI Design (Cobra)

> Carregado quando: adicionando/editando comandos Cobra.
> Referenciado por: `AGENTS.md#stack`, `folder-structure.md`.

<command-shape>
Padrão de UX: `citadel <noun> <verb> [args] [flags]`.

Exemplos:
- `citadel workspace create <name>` — cria worktree + ambiente
- `citadel workspace list`
- `citadel workspace destroy <name>`
- `citadel runtime up <workspace>`
- `citadel runtime down <workspace>`
- `citadel runtime status [<workspace>]`

Evitar verbos isolados na raiz (`citadel create ...`); sempre nomear o "noun".
</command-shape>

<file-organization>
- `internal/cli/root.go` — `rootCmd`, flags globais (`--config`, `--verbose`, `--json`).
- `internal/cli/<noun>.go` — comando-pai (`workspaceCmd`) + subcomandos no mesmo arquivo **se** couberem em <150 linhas; senão um arquivo por verb (`workspace_create.go`).
- Cada arquivo de comando registra-se no pai via `init()` **apenas** com `AddCommand` — sem outros side effects.
</file-organization>

<command-template>
```go
// internal/cli/workspace_create.go
func newWorkspaceCreateCmd(deps *Deps) *cobra.Command {
    var opts workspace.CreateOpts
    cmd := &cobra.Command{
        Use:   "create <name>",
        Short: "Create an isolated workspace (worktree + runtime)",
        Args:  cobra.ExactArgs(1),
        RunE: func(cmd *cobra.Command, args []string) error {
            opts.Name = args[0]
            return deps.Workspaces.Create(cmd.Context(), opts)
        },
    }
    cmd.Flags().StringVar(&opts.Branch, "branch", "", "branch to checkout (default: new from HEAD)")
    cmd.Flags().BoolVar(&opts.Detached, "detached", false, "create as detached HEAD")
    return cmd
}
```

Regras:
- `RunE` (retorna erro), nunca `Run`.
- Use `cmd.Context()` — passe `ctx` adiante.
- Injete dependências via struct `Deps` montada em `cmd/citadel/main.go`.
- Flags ligadas a `opts` struct, não a globals.
</command-template>

<flags>
- Globais: apenas `--config`, `--verbose/-v`, `--json`, `--no-color`.
- Por comando: nomes longos descritivos; shorts só para alta frequência (`-v`, `-f`).
- Booleanos default = `false`. Use `--no-foo` se precisar inverter algo default-true.
- Validação no `RunE` (early return com erro humano), **não** dentro do domínio.
</flags>

<output>
- Default: human-readable. Verbo + status + path.
- Com `--json`: structured (definido em `internal/cli/output/`).
- `stdout` para resultado; `stderr` para logs/erros.
- Exit codes: `0` sucesso, `1` erro de execução, `2` erro de uso (Cobra lida automaticamente).
- Sem emoji/cores por default; cores apenas se `isatty(stdout)` **e** `--no-color` não setado.
</output>

<help-text>
- `Short`: uma linha, imperativo, <60 chars.
- `Long`: opcional, parágrafo curto + exemplos via `Example` field.
- Sempre incluir 1-2 exemplos em comandos não-triviais.
</help-text>
