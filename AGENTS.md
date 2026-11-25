# lib-stdio-c

The libc backend for the `stdio` contract (`lib-stdio`). Pure `.hnl`
— everything goes through `ffi.call("libc.so.6", ...)` directly on
the `stdin`/`stdout`/`stderr` FILE* globals; no posix dep, no
generated libc adapter.

## Layout

| path | role |
| --- | --- |
| `src/index.hnl` | the backend implementation |
| `pkg.hnl` | manifest — `modules: ["stdio-c"]` |
| `scripts/hnl-gate.sh`, `ci.sh` | the gate / bupha entry |

## Contract

- Surface: `print`, `eprint`, `write`, `ewrite`, `flush`,
  `read_line`, `read_chars`, `read_all` — untyped exports per the
  `*-c` convention (a type annotation is treated as a callee and
  fails for imported arrows).
- Buffering lives in libc stdio (handl has no mutable state):
  stdout line-/block-buffered, stderr unbuffered, stdin read-ahead;
  `flush` = `fflush(NULL)`. `read_line` is one `getline` per line;
  `read_chars`/`read_all` are a `fread` loop in 64 KiB slots. Each
  call resolves the FILE* fresh (`ffi.sym` + `ffi.read` on the libc
  global) — no handl-side state.

## Verification

```bash
HNL=../host-cli/target/debug/hnl scripts/hnl-gate.sh
```

## License

MIT.

<!-- myco:agent-awareness -->

## Agent awareness

- Session start / any work discussion: run `myco status --prompt`
  before proposing next steps — know the board first.
- User describes new work → offer to capture it (`myco todo add`,
  `issue new`, `proposal new`). Don't wait to be asked.
- Picking up a task: `myco resume <ID>` is the briefing.
- Wrapping up: `myco handoff <ID>` anything resumable; keep statuses
  honest (claimed → in-progress → blocked/done).

<!-- /myco:agent-awareness -->
