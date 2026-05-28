# AGENTS.md

> Documento canônico para agentes de IA operando neste repositório. Seções marcadas com tags XML para indexação e referência cruzada (ex.: "ver `<skills-map>`").

<project>
**AI Citadel** — CLI local-first em Go para orquestrar ambientes isolados de desenvolvimento multi-repo voltados a workflows assistidos por IA. Permite que agentes desenvolvam features, fixes e refactors em paralelo via **git worktrees** (isolamento de codebase) + **Docker Compose** (isolamento de runtime/dependências), evitando conflitos de portas, colisão de estado e interferência entre agentes simultâneos. Adota **SDD (Spec-Driven Development)** como fluxo padrão.
</project>

<stack>
- **Linguagem**: Go (1.22+)
- **CLI Framework**: [Cobra](https://github.com/spf13/cobra)
- **Configuração**: YAML
- **Orquestração**: Docker Compose (via `os/exec`)
- **VCS**: Git CLI (via `os/exec`)
</stack>

<quick-commands>
| Comando | Ação |
|---|---|
| `go build ./...` | Compilar todo o módulo |
| `go test ./...` | Rodar testes |
| `go vet ./...` | Análise estática |
| `gofmt -s -w .` | Formatar |
| `go run ./cmd/citadel` | Executar a CLI local |
</quick-commands>

<core-rules>
Regras invioláveis. Aplicam-se a 100% das tasks.

1. **Limite de 150 linhas** — Nenhum arquivo `.go` deve exceder **150 linhas** (incluindo imports, excluindo blank lines de separação). Se ultrapassar, quebre por responsabilidade antes de continuar. Exceção única: arquivos gerados (`*.pb.go`, `*_gen.go`).
2. **SOLID, sem fanatismo** — SRP é obrigatório; OCP/LSP/ISP/DIP aplicados **somente** quando trouxerem clareza ou desacoplamento real. Ver `<dynamic-docs:code-standards>`.
3. **Clean Code mainstream** — nomes intencionais, funções curtas (<30 linhas), early returns, zero comentários redundantes. Ver `<dynamic-docs:code-standards>`.
4. **Design Patterns sob demanda** — use Strategy, Factory, Adapter (especialmente para `exec.Command`), Command, Repository quando resolverem um problema concreto; nunca por estética.
5. **Local-first** — nenhuma chamada de rede sem flag explícita do usuário. Sem telemetria silenciosa.
6. **Idempotência** — toda operação que toca worktree ou Docker deve ser idempotente e reversível (rollback explícito em falha).
7. **Sem workarounds** — invoque [`no-workarounds`](.claude/skills/no-workarounds) antes de qualquer "patch rápido". Root-cause sempre.
8. **Testes ao lado** — `foo.go` → `foo_test.go` no mesmo pacote. Cobertura mínima para domínio crítico (worktree, docker, git).
</core-rules>

<folder-structure>
Layout de alto nível. Detalhes e responsabilidades por pasta em `<dynamic-docs:folder-structure>`.

```
ai-citadel/
├── cmd/citadel/           # entrypoint da CLI (main.go fino)
├── internal/
│   ├── cli/               # definições Cobra (commands, flags, wiring)
│   ├── config/            # parse/validação YAML
│   ├── workspace/         # ciclo de vida de worktrees
│   ├── runtime/           # orquestração Docker Compose
│   ├── git/               # adapter para git CLI
│   ├── docker/            # adapter para docker compose CLI
│   ├── exec/              # wrapper seguro de os/exec
│   └── domain/            # tipos e entidades core
├── pkg/                   # libs públicas reutilizáveis (raro)
├── docs/agents/           # docs carregadas sob demanda por agentes
├── AGENTS.md              # este arquivo
├── CLAUDE.md              # ponteiro para AGENTS.md
└── go.mod
```
</folder-structure>

<skills-map>
Skills disponíveis em `.claude/skills/` (locais a este repo). Invoque conforme o contexto:

| Skill | Quando usar |
|---|---|
| [`golang-pro`](.claude/skills/golang-pro) | Implementação Go idiomática, concorrência, generics, perf |
| [`context7`](.claude/skills/context7) | Consultar docs autoritativas de libs (Cobra, etc.) |
| [`find-docs`](.claude/skills/find-docs) | Buscar API/syntax de libs/frameworks |
| [`no-workarounds`](.claude/skills/no-workarounds) | **Obrigatório** antes de qualquer fix rápido |
| [`find-rules`](.claude/skills/find-rules) | Descobrir convenções aplicáveis à task |
| [`find-skills`](.claude/skills/find-skills) | Buscar capacidade nova não coberta |
| [`agent-md-refactor`](.claude/skills/agent-md-refactor) | Refatorar este AGENTS.md ou docs derivadas |
| [`documentation-writer`](.claude/skills/documentation-writer) | Criar/editar docs (Diátaxis) |
| [`git-rebase`](.claude/skills/git-rebase) | Rebase com resolução inteligente |
| [`qa-execution`](.claude/skills/qa-execution) | QA real-user de fluxos da CLI |
| [`qa-report`](.claude/skills/qa-report) | Planejar QA antes da execução |
| [`skill-best-practices`](.claude/skills/skill-best-practices) | Autoria de novas skills |
| [`skill-load-tips`](.claude/skills/skill-load-tips) | Auditar skills que não carregam bem |
| [`writing-tech-post`](.claude/skills/writing-tech-post) | Posts técnicos sobre o projeto |
</skills-map>

<dynamic-docs>
Carregue **apenas** o que for relevante para a task corrente:

| Tag | Arquivo | Carregar quando |
|---|---|---|
| `code-standards` | [`docs/agents/code-standards.md`](docs/agents/code-standards.md) | Implementando feature, refatorando, code review |
| `folder-structure` | [`docs/agents/folder-structure.md`](docs/agents/folder-structure.md) | Criando arquivos novos, movendo código |
| `anti-patterns` | [`docs/agents/anti-patterns.md`](docs/agents/anti-patterns.md) | Antes de qualquer PR; quando sentir "cheiro" |
| `go-conventions` | [`docs/agents/go-conventions.md`](docs/agents/go-conventions.md) | Escrevendo Go neste repo |
| `cli-design` | [`docs/agents/cli-design.md`](docs/agents/cli-design.md) | Adicionando/editando comandos Cobra |
| `external-commands` | [`docs/agents/external-commands.md`](docs/agents/external-commands.md) | Invocando `git` ou `docker compose` |
</dynamic-docs>

<workflow>
Fluxo padrão para qualquer task não-trivial:

1. **Entender** — leia a spec/issue; se ambígua, pergunte.
2. **Descobrir rules** — invoque [`find-rules`](.claude/skills/find-rules) + carregue docs relevantes de `<dynamic-docs>`.
3. **Verificar libs** — para qualquer lib externa, invoque [`context7`](.claude/skills/context7) ou [`find-docs`](.claude/skills/find-docs).
4. **Implementar** — siga `<core-rules>`. Para Go, invoque [`golang-pro`](.claude/skills/golang-pro).
5. **Testar** — `_test.go` ao lado; rode `go test ./...` e `go vet ./...`.
6. **Validar** — antes de declarar "done", confira `<anti-patterns>` e o limite de 150 linhas.
7. **Commit** — mensagens curtas, imperativas, escopo claro (ex.: `workspace: idempotent worktree create`).
</workflow>

<anti-patterns-summary>
Versão curta. Detalhes em `<dynamic-docs:anti-patterns>`.

- ❌ Arquivos `.go` > 150 linhas
- ❌ Funções > 30 linhas sem justificativa
- ❌ `panic` fora de `main` / `init`
- ❌ `os/exec` direto fora de `internal/exec` ou adapters (`internal/git`, `internal/docker`)
- ❌ `interface{}` / `any` sem necessidade concreta
- ❌ Engolir erros (`_ = err`) — sempre tratar ou propagar com contexto
- ❌ Comentários que descrevem o **que** o código faz (deixe os nomes falarem)
- ❌ Abstrações especulativas / interfaces sem ≥2 implementações reais
- ❌ Estado global mutável
- ❌ Configuração inferida de env sem documentar no YAML schema
</anti-patterns-summary>
