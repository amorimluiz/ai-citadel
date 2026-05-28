# AI Citadel

> CLI local-first em Go para orquestrar ambientes isolados de desenvolvimento multi-repo voltados a workflows assistidos por IA.

AI Citadel atua como uma camada central de controle para que agentes de IA desenvolvam features, fixes e refactors **em paralelo e com segurança**, isolando tanto a codebase (via **git worktrees**) quanto runtimes e dependências (via **Docker Compose**). Adota **SDD (Spec-Driven Development)** como fluxo padrão, evitando conflitos de portas, colisão de estado, compartilhamento indesejado de ambiente e interferência entre agentes operando simultaneamente no mesmo sistema.

## Por quê

Times que usam agentes de IA para codar em paralelo enfrentam três problemas recorrentes:

- **Conflitos de codebase** — múltiplos agentes editando os mesmos arquivos numa única working tree.
- **Colisão de runtime** — mesma porta, mesmo volume, mesmo container name.
- **Estado vazado** — banco, cache, filas compartilhados entre tasks que deveriam ser independentes.

AI Citadel resolve isso dando a cada task seu próprio worktree + sua própria stack Compose, isolada por projeto.

## Stack

- **Go** (1.22+)
- **Cobra** — framework CLI
- **YAML** — configuração declarativa de workspaces
- **Docker Compose** — runtime isolado por workspace (via `os/exec`)
- **Git CLI** — gestão de worktrees (via `os/exec`)

## Status

🚧 **Em desenvolvimento inicial.** API e comandos sujeitos a mudança.

## Como funciona (visão geral)

```
┌──────────────────────────────────────────────────────────────┐
│                       citadel (CLI)                          │
├──────────────────────────────────────────────────────────────┤
│  workspace create │ workspace list │ workspace destroy │ ... │
│  runtime up       │ runtime down   │ runtime status    │ ... │
└──────────┬───────────────────────────────┬───────────────────┘
           │                               │
           ▼                               ▼
   ┌───────────────┐               ┌────────────────┐
   │ git worktree  │               │ docker compose │
   │  (codebase)   │               │   (runtime)    │
   └───────────────┘               └────────────────┘
```

Cada workspace = 1 worktree isolado + 1 projeto Compose dedicado (`-p <workspace>`), com portas atribuídas dinamicamente e estado próprio.

## Comandos previstos

```bash
citadel workspace create <name> [--branch <ref>]   # cria worktree + ambiente
citadel workspace list                              # lista workspaces ativos
citadel workspace destroy <name>                    # remove worktree + tear down
citadel runtime up <workspace>                      # sobe stack Compose
citadel runtime down <workspace>                    # derruba stack
citadel runtime status [<workspace>]                # status dos serviços
```

## Para contribuidores e agentes

A configuração canônica para agentes de IA operando neste repositório está em [`AGENTS.md`](AGENTS.md). Esse arquivo contém:

- Visão geral do projeto e stack
- Regras de código (limite de 150 linhas/arquivo, SOLID, Clean Code)
- Mapa de skills disponíveis
- Folder structure
- Anti-patterns
- Referências para docs detalhadas em [`docs/agents/`](docs/agents/)

## Licença

MIT — ver [`LICENSE`](LICENSE).
