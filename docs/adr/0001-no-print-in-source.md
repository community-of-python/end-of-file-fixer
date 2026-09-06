# No `print()` in library/CLI source

**Decision:** Source code never calls `print()`. CLI output goes through `sys.stdout.write(...)` /
`sys.stderr.write(...)` with an explicit `\n` in the written string.

A proposed output fix swapped `sys.stdout.write(f"Fixing {filename}")` for
`print(f"Fixing {filename}")` to add the missing trailing newline. The underlying bug was a missing
`\n`, not the choice of API, and upstream prior art (`pre-commit-hooks`) uses `print()`, so the swap
looked idiomatic. The two options on the table were replacing the `write` call with `print()`, which
gets the newline for free, or keeping the `write` call and adding the explicit `\n` to the format
string.

The second option was taken. `print()` in production code paths is treated as a smell here: it hides
the newline behaviour, mixes a convenience builtin into deliberate I/O, and obscures the destination
stream. Keeping explicit `write` calls makes both the stream and the terminator visible at the call
site, and the original bug is fixed in place by adding `\n` rather than switching APIs. The rule is
about *source*: `print()` remains fine in tests, scratch scripts, and REPL examples.

**Revisit trigger:** the project adopts a logging framework or a CLI/output library that supplies its
own print-style helper, making hand-rolled `write` calls the inconsistent choice.
