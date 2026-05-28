# Folder Structure

> Carregado quando: criando arquivos novos, movendo código, decidindo onde algo vive.
> Referenciado por: `AGENTS.md#folder-structure`.

<layout>
```
ai-citadel/
├── cmd/
│   └── citadel/
│       └── main.go            # entrypoint fino: parseia flags globais e delega para internal/cli
├── internal/
│   ├── cli/                   # Cobra wiring: comandos, flags, output formatter
│   │   ├── root.go            # rootCmd + flags globais
│   │   ├── workspace.go       # `citadel workspace ...`
│   │   ├── runtime.go         # `citadel runtime ...`
│   │   └── ...
│   ├── config/                # parse + validação YAML; sem I/O fora de Load/Save
│   ├── workspace/             # ciclo de vida de worktrees (create, list, destroy)
│   ├── runtime/               # orquestração docker compose (up, down, status)
│   ├── git/                   # adapter para `git` CLI (implementa GitRunner)
│   ├── docker/                # adapter para `docker compose` CLI
│   ├── exec/                  # wrapper seguro sobre os/exec (timeout, env, capture)
│   └── domain/                # tipos puros: Workspace, Service, Spec, Port, etc.
├── pkg/                       # libs públicas reutilizáveis (manter mínimo; default = internal/)
├── docs/
│   └── agents/                # docs sob demanda para agentes
├── AGENTS.md
├── CLAUDE.md
├── go.mod
└── go.sum
```
</layout>

<rules>
- **`cmd/` é fino** — apenas wiring. Lógica vai para `internal/`.
- **`internal/` por default** — só promova para `pkg/` se uma lib externa for de fato consumir.
- **Sem ciclos** — `domain` não importa nada de `internal/`; `cli` importa de qualquer outro pacote `internal/`; adapters (`git`, `docker`) só importam `exec` e `domain`.
- **Um pacote por diretório** — nome do pacote = nome do diretório (idiomático Go).
- **Testes ao lado** — `internal/workspace/create.go` ↔ `internal/workspace/create_test.go`.
- **Sem subpastas profundas** — máximo 2 níveis dentro de `internal/`. Se sentir necessidade de mais, repense.
</rules>

<where-does-it-go>
| Tipo de código | Pacote |
|---|---|
| Definição de comando Cobra | `internal/cli/` |
| Parsing/validação de `citadel.yaml` | `internal/config/` |
| Lógica de criar/listar/destruir worktree | `internal/workspace/` |
| Orquestração compose (up/down/logs) | `internal/runtime/` |
| Chamada a `git worktree add` | `internal/git/` (via `internal/exec/`) |
| Chamada a `docker compose up` | `internal/docker/` (via `internal/exec/`) |
| Struct `Workspace`, `Service`, enums | `internal/domain/` |
| Helper genérico (string, slice) | **não criar** — inline ou stdlib |
</where-does-it-go>
