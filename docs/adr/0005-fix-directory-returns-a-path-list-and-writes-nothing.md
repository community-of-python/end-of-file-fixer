# `fix_directory` returns a path list and writes nothing

**Decision:** `fix_directory` returns `list[pathlib.Path]` in walk order and touches no stream.
`main()` renders the `Fixing <path>` lines and derives the exit code from that list after the walk
returns.

Two alternatives were rejected when the fix behavior was extracted out of `main()`. The first was a
`FixReport` wrapper carrying counts and categories instead of a bare list: both consumers that exist
— the rendered lines and the 0/1 exit code — are satisfied by the list itself, so the wrapper would
add a type without adding behaviour. The second was keeping output streaming during the walk, which
would mean handing `fix_directory` a stream or a callback and re-fusing the I/O it was extracted to
separate.

Rendering after the walk instead of during it is the one observable consequence: the lines, their
order, the set of files fixed, and the exit code are all unchanged, only the timing differs. For a
tool that runs as a fast CI or pre-commit gate over a tree it has already pruned, seeing the report
at the end rather than incrementally costs nothing, and it buys a fix module with no I/O
dependencies — which is what lets the content-level tests run against `BytesIO` with no filesystem
and no captured stdout.

**Revisit trigger:** a consumer appears that needs more than the paths — counts, per-file actions, or
a skipped/binary breakdown — or a tree large enough that a user waits long enough to want incremental
output. The first makes a report type real; the second makes the streaming seam real.
