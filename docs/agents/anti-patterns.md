# Anti-Patterns

> Carregado quando: antes de qualquer PR; quando sentir "cheiro" de código.
> Referenciado por: `AGENTS.md#anti-patterns-summary`.

<go-specific>
- ❌ **`panic` fora de `main`/`init`** — propague `error`.
- ❌ **`interface{}` / `any`** sem necessidade concreta — use generics ou tipos específicos.
- ❌ **Engolir erros**: `_ = err`, `if err != nil { return nil }`. Sempre propague com `%w` e contexto.
- ❌ **`init()` com efeitos colaterais** (registrar handlers, abrir arquivos) — torna teste impossível.
- ❌ **Goroutines sem cancelamento** — sempre receba `ctx context.Context`.
- ❌ **`fmt.Println` para erros** — use `fmt.Fprintln(os.Stderr, ...)` ou retorne `error`.
- ❌ **Variáveis globais mutáveis** — encapsule em struct e injete.
- ❌ **`time.Sleep` em produção** para "esperar algo" — use polling com context ou canais.
</go-specific>

<architecture>
- ❌ **`os/exec` fora de `internal/exec` e adapters** — toda chamada externa deve passar pelo wrapper para garantir timeout, captura de stderr, e testabilidade.
- ❌ **`internal/cli` chamando `git`/`docker` diretamente** — passe pelos adapters.
- ❌ **`domain` importando outros pacotes `internal/`** — domínio é puro.
- ❌ **Interfaces com 1 implementação real** (e nenhum plano concreto de teste com fake) — delete.
- ❌ **Camadas inventadas** (`Service` → `Manager` → `Handler` → `Helper`) — colapse.
</architecture>

<size-complexity>
- ❌ Arquivos `.go` > **150 linhas** — quebre por responsabilidade.
- ❌ Funções > **30 linhas** — extraia.
- ❌ Funções com > **3 parâmetros** — use struct de opções.
- ❌ Nesting > **3 níveis** — early return.
- ❌ Switch/if-else gigante sobre tipo/string — considere Strategy ou map de handlers.
</size-complexity>

<documentation-comments>
- ❌ Comentários que descrevem **o quê** o código faz (`// increment counter`).
- ❌ Comentários sobre task/PR atual (`// added in #123`, `// requested by Alice`) — vai no commit/PR, não no código.
- ❌ Docstrings genéricas (`// Foo does foo stuff`).
- ❌ `// TODO` sem owner/contexto — abra issue ou delete.
- ✅ Comentários **sobre porquê** raros e específicos (invariante, workaround documentado).
</documentation-comments>

<process>
- ❌ "Patch rápido" para verde no CI — invoque [`no-workarounds`](../../.claude/skills/no-workarounds).
- ❌ Adicionar feature sem teste correspondente para domínio.
- ❌ Misturar refactor + feature no mesmo commit.
- ❌ Mudar config schema sem atualizar `internal/config/` e exemplo em `docs/`.
- ❌ Introduzir nova dependência sem justificar no PR (`go.sum` cresce, blast radius cresce).
</process>
