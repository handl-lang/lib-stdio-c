# lib-stdio-c

The libc backend for the `stdio` contract (`lib-stdio`). Pure `.hnl` —
everything goes through `ffi.call("libc.so.6", ...)` directly on the
`stdin`/`stdout`/`stderr` FILE* globals; no posix dep, no generated
libc adapter.

## Buffering

handl has no mutable state (values are immutable — no refs/cells), so
the buffers live in libc stdio, which is exactly the contract's
documented model:

- `stdout` — line-buffered on a terminal, block-buffered (~4–8 KiB)
  otherwise; N `print`s cost ~one `write(2)`.
- `stderr` — unbuffered (C default); diagnostics reach the kernel
  promptly.
- `stdin` — libc read-ahead; `read_line` is one `getline` per line
  (arbitrary length), `read_chars`/`read_all` are a `fread` loop in
  64 KiB slots.
- `flush` — `fflush(NULL)`, every output stream at once.

Each call resolves the FILE* fresh (`ffi.sym` + `ffi.read` on the
libc global), so there is no handl-side state to keep coherent.

## Surface

`print`, `eprint`, `write`, `ewrite`, `flush`, `read_line`,
`read_chars`, `read_all` — untyped exports per the `*-c` convention
(a type annotation is treated as a callee and fails for imported
arrow types). Errors raise `stdio: <op>: <strerror>`.

## Layout

```text
src/             the .hnl sources (index + tests)
pkg.hnl          package manifest — modules, deps, description, license
ci.sh            bupha entry → scripts/hnl-gate.sh
```

## Gate

```bash
HNL=../host-cli/target/debug/hnl scripts/hnl-gate.sh
```

## License

MIT.
