# External Commands (`git`, `docker compose`)

> Carregado quando: invocando `git` ou `docker compose` via `os/exec`.
> Referenciado por: `AGENTS.md#stack`, `anti-patterns.md`.

<rule>
**Toda chamada externa passa por `internal/exec`** (wrapper) e é consumida via adapter (`internal/git`, `internal/docker`). Nunca chame `exec.Command` direto em `internal/cli/`, `internal/workspace/`, ou `internal/runtime/`.

Motivo: testabilidade (substitua adapter por fake), uniformidade de timeout/log/erro, e auditoria de toda interação com o sistema.
</rule>

<exec-wrapper>
`internal/exec` deve expor algo como:

```go
type Runner interface {
    Run(ctx context.Context, cmd Spec) (Result, error)
}

type Spec struct {
    Name    string            // "git" | "docker"
    Args    []string
    Dir     string            // working dir
    Env     map[string]string // additional env (merged over os.Environ)
    Stdin   io.Reader
    Timeout time.Duration     // 0 = use ctx deadline
}

type Result struct {
    Stdout   []byte
    Stderr   []byte
    ExitCode int
}
```

Garantias do wrapper:
- Respeita `ctx` cancellation.
- Captura stdout/stderr separados.
- Erro inclui comando + exit code + tail de stderr.
- Logs em `--verbose` mostram comando completo (com env redacted).
</exec-wrapper>

<git-adapter>
`internal/git` envolve operações usadas pelo domínio:

- `Worktree.Add(ctx, path, ref)` → `git worktree add <path> <ref>`
- `Worktree.Remove(ctx, path, force bool)`
- `Worktree.List(ctx)` → parseado de `git worktree list --porcelain`
- `Branch.Create(ctx, name, from)`
- `Repo.HeadRef(ctx)`

Regras:
- Sempre `--porcelain` ou `-z` quando disponível (parsing estável).
- Nunca parsear output humano de `git status`/`git log`.
- Caminhos sempre absolutos.
</git-adapter>

<docker-adapter>
`internal/docker` envolve `docker compose`:

- `Compose.Up(ctx, projectDir, opts)` → `docker compose -p <project> up -d`
- `Compose.Down(ctx, projectDir, opts)`
- `Compose.PS(ctx, projectDir)` → JSON (`--format json`)
- `Compose.Logs(ctx, projectDir, service, follow bool)`

Regras:
- `-p <project>` sempre explícito (deriva do nome do workspace) para isolar networks/volumes.
- Use `--format json` quando disponível.
- Portas dinâmicas: leia `docker compose port` ou `ps --format json` para descobrir; nunca assuma porta fixa.
- Cleanup: `down --volumes --remove-orphans` em `destroy` de workspace.
</docker-adapter>

<security>
- **Nunca** construa comando via concatenação de string + `sh -c`. Use `[]string{...}` de args.
- Valide entradas que vão para argv (ex.: nome de workspace) com regex restrita (`^[a-z0-9][a-z0-9-]{0,62}$`).
- Não logue valores de env potencialmente sensíveis.
- Timeout default obrigatório (sugerido: 30s para git, 120s para compose up).
</security>

<idempotency>
- `worktree add` em path existente → detecte antes (`worktree list`) e retorne `ErrAlreadyExists` ou no-op conforme contrato.
- `compose up` é idempotente por design — confie.
- `compose down` em projeto inexistente → ignore exit code se stderr indicar "no such project".
- Toda operação destrutiva deve ser precedida por checagem de estado.
</idempotency>
